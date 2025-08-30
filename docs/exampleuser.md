# 🧪 Módulo de Ejemplo: Gestión de Usuarios con Autenticación Avanzada (Passkeys)

Este módulo sirve como ejemplo oficial para contribuir al proyecto. Aplica Clean Architecture, CQRS, y autenticación moderna con Passkeys. Todo nuevo módulo debe seguir esta estructura y documentación como modelo.

---

## 📁 Estructura del Módulo `/internal/exampleuser/`

```plaintext
/internal/exampleuser/
├── domain/
│   ├── user.go
│   └── repository.go
├── application/
│   ├── command/
│   │   ├── register_user.go
│   │   └── register_user_handler.go
│   └── query/
│       ├── get_user.go
│       └── get_user_handler.go
├── infrastructure/
│   └── repository/
│       └── user_repository_pg.go
├── interfaces/
│   └── handler/
│       └── user_http_handler.go
├── passkey/
│   ├── registration_flow.go
│   ├── validation.go
│   └── response.go
├── tests/
│   ├── register_user_test.go
│   └── get_user_test.go
└── README.md
```

---

## 🔐 Caso de Uso: Registro de Usuario con Passkeys

### ✳️ Comando: `RegisterUser`

```go
package command

type RegisterUser struct {
    ID       string
    Name     string
    Email    string
    Passkey  string // clave pública del autenticador
}
```

### ✳️ Handler: `RegisterUserHandler`

```go
package command

import (
    "internal/exampleuser/domain"
    "internal/exampleuser/passkey"
)

type RegisterUserHandler struct {
    repo       domain.UserRepository
    validator  *passkey.Validator
}

func NewRegisterUserHandler(r domain.UserRepository, v *passkey.Validator) *RegisterUserHandler {
    return &RegisterUserHandler{repo: r, validator: v}
}

func (h *RegisterUserHandler) Handle(cmd RegisterUser) error {
    if !h.validator.IsValid(cmd.Passkey) {
        return errors.New("passkey inválido")
    }
    user := domain.NewUser(cmd.ID, cmd.Name, cmd.Email, cmd.Passkey)
    return h.repo.Save(user)
}
```

### ✳️ Entidad: `User`

```go
package domain

type User struct {
    ID      string
    Name    string
    Email   string
    Passkey string
}

func NewUser(id, name, email, passkey string) *User {
    return &User{
        ID: id,
        Name: name,
        Email: email,
        Passkey: passkey,
    }
}
```

---

## 📡 Adaptador HTTP: `user_http_handler.go`

```go
package handler

import (
    "net/http"
    "encoding/json"
    "internal/exampleuser/application/command"
)

type UserHandler struct {
    registerHandler *command.RegisterUserHandler
}

func (h *UserHandler) RegisterUser(w http.ResponseWriter, r *http.Request) {
    var cmd command.RegisterUser
    if err := json.NewDecoder(r.Body).Decode(&cmd); err != nil {
        http.Error(w, "entrada inválida", http.StatusBadRequest)
        return
    }
    if err := h.registerHandler.Handle(cmd); err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    w.WriteHeader(http.StatusCreated)
}
```

---

## 🧪 Test de Unidad: `register_user_test.go`

```go
func TestRegisterUser_OK(t *testing.T) {
    repo := mocks.NewMockUserRepository()
    validator := passkey.NewValidator()
    handler := command.NewRegisterUserHandler(repo, validator)

    cmd := command.RegisterUser{
        ID: "123",
        Name: "Juan",
        Email: "juan@mail.com",
        Passkey: "clave-valida",
    }

    err := handler.Handle(cmd)
    assert.NoError(t, err)
    assert.Equal(t, 1, repo.Count())
}
```

---

## 📄 Documentación del Módulo (`README.md`)

```markdown
# Módulo Ejemplo: Registro de Usuario con Passkeys

Este módulo demuestra cómo estructurar un componente funcional completo en el proyecto.

- Aplica **arquitectura hexagonal y CQRS**.
- Incluye comandos, consultas, repositorio, validación y adaptadores HTTP.
- Usa una clave pública tipo **Passkey** para autenticación avanzada.

## Flujo

1. El cliente envía datos + passkey pública (webauthn, FIDO2).
2. Se valida que la passkey cumpla requisitos técnicos.
3. Se crea el usuario si todo es correcto.

## Archivos clave

- `application/command`: lógica de negocio para registro.
- `interfaces/handler`: endpoint HTTP que consume el comando.
- `passkey/`: validaciones específicas para claves FIDO2.
- `tests/`: cobertura de unidad para el caso de uso.

## Uso

Este módulo se puede copiar como base para:
- Módulos futuros de onboarding.
- Validación biométrica o passkeys.
- Adaptadores HTTP seguros.
```

---

¿Necesitas validar el segundo ejemplo o agregar una versión con un flujo de login e integración con sesión también?
