# Requisitos de Conocimiento por Fase - Proyecto EAV Híbrido

Este documento define exactamente qué información debe recibir cada Inteligencia Artificial encargada de implementar una fase específica del proyecto EAV Híbrido con Passkeys.

## Documentación Base para Todas las Fases

**Toda IA, independientemente de su fase asignada, debe recibir:**

1. **plan-eav-hibrido-definitivo.md** completo (enfocándose en su fase asignada)
2. **entregables-entre-fases.md** (secciones de la fase anterior y la actual)
3. **directrices-implementacion-ia.md** completo
4. **Secciones generales del requerimiento-EAV-Go.md**:
   - Sección 1: Introducción
   - Sección 2: Visión General de la Arquitectura
   - Sección 4: Stack Tecnológico
5. **Estructura del proyecto y estándares de código**

## Requisitos Específicos por Fase

### Fase 1: Estructura Base y Modelo de Dominio EAV

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 3: **Modelo de Datos EAV Híbrido** (COMPLETA - 3.1, 3.2.1, 3.2.2, 3.2.3, 3.2.4, 3.2.5, 3.2.6, 3.3, 3.4)
- Sección 2.2: Principios Arquitectónicos (enfatizando DDD y Arquitectura Hexagonal)

**Entregables principales:**
- Implementación de entidades de dominio (LogicalTable, ColumnDefinition, TableRow, etc.)
- Interfaces de repositorio para todas las entidades
- Definición de errores de dominio estandarizados
- Enumeraciones y tipos de valor del dominio

### Fase 2: Infraestructura Base y Patrones Core

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 8: **Gestión de Transacciones y Consistencia**
- Sección 9: **Patrones de Diseño Implementados** (9.1-9.9)
- Sección 13: Observabilidad y Monitoreo (conceptos básicos)

**Entregables principales:**
- Implementación de patrón Circuit Breaker
- Implementación de patrón Bulkhead
- Implementación del patrón Outbox para eventos
- Implementación de Unit of Work para transacciones
- Estrategia de caché multinivel

**Dependencias:**
- Código completo de Fase 1

### Fase 3: Implementación de Repositorios EAV

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 11: **Escalabilidad y Rendimiento** (11.1-11.5)
- Sección 3.3: Optimizaciones del Modelo
- Sección 3.4: Consideraciones de Integridad de Datos

**Entregables principales:**
- Implementación de todos los repositorios EAV
- Consultas optimizadas para el modelo EAV
- Índices y optimizaciones para PostgreSQL
- Estrategias de caché para entidades EAV

**Dependencias:**
- Código completo de Fases 1-2

### Fase 4: Repositorios para Autenticación y RBAC

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 3.2.3: Tablas de Autenticación y Passkeys
- Sección 3.2.5: Tablas para RBAC
- Sección 12: **Seguridad** (específicamente 12.0 y 12.1)

**Entregables principales:**
- Implementación de UserRepository, SessionRepository, PasskeyRepository
- Implementación de RoleRepository, PermissionRepository, etc.
- Integración con Redis para sesiones
- Estrategias de seguridad para almacenamiento de datos sensibles

**Dependencias:**
- Código completo de Fases 1-3

### Fase 5: Implementación del Patrón Mediator para CQRS

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 6: **Implementación del Patrón CQRS** (6.1)
- Sección 9.8: Mediator Pattern

**Entregables principales:**
- Implementación del patrón Mediator
- Pipeline de procesamiento con validación, logging, etc.
- Integración con Unit of Work
- Sistema de eventos a través del mediator

**Dependencias:**
- Código completo de Fases 1-4

### Fase 6: Implementación de Comandos CQRS para EAV

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 6.1.1: Modelo de Comandos (Escritura)
- Sección 6.3: Gestión de Eventos de Dominio

**Entregables principales:**
- Implementación de comandos para LogicalTable, ColumnDefinition, TableRow
- Implementación de comandos para gestión de restricciones
- Validadores para cada comando
- Sistema de validación de restricciones

**Dependencias:**
- Código completo de Fases 1-5

