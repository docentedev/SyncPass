# 01 — Login paso a paso (Compose + MVVM + Retrofit + .env)

> Objetivo: construir un formulario de login funcional que llegue a lo definido en `docs/api.md`.
> Restricciones y contexto: ver `docs/caso.md` §3.3, §3.4, §5.1 (MVP académico, datos 100% ficticios, stack Kotlin + Compose + M3 + MVVM + Room/SQLite + Retrofit, API 26+, 48x48dp, claro/oscuro).

## 0. A dónde hay que llegar (base: `api.md`)

```bash
curl -X POST https://172-233-15-71.ip.linodeusercontent.com/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"juan_perez","password":"Password123"}'
# -> 200 {access_token, token_type: "Bearer", expires_in: 3600, user: {...}}
```

La app debe hacer exactamente ese `POST`:

- `BASE_URL = https://172-233-15-71.ip.linodeusercontent.com/`
- `POST api/auth/login`
- body: `{ "username": "<username o email>", "password": "<password>" }`
- respuesta 200: `access_token`, `token_type`, `expires_in`, `user`

Regla innegociable de esta guía:

- **La URL jamás queda en duro en el source code.** Viene desde `.env` → `BuildConfig.API_BASE_URL` → `AppConfig`.
- Usa solo datos ficticios. Ej: `juan_perez / Password123`.

Buenas prácticas que aplica esta guía:

1. MVVM: `Screen (Compose, tonta)` → `ViewModel (estado + eventos)` → `Repository` → `Retrofit`.
2. Estado unidireccional con `StateFlow + LoginUiState` sellado o data class inmutable.
3. Componente reutilizable `AuthCard(content: @Composable ColumnScope.() -> Unit)` — el formulario va como `children` (slot API).
4. Validación en ViewModel, no en el Composable.
5. Accesibilidad caso §3.3: áreas táctiles ≥48dp, contraste, modo claro/oscuro, feedback visual inmediato, lenguaje simple.

---

## Paso 1 — Dependencias

Archivo: `gradle/libs.versions.toml`

Agrega versiones (valores sugeridos por Android Studio, oct-2026):

```toml
[versions]
# ... las que ya tienes ...
retrofit = "3.0.0"
okhttpLogging = "5.5.0"
gsonConverter = "3.0.0"
lifecycleViewmodelCompose = "2.11.0"
coroutines = "1.11.0"

[libraries]
# ... las que ya tienes ...
retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
retrofit-gson = { group = "com.squareup.retrofit2", name = "converter-gson", version.ref = "gsonConverter" }
okhttp-logging = { group = "com.squareup.okhttp3", name = "logging-interceptor", version.ref = "okhttpLogging" }
lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "lifecycleViewmodelCompose" }
coroutines-core = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-core", version.ref = "coroutines" }
coroutines-android = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-android", version.ref = "coroutines" }
```

Archivo: `app/build.gradle.kts`

```kotlin
android {
    // ...
    buildFeatures {
        compose = true
        buildConfig = true // requerido para exponer API_BASE_URL sin hardcodear
    }
}

dependencies {
    // ... las que ya tienes ...
    implementation(libs.retrofit)
    implementation(libs.retrofit.gson)
    implementation(libs.okhttp.logging)
    implementation(libs.lifecycle.viewmodel.compose)
    implementation(libs.coroutines.core)
    implementation(libs.coroutines.android)
}
```

Sincroniza Gradle: `Sync Now`.

> Por qué así: Retrofit + Gson es lo pedido en caso §5.1 (`Retrofit`). Logging solo en debug para no filtrar tokens. `lifecycle-viewmodel-compose` permite `viewModel()` en Compose sin Hilt (suficiente para MVP).

---

## Paso 2 — Variables de entorno con `.env` (nada en duro)

### 2.1 Crea `.env` (no se commitea)

En la raíz del proyecto (`SyncPass/.env`):

