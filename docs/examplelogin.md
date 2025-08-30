# 🧪 Módulo de Ejemplo: Inicio de Sesión con Passkeys + Manejo de Sesiones

Este módulo complementa el registro de usuarios y muestra cómo implementar el flujo de login con Passkeys, validación avanzada y creación de sesiones seguras.

---

## 📁 Estructura del Módulo `/internal/examplelogin/`

```plaintext
/internal/examplelogin/
├── domain/
│   ├── session.go
│   └── repository.go
├── application/
│   └── command/
│       ├── login_user.go
│       └── login_user_handler.go
├── infrastructure/
│   └── repository/
│       └── session_repository_memory.go
├── interfaces/
│   └── handler/
│       └── login_http_handler.go
├── passkey/
│   ├── assertion_flow.go
│   └── validator.go
├── tests/
│   └── login_user_test.go
└── README.md
```

---

## 🔐 Caso de Uso: Login con Passkey

### ✳️ Comando: `LoginUser`

```go
package command

type LoginUser struct {
    Email   string
    Passkey string // clave pública del usuario
    ClientIP string // opcional para validaciones contextuales
}
```

### ✳️ Handler: `LoginUserHandler`

```go
package command

import (
    "internal/examplelogin/domain"
    "internal/examplelogin/passkey"
    "errors"
)

type LoginUserHandler struct {
    repo      domain.SessionRepository
    validator *passkey.Validator
}

func NewLoginUserHandler(r domain.SessionRepository, v *passkey.Validator) *LoginUserHandler {
    return &LoginUserHandler{repo: r, validator: v}
}

func (h *LoginUserHandler) Handle(cmd LoginUser) (*domain.Session, error) {
    if !h.validator.IsValid(cmd.Email, cmd.Passkey) {
        return nil, errors.New("passkey inválido o no coincide")
    }
    return h.repo.CreateSession(cmd.Email, cmd.ClientIP)
}
```

---

## 🧠 Dominio: `Session`

```go
package domain

type Session struct {
    Token    string
    Email    string
    IssuedAt time.Time
    IP       string
}
```

### ✳️ Repositorio en Memoria (para test/demo)

```go
package repository

import (
    "time"
    "internal/examplelogin/domain"
    "math/rand"
)

type MemorySessionRepository struct {
    store map[string]domain.Session
}

func NewMemorySessionRepository() *MemorySessionRepository {
    return &MemorySessionRepository{store: make(map[string]domain.Session)}
}

func (r *MemorySessionRepository) CreateSession(email, ip string) (*domain.Session, error) {
    token := generateToken()
    session := domain.Session{Token: token, Email: email, IssuedAt: time.Now(), IP: ip}
    r.store[token] = session
    return &session, nil
}

func generateToken() string {
    return fmt.Sprintf("sess-%d", rand.Int())
}
```

---

## 📡 Adaptador HTTP: `login_http_handler.go`

```go
package handler

import (
    "encoding/json"
    "net/http"
    "internal/examplelogin/application/command"
)

type LoginHandler struct {
    loginHandler *command.LoginUserHandler
}

func (h *LoginHandler) Login(w http.ResponseWriter, r *http.Request) {
    var cmd command.LoginUser
    if err := json.NewDecoder(r.Body).Decode(&cmd); err != nil {
        http.Error(w, "entrada inválida", http.StatusBadRequest)
        return
    }
    session, err := h.loginHandler.Handle(cmd)
    if err != nil {
        http.Error(w, err.Error(), http.StatusUnauthorized)
        return
    }
    w.Header().Set("Authorization", session.Token)
    json.NewEncoder(w).Encode(session)
}
```

---

## 📄 Documentación del Módulo (`README.md`)

```markdown
# Módulo Ejemplo: Inicio de Sesión con Passkeys

Este módulo ilustra cómo implementar un flujo seguro de inicio de sesión usando Passkeys + sesiones.

## Características
- Verifica passkey contra clave pública previamente registrada.
- Usa un repositorio de sesiones (simulado en memoria).
- Expone un adaptador HTTP limpio y desacoplado.

## Flujo resumido
1. Cliente envía `email` y `passkey` (clave pública).
2. Se valida que esa combinación sea correcta.
3. Se genera un `Session.Token` único.
4. La sesión es devuelta como respuesta.

## Estructura
- `command/`: orquesta el login.
- `domain/`: define qué es una sesión.
- `passkey/`: lógica de validación de claves públicas.
- `repository/`: manejo de sesiones.
- `handler/`: expone HTTP.

Este módulo puede ser extendido para JWT, Redis, OAuth2 o MFA.
```

---

Este módulo queda como guía base para cualquier implementación de autenticación avanzada. ¿Te gustaría que ahora preparemos un `Makefile` con tareas para generar, testear e integrar módulos como este?
