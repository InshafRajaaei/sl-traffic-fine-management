# SL Traffic Fine Management System

A full-stack system for managing and paying Sri Lanka Police traffic fines. The project is organised as four independent sub-projects that work together:

| Sub-project | Tech | Purpose |
|---|---|---|
| `backend` | Spring Boot 4 / Java 21 / MySQL | REST API, JWT auth, fine & payment logic |
| `admin-portal` | React 18 + Vite | Admin dashboard for police officials |
| `web-payment` | React 18 + Vite | Public portal for drivers to pay fines online |
| `mobile-app` | Flutter (Android) | Mobile app for roadside fine processing by officers |

---

## Architecture

```
                  ┌─────────────────────────────┐
                  │         Backend API          │
                  │  Spring Boot  •  Port 8080   │
                  │  MySQL  •  JWT Auth          │
                  └──────────┬──────────────────┘
           ┌─────────────────┼──────────────────┐
           ▼                 ▼                  ▼
  ┌──────────────┐  ┌─────────────────┐  ┌──────────────┐
  │ Admin Portal │  │  Web Payment    │  │  Mobile App  │
  │ React / Vite │  │  React / Vite   │  │  Flutter     │
  │ Port 3000    │  │  Port 5173      │  │  Android     │
  └──────────────┘  └─────────────────┘  └──────────────┘
```

- **Public endpoints** (`/api/fines/**`, `/api/payments/**`) — used by the web portal and mobile app; no authentication required.
- **Admin endpoints** (`/api/admin/**`, `/api/auth/**`) — JWT-protected; used exclusively by the admin portal.

---

## Prerequisites

| Tool | Version |
|---|---|
| Java (JDK) | 21+ |
| Apache Maven | 3.9+ (or use the included `mvnw` wrapper) |
| MySQL | 8.0+ |
| Node.js | 18+ |
| npm | 9+ |
| Flutter SDK | 3.x+ |
| Android SDK / device | For mobile-app only |

---

## Getting Started

### 1 — Backend

```bash
cd backend
```

**Configure environment**

Copy the example file and fill in your values:

```bash
cp .env.example .env
```

```env
DB_USERNAME=root
DB_PASSWORD=your_mysql_password

# Generate a Base64-encoded secret (e.g. openssl rand -base64 64)
JWT_SECRET=<your-base64-secret>
JWT_EXPIRY_MS=86400000   # 24 h (optional, defaults to 86400000)
```

The database (`traffic_fines_db`) will be created automatically on first run (`createDatabaseIfNotExist=true`). The default MySQL port used is **3307**; change `spring.datasource.url` in `src/main/resources/application.properties` if your instance runs on port 3306.

**Run**

```bash
./mvnw spring-boot:run        # Linux / macOS
mvnw.cmd spring-boot:run      # Windows
```

The API starts at **http://localhost:8080**. Sample data (fine categories, districts, dummy fines, and an admin user) is seeded automatically on first start.

---

### 2 — Admin Portal

```bash
cd admin-portal
npm install
```

Copy `.env.example` to `.env.local` (only needed if the backend runs on a different URL):

```bash
cp .env.example .env.local
# VITE_API_BASE=http://localhost:8080
```

```bash
npm run dev     # dev server — http://localhost:3000
```

**Default admin credentials**

| Username | Password |
|---|---|
| `admin` | `admin123` |

---

### 3 — Web Payment Portal

```bash
cd web-payment
npm install
```

```bash
cp .env.example .env
# VITE_API_BASE=http://localhost:8080
```

```bash
npm run dev     # dev server — http://localhost:5173
```

Drivers enter their fine reference number and category code to look up and pay a fine.

---

### 4 — Mobile App (Flutter / Android)

```bash
cd mobile-app
flutter pub get
```

Edit `lib/config.dart` and set the backend URL to your computer's local IP address (so the Android device can reach it over WiFi):

```dart
static const String apiBaseUrl = 'http://192.168.x.x:8080';
```

Run `ipconfig` (Windows) or `ifconfig` / `ip addr` (macOS/Linux) to find your local IP.

```bash
flutter run              # connected device or emulator
flutter devices          # list available devices
flutter run -d <id>      # target a specific device
```

**Build a release APK**

```bash
flutter build apk --release
# Output: build/app/outputs/flutter-apk/app-release.apk
```

---

## API Overview

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/api/auth/login` | — | Admin login — returns JWT |
| `GET` | `/api/fines/lookup?ref=&category=` | — | Look up a fine |
| `POST` | `/api/payments` | — | Pay a fine |
| `GET` | `/api/admin/report/summary` | JWT | Summary stats |
| `GET` | `/api/admin/report/by-district` | JWT | Collections by district |
| `GET` | `/api/admin/report/by-category` | JWT | Collections by category |

---

## Production Builds

```bash
# Admin portal
cd admin-portal && npm run build    # → dist/

# Web payment portal
cd web-payment  && npm run build    # → dist/

# Backend fat JAR
cd backend && ./mvnw package -DskipTests
java -jar target/backend-0.0.1-SNAPSHOT.jar
```

---

## Project Structure

```
sl-traffic-fine-management/
├── backend/          # Spring Boot REST API
│   ├── src/main/java/com/trafficfines/backend/
│   │   ├── auth/         # Login / JWT issuance
│   │   ├── security/     # JWT filter & user-details service
│   │   ├── fine/         # Fine lookup controller & service
│   │   ├── payment/      # Payment controller & service
│   │   ├── admin/        # Reporting controllers & services
│   │   ├── entity/       # JPA entities (TrafficFine, Payment, …)
│   │   ├── repository/   # Spring Data repositories
│   │   ├── notification/ # Mock SMS service
│   │   └── config/       # Security config, data seeder
│   └── src/main/resources/application.properties
├── admin-portal/     # React admin dashboard
├── web-payment/      # React public payment portal
└── mobile-app/       # Flutter Android app
    └── lib/
        ├── screens/  # lookup, fine details, confirmation
        ├── services/ # HTTP client
        ├── models/   # data models
        └── config.dart
```

---

## License

This project is provided for educational purposes.
