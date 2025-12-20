# TLS/SSL Setup For Spring Boot Made Simple  
## A Step-by-Step Guide  


Securing web traffic with TLS/SSL is no longer optional; it’s a must. Spring Boot makes adding HTTPS to your app straightforward. This guide walks you through a clear, practical workflow: generate a keystore, configure Spring Boot, redirect HTTP to HTTPS, test the setup, and take tips for production readiness. No theory overload, just the exact steps you need.

---

## 1. Quick overview: what you’ll do
- Create a keystore (self-signed for testing or PKCS12/JKS for production).  
- Configure `application.properties` / `application.yml` with `server.ssl.*` properties.  
- (Optional) Redirect HTTP → HTTPS.  
- Test with a browser and curl.  
- Learn production options: CA-signed certs, Let’s Encrypt, and reverse proxy setups.  

---

## 2. Create a keystore (self-signed for dev/testing)
For local testing, create a PKCS#12 keystore (recommended over JKS for compatibility):

```bash
keytool -genkeypair \
  -alias springboot-local \
  -keyalg RSA \
  -keysize 2048 \
  -storetype PKCS12 \
  -keystore keystore.p12 \
  -validity 3650 \
  -storepass changeit \
  -keypass changeit \
  -dname "CN=localhost, OU=Dev, O=MyCompany, L=City, S=State, C=IN"
```

**Important fields:**
- `CN=localhost` if testing locally. Use your domain in production.  
- Keep a secure `storepass` / `keypass` in production (store in a secret manager).  

---

## 3. Configure Spring Boot
Add the following to `src/main/resources/application.properties` (or `application.yml` equivalent):

```properties
server.port=8443

# SSL configuration
server.ssl.key-store=classpath:keystore.p12
server.ssl.key-store-password=changeit
server.ssl.key-store-type=PKCS12
server.ssl.key-alias=springboot-local
```

If you prefer YAML:

```yaml
server:
  port: 8443
  ssl:
    key-store: classpath:keystore.p12
    key-store-password: changeit
    key-store-type: PKCS12
    key-alias: springboot-local
```

Place `keystore.p12` into `src/main/resources` so it’s packaged into the JAR.  
For external keystores use an absolute path (`file:/path/to/keystore.p12`) or an environment variable.  

---

## 4. Optional: keep HTTP for redirect or health checks
If you want your app to listen on port 8080 for a redirect (common pattern), run an embedded HTTP connector that forwards to HTTPS. For embedded Tomcat in Spring Boot, add a small config class:

```java
import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class HttpToHttpsRedirectConfig {
    @Bean
    public WebServerFactoryCustomizer<TomcatServletWebServerFactory> servletContainer() {
        return server -> server.addAdditionalTomcatConnectors(createHttpConnector());
    }
    private Connector createHttpConnector() {
        Connector connector = new Connector(TomcatServletWebServerFactory.DEFAULT_PROTOCOL);
        connector.setScheme("http");
        connector.setPort(8080);
        connector.setSecure(false);
        connector.setRedirectPort(8443);
        return connector;
    }
}
```

This listens on 8080 and redirects HTTP requests to 8443.  

---

## 5. Test your TLS setup
Start the app and test:

- **Browser:** visit `https://localhost:8443`. For a self-signed cert, you’ll see a warning expected for dev.  
- **curl (ignore cert validation for self-signed):**
  ```bash
  curl -v -k https://localhost:8443/actuator/health
  ```
- **Verify certificate chain and TLS details:**
  ```bash
  openssl s_client -connect localhost:8443 -showcerts
  ```

If everything loads, congrats, your Spring Boot app is now serving HTTPS.  

---

## 6. Production: get a CA-signed certificate
Self-signed certs are only for development. For production, use a certificate signed by a trusted CA (Let’s Encrypt is free and automatable).

**Options:**
- **Let’s Encrypt with Certbot:** best for apps hosted on VMs or servers where you can run Certbot; renewals are automated.  
- **Reverse proxy (recommended):** Terminate TLS at Nginx/Traefik/HAProxy and keep Spring Boot on HTTP (`127.0.0.1:8080`). This centralizes TLS, simplifies renewals, and offloads TLS work.  
- **Cloud-managed certificates:** Use cloud load balancers (AWS ALB, GCP HTTPS Load Balancer) to handle certs and forwarding.  

**Using Let’s Encrypt with Spring Boot directly:**
```bash
openssl pkcs12 -export -in fullchain.pem -inkey privkey.pem -out keystore.p12 -name springboot-prod -CAfile chain.pem -caname root
```

Then set `server.ssl.key-store` to the produced `keystore.p12` and use secure passwords from your secret management system.  

---

## 7. Advanced: mutual TLS (mTLS)
If you need client certificates:

Configure:
```properties
server.ssl.client-auth=need
server.ssl.trust-store=classpath:truststore.jks
server.ssl.trust-store-password=trustpass
```

Use this for high-security internal APIs or admin endpoints.  

---

## 8. Best practices & troubleshooting
- Don’t hardcode passwords in `application.properties`. Use environment variables, platform secrets, or Spring Boot’s support for externalized configs.  
- Prefer PKCS#12 for the keystore for wider compatibility with modern tools.  
- Use a reverse proxy in production for easier certificate management and adding WAF, rate-limiting, etc.  
- Monitor certificate expiry. For CA-signed certs set up alerts and automate renewals (Let’s Encrypt auto-renews every 90 days).  
- If browsers show mixed content warnings, ensure all assets (CSS, JS, images, API calls) use HTTPS.  
- For Spring Boot Actuator endpoints, secure them behind authentication and prefer HTTPS-only exposure.  

---

## Conclusion
With these steps, you’ve gone from zero to HTTPS on your Spring Boot app. Start with a self-signed keystore locally to validate your configuration, then pick a production path (reverse proxy + Let’s Encrypt or CA-signed keystore) that fits your deployment model. Secure keys, monitor expiries, and you’ll keep your traffic encrypted and your users safe.  