```dotenv
API_BASE_URL=https://172-233-15-71.ip.linodeusercontent.com/
```

### 2.2 Crea `.env.example` (sí se commitea)

`SyncPass/.env.example`:

```dotenv
# Copiar a .env y completar. Nunca commitear .env con valores reales.
API_BASE_URL=https://TU_HOST_O_IP/
```

### 2.3 Ignora `.env`

Agrega a `SyncPass/.gitignore` (y verifica `app/.gitignore`):

```gitignore
.env
```

### 2.4 Expón `.env` como `BuildConfig.API_BASE_URL`

Edita `app/build.gradle.kts` — arriba del bloque `android { }`:

```kotlin
import java.util.Properties

val envFile = rootProject.file(".env")
val envProps = Properties()
if (envFile.exists()) {
    envFile.inputStream().use { envProps.load(it) }
}
// Prioridad: .env > variable de entorno del sistema > fallback solo para compilar
val apiBaseUrl: String =
    envProps.getProperty("API_BASE_URL")
        ?: System.getenv("API_BASE_URL")
        ?: "https://172-233-15-71.ip.linodeusercontent.com/"

android {
    namespace = "com.rojas.syncpass"
    // ...
    defaultConfig {
        // ...
        buildConfigField("String", "API_BASE_URL", "\"$apiBaseUrl\"")
    }
}
```

### 2.5 Accede desde código con un solo punto

Nuevo archivo: `app/src/main/java/com/rojas/syncpass/core/config/AppConfig.kt`

```kotlin
package com.rojas.syncpass.core.config

import com.rojas.syncpass.BuildConfig

object AppConfig {
    val apiBaseUrl: String = BuildConfig.API_BASE_URL.trim().let {
        if (it.endsWith("/")) it else "$it/"
    }
}
```

> Buena práctica: todo el código usa `AppConfig.apiBaseUrl`. Si ves `"https://..."` o `"172-233..."` en un `.kt`, está mal. Busca con: `rg -n "172-233|https?://" app/src/main/java --glob '*.kt'`

---

## Paso 3 — Estructura MVVM que vamos a crear

Crea estos paquetes/archivos (todo ficticio, sin datos reales):

```
com.rojas.syncpass/
  core/config/AppConfig.kt              // paso 2.5
  data/remote/
    AuthApiService.kt                   // POST api/auth/login
    AuthDtos.kt                         // LoginRequest / LoginResponse / UserDto
    RetrofitClient.kt                   // usa AppConfig.apiBaseUrl
  data/repository/
    AuthRepository.kt                   // expone Result<LoginResponse>
  ui/components/
    AuthCard.kt                         // Card reutilizable con slot content
  ui/login/
    LoginUiState.kt                     // estado inmutable
    LoginViewModel.kt                   // validación + llamada a repo
    LoginScreen.kt                      // formulario como children de AuthCard
```

Permiso de internet — `app/src/main/AndroidManifest.xml`:

```xml
<manifest ...>
    <uses-permission android:name="android.permission.INTERNET" />
    <application ...>
```

---

## Paso 4 — DTOs + API (contrato de `api.md`)

Archivo: `data/remote/AuthDtos.kt`

```kotlin
package com.rojas.syncpass.data.remote

data class LoginRequest(
    val username: String,
    val password: String
)

data class LoginResponse(
    val access_token: String,
    val token_type: String = "Bearer",
    val expires_in: Long = 3600,
    val user: UserDto? = null
)

data class UserDto(
    val id: Int? = null,
    val username: String? = null,
    val email: String? = null
)
```

Archivo: `data/remote/AuthApiService.kt`

```kotlin
package com.rojas.syncpass.data.remote

import retrofit2.http.Body
import retrofit2.http.POST

interface AuthApiService {
    @POST("api/auth/login")
    suspend fun login(@Body body: LoginRequest): LoginResponse
}
```

Archivo: `data/remote/RetrofitClient.kt`

