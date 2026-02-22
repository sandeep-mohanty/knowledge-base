# HTTP Status Codes Explained: A Complete Guide

Learn what HTTP status codes mean, how they work, and why they matter in real-world web applications.

HTTP status codes are **three-digit numbers** that a web server sends in response to a request made by a browser. Most people have encountered the classic **404 Page Not Found** error at some point. This is an example of an HTTP **client error** status code, and there are many more like it.

In this article, we’ll explain what **HTTP status codes** are and how they work. These codes (also called **response status codes**) act as a way for the server to communicate with the web browser. They are divided into **different classes**, based on the type of response being sent.

The class of a status code is identified by its **first digit**. For example, **4xx** codes (like 404) indicate that the requested page could not be reached, while **2xx** codes mean the request was **successfully completed**.

---

### How are HTTP status codes categorized?

HTTP status codes are divided into **five main categories**. Each category helps you understand the **type of response** sent by the server, even if you don’t know the exact status code.



* **1xx (Informational):** Indicates that the request was received and the server is continuing the process.
* **2xx (Successful):** Means the request was successfully received, understood, and processed.
* **3xx (Redirection):** Tells the browser that it needs to take an additional action, such as redirecting to another URL.
* **4xx (Client Error):** Indicates a problem with the request made by the client (browser), such as a missing page.
* **5xx (Server Error):** Means the server failed to complete a valid request.

---

### 1xx (Informational)

These status codes are used to send **temporary (provisional) responses** to the client while the server is still **processing the request**. They do not indicate success or failure — only that the request has been received and is being handled.

* **100 (Continue):** This status code means the server has received the **request headers**, and the client can continue sending the **request body**.
* **101 (Switching Protocols):** This status code is sent when the server agrees to **switch protocols**, as requested by the client (for example, switching from HTTP to WebSocket).
* **102 (Processing):** This status code indicates that the server has received the request and is **still processing it**. It is commonly used when a request contains **many sub-requests** or takes a long time to complete.
* **103 (Early Hints):** This status code allows the server to send **response headers early**, before the final response is ready. It helps improve performance by letting the browser start loading resources sooner.

---

### 2xx (Success)

These status codes indicate that the request was **successfully received, understood, and processed** by the server.



* **200 (OK):** This status code means the client’s request was **successful**, and the server returned the requested data.
* **201 (Created):** This status code indicates that a **new resource** was successfully created as a result of the request (commonly used with `POST` requests).
* **202 (Accepted):** This status code means the request was **accepted for processing**, but the processing has **not been completed yet**.
* **203 (Non-Authoritative Information):** This status code indicates that the returned data was **modified by a proxy server** and does not come directly from the original server.
* **204 (No Content):** This status code means the request was **successful**, but there is **no content** to return in the response body.
* **205 (Reset Content):** This status code tells the client to **reset the view or form** from which the request was sent.
* **206 (Partial Content):** This status code indicates that the server is sending **only part of the requested content**, usually because the client requested a specific range of data.
* **207 (Multi-Status):** This status code contains **multiple response codes** for different parts of a single request. It is mainly used in **WebDAV**.
* **208 (Already Reported):** This status code indicates that the response has **already been included earlier**, preventing duplicate information.
* **226 (IM Used):** This status code indicates that the server has fulfilled a **partial GET request** using **instance manipulation**. (It is rarely used in practice.)

---

### 3xx (Redirection)

These HTTP status codes indicate that the client must take **additional action** to complete the request, usually by **redirecting to another URL**.



* **300 (Multiple Choices):** This status code indicates that the request has **multiple possible responses**, and the client can choose one of them (for example, different formats or URLs).
* **301 (Moved Permanently):** This status code means the requested page has been **permanently moved** to a new URL. Search engines update their indexes to the new URL.
* **302 (Found):** This status code indicates that the requested page has been **temporarily moved** to a different URL.
* **303 (See Other):** This status code tells the client to **retrieve the requested resource using another URL**, usually with a **GET request**.
* **304 (Not Modified):** This status code means the requested resource has **not changed** since the last request. The browser can use its **cached version**, improving performance.
* **305 (Use Proxy):** This status code indicates that the requested resource must be accessed **through a proxy server**. (This status code is deprecated and should not be used.)
* **306 (No Longer Used):** This status code is **unused and reserved**. It was originally intended for proxy switching.
* **307 (Temporary Redirect):** This status code indicates a **temporary redirect**, and the client must repeat the request to the new URL **without changing the HTTP method**.
* **308 (Permanent Redirect):** This status code means the resource has been **permanently moved** to a new URL, and the client must use the new URL **without changing the HTTP method**.

