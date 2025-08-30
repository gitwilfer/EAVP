# Entregables Entre Fases - Proyecto EAV Híbrido con Passkeys

Este documento detalla los entregables específicos que cada fase debe proporcionar a la siguiente, garantizando transiciones fluidas entre ingenieros y coherencia en el desarrollo del proyecto EAV Híbrido.

## Sesión 1: Estructura Base y Modelo de Dominio EAV → Sesión 2

**Entregables clave para la siguiente fase:**
- Código fuente completo de las entidades de dominio (LogicalTable, ColumnDefinition, TableRow, ColumnValue, etc.)
- Interfaces de todos los repositorios definidos en `/internal/domain/repository/`
- Enumeraciones y tipos de valor del dominio definidos en `/internal/domain/valueobjects/`
- Definiciones de error estandarizadas en `/internal/domain/errors/`
- Diagrama UML o similar que ilustre las relaciones entre entidades del dominio
- Documentación de extensibilidad del modelo para implementaciones futuras
- Lista de verificación de cumplimiento con los principios DDD implementados

## Sesión 2: Infraestructura Base y Patrones Core → Sesión 3

**Entregables clave para la siguiente fase:**
- Implementación completa del patrón Circuit Breaker en `/internal/infrastructure/resilience/`
- Código del patrón Bulkhead para aislamiento de recursos en `/internal/infrastructure/resilience/`
- Implementación del patrón Outbox en `/internal/infrastructure/events/`
- Implementación completa del Unit of Work para transacciones en `/internal/infrastructure/persistence/`
- Configuraciones PostgreSQL y Redis funcionales en `/configs/`
- Implementación de caché multinivel (memoria + Redis) en `/internal/infrastructure/cache/`
- Pruebas funcionales que demuestren el correcto funcionamiento de cada patrón
- Documentación detallada sobre cómo integrar estos patrones con los repositorios

## Sesión 3: Implementación de Repositorios EAV → Sesión 4

**Entregables clave para la siguiente fase:**
- Implementaciones completas de:
  - `LogicalTableRepository` en `/internal/infrastructure/persistence/postgres/`
  - `ColumnDefinitionRepository` en `/internal/infrastructure/persistence/postgres/`
  - `TableRowRepository` en `/internal/infrastructure/persistence/postgres/`
  - `ColumnValueRepository` en `/internal/infrastructure/persistence/postgres/`
  - `TableConstraintRepository` en `/internal/infrastructure/persistence/postgres/`
- Scripts SQL para índices y optimizaciones en `/db/migrations/`
- Implementación de consultas optimizadas para campos JSONB
- Implementación de estrategias de caché con invalidación para entidades EAV
- Pruebas de integración para todas las implementaciones de repositorio
- Documentación de patrones de uso con ejemplos concretos de operaciones CRUD
- Guía de troubleshooting para problemas comunes de acceso a datos

## Sesión 4: Repositorios para Autenticación y RBAC → Sesión 5

**Entregables clave para la siguiente fase:**
- Implementaciones completas de:
  - `UserRepository` en `/internal/infrastructure/persistence/postgres/`
  - `SessionRepository` en `/internal/infrastructure/persistence/postgres/`
  - `PasskeyRepository` en `/internal/infrastructure/persistence/postgres/`
  - `RoleRepository` en `/internal/infrastructure/persistence/postgres/`
  - `PermissionRepository` en `/internal/infrastructure/persistence/postgres/`
  - `UserRoleRepository` en `/internal/infrastructure/persistence/postgres/`
  - `RolePermissionRepository` en `/internal/infrastructure/persistence/postgres/`
- Integración funcional con Redis para almacenamiento de sesiones
- Implementación segura para almacenamiento de contraseñas con Argon2id
- Estrategias de caché para permisos con invalidación basada en eventos
- Pruebas de integración para todas las implementaciones de repositorio
- Diagrama de relaciones entre entidades de autenticación y RBAC
- Ejemplos de consultas para verificación de permisos optimizadas