```kotlin
package com.rojas.syncpass.data.remote

import com.rojas.syncpass.BuildConfig
import com.rojas.syncpass.core.config.AppConfig
import okhttp3.OkHttpClient
import okhttp3.logging.HttpLoggingInterceptor
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

object RetrofitClient {
    val authApi: AuthApiService by lazy {
        val logging = HttpLoggingInterceptor().apply {
            level = if (BuildConfig.DEBUG)
                HttpLoggingInterceptor.Level.BODY
            else
                HttpLoggingInterceptor.Level.NONE
        }
        val client = OkHttpClient.Builder()
            .addInterceptor(logging)
            .build()

        Retrofit.Builder()
            .baseUrl(AppConfig.apiBaseUrl) // <- nunca hardcodeada
            .client(client)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(AuthApiService::class.java)
    }
}
```

Archivo: `data/repository/AuthRepository.kt`

```kotlin
package com.rojas.syncpass.data.repository

import com.rojas.syncpass.data.remote.AuthApiService
import com.rojas.syncpass.data.remote.LoginRequest
import com.rojas.syncpass.data.remote.LoginResponse

class AuthRepository(private val api: AuthApiService) {
    suspend fun login(username: String, password: String): Result<LoginResponse> =
        runCatching {
            api.login(LoginRequest(username.trim(), password))
        }
}
```

---

## Paso 5 — `AuthCard`: Card reutilizable que recibe el formulario como children

Archivo: `ui/components/AuthCard.kt`

Requisito del caso §3.3: botón/área ≥48dp, legible con luz variable, claro/oscuro, feedback inmediato. Por eso `AuthCard` centraliza estilo y el formulario entra por slot.

```kotlin
package com.rojas.syncpass.ui.components

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.ColumnScope
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Card
import androidx.compose.material3.CardDefaults
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import com.rojas.syncpass.ui.theme.SyncPassTheme

/**
 * Card reutilizable para auth. El formulario se pasa como [content] (children).
 * No sabe nada de login: solo layout + estilo.
 */
@Composable
fun AuthCard(
    title: String,
    subtitle: String? = null,
    modifier: Modifier = Modifier,
    content: @Composable ColumnScope.() -> Unit
) {
    Card(
        modifier = modifier.fillMaxWidth(),
        elevation = CardDefaults.cardElevation(defaultElevation = 4.dp),
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.surfaceContainerHigh
        )
    ) {
        Column(
            modifier = Modifier
                .fillMaxWidth()
                .padding(20.dp),
            horizontalAlignment = Alignment.CenterHorizontally,
            verticalArrangement = Arrangement.spacedBy(12.dp)
        ) {
            Text(
                text = title,
                style = MaterialTheme.typography.headlineSmall
            )
            if (!subtitle.isNullOrBlank()) {
                Text(
                    text = subtitle,
                    style = MaterialTheme.typography.bodyMedium,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
            Spacer(Modifier.height(4.dp))
            content() // <- aquí entra el formulario hijo
        }
    }
}

@Preview(showBackground = true)
@Preview(showBackground = true, uiMode = android.content.res.Configuration.UI_MODE_NIGHT_YES)
@Composable
private fun AuthCardPreview() {
    SyncPassTheme {
        AuthCard(
            title = "SyncPass",
            subtitle = "Vista previa con contenido ficticio"
        ) {
            Text("Aquí va el formulario hijo")
        }
    }
}
```

> Patrón slot: `content: @Composable ColumnScope.() -> Unit` permite que `LoginScreen`, `RegisterScreen`, `ForgotPasswordScreen` reutilicen el mismo Card sin duplicar estilo.

---

## Paso 6 — Estado + ViewModel (validación aquí, no en la UI)

Archivo: `ui/login/LoginUiState.kt`

