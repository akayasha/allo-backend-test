# Allo Bank Backend Test - Implementation Documentation

## 📋 Project Overview

This project is a complete implementation of the Allo Bank Backend Developer Take-Home Test. It demonstrates a production-ready Spring Boot REST API that aggregates Indonesian Rupiah (IDR) exchange rate data from the Frankfurter API using advanced architectural patterns.

## 👤 Personalization

**GitHub Username:** `yourusername`
**Spread Factor:** `0.00XXX` *(calculated from username)*

> **Note:** Replace `yourusername` in `application.properties` with your actual GitHub username before running the application.

### Spread Factor Calculation

The application calculates a unique spread factor based on your GitHub username:

1. Convert GitHub username to lowercase
2. Calculate sum of Unicode values of all characters
3. Spread Factor = (Sum % 1000) / 100000.0
4. USD_BuySpread_IDR = (1 / Rate_USD) * (1 + Spread Factor)

**Example Calculation:**
```
Username: "johndoe"
Unicode Sum: j(106) + o(111) + h(104) + n(110) + d(100) + o(111) + e(101) = 743
Spread Factor: (743 % 1000) / 100000.0 = 0.00743
If USD rate = 0.00006, then USD_BuySpread_IDR = (1/0.00006) * 1.00743 = 16,790.50
```

## 🚀 Quick Start

### Prerequisites
- Java 17 or higher
- Maven 3.6 or higher
- Internet connection (for fetching data from Frankfurter API)

### Setup & Run

```bash
# 1. Clone the repository
git clone <your-repository-url>
cd allo-backend-test

# 2. Update your GitHub username in application.properties
# Edit: src/main/resources/application.properties
# Change: github.username=yourusername

# 3. Build the project
mvn clean install

# 4. Run the application
mvn spring-boot:run
```

The application will start on `http://localhost:8080` and automatically fetch all data on startup.

### Run Tests

```bash
# Run all tests
mvn test

# Run specific test class
mvn test -Dtest=LatestIDRRatesStrategyTest

# Run with coverage
mvn clean test jacoco:report
```

## 📡 API Endpoints

### Main Endpoint

```
GET /api/finance/data/{resourceType}
```

Where `{resourceType}` can be:
- `latest_idr_rates` - Latest IDR exchange rates with calculated USD buy spread
- `historical_idr_usd` - Historical IDR/USD rates (2024-01-01 to 2024-01-05)
- `supported_currencies` - List of all supported currencies

### Example cURL Commands

#### 1. Get Latest IDR Rates

```bash
curl -X GET http://localhost:8080/api/finance/data/latest_idr_rates
```

**Response:**
```json
{
  "success": true,
  "message": "Data retrieved successfully for resource type: latest_idr_rates",
  "data": {
    "base": "IDR",
    "date": "2025-12-09",
    "rates": {
      "USD": 0.00006,
      "EUR": 0.000052,
      "GBP": 0.000045,
      "JPY": 0.00938
    },
    "USD_BuySpread_IDR": 16721.17
  },
  "timestamp": "2025-12-10T14:06:41.974"
}
```

#### 2. Get Historical IDR/USD Rates

```bash
curl -X GET http://localhost:8080/api/finance/data/historical_idr_usd
```

**Response:**
```json
{
  "success": true,
  "message": "Data retrieved successfully for resource type: historical_idr_usd",
  "data": {
    "base": "IDR",
    "startDate": null,
    "endDate": null,
    "rates": {
      "2024-01-01": { "USD": 0.000064 },
      "2024-01-02": { "USD": 0.000064 },
      "2024-01-03": { "USD": 0.000064 },
      "2024-01-04": { "USD": 0.000064 },
      "2024-01-05": { "USD": 0.000064 }
    }
  },
  "timestamp": "2025-12-10T14:06:58.271"
}
```

#### 3. Get Supported Currencies

```bash
curl -X GET http://localhost:8080/api/finance/data/supported_currencies
```

