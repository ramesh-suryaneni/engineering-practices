### What is an API?  
- **API (Application Programming Interface)** defines how software components interact.  
- Acts as a **contract** between client (e.g., mobile or browser) and server.  
- Specifies:  
  - Allowed requests (endpoints, HTTP methods)  
  - Expected responses  
- **API serves as abstraction**: hides implementation details but exposes functionality (e.g., saving user data without revealing internal logic).  
- Defines **service boundaries**, allowing multiple servers and systems to communicate regardless of technology differences.

### Common API Styles: REST, GraphQL, and gRPC  
- **REST (Representational State Transfer)**:  
  - Resource-based, uses HTTP methods (GET, POST, PUT/PATCH, DELETE).  
  - Stateless: each request is independent and contains all needed info.  
  - Most common for web and mobile apps.  
- **GraphQL**:  
  - Query language with a single endpoint for all operations.  
  - Clients specify exactly what data they want (queries) or modify (mutations).  
  - Supports subscriptions for real-time communication.  
  - Minimizes round trips (e.g., one request replaces multiple REST requests).  
  - Ideal for complex UIs with nested or varying data needs.  
- **gRPC**:  
  - High-performance RPC framework using protocol buffers over HTTP/2.  
  - Supports streaming and bidirectional communication.  
  - Best suited for microservices and internal server-to-server communication.  
  - Least common for public APIs compared to REST and GraphQL.

### REST vs GraphQL: Key Differences and Examples  
| Feature                  | REST                                    | GraphQL                                |
|--------------------------|-----------------------------------------|---------------------------------------|
| Endpoints                | Multiple resource-based endpoints        | Single endpoint for all operations    |
| Request Method           | HTTP verbs (GET, POST, PUT/PATCH, DELETE) | POST requests with query language     |
| Response Structure       | Fixed per endpoint                       | Client specifies the response shape   |
| Versioning               | Explicit, e.g., /v1/, /v2/               | Usually no versioning, optional field versioning |
| Data Fetching            | Multiple requests for related resources | Single request for nested data        |
| Caching                  | HTTP caching supported                   | Application-level caching              |

- REST uses resource URLs and standard HTTP methods; multiple requests may be needed for related data.  
- GraphQL allows clients to request precise nested data in a single call, improving efficiency.  
- REST APIs version by URL; GraphQL schema evolves mainly without explicit versioning, although field versioning is possible.

### Four Essential API Design Principles  
- **Consistency**: Use consistent naming conventions, casing (camelCase vs snake_case), and predictable behavior. Avoid surprising side effects (e.g., GET request should not modify data).  
- **Simplicity**: Focus on core use cases, intuitive design, and minimize complexity so that APIs can be used without extensive documentation.  
- **Security**: Implement authentication, authorization, input validation, and rate limiting to protect APIs.  
- **Performance**: Optimize with caching strategies, pagination (limit, offset, cursor-based), minimized payloads, and reduced round trips to avoid unnecessary data transfer.

### Role of Protocols in API Design  
- Protocol choice shapes API features and performance:  
  - **HTTP**: Foundation for REST, supports status codes, caching, CRUD operations.  
  - **WebSockets**: Enables real-time, bidirectional communication (e.g., chat, video streaming).  
  - **gRPC**: Efficient microservices communication using HTTP/2 and protocol buffers.  
- Choose protocols based on interaction patterns, performance needs, payload size, client compatibility, and developer tooling.

### API Design Process  
- **Requirement Analysis**: Identify core use cases, user stories, and scope (in/out of scope features).  
- **Performance and Security**: Define bottlenecks and security constraints early.  
- **Design Approaches**:  
  - Top-down: Start from high-level requirements to define endpoints and operations (common in interviews).  
  - Bottom-up: Design API based on existing data models and capabilities (common in established companies).  
  - Contract-first: Define API contract (request/response format) before implementation.  
- **Lifecycle Management**:  
  - Design → Develop → Test → Deploy → Monitor → Maintain → Deprecate → Retire  
- Emphasizes maintainability and versioning to ensure smooth transitions and backward compatibility.

### Application Layer Protocols in Network Stack  
- Application protocols operate over transport protocols (TCP/UDP).  
- **HTTP/HTTPS**:  
  - Standard client-server request-response protocol.  
  - HTTP methods: GET, POST, PUT/PATCH, DELETE.  
  - Status codes indicate request results (200 success, 400 client error, 500 server error).  
  - HTTPS adds TLS/SSL encryption for data security in transit.  
- **WebSockets**:  
  - After initial handshake, provides persistent bidirectional communication.  
  - Eliminates unnecessary polling by server-initiated data pushes.  
  - Ideal for real-time apps like chat or live notifications.  
- **AMQP (Advanced Message Queuing Protocol)**:  
  - Enterprise messaging protocol for asynchronous communication with message queues.  
  - Producers publish messages to queues, consumers pull messages when ready.  
  - Supports different exchange types: direct, fan-out, topic-based.  
- **gRPC**:  
  - Uses HTTP/2 and protocol buffers for efficient server-to-server communication.  
  - Supports streaming and multiplexing.

### Transport Layer Protocols: TCP vs UDP  
| Feature               | TCP (Transmission Control Protocol)             | UDP (User Datagram Protocol)                |
|-----------------------|-------------------------------------------------|---------------------------------------------|
| Reliability           | Guaranteed delivery, retransmits lost packets   | No delivery guarantee, packets may be lost  |
| Connection            | Connection-oriented (3-way handshake)           | Connectionless                               |
| Ordering              | Ensures packets arrive in order                  | No ordering guarantees                       |
| Overhead              | Higher (due to connection management and ACKs) | Lower overhead, faster transmission          |
| Use Cases             | Payments, emails, authentication (critical data)| Video calls, live streaming, gaming (real-time) |