### Fase 7: Implementación de Queries CQRS para EAV

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 6.1.2: Modelo de Consultas (Lectura)
- Sección 6.2: Ejemplo de Implementación de Query Optimizada
- Sección 11.2: Optimizaciones para PostgreSQL

**Entregables principales:**
- Implementación de queries para LogicalTable, ColumnDefinition, TableRow
- Implementación de queries para restricciones
- Optimizaciones de rendimiento para consultas EAV
- Sistema de caché con invalidación

**Dependencias:**
- Código completo de Fases 1-6

### Fase 8: Comandos y Queries para Autenticación y RBAC

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 7: Sistema de Autenticación (7.1-7.5)
- Sección 12.1: Implementación de RBAC

**Entregables principales:**
- Comandos para usuarios, passkeys, sesiones y roles
- Queries para usuarios, autenticación y RBAC
- Implementación de handlers para estos comandos y queries

**Dependencias:**
- Código completo de Fases 1-7

### Fase 9: Comandos y Queries para Reportes y Migración

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 18: **Sistema de Queries Dinámicos** (18.1-18.2)
- Sección 19: **Sistema de Migración de Datos** (19.1-19.2)

**Entregables principales:**
- Implementación de tablas lógicas para reportes y migración
- Comandos y queries para sistema de reportes
- Comandos y queries para sistema de migración
- Implementación de handlers correspondientes

**Dependencias:**
- Código completo de Fases 1-8

### Fase 10: Autenticación con Passkeys

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 7: **Sistema de Autenticación con Passkeys** (7.1-7.4)
- Sección 20.3.1: Diagrama de Secuencia de Autenticación con Passkeys

**Entregables principales:**
- Implementación completa del servicio de Passkeys
- Flujos de registro y autenticación con WebAuthn
- Implementación del Factory Method para proveedores
- Adaptadores para Microsoft (estructura)

**Dependencias:**
- Código completo de Fases 1-9

### Fase 11: Autenticación Tradicional y Gestión de Sesiones

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 7.5: Gestión de Sesiones
- Sección 12.0: Seguridad de Contraseñas Tradicionales
- Sección 12.2: Middleware de Autenticación

**Entregables principales:**
- Sistema de autenticación con contraseña usando Argon2id
- Gestión de sesiones con JWT y revocación
- Sistema RBAC para autorización
- Políticas de contraseñas y protección contra ataques

**Dependencias:**
- Código completo de Fases 1-10

### Fase 12: Motor de Ejecución de Reportes

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 18: **Sistema de Queries Dinámicos** (18.3-18.4)

**Entregables principales:**
- Motor de ejecución de reportes
- Procesamiento de SQL con parámetros
- Validación de seguridad para consultas dinámicas
- Sistema de auditoría para ejecución de reportes

**Dependencias:**
- Código completo de Fases 1-11

### Fase 13: Motor de Migración de Datos

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 19: **Sistema de Migración de Datos** (19.3-19.4)

**Entregables principales:**
- Motor de migración con worker pool para procesamiento paralelo
- Sistema de transformación y validación de datos
- Gestión de errores y recuperación
- Sistema de reportes de progreso en tiempo real

**Dependencias:**
- Código completo de Fases 1-12

### Fase 14: API HTTP Base y Middleware

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 10: **API y Contratos** (10.1-10.2)
- Sección 12.3: Middleware de Autenticación
- Sección 12.2: Protección contra Ataques Comunes

**Entregables principales:**
- Configuración del servidor HTTP con chi router
- Implementación de middleware base (logger, recuperación, trazas)
- Middleware de seguridad (autenticación, CSRF, cabeceras)
- Gestión centralizada de errores HTTP

**Dependencias:**
- Código completo de Fases 1-13

### Fase 15: API HTTP para EAV y Autenticación

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 10.1: API RESTful para Datos EAV
- Sección 10.2: API para Autenticación
- Sección 10.3: Documentación de API con OpenAPI/Swagger

**Entregables principales:**
- Handlers para EAV (LogicalTable, ColumnDefinition, TableRow)
- Handlers para autenticación (Passkey, Password, Session)
- Handlers para RBAC (Role, Permission, UserRole)
- Anotaciones Swagger para todos los endpoints

**Dependencias:**
- Código completo de Fases 1-14