## Sesión 5: Implementación del Patrón Mediator para CQRS → Sesión 6

**Entregables clave para la siguiente fase:**
- Implementación completa del patrón Mediator en `/internal/application/mediator/`
- Pipeline de procesamiento con validación, logging y manejo de errores
- Interfaces para controladores de comandos y queries
- Implementación de resolución de dependencias para handlers
- Integración del Unit of Work con Mediator para transacciones atómicas
- Sistema de eventos CQRS con publicación a través del mediator
- Pruebas unitarias completas del patrón Mediator
- Diagrama de secuencia que ilustre el flujo completo de procesamiento
- Ejemplos de handlers para comandos y queries simples
- Guía de implementación para añadir nuevos comandos y queries

## Sesión 6: Implementación de Comandos CQRS para EAV → Sesión 7

**Entregables clave para la siguiente fase:**
- Implementación completa de comandos para LogicalTable en `/internal/application/commands/`
- Implementación completa de comandos para ColumnDefinition en `/internal/application/commands/`
- Implementación completa de comandos para TableRow en `/internal/application/commands/`
- Implementación de comandos para gestión de restricciones
- Validadores para cada comando usando validator/v10
- Sistema de validación de restricciones (unicidad, llave foránea, check)
- Implementación de handlers para cada comando en `/internal/application/handlers/`
- Integración completa con el sistema de eventos vía Outbox
- Pruebas unitarias para todos los comandos y sus handlers
- Ejemplos de uso para escenarios comunes de actualización de datos
- Documentación de casos de error y manejo de excepciones

## Sesión 7: Implementación de Queries CQRS para EAV → Sesión 8

**Entregables clave para la siguiente fase:**
- Implementación completa de queries para LogicalTable en `/internal/application/queries/`
- Implementación completa de queries para ColumnDefinition en `/internal/application/queries/`
- Implementación completa de queries para TableRow en `/internal/application/queries/`
- Implementación de queries para consulta de restricciones
- Implementación de handlers para cada query en `/internal/application/handlers/`
- Sistema de paginación keyset-based para consultas de alto volumen
- Optimizaciones de rendimiento con caché para consultas frecuentes
- Pruebas unitarias para todas las queries y sus handlers
- Consultas específicas para propiedades JSON utilizando JSONB
- Ejemplos de proyecciones selectivas para reducir tamaño de respuesta
- Documentación de rendimiento y estrategias de optimización para cada query

## Sesión 8: Comandos y Queries para Autenticación y RBAC → Sesión 9

**Entregables clave para la siguiente fase:**
- Implementación completa de:
  - Comandos para gestión de usuarios en `/internal/application/commands/auth/`
  - Comandos para gestión de passkeys en `/internal/application/commands/auth/`
  - Comandos para gestión de sesiones en `/internal/application/commands/auth/`
  - Comandos para gestión RBAC en `/internal/application/commands/auth/`
  - Queries para usuarios y autenticación en `/internal/application/queries/auth/`
  - Queries para RBAC en `/internal/application/queries/auth/`
- Handlers para todos los comandos y queries implementados
- Implementación del flujo de cambio de contraseña con validación
- Sistema completo de verificación de permisos basado en roles
- Pruebas unitarias para todos los comandos, queries y handlers
- Diagramas de secuencia para flujos críticos (asignación de roles, verificación de permisos)
- Ejemplos de uso para verificación de permisos en diferentes escenarios
- Documentación de seguridad para gestión de sesiones y contraseñas

## Sesión 9: Comandos y Queries para Reportes y Migración → Sesión 10

**Entregables clave para la siguiente fase:**
- Implementación de tablas lógicas EAV para reportes:
  - `REPORT_DEFINITIONS`, `REPORT_QUERIES`, `REPORT_RESPONSE_CONTRACTS`, etc.
