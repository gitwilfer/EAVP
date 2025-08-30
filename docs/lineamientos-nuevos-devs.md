# 🧭 Lineamientos para Nuevos Desarrolladores – Proyecto Go EAV Híbrido

Bienvenido al proyecto. Este documento define las **reglas, cultura y expectativas** que todo nuevo desarrollador debe conocer antes de escribir una sola línea de código.

---

## 📍 1. Propósito del Proyecto

Este sistema es un **núcleo empresarial universal** basado en un modelo EAV (Entidad-Atributo-Valor), diseñado para ser altamente **configurable, duradero, y transversal**. Su misión es servir como base para cualquier organización, sin importar su industria.

Toda lógica de negocio debe desarrollarse como una extensión del núcleo, **sin modificar su estructura central**.

---

## 🔑 2. Principios Técnicos Irrenunciables

* ✅ Clean Architecture + Hexagonal
* ✅ CQRS (comandos para escribir, queries para leer)
* ✅ Validaciones estrictas (`validator/v10`)
* ✅ Seguridad moderna: Passkeys (FIDO2), sesiones desacopladas
* ✅ Logging estructurado con `zap` (💡 NUEVO)
* ✅ Instrumentación mínima con trazas y métricas (💡 NUEVO)
* ✅ Patrón Outbox obligatorio para side effects (💡 NUEVO)
* 🚫 No ORMs automáticos
* 🚫 No lógica de negocio en adaptadores (`handler.go`)

---

## 🧠 3. Lecturas obligatorias antes de desarrollar

| Archivo                       | ¿Para qué sirve?                                      |
| ----------------------------- | ----------------------------------------------------- |
| `docs/arquitectura.md`        | Entender el diseño, capas y patrones permitidos       |
| `docs/estilo-codigo.md`       | Saber cómo escribir código aceptado                   |
| `docs/directorio-codigo.md`   | Conocer la estructura de carpetas y módulos           |
| `CONTRIBUTING.md`             | Cómo se colabora: PRs, pruebas, revisiones            |
| `internal/example*/README.md` | Ejemplos aprobados de cómo desarrollar nuevos módulos |

---

## 🧰 4. Herramientas de desarrollo

* `make format` → Aplica `goimports`
* `make lint` → Ejecuta `golangci-lint` + linters oficiales
* `make test` → Corre pruebas del sistema completo
* `make exampleuser` → Corre módulo de registro con Passkeys
* `make examplelogin` → Corre módulo de login con sesión segura

---

## 🧪 5. Cómo crear un nuevo módulo

Checklist obligatorio:

* [ ] Crear carpeta en `internal/nombremodulo/`
* [ ] Separar en `domain/`, `application/`, `interfaces/`, `infrastructure/`, `tests/`
* [ ] Crear `README.md` explicando propósito y diseño
* [ ] Agregar entrada en `docs/directorio-codigo.md`
* [ ] Incluir al menos una prueba unitaria
* [ ] Validar con `make lint` y `make test`
* [ ] Incluir logs estructurados con `zap` (💡 NUEVO)
* [ ] Instrumentar comandos/queries con trazas (💡 NUEVO)
* [ ] Usar patrón Outbox si hay side effects (💡 NUEVO)
* [ ] Subir PR con etiqueta `@arquitectura`

---

## ❌ 6. Prácticas prohibidas

* ❌ Push directo a `main`
* ❌ Código sin pruebas ni documentación
* ❌ Uso de `fmt.Println` en producción (usar `zap`)
* ❌ Repetición de lógica en múltiples módulos
* ❌ Introducir dependencias sin revisión arquitectónica
* ❌ Interfaces sin más de una implementación (innecesarias)

---

## 💬 7. Cultura del equipo

> Aquí no improvisamos estructuras. Cada módulo, cada función y cada test debe reflejar un estándar profesional. Buscamos claridad, previsibilidad y mantenimiento a largo plazo. Tu código debe ser tan claro que otro lo pueda extender sin miedo. Si tienes dudas, pregunta y documenta.

---

Este documento es parte del contrato técnico del proyecto. Su lectura y cumplimiento es obligatoria para cualquier colaborador técnico.
