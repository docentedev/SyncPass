# SyncPass

MVP académico — DSY1105 Desarrollo de Aplicaciones Móviles, Duoc UC.
Digitaliza la acreditación de asistencia en actividades comunitarias (QR + offline-first) en reemplazo de planillas en papel.

## Stack

Kotlin, Android Studio, Jetpack Compose, Material Design 3, MVVM, Room/SQLite, Retrofit + API REST Spring Boot. Min SDK 28. Ver restricciones en `docs/caso.md`.

## Estructura

```
app/src/main/java/com/rojas/syncpass/
  core/config/      # AppConfig (lee BuildConfig, nunca URL en duro)
  data/remote/      # AuthApiService, DTOs, RetrofitClient
  data/repository/  # AuthRepository
  ui/components/    # AuthCard (reutilizable, recibe formulario como children)
  ui/login/         # LoginScreen, LoginViewModel, LoginUiState
docs/
  caso.md           # caso transcrito desde caso.pdf
  api.md            # contrato POST api/auth/login
  01_login.md       # guía paso a paso login + conceptos + troubleshooting
```

## Puesta en marcha

1. Clonar y abrir en Android Studio.
2. Copiar entorno:
   ```bash
   cp .env.example .env
   ```
   Edita `API_BASE_URL` en `.env`. Nunca commitees `.env`.
3. SDK necesario: compilar contra API 37 (ver troubleshooting en `docs/01_login.md` si ves error AAR metadata).
4. Sync Gradle + Run en emulador/dispositivo con internet.

## Contrato API

```bash
curl -X POST https://172-233-15-71.ip.linodeusercontent.com/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"juan_perez","password":"Password123"}'
```

Detalle en `docs/api.md`. La URL se inyecta vía `.env` → `BuildConfig.API_BASE_URL` → `AppConfig`; ningún `.kt` la contiene en duro.

## Reglas

- Solo datos 100% ficticios en código, previews, capturas y repo.
- Sin credenciales/tokens reales ni accesos a sistemas internos.
- Guía de construcción: `docs/01_login.md`.