---

### 4xx (Client Errors)

These HTTP status codes indicate that the **error occurred because of the client’s request** (for example, a bad request, missing permissions, or invalid data).



* **400 (Bad Request):** The server could not understand the request due to **invalid syntax**, **bad formatting**, or **incorrect request data**.
* **401 (Unauthorized):** Authentication is required, but the client **did not provide valid credentials**.
* **402 (Payment Required):** This status code is **reserved for future use**. Some services use it to indicate that **payment is required**.
* **403 (Forbidden):** The client is authenticated, but **does not have permission** to access the requested resource.
* **404 (Not Found):** The requested page or resource **could not be found** on the server.
* **405 (Method Not Allowed):** The HTTP method (such as `GET` or `POST`) is **not allowed** for the requested resource.
* **406 (Not Acceptable):** The server cannot generate a response that matches the **content type requested by the client**.
* **407 (Proxy Authentication Required):** Authentication is required to access the resource through a **proxy server**.
* **408 (Request Timeout):** The server **timed out** while waiting for the client’s request.
* **409 (Conflict):** The request could not be completed due to a **conflict with the current state** of the resource.
* **410 (Gone):** The requested resource is **permanently removed** and will not be available again.
* **411 (Length Required):** The server requires the **Content-Length** header, but it was not provided.
* **412 (Precondition Failed):** The server does not meet one or more **preconditions** specified in the request headers.
* **413 (Payload Too Large):** The request is too large for the server to process. (Previously called **Request Entity Too Large**)
* **414 (URI Too Long):** The requested URL is **too long** for the server to handle.
* **415 (Unsupported Media Type):** The server does not support the **media type** sent by the client.
* **416 (Range Not Satisfiable):** The requested range of data is **invalid or outside the available size**.
* **417 (Expectation Failed):** The server cannot meet the expectations specified in the **Expect** request header.
* **421 (Misdirected Request):** The request was sent to a server that **cannot produce a valid response**.
* **422 (Unprocessable Entity):** The request is well-formed but contains **semantic errors**.
* **423 (Locked):** The requested resource is **locked** and cannot be accessed.
* **424 (Failed Dependency):** The request failed because it **depends on another request that failed**.
* **425 (Too Early):** The server is unwilling to process the request due to the risk of **replay attacks**.
* **426 (Upgrade Required):** The client must **upgrade to a different protocol** to complete the request.
* **428 (Precondition Required):** The server requires certain **preconditions**, but the client did not provide them.
* **429 (Too Many Requests):** The client has sent **too many requests** in a given amount of time (rate limiting).
* **431 (Request Header Fields Too Large):** One or more request headers are **too large**, or the total header size is too big.
* **451 (Unavailable For Legal Reasons):** The requested resource is unavailable due to **legal restrictions**.

---

### 5xx (Server Errors)

These HTTP status codes indicate that the **server failed to fulfill a valid client request**. The problem occurs **on the server side**, not because of the client.



* **500 (Internal Server Error):** This status code means the server encountered an **unexpected condition** and could not complete the request.
* **501 (Not Implemented):** The server does not support the **request method** or is unable to process the request.
* **502 (Bad Gateway):** The server received an **invalid response** from an upstream server (such as another server or API).
* **503 (Service Unavailable):** The server is currently **unavailable**, usually due to **maintenance** or **overload**.
* **504 (Gateway Timeout):** The server did not receive a **timely response** from an upstream server.
* **505 (HTTP Version Not Supported):** The server does not support the **HTTP protocol version** used in the request.
* **507 (Insufficient Storage):** The server does not have enough **storage space** to complete the request.
* **508 (Loop Detected):** The server detected an **infinite loop** while processing the request.
* **510 (Not Extended):** The request cannot be fulfilled because it does not meet the **required extension policies**.
* **511 (Network Authentication Required):** The client must **authenticate** to gain access to the network before the request can be processed.

---

HTTP status codes play an important role in how the web works. By understanding these status codes, developers, students, and website owners can quickly identify whether a request was successful, redirected, or failed due to a client-side or server-side issue. They are an essential part of building and maintaining reliable web applications.

Would you like me to find a **troubleshooting guide for the most common 5xx errors** to help you debug server-side issues?
```