```kotlin
package com.rojas.syncpass.ui.login

data class LoginUiState(
    val username: String = "",
    val password: String = "",
    val isPasswordVisible: Boolean = false,
    val isLoading: Boolean = false,
    val usernameError: String? = null,
    val passwordError: String? = null,
    val loginError: String? = null,
    val isSuccess: Boolean = false
) {
    val canSubmit: Boolean =
        username.isNotBlank() && password.length >= 6 && !isLoading
}
```

Archivo: `ui/login/LoginViewModel.kt`

```kotlin
package com.rojas.syncpass.ui.login

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.rojas.syncpass.data.remote.RetrofitClient
import com.rojas.syncpass.data.repository.AuthRepository
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.update
import kotlinx.coroutines.launch

class LoginViewModel(
    private val repository: AuthRepository = AuthRepository(RetrofitClient.authApi)
) : ViewModel() {

    private val _uiState = MutableStateFlow(LoginUiState())
    val uiState: StateFlow<LoginUiState> = _uiState.asStateFlow()

    fun onUsernameChange(value: String) {
        _uiState.update { it.copy(username = value, usernameError = null, loginError = null) }
    }

    fun onPasswordChange(value: String) {
        _uiState.update { it.copy(password = value, passwordError = null, loginError = null) }
    }

    fun onTogglePasswordVisibility() {
        _uiState.update { it.copy(isPasswordVisible = !it.isPasswordVisible) }
    }

    fun onLoginClick() {
        val current = _uiState.value
        val usernameError = if (current.username.isBlank()) "Ingresa tu usuario o correo" else null
        val passwordError = if (current.password.length < 6) "Mínimo 6 caracteres" else null
        if (usernameError != null || passwordError != null) {
            _uiState.update { it.copy(usernameError = usernameError, passwordError = passwordError) }
            return
        }
        _uiState.update { it.copy(isLoading = true, loginError = null) }
        viewModelScope.launch {
            val result = repository.login(current.username, current.password)
            result
                .onSuccess { response ->
                    // TODO paso siguiente: guardar response.access_token en DataStore/Room
                    _uiState.update { it.copy(isLoading = false, isSuccess = true) }
                }
                .onFailure { e ->
                    _uiState.update {
                        it.copy(
                            isLoading = false,
                            loginError = "No se pudo iniciar sesión. Revisa tus datos o tu conexión."
                        )
                    }
                }
        }
    }

    fun onErrorShown() {
        _uiState.update { it.copy(loginError = null) }
    }
}
```

> No guardes el token en memoria ni en `SharedPreferences` sin cifrar en la versión final. Para este MVP basta con `isSuccess`; el almacenamiento seguro (DataStore + offline-first con Room) es el siguiente documento.

---

## Paso 7 — `LoginScreen`: formulario como hijo de `AuthCard`

Archivo: `ui/login/LoginScreen.kt`

