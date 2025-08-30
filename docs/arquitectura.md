# 🏗️ Arquitectura del Proyecto – EAV Híbrido con Clean Architecture y Passkeys

Este documento describe la arquitectura oficial del sistema, los principios que la sustentan, los patrones aprobados y las decisiones técnicas estratégicas que guían su desarrollo.

---

## 🎯 Visión General

Este proyecto implementa una arquitectura **modular, mantenible y extensible** basada en los principios de:

* **Clean Architecture** (por Robert C. Martin)
* **Arquitectura Hexagonal (Ports & Adapters)**
* **CQRS (Command Query Responsibility Segregation)**
* **EAV (Entidad-Atributo-Valor) para datos dinámicos**
* **Autenticación moderna con Passkeys/WebAuthn**

La arquitectura está diseñada para facilitar:

* Escalabilidad
* Separación de responsabilidades
* Testabilidad total
* Control estricto del flujo y validación
* Observabilidad y confiabilidad avanzada (💡 NUEVO)

---

## 🧱 Diagrama Lógico de Capas

```plaintext
+-------------------------+
|     Adaptadores HTTP    | ← interfaces/handler/
+-------------------------+
|      Casos de Uso       | ← application/command/, application/query/
+-------------------------+
|        Dominio          | ← domain/ (entidades, interfaces, validaciones)
+-------------------------+
|  Infraestructura (DB,   |
|     redes, sesión...)   | ← infrastructure/repository/, auth/
+-------------------------+
```

---

## 🧩 Componentes Clave

### `domain/`

Define:

* Entidades centrales (`User`, `Session`, `Schema`, etc.)
* Interfaces (puertos) para repositorios y validadores
* Reglas de negocio básicas (sin dependencias externas)
* **Validaciones estrictas con tipos fuertes (💡 RECOMENDADO)**

### `application/`

* Casos de uso divididos en:

  * `command/`: mutaciones
  * `query/`: solo lectura
* Cada caso de uso tiene su handler
* No contiene lógica de transporte (HTTP) ni persistencia (DB)
* Debe estar completamente instrumentado con `metrics` y `tracing` (💡 NUEVO)

### `interfaces/`

* Adaptadores de entrada: HTTP, Web, CLI
* Transforman requests a comandos o queries
* No contienen lógica de negocio

### `infrastructure/`

* Adaptadores de salida:

  * Repositorios (Postgres, Redis...)
  * API externas
  * Validadores de Passkey
* Implementan interfaces del dominio
* Logging estructurado con `zap` es **obligatorio** (💡 NUEVO)

---

## 🔐 Autenticación con Passkeys

* Se usa `Passkey` (clave pública) para validar identidad
* Flujo implementado en módulos `exampleuser/` y `examplelogin/`
* Validaciones se realizan mediante adaptadores específicos (`passkey/`)
* La sesión es generada y almacenada en `SessionRepository`

---

## 📦 Patrones Aprobados

| Patrón / Técnica     | Estado        | Notas                                      |
| -------------------- | ------------- | ------------------------------------------ |
| Clean Architecture   | ✅ Usado       | Separación estricta de capas               |
| CQRS                 | ✅ Usado       | Comandos y consultas separados             |
| Hexagonal            | ✅ Usado       | Adaptadores para entrada/salida            |
| Circuit Breaker      | ✅ Recomendado | Ej: `gobreaker` en integraciones           |
| Outbox Pattern       | ✅ Reforzado   | Debe usarse donde haya efectos colaterales |
| Dependency Injection | ✅ Manual      | Via constructores, sin contenedor mágico   |
| Observabilidad Total | ✅ Obligatorio | Tracing, logs y métricas por defecto       |

---

## ⚠️ Decisiones Estratégicas

* **No se permite lógica de negocio en `handler`**
* **No se usan ORMs automáticos** (solo SQL limpio o herramientas explícitas)
* **No se usa `interface{}` salvo donde se justifique bien (ej: JSON genérico)**
* **Las dependencias externas deben ser justificadas y documentadas** en este archivo
* **Cada PR debe mantener esta arquitectura o será rechazado**
* **Todo nuevo módulo con efectos colaterales debe usar patrón Outbox (💡 NUEVO)**
* **Cada módulo nuevo debe exponer trazabilidad e instrumentación mínima viable**

---

## 📚 Documentación Complementaria

* `docs/estilo-codigo.md`: Convenciones de estilo y formato
* `docs/directorio-codigo.md`: Índice de módulos y responsabilidades
* `CONTRIBUTING.md`: Reglas para participar en el desarrollo

---

Este documento es parte del contrato técnico del proyecto y debe ser actualizado ante cualquier cambio estructural o arquitectónico importante.
