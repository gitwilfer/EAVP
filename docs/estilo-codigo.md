# 🎨 Guía de Estilo de Código – Proyecto Go EAV Híbrido

Esta guía define las **reglas obligatorias** y convenciones idiomáticas para escribir código Go limpio, consistente y profesional en este proyecto.

Está basada en:

* [Effective Go](https://go.dev/doc/effective_go)
* [Uber Go Style Guide](https://github.com/uber-go/guide)
* Experiencia en arquitectura limpia, CQRS, Passkeys, y control de sesiones

---

## 🔤 1. Convenciones de Nombres

### Variables

* Usa nombres cortos y significativos:

  * `ctx`, `cfg`, `svc`, `err`, `u`, `repo`
* Evita redundancia: `user.Name` es suficiente, no `user.UserName`

### Funciones

* Exportadas: empiezan con mayúscula y son verbos: `CreateUser`, `FetchData`
* Booleanas: `IsValid`, `HasPermission`, `CanLogin`

### Paquetes

* Todo en minúscula, sin guiones ni snake\_case: `auth`, `eav`, `metadata`
* El nombre del paquete debe ser coherente con su propósito: `validator`, `handler`, `repository`

---

## 🧱 2. Organización de Código

Cada módulo (`internal/X/`) debe estructurarse así:

```
domain/         # Entidades y contratos (interfaces)
application/
  command/      # Comandos (acciones que modifican estado)
  query/        # Consultas (acciones de solo lectura)
interfaces/     # Adaptadores de entrada (HTTP, CLI, etc.)
infrastructure/ # Adaptadores de salida (DB, APIs externas)
tests/          # Pruebas del módulo
README.md       # Descripción obligatoria del módulo
```

---

## 🧩 3. Uso de Interfaces

* Define interfaces **donde sea necesario desacoplar** (repositorios, servicios externos).
* Las interfaces deben vivir en el **paquete que las consume**, no donde se implementan.

```go
// Correcto: en application/query

//go:generate mockgen -source=repository.go -destination=mock_repository.go

package query

type UserRepository interface {
    FindByEmail(email string) (*User, error)
}
```

---

## ⚠️ 4. Manejo de Errores

* Nunca ignores errores: `if err != nil` es obligatorio
* Usa wrapping para contexto: `fmt.Errorf("fallo al guardar usuario: %w", err)`
* No uses `panic` excepto en `main` o inicialización crítica
* Define errores de dominio como variables:

```go
var ErrUserNotFound = errors.New("usuario no encontrado")
```

---

## 🔁 5. Control de Flujo y Concurrencia

* Usa `context.Context` en toda función pública que pueda cancelarse
* No uses `context.Background()` salvo en `main` o tests
* Para múltiples goroutines, usa `sync/errgroup` o `chan` con `select`

---

## 📚 6. Documentación del Código

* Toda función o struct exportada debe tener comentario tipo `godoc`

```go
// RegisterUser crea un nuevo usuario con passkey.
func RegisterUser(...) error { ... }
```

* No repitas el nombre de la función en la descripción si no es exportada

---

## 🧪 7. Estilo de Pruebas

* Usa `testify/assert` o `require`
* Prefiere `table-driven tests` cuando hay múltiples entradas/salidas

```go
func TestValidatePasskey(t *testing.T) {
    tests := []struct {
        name  string
        input string
        valid bool
    }{
        {"válido", "clave-ok", true},
        {"inválido", "", false},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            res := validator.IsValid(tt.input)
            assert.Equal(t, tt.valid, res)
        })
    }
}
```

---

## 🧹 8. Formato y Lint

* El código debe pasar sin errores:

  * `golangci-lint`
  * `goimports`
  * `staticcheck`
* El orden de imports debe ser:

```go
import (
    "std"
    "external"
    "internal"
)
```

* Usa `make format` y `make lint` antes de cada `git push`

---

## 📌 9. Cosas que están Prohibidas

* Uso de `fmt.Println` en código productivo (usa `zaplogger`)
* Nombres como `data1`, `temp`, `foo`
* `interface{}` salvo en genéricos o handlers muy abstractos
* Lógica de negocio en adaptadores (nunca dentro de `handler.go` directamente)

---

Este documento **es vinculante**. Toda violación a estas reglas puede causar el rechazo inmediato de un Pull Request. Las herramientas automáticas refuerzan su cumplimiento.