```kotlin
package com.rojas.syncpass.ui.login

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.text.KeyboardActions
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Visibility
import androidx.compose.material.icons.filled.VisibilityOff
import androidx.compose.material3.Button
import androidx.compose.material3.CircularProgressIndicator
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.SnackbarHost
import androidx.compose.material3.SnackbarHostState
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.runtime.remember
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.input.ImeAction
import androidx.compose.ui.text.input.KeyboardType
import androidx.compose.ui.text.input.PasswordVisualTransformation
import androidx.compose.ui.text.input.VisualTransformation
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.lifecycle.viewmodel.compose.viewModel
import com.rojas.syncpass.ui.components.AuthCard
import com.rojas.syncpass.ui.theme.SyncPassTheme

@Composable
fun LoginScreen(
    viewModel: LoginViewModel = viewModel(),
    onLoginSuccess: () -> Unit = {}
) {
    val state by viewModel.uiState.collectAsState()
    val snackbar = remember { SnackbarHostState() }

    LaunchedEffect(state.loginError) {
        state.loginError?.let {
            snackbar.showSnackbar(it)
            viewModel.onErrorShown()
        }
    }
    LaunchedEffect(state.isSuccess) {
        if (state.isSuccess) onLoginSuccess()
    }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        AuthCard(
            title = "SyncPass",
            subtitle = "Inicia sesión para acreditar asistencia"
        ) {
            // --- Todo lo de abajo es el "children" del Card ---

            OutlinedTextField(
                value = state.username,
                onValueChange = viewModel::onUsernameChange,
                label = { Text("Usuario o correo") },
                singleLine = true,
                isError = state.usernameError != null,
                supportingText = { state.usernameError?.let { Text(it) } },
                keyboardOptions = KeyboardOptions(
                    keyboardType = KeyboardType.Email,
                    imeAction = ImeAction.Next
                ),
                modifier = Modifier.fillMaxWidth()
            )

            OutlinedTextField(
                value = state.password,
                onValueChange = viewModel::onPasswordChange,
                label = { Text("Contraseña") },
                singleLine = true,
                isError = state.passwordError != null,
                supportingText = { state.passwordError?.let { Text(it) } },
                visualTransformation = if (state.isPasswordVisible)
                    VisualTransformation.None
                else
                    PasswordVisualTransformation(),
                trailingIcon = {
                    IconButton(onClick = viewModel::onTogglePasswordVisibility) {
                        Icon(
                            imageVector = if (state.isPasswordVisible)
                                Icons.Filled.VisibilityOff
                            else
                                Icons.Filled.Visibility,
                            contentDescription = if (state.isPasswordVisible)
                                "Ocultar contraseña" else "Mostrar contraseña"
                        )
                    }
                },
                keyboardOptions = KeyboardOptions(
                    keyboardType = KeyboardType.Password,
                    imeAction = ImeAction.Done
                ),
                keyboardActions = KeyboardActions(
                    onDone = { viewModel.onLoginClick() }
                ),
                modifier = Modifier.fillMaxWidth()
            )

            Button(
                onClick = viewModel::onLoginClick,
                enabled = state.canSubmit,
                modifier = Modifier
                    .fillMaxWidth()
                    .height(48.dp) // mínimo táctil caso §3.3 / §5.1
            ) {
                if (state.isLoading) {
                    CircularProgressIndicator(
                        color = MaterialTheme.colorScheme.onPrimary
                    )
                } else {
                    Text("Ingresar")
                }
            }
        }
        SnackbarHost(hostState = snackbar)
    }
}

@Preview(showBackground = true)
@Composable
private fun LoginScreenPreview() {
    SyncPassTheme {
        LoginScreen(viewModel = LoginViewModel())
    }
}
```

---

## Paso 8 — Conecta en `MainActivity`

Reemplaza el `Greeting` de plantilla en `MainActivity.kt`:

```kotlin
package com.rojas.syncpass

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import com.rojas.syncpass.ui.login.LoginScreen
import com.rojas.syncpass.ui.theme.SyncPassTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            SyncPassTheme {
                LoginScreen(
                    onLoginSuccess = {
                        // TODO 02: navegar a Home / lista de eventos
                    }
                )
            }
        }
    }
}
```

Borra `Greeting` y `GreetingPreview` de la plantilla.

---

## Paso 9 — Probar

1. Verifica que `.env` existe y `BuildConfig` se generó:
   `Build > Rebuild Project` y luego `rg -n "API_BASE_URL" app/build/generated -g '*.java' | head`
2. Ejecuta en emulador/dispositivo con internet.
3. Prueba camino feliz con datos ficticios de `api.md`:
   - usuario: `juan_perez`
   - clave: `Password123`
   - Esperado: botón muestra loading → `isSuccess = true` (luego navegará).
4. Prueba errores:
   - campo vacío → mensaje bajo el campo.
   - clave <6 → `Mínimo 6 caracteres`, botón deshabilitado.
   - sin internet → Snackbar `No se pudo iniciar sesión...`.
5. Contrasta con curl: la app debe enviar el mismo body que `docs/api.md`.
6. Revisa modo claro/oscuro y rotación: el estado sobrevive porque vive en `ViewModel`, no en `remember`.