**Response:**
```json
{
  "success": true,
  "message": "Data retrieved successfully for resource type: supported_currencies",
  "data": {
    "USD": "United States Dollar",
    "EUR": "Euro",
    "IDR": "Indonesian Rupiah",
    "GBP": "British Pound",
    "JPY": "Japanese Yen"
  },
  "timestamp": "2025-12-10T14:07:06.673"
}
```

#### 4. Error Handling Example

```bash
curl -X GET http://localhost:8080/api/finance/data/invalid_type
```

**Response:**
```json
{
  "success": false,
  "message": "Resource not found",
  "error": "Resource type 'invalid_type' not found",
  "timestamp": "2025-12-10T14:10:00.123"
}
```

## 🏗️ Architecture Overview

### Design Patterns Implemented

#### 1. **Strategy Pattern** (Constraint A)
- **Interface:** `IDRDataFetcher`
- **Implementations:**
    - `LatestIDRRatesStrategy` - Fetches latest rates and calculates USD buy spread
    - `HistoricalIDRUSDStrategy` - Fetches historical IDR/USD rates
    - `SupportedCurrenciesStrategy` - Fetches supported currencies list
- **Benefits:** Easy extensibility, clean separation of concerns, testability

#### 2. **Factory Bean Pattern** (Constraint B)
- **Implementation:** `WebClientFactoryBean` implements `FactoryBean<WebClient>`
- **Features:**
    - Externalizes API base URL via `@Value`
    - Configures timeouts and default headers
    - Provides centralized client configuration
- **Location:** `src/main/java/com/allo/backend/factory/WebClientFactoryBean.java`

#### 3. **Application Runner** (Constraint C)
- **Implementation:** `DataInitializationRunner` implements `ApplicationRunner`
- **Features:**
    - Fetches all data exactly once on startup
    - Loads data into thread-safe in-memory store
    - Ensures data is immutable after initialization
- **Location:** `src/main/java/com/allo/backend/runner/DataInitializationRunner.java`

#### 4. **In-Memory Data Store**
- **Implementation:** `InMemoryDataStore` service
- **Features:**
    - Thread-safe using `ConcurrentHashMap`
    - Immutable data access via defensive copying
    - Singleton scope ensures single instance
- **Location:** `src/main/java/com/allo/backend/service/InMemoryDataStore.java`

### Project Structure

```
src/
├── main/
│   ├── java/com/allo/backend/
│   │   ├── controller/              # REST controllers
│   │   │   └── FinanceController.java
│   │   ├── strategy/                # Strategy pattern implementations
│   │   │   ├── IDRDataFetcher.java  (interface)
│   │   │   ├── LatestIDRRatesStrategy.java
│   │   │   ├── HistoricalIDRUSDStrategy.java
│   │   │   └── SupportedCurrenciesStrategy.java
│   │   ├── service/                 # Business logic
│   │   │   ├── FinanceDataService.java
│   │   │   └── InMemoryDataStore.java
│   │   ├── factory/                 # Factory beans
│   │   │   └── WebClientFactoryBean.java
│   │   ├── runner/                  # Application runners
│   │   │   └── DataInitializationRunner.java
│   │   ├── model/                   # Domain models
│   │   │   ├── LatestRates.java
│   │   │   └── HistoricalRates.java
│   │   ├── dto/                     # Data transfer objects
│   │   │   ├── ApiResponse.java
│   │   │   ├── LatestRatesDTO.java
│   │   │   └── HistoricalRatesDTO.java
│   │   └── exception/               # Exception handling
│   │       ├── ExternalApiException.java
│   │       ├── ResourceNotFoundException.java
│   │       └── GlobalExceptionHandler.java
│   └── resources/
│       └── application.properties
└── test/
    └── java/com/allo/backend/
        ├── strategy/                # Strategy unit tests
        │   ├── LatestIDRRatesStrategyTest.java
        │   ├── HistoricalIDRUSDStrategyTest.java
        │   └── SupportedCurrenciesStrategyTest.java
        └── controller/              # Integration tests
            └── FinanceControllerIntegrationTest.java
```

