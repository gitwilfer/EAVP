# 📦 Estructura Base del Proyecto Go – Arquitectura EAV Híbrida

Este documento define la estructura oficial de carpetas y archivos para proyectos Go desarrollados bajo los principios de Clean Architecture, CQRS, DDD y control estricto de estándares. También se aplican los principios de Arquitectura Hexagonal (Ports & Adapters), aunque no se refleje en nombres de carpetas explícitos.

> 📌 **Nota:** La arquitectura hexagonal está implementada funcionalmente mediante la separación entre:
>
> * Adaptadores externos: `handler.go`
> * Lógica de aplicación: `service.go`, `command/`, `query/`
> * Adaptadores de infraestructura: `repository.go`
> * Interfaces: definidas en `domain/` o a nivel de módulo

---

## 🗂️ Estructura General del Proyecto

```plaintext
├── api/                    # Contratos de API (OpenAPI, gRPC, etc.)
│   └── openapi.yaml        
├── build/                 # Archivos de construcción y CI/CD
│   └── Dockerfile
├── cmd/                   # Entrypoints para ejecutables
│   └── eav-api/           # main.go para API principal
├── configs/               # Archivos de configuración YAML/JSON
│   └── config.yaml
├── deployments/           # Despliegue (Kubernetes, Helm, etc.)
│   └── k8s/
├── docs/                  # Documentación técnica y arquitectónica
│   ├── arquitectura.md
│   ├── directorio-codigo.md
│   └── estilo-codigo.md
├── internal/              # Código interno (no exportable)
│   ├── auth/              # Autenticación y Passkeys
│   ├── metadata/          # Gestión del esquema EAV
│   ├── data/              # CRUD y queries de datos EAV
│   ├── exampleuser/       # Ejemplo: registro de usuario con Passkeys
│   ├── examplelogin/      # Ejemplo: inicio de sesión con Passkeys y sesiones
│   └── common/            # Utilidades comunes internas
├── pkg/                   # Código reusable (validadores, middlewares, etc.)
├── scripts/               # Scripts de desarrollo y mantenimiento
├── test/                  # Pruebas de integración y e2e
├── go.mod
├── go.sum
├── README.md
├── Makefile
└── CONTRIBUTING.md
```

---

## ✅ Reglas de Estructura

1. Cada módulo debe tener un `README.md` explicativo.
2. Cada paquete debe seguir la separación por capas: dominio, aplicación, interfaces, infraestructura.
3. Todo nuevo módulo debe registrarse en `docs/directorio-codigo.md`.
4. No se deben crear carpetas arbitrarias fuera del layout aquí descrito.

---

Este documento es parte del contrato técnico de arquitectura y debe mantenerse actualizado con cada cambio estructural significativo.
