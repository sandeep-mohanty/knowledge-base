# B2C Assertion Grant - OAuth2 Client Credentials Implementation

## Tech Stack
- Java 21
- Spring Boot 3.5
- Groovy 4
- WebClient (Reactor Netty)

---

## Project Structure

```
src/main/groovy/com/example/oauth2/
├── B2CAssertionGrantProperties.groovy
├── B2CAssertionGrantClientConfig.groovy
├── OAuth2ClientCredentialsService.groovy
├── OAuth2TokenResponse.groovy
└── OAuth2TokenException.groovy
```

---

## `application.yml`

```yaml
b2c:
 assertion-grant:
   token-uri: https://your-auth-server.com/oauth2/token
   client-id: your-client-id
   client-secret: your-client-secret
   scope: read write
   timeout:
     connect: 5000   # ms - TCP connection establishment
     read: 20000     # ms - time to read data from server
     write: 5000     # ms - time to send request to server
     response: 20000 # ms - total time to receive full response
```

---

## `B2CAssertionGrantProperties.groovy`

```groovy
package com.example.oauth2

import groovy.transform.CompileStatic
import org.springframework.boot.context.properties.ConfigurationProperties

@CompileStatic
@ConfigurationProperties(prefix = "b2c.assertion-grant")
class B2CAssertionGrantProperties {
   String tokenUri
   String clientId
   String clientSecret
   String scope
   Timeout timeout = new Timeout()

   @CompileStatic
   static class Timeout {
       int connect = 5000
       int read = 20000
       int write = 5000
       int response = 20000
   }
}
```

---

## `B2CAssertionGrantClientConfig.groovy`

```groovy
package com.example.oauth2

import groovy.transform.CompileStatic
import io.netty.channel.ChannelOption
import io.netty.handler.timeout.ReadTimeoutHandler
import io.netty.handler.timeout.WriteTimeoutHandler
import org.springframework.boot.context.properties.EnableConfigurationProperties
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import org.springframework.http.HttpHeaders
import org.springframework.http.MediaType
import org.springframework.http.client.reactive.ReactorClientHttpConnector
import org.springframework.util.LinkedMultiValueMap
import org.springframework.util.MultiValueMap
import org.springframework.web.reactive.function.client.WebClient
import reactor.netty.http.client.HttpClient

import java.time.Duration
import java.util.concurrent.TimeUnit

@CompileStatic
@Configuration
@EnableConfigurationProperties(B2CAssertionGrantProperties)
class B2CAssertionGrantClientConfig {

   @Bean
   WebClient b2cAssertionGrantWebClient(WebClient.Builder builder, B2CAssertionGrantProperties properties) {
       def httpClient = HttpClient.create()
           .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, properties.timeout.connect)
           .responseTimeout(Duration.ofMillis(properties.timeout.response))
           .doOnConnected { conn ->
               conn.addHandlerLast(new ReadTimeoutHandler(properties.timeout.read, TimeUnit.MILLISECONDS))
               conn.addHandlerLast(new WriteTimeoutHandler(properties.timeout.write, TimeUnit.MILLISECONDS))
           }

       builder
           .baseUrl(properties.tokenUri)
           .clientConnector(new ReactorClientHttpConnector(httpClient))
           .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_FORM_URLENCODED_VALUE)
           .filter { request, next ->
               def updatedRequest = request.mutate()
                   .body(buildBaseFormBody(properties))
                   .build()
               next.exchange(updatedRequest)
           }
           .build()
   }

   private MultiValueMap<String, String> buildBaseFormBody(B2CAssertionGrantProperties properties) {
       def formBody = new LinkedMultiValueMap<String, String>()
       formBody.add("grant_type", "client_credentials")
       formBody.add("client_id", properties.clientId)
       formBody.add("client_secret", properties.clientSecret)
       formBody.add("scope", properties.scope)
       formBody
   }
}
```

---

## `OAuth2ClientCredentialsService.groovy`