- TCP is slower but reliable; UDP is faster but does not guarantee delivery.  
- TCP’s three-way handshake establishes a connection before data transfer starts.  
- UDP’s lightweight nature suits scenarios where losing some data is acceptable.

### RESTful API Design Best Practices  
- **Resource Modeling**: Use nouns representing business domain entities (e.g., /products, /orders) rather than verbs (avoid /getProducts).  
- **URLs**:  
  - Collections: `/products` returns all products.  
  - Individual items: `/products/{id}` returns a specific product.  
  - Nested resources: `/products/{id}/reviews` returns reviews of a product.  
- **Filtering, Sorting, Pagination** via query parameters to limit response size and improve performance:  
  - Filtering: `/products?category=electronics&inStock=true`  
  - Sorting: `/products?sort=price_asc`  
  - Pagination: `/products?page=3&limit=10` or using offset: `/products?offset=20&limit=10`  
  - Cursor-based pagination also common for scalable APIs.  
- **HTTP Methods for CRUD**:  
  - GET: Retrieve resources (safe and idempotent).  
  - POST: Create resources (not idempotent).  
  - PUT: Replace entire resource.  
  - PATCH: Partial update.  
  - DELETE: Remove resource.  
- **HTTP Status Codes**:  
  - 200 OK: Successful GET or other requests.  
  - 201 Created: Resource successfully created (POST).  
  - 204 No Content: Successful request with no response body.  
  - 300s: Redirection.  
  - 400s: Client errors (400 Bad Request, 401 Unauthorized, 404 Not Found).  
  - 500s: Server errors.  
- **Versioning**: Use URL versioning (e.g., `/api/v1/products`) to preserve backward compatibility during upgrades.

### GraphQL API Design Fundamentals  
- Developed by Facebook to reduce excessive and multiple REST requests by allowing clients to request exactly the data needed from a single endpoint.  
- **Schema Design and Type System**:  
  - Schema acts as a contract between client and server.  
  - Defines types (e.g., User, Post) with fields and nested types.  
  - Queries correspond to GET operations; mutations correspond to create/update/delete operations.  
- **Query Example**:  
  - Clients specify exact fields required, e.g., user’s name, posts’ titles only.  
- **Error Handling**:  
  - All responses return HTTP 200 status.  
  - Errors are communicated in an `errors` field in the response JSON with status codes and messages.  
  - Partial data may be returned alongside errors.  
- **Best Practices**:  
  - Keep schemas small and modular.  
  - Limit query depth to avoid performance issues.  
  - Use meaningful naming conventions.  
  - Use input types for mutations.

### Authentication Overview and Methods  
- **Purpose**: Verify identity of users or systems accessing APIs before authorization.  
- **Basic Authentication**:  
  - Username and password encoded in Base64.  
  - Considered insecure unless used over HTTPS.  
- **Bearer Tokens**:  
  - Access tokens sent with each request, more secure and stateless.  
  - Standard for modern API authentication.  
- **OAuth2 with JWT Tokens**:  
  - OAuth2 protocol allows login via trusted providers (Google, GitHub).  
  - Providers issue JWT tokens containing user info and expiration.  
  - JWT tokens are signed and stateless, supporting scalable APIs.  
- **Access and Refresh Tokens**:  
  - Short-lived access tokens for API calls.  
  - Long-lived refresh tokens to renew access tokens without re-login.  
  - Refresh tokens typically stored server-side for security.  
- **Single Sign-On (SSO)**:  
  - One login grants access to multiple services.  
  - Uses OAuth2 (modern, JSON-based) or SAML (legacy, XML-based).  
  - Common in enterprise and large ecosystems (e.g., Google services).

### Authorization Concepts and Models  
- **Authorization** determines what authenticated users can do and which resources they can access.  
- Example: GitHub repository permissions vary by user role (read-only, write, admin).  
- **Common Authorization Models**:  
  - **Role-Based Access Control (RBAC)**: Users assigned roles (admin, editor, viewer) with predefined permissions.  
  - **Attribute-Based Access Control (ABAC)**: Access based on user/resource attributes and environment conditions (e.g., department, time of day). More flexible but complex and prone to conflicts.  
  - **Access Control Lists (ACLs)**: Permissions assigned per resource specifying which users have what access (read, write). Used in systems like Google Drive.  
- Real-world systems often combine these models for security and flexibility.  
- **Enforcement Technologies**:  
  - OAuth2 delegated authorization issues tokens with scopes reflecting permissions.  
  - JWT/bearer tokens carry identity and claims, checked by servers to enforce permission logic.

### Seven Proven API Security Techniques  
- **Rate Limiting**: Limits number of requests per client/timeframe to prevent abuse and DDoS attacks. Can be set per endpoint, per user/IP, or globally.  
- **CORS (Cross-Origin Resource Sharing)**: Controls which domains can access your API from browsers to prevent malicious cross-site requests.  
- **SQL/NoSQL Injection Prevention**: Use parameterized queries or ORM safeguards to avoid injection attacks that compromise databases.  
- **Firewalls**: Filter incoming traffic to block suspicious patterns or known attack vectors (e.g., AWS WAF).  
- **VPNs**: Restrict access to APIs to specific internal networks (useful for private/internal APIs).  
- **CSRF (Cross-Site Request Forgery) Protection**: Use CSRF tokens alongside cookies to prevent unauthorized actions from malicious sites on behalf of logged-in users.  
- **XSS (Cross-Site Scripting) Prevention**: Sanitize user input to prevent injection of malicious scripts that execute in other users’ browsers.  