## 🛠️ Architectural Rationale

### 1. Polymorphism Justification: Why Strategy Pattern?

The Strategy Pattern was chosen over simple conditional blocks (if/else or switch statements) for several critical reasons:

**Extensibility:**
- **Open/Closed Principle:** Adding new resource types (e.g., `crypto_rates`, `bond_rates`) requires only creating a new strategy class implementing `IDRDataFetcher` interface
- **Zero Modification:** No changes needed to existing controller, service, or other strategy implementations
- **Plugin Architecture:** New strategies can be added dynamically without recompiling existing code
- **Future-Proof:** Easy to add features like caching, rate limiting, or authentication per resource type

**Maintainability:**
- **Single Responsibility:** Each strategy class has one clear purpose - fetch and transform specific data
- **Isolation:** Changes to one resource type don't affect others
- **Reduced Complexity:** Eliminates large conditional blocks, reducing cyclomatic complexity from O(n) to O(1)
- **Clear Dependencies:** Each strategy explicitly declares its dependencies via constructor injection

**Testability:**
- **Unit Testing:** Each strategy can be tested in complete isolation with mocked dependencies
- **Mock Injection:** Easy to inject mock WebClient for testing without affecting other strategies
- **Coverage:** Individual strategies can achieve 100% test coverage independently
- **Test Clarity:** Tests are focused and easy to understand

**Code Organization:**
- **Map-Based Lookup:** Spring auto-wires all strategies into a `Map<String, IDRDataFetcher>` keyed by resource type
- **No Conditionals:** Controller uses direct map lookup: `strategies.get(resourceType).fetchData()`
- **Type Safety:** Compile-time validation ensures all strategies implement the interface correctly
- **Self-Documenting:** Strategy names clearly indicate their purpose

**Real-World Example:**
```java
// Without Strategy Pattern (Bad)
public Object getFinanceData(String type) {
    if (type.equals("latest_idr_rates")) {
        // 50 lines of code
    } else if (type.equals("historical_idr_usd")) {
        // 40 lines of code
    } else if (type.equals("supported_currencies")) {
        // 30 lines of code
    }
    // Adding new type requires modifying this method
}

// With Strategy Pattern (Good)
public Object getFinanceData(String type) {
    return strategies.get(type).fetchData(); // 1 line, extensible
}
```

### 2. Client Factory: Why FactoryBean for WebClient?

Using `FactoryBean<WebClient>` implementation provides significant advantages over a standard `@Bean` method:

**Advanced Lifecycle Management:**
- **Complex Initialization:** FactoryBean provides hooks like `afterPropertiesSet()` for complex setup logic
- **Lazy Loading:** Can control when the WebClient is actually instantiated vs when the factory bean is created
- **Singleton vs Prototype:** Built-in support for managing object scope and lifecycle
- **Dependency Ordering:** Clear control over initialization order relative to other beans

**Configuration Externalization:**
- **Property Injection:** Clean separation of configuration (`@Value` annotations) from client creation logic
- **Profile Support:** Easy to create different client configurations for dev, test, prod environments
- **Runtime Configuration:** Can modify client behavior based on runtime conditions or feature flags
- **Environment-Aware:** Access to `Environment` bean for complex conditional configuration

**Production Features:**
- **Connection Pooling:** Centralized place to configure connection pool size, timeouts, keep-alive
- **Resilience Patterns:** Easy to add retry logic, circuit breakers, fallbacks at factory level
- **Monitoring:** Can instrument client creation with metrics, logging, distributed tracing
- **Security:** Centralized place for SSL/TLS configuration, certificate management, authentication headers