Comando útil para confirmar que no hardcodeaste la URL:

```bash
rg -n "172-233-15-71|linodeusercontent|BASE_URL\s*=" app/src/main --glob '*.kt'
# Esperado: solo aparece en AppConfig.kt vía BuildConfig, nunca literal en RetrofitClient u otros.
```

---

## Checklist de aceptación (para dar por listo el 01)

- [ ] `POST` llega a `api/auth/login` y con 200 marca `isSuccess`.
- [ ] `AuthCard` reutilizable con `content: @Composable ColumnScope.() -> Unit`; `LoginScreen` le pasa el formulario como hijo.
- [ ] Ningún `.kt` contiene la URL literal; todo pasa por `.env` → `BuildConfig` → `AppConfig`. `.env` está en `.gitignore`, `.env.example` sí commiteado.
- [ ] Validación en ViewModel, estados `loading / error / success` con feedback visual inmediato.
- [ ] Botón y campos ≥48dp, funciona en claro/oscuro, lenguaje simple.
- [ ] Solo datos ficticios en código, previews, capturas y logs (caso §5.1).
- [ ] `INTERNET` declarado; sin crash sin conexión.

## Siguiente (no parte de este doc)

- `02_home.md`: guardar `access_token` (DataStore), lista de proyectos/eventos, caché Room + `is_synced`, batch sync (caso §3.4).

---

## Anexo — Explicación de conceptos (por qué la guía está hecha así)

Esta sección no es código nuevo: es el "por qué" detrás de cada decisión, para que puedas defenderla en la evaluación.

### 1. MVVM (Model-View-ViewModel)

- **View (UI):** `LoginScreen.kt`. Solo dibuja y reenvía eventos (`onUsernameChange`, `onLoginClick`). No valida, no llama a red. Se dice que es "tonta" a propósito: así es testeable y sobrevive a rotación.
- **ViewModel:** `LoginViewModel.kt`. Guarda el estado, valida reglas de negocio y orquesta la llamada al repositorio. Sobrevive a cambios de configuración (rotar el teléfono) porque no lo crea Compose, lo crea el sistema.
- **Model:** `AuthRepository + Retrofit + DTOs`. Sabe cómo obtener/guardar datos, no sabe nada de pantallas.

Flujo en una sola dirección (UDF — Unidirectional Data Flow):

```
Evento UI -> ViewModel.update(state) -> StateFlow emite -> UI recompone
```

Si rompes esto (ej: llamar a Retrofit directo desde el `Button`), mezclas capas y no puedes testear ni reutilizar.

### 2. `UiState` + `StateFlow`

- `LoginUiState` es una `data class` inmutable con todo lo que la pantalla necesita: valores, errores, `isLoading`, `isSuccess`. Una sola fuente de verdad.
- `MutableStateFlow` en privado + `StateFlow` público en solo lectura: la UI observa pero no muta directamente. Solo el ViewModel hace `_uiState.update { it.copy(...) }`.
- `collectAsState()` convierte ese Flow en `State` de Compose: cada emisión dispara recomposición solo de lo que cambió.
- `canSubmit` derivado evita lógica duplicada en el botón.

Por qué no `var` sueltos con `remember`: se pierden al rotar y se desincronizan entre campos.

### 3. Slot API / `children` en Compose (`AuthCard`)

```kotlin
fun AuthCard(..., content: @Composable ColumnScope.() -> Unit)
```

- Es el equivalente a `children` en React/Web: el padre define marco (Card, padding, título, elevación, colores M3) y el hijo define contenido (formulario login, registro, recuperar clave).
- `ColumnScope.()` como receptor permite que el hijo use `Modifier.weight()`, `align()`, etc., como si estuviera dentro del `Column` del Card.
- Beneficio caso §3.3: centralizas accesibilidad (padding 20dp, spacing 12dp, `surfaceContainerHigh` que se adapta a claro/oscuro) en un solo lugar. Si mañana cambia el diseño, cambias `AuthCard`, no 5 pantallas.