### Fase 16: API HTTP para Reportes y Migración

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 18.3: API para Reportes Dinámicos
- Sección 19.3: API para Migración de Datos
- Sección 10.3: Documentación de API con OpenAPI/Swagger

**Entregables principales:**
- Handlers para reportes (definición, ejecución, etc.)
- Handlers para migración (configuración, estado, errores)
- Endpoints para estadísticas y monitoreo
- Anotaciones Swagger para todos los endpoints

**Dependencias:**
- Código completo de Fases 1-15

### Fase 17: Integración y Refinamiento de Documentación API

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 10.3: Documentación de API con OpenAPI/Swagger
- Sección 10.4: Gestión de Versiones de API

**Entregables principales:**
- Configuración completa del generador swaggo/swag
- UI personalizada de Swagger
- Especificaciones OpenAPI finales
- Pruebas automatizadas para verificar la documentación

**Dependencias:**
- Código completo de Fases 1-16

### Fase 18: Observabilidad y Monitoreo

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 13: **Observabilidad y Monitoreo** (13.1-13.5)

**Entregables principales:**
- Configuración avanzada de logging estructurado
- Métricas completas con Prometheus
- Trazas distribuidas con OpenTelemetry
- Health checks y readiness probes

**Dependencias:**
- Código completo de Fases 1-17

### Fase 19: Testing - Pruebas Unitarias y de Integración

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 14: **Estrategia de Pruebas** (14.1-14.2)

**Entregables principales:**
- Mocks para todas las interfaces
- Pruebas unitarias para entidades de dominio
- Pruebas unitarias para comandos y queries
- Pruebas de integración con testcontainers

**Dependencias:**
- Código completo de Fases 1-18

### Fase 20: Testing - E2E y Rendimiento

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 14: **Estrategia de Pruebas** (14.3-14.5)

**Entregables principales:**
- Pruebas E2E para flujos principales
- Pruebas de API con httptest
- Pruebas de rendimiento para operaciones críticas
- Pruebas de carga con k6

**Dependencias:**
- Código completo de Fases 1-19

### Fase 21: Containerización y Configuración Local

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 15: **Implementación DevOps** (15.3.1)

**Entregables principales:**
- Dockerfile optimizado con multi-stage building
- Docker Compose para entorno completo
- Scripts de inicialización para bases de datos
- Documentación para desarrollo local

**Dependencias:**
- Código completo de Fases 1-20

### Fase 22: Despliegue Kubernetes

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 15: **Implementación DevOps** (15.3.2, 15.4, 15.5)

**Entregables principales:**
- Manifiestos Kubernetes completos
- Helm charts para despliegue simplificado
- Configuración de autoscaling y health checks
- Gestión de secretos

**Dependencias:**
- Código completo de Fases 1-21

### Fase 23: CI/CD y Automatización

**Secciones críticas del requerimiento-EAV-Go.md:**
- Sección 15: **Implementación DevOps** (15.1, 15.2)
- Sección 16: Riesgos y Mitigaciones
- Sección 20.5: Estrategia de Gestión de Versiones

**Entregables principales:**
- Configuración de GitHub Actions para CI/CD
- Configuración de ArgoCD para GitOps
- Scripts de migración de base de datos
- Estrategias de despliegue seguro

**Dependencias:**
- Código completo de Fases 1-22

## Consideraciones Especiales

1. **Correlación entre fases**: Algunas fases son altamente interdependientes (6-7, 10-11, 12-13, etc.). La IA asignada a estas fases debe revisar cuidadosamente ambas secciones del plan.

2. **Zonas protegidas**: Seguir estrictamente las directrices sobre zonas protegidas del código. Una IA no debe modificar código fuera de su alcance.

3. **Formato de entrega**: Cada IA debe entregar su trabajo en el formato consolidado especificado en las directrices, con marcadores claros para cada archivo.

4. **Continuidad**: Mantener absoluta coherencia con el trabajo realizado en fases anteriores, siguiendo patrones y convenciones establecidos.

5. **Validación**: Antes de entregar, cada IA debe ejecutar las validaciones obligatorias (lint, tests) para asegurar conformidad.