**Testability:**
- **Mock Factory:** In tests, replace factory bean with one that creates mock/stub clients
- **Profile Override:** Use `@Profile("test")` to provide test-specific factory implementation
- **Integration Testing:** Easy to provide real client in integration tests vs mock in unit tests
- **Consistent Behavior:** All strategies use the same client instance, ensuring consistency

**Real-World Scenario:**
```java
// Production FactoryBean creates WebClient with:
// - 30-second timeout
// - Connection pooling (max 500 connections)
// - Circuit breaker (fail after 5 consecutive errors)
// - Retry logic (3 attempts with exponential backoff)
// - Request/response logging
// - Distributed tracing headers

// Test FactoryBean creates WebClient with:
// - 5-second timeout
// - No connection pooling
// - No retries
// - Detailed logging for debugging
```

**Why Not @Bean?**
- `@Bean` methods are simpler but lack the flexibility for complex initialization logic
- FactoryBean makes it explicit that WebClient is a "managed resource" with special lifecycle needs
- Aligns with Spring's philosophy for creating complex, configurable beans
- Better separation of concerns: factory bean handles "how to create" vs "what to do with it"

### 3. Startup Runner Choice: ApplicationRunner vs @PostConstruct

`ApplicationRunner` was chosen over `@PostConstruct` for data initialization due to critical architectural considerations:

**Application Lifecycle Awareness:**
- **Full Context Initialization:** ApplicationRunner executes AFTER the entire Spring context is fully loaded and ready
- **Bean Availability:** Guarantees all beans (WebClient, strategies, data store) are initialized and injected
- **Startup Order:** `@PostConstruct` runs during bean creation, which may be too early - other required beans might not exist yet
- **Application Ready:** Ensures the application is in a consistent, ready state before fetching data

**Graceful Error Handling:**
- **Startup Failures:** If external API is down, ApplicationRunner can fail the entire application startup (fail-fast)
- **Retry Strategy:** Can implement sophisticated retry logic with delays before declaring failure
- **Fallback Options:** Can load cached/backup data if external API is unavailable
- **User Feedback:** Clear error messages indicate startup failure vs runtime failure

**Production Operations:**
- **Health Checks:** Integration with Spring Boot Actuator health indicators
- **Readiness Probes:** In Kubernetes/Cloud environments, container only becomes "ready" after runner succeeds
- **Liveness Probes:** Separate application "alive" (process running) from "ready" (data loaded)
- **Graceful Degradation:** Can start application with empty cache and retry data loading in background

**Command-Line Integration:**
- **Application Arguments:** Access to `ApplicationArguments` for parsing command-line flags
- **Flexible Execution:** Can add `--skip-data-load` flag for development or testing
- **Environment-Specific Behavior:** Different behavior in dev vs prod based on arguments or profiles
- **Scripting:** Easy to control runner behavior via environment variables or startup scripts

**Blocking Semantics:**
- **Explicit Blocking:** ApplicationRunner clearly signals "this blocks application startup"
- **No Surprises:** Developers know data loading must complete before app serves traffic
- **Predictable State:** Application is in consistent state when it starts accepting requests
- **Avoid Race Conditions:** No risk of API requests arriving before data is loaded

**Testability:**
- **Disable in Tests:** Easy to disable runner in unit tests using `@ConditionalOnProperty`
- **Integration Tests:** Can enable runner in integration tests to verify full startup sequence
- **Mock Data:** Test profiles can skip external API calls and load mock data instead
- **Startup Time:** Can measure and test application startup performance including data loading

**Real-World Production Scenario:**
```
Container Startup Sequence:
1. JVM starts
2. Spring context initializes (all beans created)
3. @PostConstruct methods run (too early for data loading!)
4. ApplicationRunner executes (perfect time!)
   - Attempts to fetch data from external API
   - If successful: Data stored, app becomes "ready"
   - If failure: Container restart (Kubernetes), alert (monitoring)
5. Readiness probe passes
6. Load balancer adds container to pool
7. Application starts receiving traffic

Benefits:
- No requests hit app before data is loaded
- Failed data loads prevent broken app from serving traffic
- Clear separation between "app started" and "app ready"
- Easy to debug startup issues
```

