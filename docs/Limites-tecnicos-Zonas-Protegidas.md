# 🚧 Límites Técnicos y Zonas Intocables – Proyecto Go EAV Híbrido

Este documento define los **límites técnicos obligatorios** y las **zonas del sistema que no pueden ser modificadas arbitrariamente**. Todo desarrollador debe respetar estas directrices como parte del contrato técnico del equipo.

---

## 🔐 1. Zonas Intocables (Prohibido Modificar sin Aprobación Arquitectónica)

| Zona                          | Descripción                                                                                 |
| ----------------------------- | ------------------------------------------------------------------------------------------- |
| `metadata` (modelo EAV)       | Estructura base para el modelo EAV. No puede ser modificada sin rediseño.                   |
| `data`                        | Lógica de persistencia y consultas EAV. No se permite reestructurar.                        |
| `exampleuser`, `examplelogin` | Son referencia técnica. No deben ser alterados salvo para actualizar estilo o dependencias. |
| `reportes dinámicos`          | Contratos, parámetros y flujo base no deben cambiar. Está cerrado a cambios estructurales.  |
| `auth/passkey`                | Autenticación moderna obligatoria. No se permite volver a contraseñas simples.              |

---

## 🧭 2. Lineamientos Técnicos

### 🔹 Estructura

* Toda nueva funcionalidad debe seguir Clean Architecture + Hexagonal.
* Debe dividirse por capas: `domain/`, `application/`, `infrastructure/`, `interfaces/`, `tests/`
* Cada módulo nuevo se registra en `docs/directorio-codigo.md`

### 🔹 Código y estilo

* Todos los módulos deben usar `make lint`, `make test`, `make format`
* No se aceptan funciones sin comentarios `godoc`
* El formato, nombres y estilo deben seguir `docs/estilo-codigo.md`

### 🔹 Seguridad y validaciones

* Las validaciones deben implementarse con `validator/v10`
* No se puede ignorar errores (`errcheck` lo valida)
* Se prohíbe `fmt.Println` fuera de debugging

### 🔹 Dependencias

* Toda nueva dependencia externa debe ser justificada en `docs/arquitectura.md`
* No se permite usar ORMs automáticos, DI mágicos o frameworks pesados

### 🔹 Autenticación y sesión

* Passkeys es el único método autorizado de autenticación
* El manejo de sesiones debe ser desacoplado (`SessionRepository`)

---

## 📌 3. Protocolo para Propuestas de Cambio

Si un desarrollador considera que alguna zona protegida debe ser modificada, debe:

1. Crear un documento de propuesta (`docs/propuestas/xxx-cambio.md`)
2. Exponer la justificación técnica, impacto y alternativas
3. Enviarlo como PR marcado `@arquitectura`
4. Esperar revisión y validación antes de implementarlo

---

Este documento protege la estabilidad, mantenibilidad y visión del proyecto. Todo incumplimiento puede llevar al rechazo de cambios o desvinculación técnica del equipo.