Mal uso común: pasarle `username/password` al Card. No: el Card no debe saber nada de login.

### 4. Compose básico usado aquí

- `@Composable`: función que describe UI, no la crea imperativamente. Se re-ejecuta (recompone) cuando cambia su estado observado.
- `Modifier`: cadena de decoradores (tamaño, padding, click). Orden importa. `fillMaxWidth().height(48.dp)` garantiza área táctil mínima pedida en caso §5.1.
- `remember { SnackbarHostState() }`: memoria local de composición (sobrevive a recomposiciones, no a rotación — por eso el error vive en ViewModel, no en remember).
- `LaunchedEffect(key)`: efecto secundario atado al ciclo de composición. Aquí muestra el Snackbar una vez por cada `loginError` nuevo y luego avisa `onErrorShown()` para no repetirlo.
- `@Preview`: render sin emulador. Ponemos preview claro + oscuro para validar caso §3.3 sin correr la app.
- `MaterialTheme.colorScheme / typography`: nunca colores hex en duro; así soportas modo oscuro gratis.

### 5. Retrofit + DTOs + Repository

- `AuthApiService`: interfaz con anotaciones (`@POST`, `@Body`). Retrofit la implementa por ti con proxies. `suspend` la hace compatible con corrutinas (no bloquea el hilo UI).
- `LoginRequest / LoginResponse / UserDto`: DTOs (Data Transfer Objects) que calcan el JSON de `api.md`. `snake_case` (`access_token`) se deja igual al backend para no configurar `@SerializedName`; en dominio interno ya usarías `camelCase`.
- `GsonConverterFactory`: convierte JSON <-> data class automáticamente.
- `OkHttp + HttpLoggingInterceptor`: capa HTTP real. Logging en `BODY` solo si `BuildConfig.DEBUG`, si no `NONE` para no filtrar `access_token` en release.
- `RetrofitClient` como `object` + `by lazy`: singleton perezoso. Para MVP basta; en app real usarías Hilt/Koin.
- `AuthRepository` + `runCatching -> Result`: aísla el origen de datos. El ViewModel no sabe si viene de Retrofit, Room o fake para tests. Mañana agregas caché offline sin tocar el ViewModel.

### 6. Corrutinas (`viewModelScope.launch`)

- `viewModelScope` lanza trabajo asíncrono atado a la vida del ViewModel: si el usuario sale, se cancela solo, no fuga memoria.
- `suspend fun login()` se ejecuta fuera del hilo principal; Retrofit-OkHttp ya lo hace en IO. La UI solo ve `isLoading=true -> false`.
- `Result.onSuccess / onFailure`: manejo explícito sin `try/catch` regado. El mensaje al usuario es genérico ("Revisa tus datos o tu conexión") para no exponer detalles técnicos ni datos sensibles.

### 7. `.env` → `BuildConfig` → `AppConfig` (nada en duro)

- `.env`: archivo local no versionado con secretos/entornos. Cada dev/CI tiene el suyo.
- `.env.example`: plantilla versionada sin valores reales, documenta qué variables existen.
- `buildConfigField("String","API_BASE_URL",...)`: Gradle lee `.env` en tiempo de compilación y genera `BuildConfig.API_BASE_URL` como constante tipada. No es visible en el `.kt`, no se hardcodea.
- `AppConfig`: único punto de acceso en código + normaliza trailing `/` (Retrofit exige base URL terminada en `/`). Si ves la IP literal fuera del `build.gradle.kts` o `.env`, es deuda técnica.
- Prioridad `.env > System.getenv > fallback`: permite compilar en CI sin `.env` (vía secrets del runner) y en local con `.env`.

Por qué no `local.properties`: sirve, pero `.env` es estándar multiplataforma y el caso pide explícitamente "por ejemplo desde un .env". El mecanismo es el mismo: archivo ignorado + generación en build.