**Why Not @PostConstruct?**
- `@PostConstruct` runs during bean initialization, before full application context is ready
- No access to application arguments or environment configuration
- Harder to disable in specific profiles or tests
- Doesn't integrate with container orchestration readiness probes
- Less explicit about blocking startup sequence
- Cannot gracefully handle external API failures during startup

##  Testing

### Test Coverage

- **Unit Tests:** All three strategy implementations with mocked WebClient
- **Integration Tests:** Full application startup and data loading verification
- **Edge Cases:** Error handling, null responses, network failures
- **Calculation Tests:** Spread factor calculation with various usernames

### Running Tests

```bash
# All tests
mvn test

# Specific strategy tests
mvn test -Dtest=LatestIDRRatesStrategyTest
mvn test -Dtest=HistoricalIDRUSDStrategyTest
mvn test -Dtest=SupportedCurrenciesStrategyTest

# Integration tests
mvn test -Dtest=FinanceControllerIntegrationTest

# With coverage report
mvn clean test jacoco:report
```

### Test Results

All tests pass successfully:
- `LatestIDRRatesStrategyTest` - Spread calculation and data transformation
- `HistoricalIDRUSDStrategyTest` - Historical data fetching
- `SupportedCurrenciesStrategyTest` - Currency list fetching
- `FinanceControllerIntegrationTest` - End-to-end API testing

##  Key Features

### Production-Ready Features

1. **Error Handling**
    - Global exception handler with `@ControllerAdvice`
    - Custom exceptions for different error scenarios
    - Graceful degradation for API failures

2. **Configuration Management**
    - Externalized configuration via `application.properties`
    - Environment-specific settings
    - Type-safe property binding

3. **Thread Safety**
    - `ConcurrentHashMap` for in-memory storage
    - Immutable data access patterns
    - No shared mutable state

4. **Logging**
    - Structured logging with SLF4J
    - Request/response logging
    - Debug-level spread factor calculation logs

5. **API Design**
    - RESTful endpoint design
    - Consistent response format (`ApiResponse<T>`)
    - Proper HTTP status codes

## 🔧 Configuration

### Application Properties

```properties
# Server Configuration
server.port=8080

# Frankfurter API Configuration
frankfurter.api.base-url=https://api.frankfurter.app
frankfurter.api.timeout=10000

# GitHub Username (MUST BE CHANGED)
github.username=yourusername

# Logging
logging.level.com.allo.backend=DEBUG
```

### Important Configuration Notes

1. **Change GitHub Username:** Update `github.username` in `application.properties` before running
2. **API Timeout:** Default 10 seconds, adjust based on network conditions
3. **Port Configuration:** Default 8080, change if port is already in use

## Troubleshooting

### Common Issues

1. **Application Won't Start**
    - Check if port 8080 is available: `netstat -ano | findstr :8080` (Windows) or `lsof -i :8080` (Linux/Mac)
    - Verify internet connection for API access
    - Check application logs for detailed error messages

2. **Tests Failing**
    - Ensure stable internet connection for integration tests
    - Check if GitHub username is set in `application.properties`
    - Run tests individually to identify specific failures

3. **Data Not Loading**
    - Check logs for "Data initialization" messages
    - Verify Frankfurter API is accessible: `curl https://api.frankfurter.app/latest?base=IDR`
    - Application fetches data only once on startup - restart to refresh

4. **Spread Factor Incorrect**
    - Verify GitHub username is correctly set
    - Check logs for spread factor calculation details
    - Ensure username is lowercase in calculation

## License

This project is part of the Allo Bank technical assessment.

---

**Implementation Date:** December 2025
**Spring Boot Version:** 3.2.0
**Java Version:** 17

