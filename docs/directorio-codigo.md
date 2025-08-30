# 🧭 Directorio del Código – Proyecto EAV Híbrido

Este documento sirve como guía para navegar por la estructura del repositorio, definiendo claramente el propósito de cada carpeta y archivo.

---

## 📁 Estructura Principal

```
/internal/
├── auth/            # Módulo de autenticación de usuarios y sesiones
├── metadata/        # Módulo de definición del esquema EAV dinámico
├── data/            # Módulo de lectura/escritura de datos EAV
/pkg/                # Librerías reutilizables (ej. conexión DB)
/docs/               # Documentación técnica y lineamientos
```

---

## 📦 Módulos por Dominio

### `/internal/auth/`

* **Responsabilidad**: Autenticación, sesiones y usuarios.
* **Subcarpetas**:
  - `domain/`: entidades `User`, `Session` y contrato `AuthRepository`
  - `application/command/`: comandos CQRS `CreateUser`, `CreateSession`
  - `application/query/`: query `GetUserByEmail`
  - `infrastructure/`: implementación real PostgreSQL
  - `interfaces/`: CLI para pruebas
  - `tests/`: pruebas unitarias con `testify`

---

### `/internal/metadata/`

* **Responsabilidad**: Definición del modelo lógico (tablas, columnas, propiedades).
* **Subcarpetas**:
  - `domain/`: entidades `LogicalTable`, `ColumnDefinition`, `TableProperty`
  - `application/command/`: comandos `CreateLogicalTable`, `AddColumnToTable`
  - `application/query/`: queries `GetLogicalTable`, `ListLogicalTables`
  - `infrastructure/`: persistencia PostgreSQL real
  - `interfaces/`: CLI para validaciones
  - `tests/`: pruebas unitarias

---

### `/internal/data/`

* **Responsabilidad**: CRUD dinámico de datos EAV.
* **Estado actual**: en construcción.
* **Plan futuro**: implementar `WriteEntityCommand`, `ReadEntityQuery`, `OutboxHandler`

---

## 🔧 Biblioteca de Código Reutilizable

### `/pkg/database/postgres.go`

* Crea conexiones a PostgreSQL
* Usada por repositorios `metadata` y `auth`

---

## 📘 Documentación Relacionada

- `estructura_proyecto_go.md`: define jerarquía
- `estilo-codigo.md`: guía de estilo Go
- `lineamientos-nuevos-devs.md`: cómo empezar y reglas internas
- `Plan de Ejecución Técnica y Secuencial...`: roadmap del desarrollo

---

## 🧪 Tests

Cada módulo tiene su carpeta `tests/` con pruebas table-driven y uso de `testify`.

---

## 📌 Convención General

- Cada módulo respeta Clean Architecture + CQRS
- No se permite lógica de dominio en infraestructura
- Cada carpeta debe tener `README.md` local