### 8. Validación y UX según caso §3.3

- Validar en ViewModel (`username.isBlank()`, `password.length < 6`) y exponer `usernameError/passwordError` + `supportingText`: la UI solo muestra, no decide.
- `canSubmit` deshabilita el botón: feedback preventivo, no solo correctivo.
- `OutlinedTextField` con `KeyboardOptions` (`Email/Password`, `ImeAction.Next/Done`) + `KeyboardActions(onDone)`: flujo con una mano en terreno.
- Mostrar/ocultar clave con `PasswordVisualTransformation` + `contentDescription` accesible.
- `Snackbar` para error de red + `CircularProgressIndicator` en botón para `isLoading`: "retroalimentación visual inmediata ante cada acción" que pide el caso.
- 48dp de alto en botón: mínimo táctil §5.1 para operar con una mano y con luz variable.

### 9. Qué NO hace este doc (y por qué)

- No guarda el token: hacerlo en `SharedPreferences` sin cifrar violaría buenas prácticas. Va en `02` con DataStore cifrado + Room para offline-first (caso §3.4: `is_synced`, `timestamps`, batch sync).
- No usa Hilt/Navigation: innecesario para un MVP de una pantalla; agrega fricción sin aporte. Se introduce cuando haya 2+ destinos.
- No loguea `password` ni `access_token`: el interceptor en release está en `NONE` y los previews usan datos ficticios (caso §5.1: prohibición de datos reales).

Glosario rápido:

| Término | En una frase |
|---|---|
| DTO | Molde del JSON que viaja por red, no es tu modelo de dominio. |
| Repository | Portero: la UI pide "login", él decide si va a red, caché o fake. |
| StateFlow | Tubo observable de estado; emite, la UI recompone. |
| Slot | Hueco `content` donde el padre recibe hijos sin conocerlos. |
| BuildConfig | Clase generada en compilación con constantes por variante (aquí, la URL). |
| UDF | Los datos bajan, los eventos suben; nunca al revés. |

---

## Anexo — Troubleshooting: `3 issues were found when checking AAR metadata`

### Síntoma

Al sincronizar/compilar aparece:

```text
Dependency 'androidx.core:core-ktx:1.19.1' requires ... compile against version 37 or later ...
:app is currently compiled against android-36.1.
Recommended action: Update this project to use a newer compileSdk of 37.
```

Afecta también a `androidx.core:core:1.19.1` y `androidx.lifecycle:lifecycle-runtime-compose-android:2.11.0`.

### Qué pasó

- `app/build.gradle.kts` declara `compileSdk = release(36) { minorApiLevel = 1 }` → `android-36.1`.
- `gradle/libs.versions.toml` trae `coreKtx = 1.19.1` y `lifecycleRuntimeKtx = 2.11.0`, que en su `AAR metadata` exigen `compileSdk >= 37`.
- Gradle valida esto en el check de metadata y falla. No es error de código, es incompatibilidad librería vs `compileSdk`.
- Recordar: `compileSdk` (con qué compilas) ≠ `targetSdk` (comportamiento runtime) ≠ `minSdk` (quién puede instalar, aquí `28`).

### Cómo resolverlo (2 opciones)

**Opción A — Recomendada: subir `compileSdk` a 37:**

1. Tools > SDK Manager > instalar `Android API 37` (Platform + Build-Tools).
2. En `app/build.gradle.kts`:
```kotlin
compileSdk {
    version = release(37)
}
```
3. Dejar `targetSdk = 36` y `minSdk = 28` como están. Sync + Rebuild.

**Opción B — Bajar librerías compatibles con 36 (si no puedes instalar SDK 37):**

En `gradle/libs.versions.toml`:
```toml
coreKtx = "1.17.0"
lifecycleRuntimeKtx = "2.9.2"
```
Luego Sync.

*Fuente contrato: `docs/api.md`. Contexto negocio/restricciones: `docs/caso.md`.*