```groovy
package com.example.oauth2

import groovy.transform.CompileStatic
import groovy.util.logging.Slf4j
import org.springframework.beans.factory.annotation.Qualifier
import org.springframework.http.HttpStatus
import org.springframework.stereotype.Service
import org.springframework.util.LinkedMultiValueMap
import org.springframework.util.MultiValueMap
import org.springframework.web.reactive.function.BodyInserters
import org.springframework.web.reactive.function.client.WebClient
import org.springframework.web.reactive.function.client.WebClientRequestException
import org.springframework.web.reactive.function.client.WebClientResponseException
import reactor.core.publisher.Mono

@Slf4j
@Service
@CompileStatic
class OAuth2ClientCredentialsService {

   private final WebClient webClient
   private final B2CAssertionGrantProperties properties

   OAuth2ClientCredentialsService(@Qualifier("b2cAssertionGrantWebClient") WebClient webClient,
                                  B2CAssertionGrantProperties properties) {
       this.webClient = webClient
       this.properties = properties
   }

   boolean isB2CClientConfigured() {
       def errors = []

       if (!properties.tokenUri) {
           errors << "b2c.assertion-grant.token-uri is missing or empty"
       }
       if (!properties.clientId) {
           errors << "b2c.assertion-grant.client-id is missing or empty"
       }
       if (!properties.clientSecret) {
           errors << "b2c.assertion-grant.client-secret is missing or empty"
       }
       if (!properties.scope) {
           errors << "b2c.assertion-grant.scope is missing or empty"
       }

       if (errors) {
           log.warn("B2C assertion grant client is not properly configured: {}", errors.join(', '))
           return false
       }

       true
   }

   OAuth2TokenResponse fetchToken(String username) {
       log.info("Requesting b2c assertion grant token for user: {}", username)

       webClient.post()
               .body(BodyInserters.fromFormData(buildRequestBody(username)))
               .retrieve()
               .onStatus(HttpStatus.UNAUTHORIZED::equals) { response ->
                   Mono.error(new OAuth2TokenException("Unauthorized: invalid client credentials"))
               }
               .onStatus(HttpStatus.BAD_REQUEST::equals) { response ->
                   response.bodyToMono(String)
                       .flatMap { body ->
                           Mono.error(new OAuth2TokenException("Bad request: ${body}"))
                       }
               }
               .onStatus({ status -> status.is5xxServerError() }) { response ->
                   Mono.error(new OAuth2TokenException("Auth server error: ${response.statusCode()}"))
               }
               .bodyToMono(OAuth2TokenResponse)
               .doOnSuccess { token ->
                   log.info("Successfully retrieved b2c assertion grant token for user: {}", username)
               }
               .doOnError(WebClientResponseException) { ex ->
                   log.error("HTTP error fetching b2c assertion grant token for user: {} - status: {}, body: {}",
                           username, ex.statusCode, ex.responseBodyAsString)
               }
               .doOnError(WebClientRequestException) { ex ->
                   log.error("Network error fetching b2c assertion grant token for user: {} - error: {}",
                           username, ex.message)
               }
               .onErrorMap(WebClientRequestException) { ex ->
                   new OAuth2TokenException("Network error connecting to auth server: ${ex.message}", ex)
               }
               .block()
   }

   private MultiValueMap<String, String> buildRequestBody(String username) {
       def formBody = new LinkedMultiValueMap<String, String>()
       formBody.add("username", username)
       formBody
   }
}
```

---

## `OAuth2TokenResponse.groovy`

```groovy
package com.example.oauth2

import com.fasterxml.jackson.annotation.JsonProperty
import groovy.transform.CompileStatic

@CompileStatic
class OAuth2TokenResponse {

   @JsonProperty("access_token")
   String accessToken

   @JsonProperty("token_type")
   String tokenType

   @JsonProperty("expires_in")
   Long expiresIn

   @JsonProperty("scope")
   String scope
}
```

---

## `OAuth2TokenException.groovy`

```groovy
package com.example.oauth2

import groovy.transform.CompileStatic

@CompileStatic
class OAuth2TokenException extends RuntimeException {

   OAuth2TokenException(String message) {
       super(message)
   }

   OAuth2TokenException(String message, Throwable cause) {
       super(message, cause)
   }
}
```

---

## `build.gradle`

```groovy
dependencies {
   implementation 'org.springframework.boot:spring-boot-starter-webflux'
   implementation 'org.apache.groovy:groovy:4.0.21'
   annotationProcessor 'org.springframework.boot:spring-boot-configuration-processor'
}
```

---

## Usage Example

```groovy
if (oAuth2ClientCredentialsService.isB2CClientConfigured()) {
   def tokenResponse = oAuth2ClientCredentialsService.fetchToken(username)
   // proceed with token
} else {
   // handle misconfiguration gracefully
}
```

---

## Design Decisions

| Decision | Reason |
|---|---|
| `WebClient` over `RestClient` | Already used in the project; no point mixing two HTTP clients |
| `B2CAssertionGrantProperties` | Consistent naming across all B2C assertion grant classes |
| `B2CAssertionGrantClientConfig` | Captures domain (B2C), grant type (AssertionGrant) and purpose (ClientConfig); mirrors `b2c.assertion-grant` from YAML |
| `@Qualifier("b2cAssertionGrantWebClient")` | Avoids ambiguity with other `WebClient` beans; name mirrors `b2c.assertion-grant` from YAML |
| Static form body in bean | `grant_type`, `client_id`, `client_secret`, `scope` are known at startup; configured once in the bean |
| Dynamic `username` in service | Changes per request; passed at call time |
| `isB2CClientConfigured()` | Allows caller to check configuration before making the token call; avoids unnecessary HTTP calls |
| `.block()` | Token is required before proceeding; blocking is intentional |
| `@CompileStatic` | Java-speed performance and compile-time type safety in Groovy 4 |
| Timeouts via Netty `HttpClient` | WebClient uses Reactor Netty underneath; this is the correct way to configure low-level timeouts |
| `OAuth2TokenException` | Clean domain exception; decouples callers from WebClient internals |

---

## Timeout Configuration

| Timeout | Value | Reason |
|---|---|---|
| `connect` | 5000ms | TCP handshake should always be fast |
| `write` | 5000ms | Sending a small form body should always be fast |
| `read` | 20000ms | Covers worst case 15s auth server response with buffer |
| `response` | 20000ms | Total response time, aligned with read timeout |

---

## Exception Handling

| Scenario | Handling |
|---|---|
| `401 Unauthorized` | Mapped to `OAuth2TokenException` with clear message |
| `400 Bad Request` | Response body extracted and included in `OAuth2TokenException` |
| `5xx Server Error` | Mapped to `OAuth2TokenException` with status code |
| Network/timeout error | `WebClientRequestException` mapped to `OAuth2TokenException` |
| All errors | Logged with user context before propagating |