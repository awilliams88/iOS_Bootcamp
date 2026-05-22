# Networking — API Client Architecture

---

## The Endpoint Pattern

Model each API endpoint as a value type:

```swift
// Endpoint descriptor
struct Endpoint {
    let path: String
    let method: HTTPMethod
    let queryItems: [URLQueryItem]?
    let headers: [String: String]?
    let body: Encodable?
    
    init(
        path: String,
        method: HTTPMethod = .get,
        queryItems: [URLQueryItem]? = nil,
        headers: [String: String]? = nil,
        body: Encodable? = nil
    ) {
        self.path = path
        self.method = method
        self.queryItems = queryItems
        self.headers = headers
        self.body = body
    }
}

// Define endpoints as static factories
extension Endpoint {
    static func users(page: Int, pageSize: Int = 20) -> Endpoint {
        Endpoint(
            path: "/users",
            queryItems: [
                URLQueryItem(name: "page", value: "\(page)"),
                URLQueryItem(name: "page_size", value: "\(pageSize)")
            ]
        )
    }
    
    static func user(id: Int) -> Endpoint {
        Endpoint(path: "/users/\(id)")
    }
    
    static func createUser(_ user: CreateUserRequest) -> Endpoint {
        Endpoint(path: "/users", method: .post, body: user)
    }
    
    static func updateUser(id: Int, update: UpdateUserRequest) -> Endpoint {
        Endpoint(path: "/users/\(id)", method: .patch, body: update)
    }
    
    static func deleteUser(id: Int) -> Endpoint {
        Endpoint(path: "/users/\(id)", method: .delete)
    }
}
```

---

## Protocol-Based API Client

```swift
// Protocol for testability
protocol APIClientProtocol {
    func fetch<T: Decodable>(_ endpoint: Endpoint) async throws -> T
    func send(_ endpoint: Endpoint) async throws
}

// Production implementation
final class APIClient: APIClientProtocol {
    private let baseURL: URL
    private let session: URLSession
    private let decoder: JSONDecoder
    private let tokenProvider: TokenProvider
    
    init(
        baseURL: URL,
        session: URLSession = .shared,
        decoder: JSONDecoder = .default,
        tokenProvider: TokenProvider
    ) {
        self.baseURL = baseURL
        self.session = session
        self.decoder = decoder
        self.tokenProvider = tokenProvider
    }
    
    func fetch<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        let request = try buildRequest(for: endpoint)
        let (data, response) = try await session.data(for: request)
        try validate(response)
        return try decoder.decode(T.self, from: data)
    }
    
    func send(_ endpoint: Endpoint) async throws {
        let request = try buildRequest(for: endpoint)
        let (_, response) = try await session.data(for: request)
        try validate(response)
    }
    
    private func buildRequest(for endpoint: Endpoint) throws -> URLRequest {
        var components = URLComponents(url: baseURL.appendingPathComponent(endpoint.path),
                                       resolvingAgainstBaseURL: true)!
        components.queryItems = endpoint.queryItems
        
        guard let url = components.url else { throw NetworkError.invalidURL }
        
        var request = URLRequest(url: url)
        request.httpMethod = endpoint.method.rawValue
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.setValue("application/json", forHTTPHeaderField: "Accept")
        
        if let token = tokenProvider.currentToken {
            request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }
        
        endpoint.headers?.forEach { request.setValue($1, forHTTPHeaderField: $0) }
        
        if let body = endpoint.body {
            request.httpBody = try JSONEncoder().encode(AnyEncodable(body))
        }
        return request
    }
    
    private func validate(_ response: URLResponse) throws {
        guard let http = response as? HTTPURLResponse else { throw NetworkError.noData }
        switch http.statusCode {
        case 200...299: return
        case 401: throw NetworkError.unauthorized
        case 404: throw NetworkError.notFound
        case 422: throw NetworkError.validationFailed
        case 500...599: throw NetworkError.serverError(http.statusCode)
        default: throw NetworkError.badStatusCode(http.statusCode)
        }
    }
}

// Helper for encoding Any Encodable
struct AnyEncodable: Encodable {
    private let encode: (Encoder) throws -> Void
    init(_ value: Encodable) { encode = value.encode }
    func encode(to encoder: Encoder) throws { try encode(encoder) }
}
```

---

## Mock API Client for Testing

```swift
// Mock for unit tests
final class MockAPIClient: APIClientProtocol {
    // Configure responses per endpoint path
    var responses: [String: Result<Data, Error>] = [:]
    var capturedRequests: [Endpoint] = []
    
    func fetch<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        capturedRequests.append(endpoint)
        
        guard let result = responses[endpoint.path] else {
            throw NetworkError.notFound
        }
        switch result {
        case .success(let data):
            return try JSONDecoder.default.decode(T.self, from: data)
        case .failure(let error):
            throw error
        }
    }
    
    func send(_ endpoint: Endpoint) async throws {
        capturedRequests.append(endpoint)
        if let result = responses[endpoint.path], case .failure(let error) = result {
            throw error
        }
    }
}

// Test usage
class UserServiceTests: XCTestCase {
    func testFetchUsers() async throws {
        let mock = MockAPIClient()
        let users = [User(id: 1, name: "Alice")]
        mock.responses["/users"] = .success(try JSONEncoder().encode(users))
        
        let service = UserService(client: mock)
        let result = try await service.fetchUsers(page: 1)
        
        XCTAssertEqual(result.count, 1)
        XCTAssertEqual(result[0].name, "Alice")
        XCTAssertEqual(mock.capturedRequests[0].path, "/users")
    }
}
```