- Implementación de tablas lógicas EAV para migración:
  - `MIGRATION_CONFIGURATIONS`, `MIGRATION_FIELD_MAPPINGS`, etc.
- Comandos completos para:
  - Sistema de reportes en `/internal/application/commands/reports/`
  - Sistema de migración en `/internal/application/commands/migration/`
- Queries completas para:
  - Sistema de reportes en `/internal/application/queries/reports/`
  - Sistema de migración en `/internal/application/queries/migration/`
- Handlers para todos los comandos y queries implementados
- Pruebas unitarias completas para ambos subsistemas
- Diagrama de relaciones entre entidades lógicas de reportes y migración
- Ejemplos de definición de reportes y configuraciones de migración
- Documentación de la relación entre el modelo EAV y los subsistemas implementados

## Sesión 10: Autenticación con Passkeys → Sesión 11

**Entregables clave para la siguiente fase:**
- Implementación completa del servicio de Passkeys en `/internal/application/services/auth/`
- Integración funcional con WebAuthn mediante go-webauthn/webauthn
- Implementación del flujo de registro de passkeys:
  - Generación de opciones y desafío
  - Almacenamiento de estado de registro
  - Verificación y almacenamiento de credenciales
- Implementación del flujo de autenticación con passkeys:
  - Generación de opciones y desafío
  - Verificación de aserción
  - Creación de sesión
- Implementación del patrón Factory Method para diferentes proveedores
- Adaptadores para Microsoft (estructura preparada)
- Integración con métricas y trazas para monitoreo del proceso
- Pruebas unitarias y de integración para todos los flujos
- Diagramas de secuencia detallados para registro y autenticación
- Documentación de troubleshooting para problemas comunes con WebAuthn

## Sesión 11: Autenticación Tradicional y Gestión de Sesiones → Sesión 12

**Entregables clave para la siguiente fase:**
- Implementación completa del sistema de autenticación con contraseña:
  - Servicio de hashing con Argon2id en `/internal/application/services/auth/`
  - Validación segura de contraseñas
  - Políticas configurables de contraseñas
  - Sistema de restablecimiento de contraseñas
- Sistema completo de limitación de intentos y bloqueo temporal
- Implementación de la gestión de sesiones:
  - Generación de JWT con firma
  - Almacenamiento de sesiones activas en Redis
  - Revocación global de tokens
  - Rotación de claves de firma
  - Refresh tokens con manejo seguro
- Implementación del sistema RBAC para autorización:
  - Verificación de permisos por acción y recurso
  - Caché optimizada de permisos
- Pruebas unitarias y de integración para todos los componentes
- Diagrama completo de los flujos de autenticación y autorización
- Ejemplos de uso para escenarios comunes de autenticación
- Guía de integración del sistema de autenticación con el API

## Sesión 12: Motor de Ejecución de Reportes → Sesión 13

**Entregables clave para la siguiente fase:**
- Implementación completa del motor de reportes en `/internal/application/services/reports/`
- Sistema de procesamiento de SQL con parámetros seguros
- Validación de seguridad para consultas dinámicas
- Transformación de resultados según contratos predefinidos
- Sistema de paginación para resultados grandes
- Implementación de seguridad para ejecución de reportes:
  - Validación de permisos
  - Protección contra inyección SQL
  - Limitación de recursos
- Sistema de auditoría para ejecución de reportes
- Soporte para diferentes formatos de salida (JSON, CSV)
- Caché de resultados con invalidación basada en tiempo
- Pruebas unitarias y de integración para el motor completo
- Ejemplos de reportes complejos con parámetros y transformaciones
- Documentación de rendimiento y optimización de consultas

## Sesión 13: Motor de Migración de Datos → Sesión 14

**Entregables clave para la siguiente fase:**
- Implementación completa del motor de migración en `/internal/application/services/migration/`
- Worker pool para procesamiento paralelo:
  - Control de progreso en tiempo real
  - Gestión de errores y recuperación
  - Procesador de eventos batch
