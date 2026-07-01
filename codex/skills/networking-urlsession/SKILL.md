---
name: networking-urlsession
description: >-
  Make network requests in Swift with async/await URLSession — GET/POST with
  URLRequest, JSON bodies, decoding responses into Codable models, checking
  HTTPURLResponse status codes, robust error handling, uploads/downloads, auth
  headers, and a reusable API client. Use when calling a REST/HTTP API, fetching
  or posting JSON, handling network errors, or replacing dataTask completion
  handlers on any Apple platform (iOS 26+, macOS 26+, watchOS 26+).
---

# Networking with URLSession

Use the modern `async`/`await` `URLSession` APIs. Avoid `dataTask(with:completionHandler:)` in new code.
Pair this with `foundation-essentials` (Codable) and `swift-concurrency` (Task, cancellation).

## Simple GET → decoded model

```swift
func fetchArticles() async throws -> [Article] {
    let url = URL(string: "https://api.example.com/articles")!
    let (data, response) = try await URLSession.shared.data(from: url)
    try validate(response)
    return try JSONDecoder().decode([Article].self, from: data)
}
```

## Validate the response — always

`data(from:)` does **not** throw on 4xx/5xx. You must check the status code yourself:

```swift
func validate(_ response: URLResponse) throws {
    guard let http = response as? HTTPURLResponse else {
        throw APIError.invalidResponse
    }
    guard (200..<300).contains(http.statusCode) else {
        throw APIError.http(status: http.statusCode)
    }
}
```

## POST with a JSON body, headers, auth

```swift
func createArticle(_ draft: ArticleDraft, token: String) async throws -> Article {
    var request = URLRequest(url: URL(string: "https://api.example.com/articles")!)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
    request.httpBody = try JSONEncoder().encode(draft)

    let (data, response) = try await URLSession.shared.data(for: request)
    try validate(response)
    return try JSONDecoder().decode(Article.self, from: data)
}
```

## A small reusable API client

```swift
struct APIClient {
    var baseURL: URL
    var session: URLSession = .shared
    var decoder: JSONDecoder = {
        let d = JSONDecoder(); d.keyDecodingStrategy = .convertFromSnakeCase
        d.dateDecodingStrategy = .iso8601; return d
    }()

    func get<T: Decodable>(_ path: String, as type: T.Type) async throws -> T {
        let (data, response) = try await session.data(from: baseURL.appending(path: path))
        try validate(response)
        return try decoder.decode(T.self, from: data)
    }

    func post<Body: Encodable, T: Decodable>(_ path: String, body: Body, as: T.Type) async throws -> T {
        var req = URLRequest(url: baseURL.appending(path: path))
        req.httpMethod = "POST"
        req.setValue("application/json", forHTTPHeaderField: "Content-Type")
        req.httpBody = try JSONEncoder().encode(body)
        let (data, response) = try await session.data(for: req)
        try validate(response)
        return try decoder.decode(T.self, from: data)
    }
}
```

## Typed errors

```swift
enum APIError: Error, LocalizedError {
    case invalidResponse
    case http(status: Int)
    case decoding(Error)

    var errorDescription: String? {
        switch self {
        case .invalidResponse:    return "The server response was invalid."
        case .http(let status):   return "Request failed (HTTP \(status))."
        case .decoding(let e):    return "Could not read the response: \(e.localizedDescription)"
        }
    }
}
```

Catch `URLError` for transport problems (offline, timeout) — `URLError.notConnectedToInternet`,
`.timedOut`, `.cancelled`.

## Query parameters

Build URLs safely with `URLComponents`:

```swift
var comps = URLComponents(string: "https://api.example.com/search")!
comps.queryItems = [.init(name: "q", value: query), .init(name: "page", value: "\(page)")]
let url = comps.url!
```

## Downloads & uploads (large payloads)

```swift
// Download to a file URL (streamed to disk, not memory):
let (tempURL, response) = try await URLSession.shared.download(from: bigFileURL)
try validate(response)
try FileManager.default.moveItem(at: tempURL, to: destination)

// Upload data/file:
let (data, response) = try await URLSession.shared.upload(for: request, from: bodyData)
```

## Configuration & lifecycle

```swift
let config = URLSessionConfiguration.default
config.timeoutIntervalForRequest = 30
config.waitsForConnectivity = true            // wait instead of failing immediately when offline
let session = URLSession(configuration: config)
```

Reuse a session; don't create one per request. `.shared` is fine for simple cases.

## Cancellation

Network calls participate in structured concurrency — cancelling the `Task` cancels the request and surfaces
`URLError(.cancelled)`. In SwiftUI use `.task { }` so navigating away cancels in-flight requests automatically.

## Pitfalls

- **Forgetting status validation** — a 404 returns "successful" `data`; always `validate`.
- **Decoding on the wrong shape** — print `DecodingError` to find the failing key.
- **Blocking the main actor** — `await` suspends; don't wrap calls in synchronous waits.
- **Creating a session per call** — leaks connections; reuse one.

## Quick reference

| Need | API |
| --- | --- |
| GET data | `try await session.data(from: url)` |
| Request with method/headers/body | `URLRequest` + `session.data(for: request)` |
| Validate | cast to `HTTPURLResponse`, check `statusCode` |
| Decode | `JSONDecoder().decode(T.self, from: data)` |
| Query params | `URLComponents` + `queryItems` |
| Download file | `session.download(from:)` |
| Upload | `session.upload(for:from:)` |
| Transport errors | catch `URLError` |
