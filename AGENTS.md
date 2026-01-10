# AGENTS.md

This file provides guidance to Qoder (qoder.com) when working with code in this repository.

## Project Overview

newbee-mall is a Spring Boot-based e-commerce system with:
- **Frontend mall**: Customer-facing shopping interface
- **Backend admin**: Management dashboard for administrators

**Tech Stack**: Spring Boot 2.7.5, MyBatis, Thymeleaf, MySQL, jQuery, Bootstrap

## Build and Run Commands

### Start the application
```bash
mvn spring-boot:run
```

### Build the project
```bash
mvn clean package
```

### Run tests
```bash
mvn test
```

### Build without tests
```bash
mvn clean package -DskipTests
```

### Run the JAR directly
```bash
java -jar target/newbee-mall-1.0.0-SNAPSHOT.jar
```

## Database Setup

1. Create database:
```sql
CREATE DATABASE newbee_mall_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

2. Import schema:
```bash
mysql -u root -p newbee_mall_db < src/main/resources/newbee_mall_schema.sql
```

3. Configure connection in `src/main/resources/application.properties`:
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/newbee_mall_db?useUnicode=true&serverTimezone=Asia/Shanghai&characterEncoding=utf8&autoReconnect=true&useSSL=false&allowMultiQueries=true
spring.datasource.username=root
spring.datasource.password=your_password
```

## Access URLs

- **Frontend mall**: http://localhost:28089
- **Backend admin**: http://localhost:28089/admin/login
- **Default port**: 28089 (configured in application.properties)

## Architecture

### Package Structure

```
ltd.newbee.mall/
├── common/              # Constants and enums
├── config/              # Spring configuration (WebMvcConfigurer)
├── controller/          # Request handlers
│   ├── admin/           # Admin panel controllers
│   ├── mall/            # Frontend mall controllers
│   ├── common/          # Shared controllers (upload, captcha)
│   └── vo/              # View objects for responses
├── dao/                 # MyBatis mapper interfaces
├── entity/              # Database entity classes
├── interceptor/         # Login interceptors for admin and mall users
├── service/             # Business logic layer
│   └── impl/            # Service implementations
└── util/                # Utility classes (MD5, Result, Pagination)
```

### Key Architecture Patterns

1. **Three-tier architecture**: Controller → Service → DAO
2. **Session-based authentication**: User info stored in HttpSession with interceptors for access control
3. **Unified response format**: `Result` class wraps all API responses
4. **MyBatis XML mappings**: SQL queries in `src/main/resources/mapper/*.xml`
5. **Thymeleaf server-side rendering**: Templates in `src/main/resources/templates/`

### Authentication Flow

- **Mall users**: Session key `Constants.MALL_USER_SESSION_KEY` ("newBeeMallUser")
- **Admin users**: Separate session and interceptor (`AdminLoginInterceptor`)
- Login interceptors check session and redirect to login page if not authenticated
- Password hashing uses MD5 (see `MD5Util`)

### Order Status Flow

Orders follow this status progression (defined in `NewBeeMallOrderStatusEnum`):
- 0: Pending payment
- 1: Paid
- 2: Preparing for shipment
- 3: Shipped
- 4: Transaction complete
- -1: Manually closed
- -2: Timeout closed
- -3: Merchant closed

### File Upload

- Upload path configured in `Constants.FILE_UPLOAD_DIC`
- Default: `D:\upload\` (Windows) or `/opt/image/upload/` (Linux)
- Modify based on deployment environment

### Important Configuration Constants

Key business rules in `ltd.newbee.mall.common.Constants`:
- `INDEX_CAROUSEL_NUMBER = 5`: Homepage carousel items
- `SHOPPING_CART_ITEM_TOTAL_NUMBER = 13`: Max cart items
- `SHOPPING_CART_ITEM_LIMIT_NUMBER = 5`: Max quantity per item
- `GOODS_SEARCH_PAGE_LIMIT = 10`: Search results per page
- `ORDER_SEARCH_PAGE_LIMIT = 3`: Orders per page

## Development Guidelines

### Adding New Features

1. **Entity**: Create class in `entity/` package
2. **Mapper**: Add interface in `dao/` and XML in `resources/mapper/`
3. **Service**: Create interface in `service/` and implementation in `service/impl/`
4. **Controller**: Add controller in `controller/admin/` or `controller/mall/`
5. **Template**: Add Thymeleaf template in `resources/templates/`

### Code Conventions

- Use `@Resource` for dependency injection (not `@Autowired`)
- Service methods return `String` with result messages from `ServiceResultEnum`
- Controller methods return `Result` objects via `ResultGenerator`
- Use transactions with `@Transactional` at service layer for multi-step operations
- Exception handling via `NewBeeMallException` for business errors

### MyBatis Mapping

- Mapper interfaces: `ltd.newbee.mall.dao.*Mapper`
- XML files: `src/main/resources/mapper/*Mapper.xml`
- Mapper scanning configured via `@MapperScan("ltd.newbee.mall.dao")` in main application class

### Pagination

Use `PageQueryUtil` and `PageResult` for paginated queries:
- `PageQueryUtil` wraps request params (page, limit)
- `PageResult` wraps response data (list, totalCount, currPage)