- Sistema de validación de datos durante migración
- Implementación de transformaciones:
  - Conversión de tipos
  - Formateo de datos
  - Validaciones personalizadas
- Estrategias para manejo de errores:
  - Continuar con errores
  - Detener en error
  - Umbrales configurables
- Sistema de reportes de progreso en tiempo real
- Backoff exponencial para reintentos
- Pruebas unitarias y de integración del motor completo
- Ejemplos de configuraciones de migración complejas
- Documentación de patrones de resiliencia aplicados
- Guía de troubleshooting para problemas comunes de migración

## Sesión 14: API HTTP Base y Middleware → Sesión 15

**Entregables clave para la siguiente fase:**
- Configuración completa del servidor HTTP con chi router en `/internal/interfaces/http/`
- Implementación de middleware base:
  - Logger middleware para registro estructurado
  - Recuperación de pánicos
  - Trazas y correlación de solicitudes
  - Métricas de solicitudes HTTP
  - Timeouts configurables
- Middleware de seguridad:
  - Autenticación JWT
  - Control de sesiones
  - Rate limiting
  - Protección CSRF
  - Cabeceras de seguridad HTTP
- Middleware de autorización RBAC
- Configuración CORS completa
- Gestión centralizada de errores HTTP
- Base para documentación OpenAPI
- Pruebas unitarias para todos los middleware
- Ejemplos de uso de cada middleware
- Documentación detallada de la configuración HTTP

## Sesión 15: API HTTP para EAV y Autenticación → Sesión 16

**Entregables clave para la siguiente fase:**
- Implementación completa de handlers para EAV:
  - `LogicalTableHandler` en `/internal/interfaces/http/handlers/`
  - `ColumnDefinitionHandler` en `/internal/interfaces/http/handlers/`
  - `TableRowHandler` en `/internal/interfaces/http/handlers/`
- Implementación completa de handlers para autenticación:
  - `PasskeyRegistrationHandler` en `/internal/interfaces/http/handlers/auth/`
  - `PasskeyAuthenticationHandler` en `/internal/interfaces/http/handlers/auth/`
  - `PasswordAuthHandler` en `/internal/interfaces/http/handlers/auth/`
  - `SessionHandler` en `/internal/interfaces/http/handlers/auth/`
- Implementación completa de handlers para RBAC:
  - `RoleHandler` en `/internal/interfaces/http/handlers/rbac/`
  - `PermissionHandler` en `/internal/interfaces/http/handlers/rbac/`
  - `UserRoleHandler` en `/internal/interfaces/http/handlers/rbac/`
- Validación de inputs con validator/v10
- Implementación de ruteo con chi router
- Versionado de API (v1)
- Transformación de errores de dominio a HTTP
- Todos los handlers con anotaciones Swagger completas
- Pruebas unitarias para todos los handlers
- Ejemplos de solicitudes HTTP para cada endpoint (curl/httpie)
- Documentación de códigos de respuesta y formatos

## Sesión 16: API HTTP para Reportes y Migración → Sesión 17

**Entregables clave para la siguiente fase:**
- Implementación completa de handlers para reportes:
  - `ReportDefinitionHandler` en `/internal/interfaces/http/handlers/reports/`
  - `ReportParameterHandler` en `/internal/interfaces/http/handlers/reports/`
  - `ReportQueryHandler` en `/internal/interfaces/http/handlers/reports/`
  - `ReportExecutionHandler` en `/internal/interfaces/http/handlers/reports/`
  - `ReportPermissionHandler` en `/internal/interfaces/http/handlers/reports/`
- Implementación completa de handlers para migración:
  - `MigrationConfigurationHandler` en `/internal/interfaces/http/handlers/migration/`
  - `MigrationFieldMappingHandler` en `/internal/interfaces/http/handlers/migration/`
  - `MigrationJobHandler` en `/internal/interfaces/http/handlers/migration/`
  - `MigrationStatusHandler` en `/internal/interfaces/http/handlers/migration/`
  - `MigrationErrorHandler` en `/internal/interfaces/http/handlers/migration/`
