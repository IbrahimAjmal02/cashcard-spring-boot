# Family Cash Card API

A REST API for managing cash cards, built with **Spring Boot**, **Spring Data**, and **Spring Security**, and developed test-first (TDD).

Each authenticated user can create, view, update, and delete **only their own** cash cards.

> Built while completing Spring Academy's *Building a REST API with Spring Boot* course.

---

## Tech Stack

| Area | Technology |
|---|---|
| Language | Java |
| Framework | Spring Boot (Spring Web / Spring MVC) |
| Data access | Spring Data repositories (`CrudRepository`, `PagingAndSortingRepository`) |
| Database | H2 (embedded, in-memory) |
| Security | Spring Security: HTTP Basic auth, BCrypt password hashing, owner-based access control |
| Testing | JUnit 5, AssertJ, `TestRestTemplate`, JsonPath, `@JsonTest` |
| Build | Gradle (wrapper included) |

---

## API Endpoints

All endpoints require HTTP Basic authentication. Users can only access cards they own.

| Method | Endpoint | Description | Success | Errors |
|---|---|---|---|---|
| `GET` | `/cashcards/{id}` | Get one cash card | `200 OK` + card | `404` if missing or not yours |
| `GET` | `/cashcards` | List your cash cards (paged and sortable) | `200 OK` + array | |
| `POST` | `/cashcards` | Create a cash card | `201 Created` + `Location` header | |
| `PUT` | `/cashcards/{id}` | Replace a cash card's amount | `204 No Content` | `404` if missing or not yours |
| `DELETE` | `/cashcards/{id}` | Delete a cash card | `204 No Content` | `404` if missing or not yours |

### Paging and sorting

`GET /cashcards` supports query parameters:

```
/cashcards?page=0&size=10&sort=amount,desc
```

- `page`: page number, starting at 0
- `size`: cards per page
- `sort`: `field,direction` (`asc` or `desc`); defaults to amount ascending

### Example card

```json
{
  "id": 99,
  "amount": 123.45,
  "owner": "sarah1"
}
```

When creating a card, only `amount` is taken from the request. The server generates the `id`
and sets the `owner` to the logged-in user, so clients can't overwrite other cards or create
cards in someone else's name.

---

## Getting Started

### Prerequisites

- **Java 17 or newer** (check with `java -version`)
- Git

You don't need to install Gradle or a database. The Gradle wrapper and the embedded H2 database are included.

### Clone the repository

```bash
git clone https://github.com/IbrahimAjmal02/cashcard-spring-boot.git
cd cashcard-spring-boot
```

### Run the tests

```bash
./gradlew test
```

On Windows, use `gradlew.bat test`.

### Run the app

```bash
./gradlew bootRun
```

The API starts at `http://localhost:8080`. The in-memory database starts empty each time the app runs.

### Try it out

A test user is configured in `SecurityConfig`: **`sarah1`** / **`abc123`**.

Create a card:

```bash
curl -i -u sarah1:abc123 \
  -H "Content-Type: application/json" \
  -d '{"amount": 250.00}' \
  http://localhost:8080/cashcards
```

The response includes a `Location` header such as `/cashcards/1`. Fetch it:

```bash
curl -u sarah1:abc123 http://localhost:8080/cashcards/1
```

List your cards, highest amount first:

```bash
curl -u sarah1:abc123 "http://localhost:8080/cashcards?page=0&size=10&sort=amount,desc"
```

Update a card:

```bash
curl -i -u sarah1:abc123 -X PUT \
  -H "Content-Type: application/json" \
  -d '{"amount": 300.00}' \
  http://localhost:8080/cashcards/1
```

Delete a card:

```bash
curl -i -u sarah1:abc123 -X DELETE http://localhost:8080/cashcards/1
```

---

## Project Structure

```
src/
├── main/java/example/cashcard/
│   ├── CashCardApplication.java   # Entry point
│   ├── CashCard.java              # Data model (record)
│   ├── CashCardController.java    # REST endpoints
│   ├── CashCardRepository.java    # Spring Data repository
│   └── SecurityConfig.java        # Security filter chain, users, password encoder
├── main/resources/
│   ├── application.properties
│   └── schema.sql                 # Creates the CASH_CARD table
└── test/java/example/cashcard/
    ├── CashCardApplicationTests.java  # Integration tests for every endpoint
    └── CashCardJsonTest.java          # JSON serialization tests
```

---

## What I Learned

- **Spring Boot fundamentals:** auto-configuration, component scanning, and dependency injection (constructor injection, `@Configuration` and `@Bean`)
- **REST design decisions:** POST vs. PUT for create, PUT vs. PATCH for update, correct status codes (`201`, `204`, `404`), and the `Location` header
- **Spring Data:** built-in CRUD methods, derived query methods (`findByIdAndOwner`), and paging/sorting with `Pageable`
- **Spring Security:** the filter chain, HTTP Basic auth, BCrypt hashing, ownership checks with `Principal`, and when it's reasonable to disable CSRF for non-browser API clients
- **Test-driven development:** writing failing tests first, integration testing with `TestRestTemplate`, and isolating tests that change data with `@DirtiesContext`

---

## Possible Next Steps

- Replace H2 with PostgreSQL (via Docker) and manage the schema with Flyway migrations
- Replace in-memory users with JWT or Keycloak-based authentication
- Add request validation and OpenAPI/Swagger documentation
- Build a frontend (e.g., Angular) that consumes the API

---

## License

Apache 2.0. See [LICENSE.txt](LICENSE.txt).
