# BankApp Project Documentation

## Overview
This is a Spring Boot banking web application that supports user registration, login, account dashboard, deposits, withdrawals, transfers, and transaction history.

Key technologies:
- Spring Boot
- Spring MVC
- Spring Security
- Spring Data JPA
- Thymeleaf
- MySQL

## Project startup
The application begins execution in `src/main/java/com/example/bankapp/BankappApplication.java`.

```java
public static void main(String[] args) {
    SpringApplication.run(BankappApplication.class, args);
}
```

This launches the Spring Boot application context and scans for beans, controllers, repositories, services, and configuration classes.

### Startup order / first code run
1. `BankappApplication.main()` is called.
2. `SpringApplication.run(...)` initializes Spring Boot.
3. Spring scans packages under `com.example.bankapp`.
4. Beans are created:
   - `SecurityConfig`
   - `AccountService`
   - `BankController`
   - `AccountRepository`
   - `TransactionRepository`
5. Spring Security configures the HTTP security filter chain.
6. The web server starts and listens for HTTP requests.

## Main components

### 1. Security configuration
File: `src/main/java/com/example/bankapp/config/SecurityConfig.java`

- Allows anonymous access to `/register`.
- Requires authentication for all other endpoints.
- Uses a custom login page at `/login`.
- Uses `AccountService` as the `UserDetailsService`.
- Uses `BCryptPasswordEncoder` to encode passwords.

### 2. Web controller
File: `src/main/java/com/example/bankapp/controller/BankController.java`

This controller handles the user-facing bank operations:

- `GET /register` → registration form view.
- `POST /register` → creates a new user account.
- `GET /login` → login page.
- `GET /dashboard` → displays account details for the authenticated user.
- `POST /deposit` → adds money to the authenticated account.
- `POST /withdraw` → withdraws money from the authenticated account.
- `GET /transactions` → displays transaction history.
- `POST /transfer` → sends money to another registered user.

### 3. Business logic service
File: `src/main/java/com/example/bankapp/service/AccountService.java`

`AccountService` contains core business rules and also integrates with Spring Security.

Key methods:
- `registerAccount(username, password)`
  - Checks if username exists.
  - Encodes password.
  - Sets initial balance to zero.
  - Saves the account.
- `deposit(account, amount)`
  - Updates account balance.
  - Stores a `Deposit` transaction.
- `withdraw(account, amount)`
  - Checks for sufficient balance.
  - Updates balance.
  - Stores a `Withdrawal` transaction.
- `transferAmount(fromAccount, toUsername, amount)`
  - Verifies sufficient funds.
  - Loads the recipient account.
  - Updates both accounts.
  - Saves transfer transactions for sender and receiver.
- `getTransactionHistory(account)`
  - Loads transaction records by account ID.
- `loadUserByUsername(username)`
  - Called by Spring Security during login.
  - Returns account details and granted authority.

### 4. Domain model
Files:
- `src/main/java/com/example/bankapp/model/Account.java`
- `src/main/java/com/example/bankapp/model/Transaction.java`

#### Account
- JPA entity representing a user account.
- Fields: `id`, `username`, `password`, `balance`, `transactions`.
- Implements `UserDetails` for Spring Security.
- `transactions` is a list of `Transaction` objects.

#### Transaction
- JPA entity representing a deposit, withdrawal, or transfer.
- Fields: `id`, `amount`, `type`, `timestamp`, and `account`.

### 5. Persistence
Files:
- `src/main/java/com/example/bankapp/repository/AccountRepository.java`
- `src/main/java/com/example/bankapp/repository/TransactionRepository.java`

Repositories extend `JpaRepository`.

- `AccountRepository.findByUsername(String username)`
- `TransactionRepository.findByAccountId(Long accountId)`

### 6. Configuration
File: `src/main/resources/application.properties`

The app is configured to use MySQL:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/bankappdb?useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=Test@123
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

`spring.jpa.hibernate.ddl-auto=update` instructs Hibernate to create or update database tables automatically.

## User flow and process

### Registration flow
1. User opens `/register`.
2. `BankController.showRegistrationForm()` returns `register.html`.
3. User submits username and password to `POST /register`.
4. `BankController.registerAccount(...)` calls `AccountService.registerAccount(...)`.
5. The account is saved in MySQL.
6. User is redirected to `/login`.

### Login flow
1. User opens `/login`.
2. Spring Security handles the login POST request.
3. `AccountService.loadUserByUsername(username)` loads account data.
4. If credentials match, user is redirected to `/dashboard`.

### Dashboard flow
1. Authenticated user requests `/dashboard`.
2. `BankController.dashboard(...)` reads the username from `SecurityContextHolder`.
3. The account is loaded using `accountService.findAccountByUsername(username)`.
4. The `dashboard.html` view renders account details.

### Deposit/withdraw/transfer
- For actions like deposit, withdraw, and transfer, controller methods call service-layer business logic.
- The account balance is updated and transaction records are saved.
- On success, users are redirected back to the dashboard.

### Transaction history
1. User requests `/transactions`.
2. Controller loads the authenticated account.
3. `AccountService.getTransactionHistory(account)` returns the list of past transactions.
4. `transactions.html` renders the list.

## Key files and responsibilities
- `pom.xml` — Maven dependencies and build configuration.
- `BankappApplication.java` — app startup.
- `SecurityConfig.java` — auth, login, and route security.
- `BankController.java` — web request handling.
- `AccountService.java` — business logic and security integration.
- `Account.java` — user account entity and security principal.
- `Transaction.java` — transaction entity.
- `AccountRepository.java` / `TransactionRepository.java` — database access.
- `application.properties` — database and JPA settings.

## How to run
1. Ensure MySQL is running and accessible on `localhost:3306`.
2. Create database `bankappdb` or allow Hibernate to create it.
3. Update `application.properties` if your credentials differ.
4. Run with Maven:

```bash
./mvnw spring-boot:run
```

5. Open browser at `http://localhost:8080/register`.

## Diagram
See `architecture-diagram.svg` for the architecture and request flow diagram.

## Notes
- Passwords are stored encrypted with BCrypt.
- Security is enforced on every endpoint except `/register`.
- `AccountService` is the central business-layer bean.
- The UI uses Thymeleaf templates in `src/main/resources/templates`.