- Endpoints para estadísticas y monitoreo
- Soporte para streaming de resultados grandes
- Handlers para uploads de archivos de migración
- Todos los handlers con anotaciones Swagger completas
- Pruebas unitarias para todos los handlers
- Pruebas de integración para flujos completos
- Ejemplos de solicitudes HTTP para cada endpoint
- Documentación de formatos de entrada/salida para cada endpoint

## Sesión 17: Integración y Refinamiento de Documentación API → Sesión 18

**Entregables clave para la siguiente fase:**
- Configuración completa del generador swaggo/swag en `/cmd/`
- Integración con el servidor HTTP existente
- Implementación de la UI personalizada de Swagger:
  - Interfaz con tema del proyecto
  - Agrupación lógica de endpoints
  - Funcionalidad Try-it-out completamente funcional
- Validación y refinamiento de toda la documentación existente:
  - Coherencia entre endpoints
  - Completitud en todos los parámetros
  - Mejora de ejemplos y descripciones
- Especificaciones OpenAPI finales en `/api/openapi.json` y `/api/openapi.yaml`
- Scripts de generación y validación automatizada en `/scripts/`
- Endpoints específicos para acceso a documentación de diferentes versiones
- Pruebas automatizadas para verificar la integridad de la documentación
- Manual de mantenimiento y actualización de la documentación
- Ejemplos de uso de la API documentada con Postman/Insomnia

## Sesión 18: Observabilidad y Monitoreo → Sesión 19

**Entregables clave para la siguiente fase:**
- Configuración avanzada de logging estructurado con Zap:
  - Logging contextual en `/internal/infrastructure/logging/`
  - Rotación de logs
  - Niveles configurables
  - Integración con sistemas centralizados
- Implementación completa de métricas con Prometheus:
  - Métricas HTTP (latencia, errores, solicitudes)
  - Métricas de bases de datos (queries, conexiones)
  - Métricas de caché (hit ratio, evictions)
  - Métricas de negocio (operaciones EAV, autenticaciones)
  - Métricas para circuit breaker y bulkhead
- Configuración de OpenTelemetry para trazas distribuidas:
  - Propagación de contexto entre servicios
  - Sampling configurable
  - Correlación entre logs, métricas y trazas
- Health checks y readiness probes en `/internal/interfaces/http/handlers/health/`
- Dashboards Grafana preconfigurados en `/deployment/grafana/`
- Configuración de alertas en `/deployment/prometheus/`
- Documentación completa del sistema de observabilidad
- Ejemplos de análisis y troubleshooting usando las herramientas implementadas

## Sesión 19: Testing - Pruebas Unitarias y de Integración → Sesión 20

**Entregables clave para la siguiente fase:**
- Configuración completa del framework de testing en `/test/`
- Implementación de mocks para todas las interfaces:
  - Repositorios mock en `/test/mocks/repositories/`
  - Servicios mock en `/test/mocks/services/`
  - Elementos de infraestructura mock en `/test/mocks/infrastructure/`
- Pruebas unitarias para todas las entidades de dominio en `/internal/domain/`
- Pruebas unitarias para todos los comandos y queries
- Pruebas unitarias para todos los servicios de aplicación
- Pruebas unitarias para todos los validadores
- Configuración de testcontainers para pruebas de integración:
  - Contenedor PostgreSQL
  - Contenedor Redis
- Pruebas de integración para repositorios en `/test/integration/`
- Pruebas de integración para servicios que requieren BD
- Configuración de cobertura de código
- Scripts para ejecución de suites de pruebas en `/scripts/test/`
- Documentación de metodología de pruebas
- Guía para añadir nuevas pruebas manteniendo la coherencia

## Sesión 20: Testing - E2E y Rendimiento → Sesión 21

