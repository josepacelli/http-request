# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**http-request** is a lightweight convenience library for making HTTP requests using Java's built-in `HttpURLConnection`. It provides a fluent interface to simplify HTTP operations and is intentionally designed as a single-class library with zero production dependencies. This is a modernized fork by josepacelli for Maven Central publishing.

**Key characteristics:**
- Targets Java 1.8+ (modernized from Java 1.5)
- MIT Licensed, published to Maven Central as `io.github.josepacelli:http-request:1.0.0`
- Single main class design: `HttpRequest.java` with inner static classes for configuration
- No production dependencies (significant design goal)
- Test suite uses Jetty 9.4 for realistic HTTP testing (updated from Jetty 8)

## Build & Development

**Build the project:**
```bash
mvn clean install
```

**Run all tests:**
```bash
mvn test
```

**Run a specific test class:**
```bash
mvn test -Dtest=HttpRequestTest
```

**Run a specific test method:**
```bash
mvn test -Dtest=HttpRequestTest#testMethod
```

**Build javadocs:**
```bash
mvn javadoc:javadoc
```

**Clean build artifacts:**
```bash
mvn clean
```

## Project Structure

```
http-request/
├── lib/                                      # Main library module
│   ├── src/main/java/com/github/kevinsawicki/http/
│   │   ├── HttpRequest.java                 # Main class (fluent HTTP API)
│   │   └── EncodeTest.java                  # Encoding utilities test
│   ├── src/test/java/com/github/kevinsawicki/http/
│   │   ├── HttpRequestTest.java             # Main integration tests
│   │   ├── EncodeTest.java                  # Encoding tests
│   │   └── ServerTestCase.java              # Test server utilities
│   └── pom.xml                              # Module POM
├── pom.xml                                  # Parent POM
├── README.md                                # User-facing documentation
└── LICENSE.md                               # MIT License
```

## Architecture

**Single-Class Design Philosophy:**
The library intentionally consolidates all functionality into `HttpRequest.java`. This design choice:
- Eliminates external dependencies (no Apache HttpComponents, etc.)
- Makes the library easily portable (single file can be copied into projects)
- Simplifies Android deployment where minimizing dependencies matters
- Uses inner static classes for nested functionality (e.g., `ConnectionFactory`, `UploadProgress`, `HttpRequestException`)

**Key Patterns:**
1. **Fluent Interface**: Methods chain by returning `this` or new `HttpRequest` instances
   - Example: `HttpRequest.get(url).accept("application/json").header("X-Custom", "value")`

2. **Static Factory Methods**: `HttpRequest.get()`, `HttpRequest.post()`, etc. create instances

3. **Runtime Exception Wrapping**: Low-level IOExceptions are wrapped in `HttpRequestException` (extends `RuntimeException`) to avoid checked exception burden

4. **Synchronous-Only API**: All methods block; async use requires external threading (e.g., Android `AsyncTask`)

## Testing Strategy

**Test Infrastructure:**
- Uses JUnit 4 as the test framework
- `ServerTestCase` provides an embedded Jetty HTTP server for integration tests
- Tests cover: GET/POST/multipart requests, encoding, authentication, proxies, compression, HTTPS

**Important Test Note:**
Tests require a real HTTP server (Jetty) and make actual HTTP requests. This validates behavior against realistic scenarios rather than mocking `HttpURLConnection` (which has inconsistent mock implementations across Java versions).

## Common Development Tasks

**Adding a new HTTP method:**
- Add static factory method to `HttpRequest` (e.g., `public static HttpRequest patch(String url)`)
- Add instance method for the operation (e.g., `public String body()`, `public int code()`)
- Add corresponding test case to `HttpRequestTest.java`

**Modifying request/response handling:**
- Changes likely affect the constructor, `connect()` method, or response reading logic
- Ensure both successful and error cases are tested (including non-2xx status codes)

**Releases:**
- Uses `maven-release-plugin` with Sonatype Central Publishing Plugin for Maven Central
- Current version is in `lib/pom.xml` (currently 1.0.0 for josepacelli fork)
- Maven Central coordinates: `io.github.josepacelli:http-request:1.0.0`
- Release artifacts include source JAR, javadoc JAR, and bundle manifest (via maven-bundle-plugin)
- To publish: `mvn --batch-mode -P release deploy` (requires Sonatype account, GPG key, and credentials in `~/.m2/settings.xml`)