---

## Environment-Based Configuration

```swift
enum APIEnvironment {
    case development
    case staging
    case production
    
    var baseURL: URL {
        switch self {
        case .development:  return URL(string: "https://dev-api.example.com/v1")!
        case .staging:      return URL(string: "https://staging-api.example.com/v1")!
        case .production:   return URL(string: "https://api.example.com/v1")!
        }
    }
    
    var isCertPinningEnabled: Bool {
        self == .production
    }
}

// Read from build configuration
extension APIEnvironment {
    static var current: APIEnvironment {
        #if DEBUG
        return .development
        #elseif STAGING
        return .staging
        #else
        return .production
        #endif
    }
}
```

---

## JSON Decoding Strategies

```swift
extension JSONDecoder {
    static var `default`: JSONDecoder {
        let decoder = JSONDecoder()
        decoder.keyDecodingStrategy = .convertFromSnakeCase  // user_name → userName
        decoder.dateDecodingStrategy = .iso8601
        return decoder
    }
    
    // Custom date strategy for non-standard formats
    static var customDate: JSONDecoder {
        let decoder = JSONDecoder()
        let formatter = DateFormatter()
        formatter.dateFormat = "yyyy-MM-dd'T'HH:mm:ssZ"
        decoder.dateDecodingStrategy = .formatted(formatter)
        return decoder
    }
}

// Polymorphic decoding — decode different types based on a field
struct Feed: Decodable {
    let items: [FeedItem]
}

enum FeedItem: Decodable {
    case post(Post)
    case ad(Advertisement)
    case story(Story)
    
    enum CodingKeys: String, CodingKey { case type }
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        let type = try container.decode(String.self, forKey: .type)
        
        switch type {
        case "post":  self = .post(try Post(from: decoder))
        case "ad":    self = .ad(try Advertisement(from: decoder))
        case "story": self = .story(try Story(from: decoder))
        default:      throw DecodingError.dataCorruptedError(forKey: .type, in: container, debugDescription: "Unknown type: \(type)")
        }
    }
}
```

---

## GraphQL Overview

```swift
// GraphQL uses POST with a query string body
struct GraphQLRequest: Encodable {
    let query: String
    let variables: [String: AnyEncodable]?
}

struct GraphQLResponse<T: Decodable>: Decodable {
    struct GraphQLError: Decodable {
        let message: String
    }
    let data: T?
    let errors: [GraphQLError]?
}

func graphQLFetch<T: Decodable>(
    query: String,
    variables: [String: AnyEncodable] = [:],
    responseType: T.Type
) async throws -> T {
    let body = GraphQLRequest(query: query, variables: variables)
    var request = URLRequest(url: graphQLEndpoint)
    request.httpMethod = "POST"
    request.httpBody = try JSONEncoder().encode(body)
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    
    let (data, _) = try await URLSession.shared.data(for: request)
    let response = try JSONDecoder().decode(GraphQLResponse<T>.self, from: data)
    
    if let errors = response.errors, !errors.isEmpty {
        throw NetworkError.graphQLErrors(errors.map(\.message))
    }
    guard let result = response.data else { throw NetworkError.noData }
    return result
}
```

---

## Interview Q&A

**Q: How do you design a testable network layer?**  
A: Define a `protocol` for the network client with `fetch` and `send` methods. The production implementation uses URLSession; tests inject a mock that returns predetermined data. This way, unit tests don't hit the network, are fast, and can test error scenarios easily.

**Q: What's the endpoint pattern and why is it better than hardcoded strings?**  
A: An endpoint struct/enum encapsulates the path, method, headers, query items, and body for each API call. It centralizes API definitions, makes them discoverable (autocomplete), ensures consistency, and makes changes easy (one place to update a path).

**Q: How would you handle API versioning?**  
A: Include the version in the base URL (`/v1/`, `/v2/`) or in a header (`Accept: application/vnd.api+json;version=2`). For incremental migration, some clients use per-endpoint version headers. The base URL approach is simpler and more widely supported.

**Q: How do you handle snake_case JSON with Swift's camelCase naming?**  
A: Set `decoder.keyDecodingStrategy = .convertFromSnakeCase`. This automatically maps `user_name` → `userName`, `created_at` → `createdAt`, etc. For edge cases or non-standard mappings, override with explicit `CodingKeys`.

**Q: What's the difference between REST and GraphQL from an iOS client perspective?**  
A: REST uses multiple endpoints (one per resource), returns fixed data shapes, and uses HTTP verbs semantically. GraphQL uses a single endpoint with POST queries, allows clients to specify exactly what data they need (avoiding over/under-fetching), and returns errors inline in the response body rather than via HTTP status codes.

---

## Quick Revision

- Endpoint pattern: struct encapsulating path, method, headers, body, query
- Protocol-based client: `APIClientProtocol` → production + mock implementations
- Mock client: returns pre-configured Data, captures requests for assertion
- Environment: `#if DEBUG / STAGING / else` to pick base URL
- `convertFromSnakeCase`: automatic snake_case → camelCase decoding
- Polymorphic decoding: `type` discriminator field → decode into enum case
- GraphQL: single POST endpoint, query in body, errors inside response not status code