**Entregables clave para la siguiente fase:**
- Configuración completa para pruebas E2E en `/test/e2e/`
- Pruebas E2E para flujos principales:
  - Registro y autenticación completa
  - Gestión del modelo EAV
  - Ejecución de reportes
  - Migración de datos
- Pruebas de API con httptest:
  - Pruebas para todos los endpoints
  - Pruebas de seguridad y autorización
  - Pruebas de validación de inputs
- Pruebas de rendimiento para operaciones críticas:
  - Benchmarks para consultas EAV
  - Benchmarks para autenticación
  - Benchmarks para ejecución de reportes
  - Análisis de rendimiento con pprof
- Pruebas de carga con k6:
  - Escenarios realistas
  - Simulación de carga gradual
  - Análisis de resultados
- Verificación de memory leaks
- Scripts para ejecución automatizada en `/scripts/test/performance/`
- Documentación de líneas base de rendimiento
- Guía de análisis y optimización de rendimiento

## Sesión 21: Containerización y Configuración Local → Sesión 22

**Entregables clave para la siguiente fase:**
- Dockerfile optimizado en la raíz del proyecto:
  - Etapa de compilación
  - Etapa de ejecución mínima
  - Optimizaciones de tamaño y seguridad
- Docker Compose completo en `/deployment/docker-compose/`:
  - Servicio API
  - PostgreSQL
  - Redis
  - Prometheus
  - Grafana
  - Jaeger para trazas
  - Loki para logs
- Scripts de inicialización para bases de datos en `/deployment/init/`
- Configuración de variables de entorno en `.env.example`
- Gestión de secretos para desarrollo
- Scripts de conveniencia en `/scripts/`:
  - Comandos make en `Makefile`
  - Scripts de inicio rápido
  - Scripts de reinicio
- Documentación completa para desarrollo local
- Pruebas de integración del entorno completo
- Guía de troubleshooting para entorno de desarrollo
- Ejemplos de configuración para diferentes escenarios

## Sesión 22: Despliegue Kubernetes → Sesión 23

**Entregables clave para la siguiente fase:**
- Manifiestos Kubernetes completos en `/deployment/kubernetes/`:
  - Deployments para todos los servicios
  - StatefulSets para bases de datos
  - ConfigMaps para configuración
  - Secrets para información sensible
  - Services para comunicación interna
  - Ingress para exposición externa
- Helm charts para despliegue simplificado en `/deployment/helm/`:
  - Plantillas para todos los recursos
  - Valores por defecto
  - Opciones de configuración
- Configuración de autoscaling horizontal (HPA)
- Implementación de sondas de readiness y liveness
- Estrategias de rolling update
- Configuración de recursos (CPU/memoria)
- Configuración de PersistentVolumes para datos
- Configuración de NetworkPolicies para seguridad
- Scripts de despliegue en `/scripts/deploy/`
- Pruebas de verificación post-despliegue
- Documentación detallada del proceso de despliegue
- Guía de troubleshooting para problemas comunes en Kubernetes

## Sesión 23: CI/CD y Automatización

**Entregables para operaciones:**
- Configuración de GitHub Actions en `/.github/workflows/`:
  - Build y test automatizados
  - Análisis estático de código
  - Verificación de seguridad
  - Construcción y publicación de imágenes Docker
  - Despliegue automatizado en entornos
- Configuración de ArgoCD para GitOps en `/deployment/argocd/`:
  - Configuración de aplicaciones
  - Sincronización automática
  - Políticas de rollback
- Scripts de migración de base de datos para despliegues en `/db/migrations/`
- Implementación de canary deployments y blue/green
- Configuración de monitoreo post-despliegue
- Automatización de rollbacks
- Pipelines separados para diferentes entornos
- Gestión de secretos en CI/CD
- Verificaciones de seguridad automatizadas
- Documentación completa del pipeline CI/CD
- Manual de operaciones y procedimientos de emergencia
- Diagrama de flujo completo del proceso CI/CD
