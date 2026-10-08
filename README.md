# MORPHO UMBRELLA 

College-project Java backend that matches the live web app.

## Where Spring Boot is used
| Part | Spring module |
|---|---|
| REST API (`controller/`) | Spring Web |
| Database access (`model/`, `repository/`) | Spring Data JPA + MySQL |
| Password hashing (BCrypt), JWT login, protected routes (`security/`) | Spring Security |
| Input validation (`dto/Dtos.java`) | Bean Validation |
| Demo Mode simulator (`service/DemoSimulator.java`) | Spring scheduling |

## Run
1. Install Java 17+, Maven and MySQL.
2. `CREATE DATABASE smart_umbrella;`
3. Edit `src/main/resources/application.properties` (DB password, JWT secret).
4. `mvn spring-boot:run` → http://localhost:8080 (tables are created from `schema.sql`).

## Architecture
```
Smart Umbrella → ESP32 → GSM/SIM → Internet → HardwareController → IngestService → MySQL → Web dashboard
                                         DemoSimulator (mock data) ──┘
```
`IngestService` is the only class that writes umbrella data. The real hardware and the demo
simulator both call it, so turning off `app.demo-mode` is the only change needed when the hardware is ready.

## API
| Method | Path | Auth | Purpose |
|---|---|---|---|
| POST | /api/auth/register | – | Create account + umbrella + primary contact |
| POST | /api/auth/login | – | Email or mobile + password → JWT |
| POST | /api/device/register | JWT | Pair umbrella, returns device key once |
| POST | /api/device/location | Device key | GPS fix `{latitude, longitude, accuracy}` |
| POST | /api/device/sos | Device key | SOS press `{latitude?, longitude?, note?}` |
| POST | /api/device/haptic | Device key | `{type: obstacle/left/right/stop/danger/emergency}` |
| POST | /api/device/heartbeat | Device key | `{battery, gsm, gps, firmware}` |
| GET | /api/device/status | JWT | Umbrella status (SIM masked) |
| GET | /api/device/location | JWT | Latest location |
| GET | /api/alerts?type= | JWT | Alert history |
| GET/POST/PUT/DELETE | /api/emergency-contacts | JWT | Manage contacts |

Device key = headers `X-Device-Id: SAU-1001` and `X-Device-Key: sau_…`. Only its SHA-256 hash is stored.

## Security
- BCrypt password hashing; JWT stateless sessions.
- Every user query is filtered by the logged-in user id (users only see their own umbrella).
- SIM numbers are masked in API responses.
- All request bodies are validated.

## Test with curl
```
curl -X POST localhost:8080/api/device/sos -H "X-Device-Id: SAU-1001" -H "X-Device-Key: sau_xxx" \
     -H "Content-Type: application/json" -d '{"latitude":13.08,"longitude":80.27}'
```
See `esp32/smart_umbrella.ino` for the hardware starter sketch.
