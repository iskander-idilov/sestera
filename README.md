# SESTERA — Personal Finance Tracker

**SESTERA** is a web application for managing personal finances. Track income and expenses, categorize transactions, monitor real-time stock quotes, and read the latest financial news.


---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Java 17, Spring Boot 4.0.6 |
| Security | Spring Security (BCrypt, role-based access) |
| Database | PostgreSQL 16 |
| Migrations | Liquibase |
| ORM | Spring Data JPA / Hibernate |
| Frontend | Thymeleaf + Thymeleaf Layout Dialect |
| Mapping | MapStruct |
| Boilerplate | Lombok |
| External API | Finnhub.io (stocks and news) |
| Containerization | Docker / Docker Compose |
| Build | Gradle |

---

## Features

### Authentication and authorization
- Registration and login via a Spring Security form
- Passwords hashed with BCrypt
- Role-based access model: `USER` and `ADMIN`
- `/access-denied` page for forbidden sections

### Dashboard (Home page)
- Current account balance
- Summary of total income and expenses
- Expense breakdown by category
- Stock quote widget (AAPL, GOOGL, MSFT, TSLA, AMZN) — data loaded from the Finnhub API and cached in memory on application startup

### Transactions
- Add income or expense transactions with an amount, type, and category
- Edit and delete existing transactions (editing recalculates the account balance for both the old and new amount/type)
- Pagination (10 records per page)
- Search transactions by keyword
- Account balance is recalculated automatically on every add, edit, and delete

### Categories
- Create and manage custom transaction categories

### Financial news
- Up to 20 latest financial news items from Finnhub
- Headline, short description, image, and publish time (Asia/Almaty timezone)
- Cached in memory with an asynchronous refresh on startup

### Profile settings
- Change password (current password verification + confirmation)
- Change email (password confirmation required)

### Admin panel
- View all registered users at `/admin/users`
- Restricted to users with the `ADMIN` role

---

## Project Structure

```
src/main/java/finance/
├── config/          # Security, cache warmup, MVC configuration, interceptors
├── controller/      # Web controllers (Home, Login, Register, Transactions, Categories, News, Settings, Admin)
├── dto/             # Data Transfer Objects + MapStruct mappers
├── entity/          # JPA entities (User, Account, Transaction, Category, Role, TransactionType)
├── repository/      # Spring Data repositories + custom JPQL/Criteria queries
├── security/        # CustomUserDetailsService
└── service/         # Service interfaces and implementations (Account, Transaction, Category, Stock, News)

src/main/resources/
├── db/changelog/    # Liquibase migration scripts
├── templates/       # Thymeleaf HTML templates
└── static/images/   # Static assets
```

---

## Running the Project

### Requirements
- Docker and Docker Compose
- Java 17 (for local development without Docker)
- Gradle

### Environment variables

All secrets (DB password, Finnhub API key, etc.) are stored in `.env` and are not committed to the repository.

1. Copy the example configuration:
   ```bash
   cp .env.example .env
   ```
2. Open `.env` and fill in the values (at minimum `DB_PASSWORD` and `FINNHUB_API_KEY` — get a Finnhub key at [finnhub.io](https://finnhub.io/)).
3. Start the stack:
   ```bash
   docker compose up
   ```

> To run the application from an IDE (not via Docker), export the variables from `.env` into your session/IDE environment before starting (e.g. via the EnvFile plugin in IntelliJ IDEA, or `export $(cat .env | xargs)` in a shell).

### Running with Docker (full stack)

```bash
docker-compose up --build
```

The application will be available at `http://localhost:8080`.

> **Important:** after changing source code you need to rebuild:
> ```bash
> ./gradlew build
> docker-compose up --build
> ```

### Running for development (via IDE)

```bash
# Start only the infrastructure
docker-compose up postgres

# Run the application from an IDE (IntelliJ IDEA recommended)
```

---

## Configuration

Main settings live in `application.properties`; secret values are substituted from environment variables (see `.env.example`):

| Parameter | Environment variable | Default |
|---|---|---|
| `server.port` | — | `8080` |
| `spring.datasource.url` | `DB_URL` | `jdbc:postgresql://localhost:5432/BitLab` |
| `spring.datasource.username` | `DB_USERNAME` | `postgres` |
| `spring.datasource.password` | `DB_PASSWORD` | — (required) |
| `finnhub.api.key` | `FINNHUB_API_KEY` | — (required) |
| `finnhub.news.limit` | — | `20` |

---

## Database Migrations

Migrations are managed with Liquibase and applied automatically on startup. Migration files live in:

```
src/main/resources/db/changelog/changes/
```

To run migrations manually via Gradle:

```bash
./gradlew update
```

---

## Testing

```bash
./gradlew test
```

Tests use a separate PostgreSQL instance (`localhost:5433`), described in `docker-compose.yml` (`postgres-test`), and a dedicated `src/test/resources/application.properties`.

### What's actually covered

Current automated test coverage is intentionally minimal:

- **`AppTests`** — a single Spring Boot smoke test that verifies the application context loads (`contextLoads`). Requires a running database (`postgres-test`).
- **`TransactionServiceImplTest`** — unit tests (Mockito, no Spring context) for `TransactionServiceImpl`'s balance arithmetic: adding an income/expense transaction, removing one, and editing one (verifying the balance is correctly unwound from the old amount/type and reapplied with the new values).

Not covered by automated tests: controllers (Transaction, Category, Settings, Admin, Register, Login, Home), CSV export, category management, account/user services, authentication and authorization rules, the Finnhub-backed stock and news services, and Liquibase migrations. Manual testing through the UI is currently required for these areas.

---

## External API Integration

### Finnhub.io
- **Stock quotes:** `GET /api/v1/quote?symbol={ticker}&token={key}`
  Tracked tickers: `AAPL`, `GOOGL`, `MSFT`, `TSLA`, `AMZN`
- **Financial news:** `GET /api/v1/news?category=general&token={key}`

Data is loaded asynchronously on application startup and held in a volatile in-memory cache. An `AtomicBoolean` guard prevents concurrent refresh requests.

---

## Author

**Iskander Idilov**
