# Plan Detallado de Implementación - Proyecto EAV Híbrido con Passkeys

## Metodología de Entrega y Continuidad

Para garantizar el éxito en la implementación del proyecto, cada sesión:

1. Producirá un **archivo consolidado** siguiendo estrictamente el formato especificado
2. Incluirá **rutas completas** para cada archivo generado
3. Contendrá un **resumen detallado** del trabajo completado
4. Proporcionará **instrucciones específicas** para la siguiente sesión
5. Implementará **verificación continua** contra los requerimientos

Cada entregable será autocontenido y completamente funcional, permitiendo continuar con el trabajo en sesiones subsecuentes sin pérdida de contexto.

## Revisión del Modelo EAV Híbrido

El proyecto implementa un modelo EAV Híbrido que sirve como base para todas las entidades del sistema. Es importante destacar que **todas las entidades lógicas** (incluidos Reportes, Migración, etc.) se implementarán utilizando este modelo EAV subyacente, sin necesidad de crear tablas físicas separadas para cada tipo de entidad.

### Estructura Base de Tablas Físicas:

#### Tablas Relacionales Convencionales:
- **logical_tables**: Define los tipos de entidades en el sistema
- **column_definitions**: Define los atributos disponibles para cada tipo de entidad
- **table_column_properties**: Define propiedades de las columnas en cada tabla lógica

#### Tablas Centrales EAV Híbridas:
- **table_rows**: Almacena instancias de entidades con soporte para extended_properties (JSONB)
- **column_values**: Almacena valores de atributos para entidades

#### Tablas de Autenticación y Passkeys:
- **app_users**: Almacena información de usuarios
- **user_passkeys**: Almacena credenciales de WebAuthn

#### Tabla de Eventos de Dominio para Patrón Outbox:
- **domain_events**: Almacena eventos de dominio para publicación confiable

#### Tablas para RBAC:
- **roles**: Define roles en el sistema
- **permissions**: Define permisos disponibles en el sistema
- **user_roles**: Asocia usuarios con roles
- **role_permissions**: Asocia roles con permisos

#### Tablas para Integridad Referencial:
- **table_constraints**: Define restricciones a nivel de tabla lógica (PRIMARY KEY, UNIQUE, FOREIGN KEY, CHECK)
- **constraint_columns**: Define qué columnas participan en cada restricción (permite llaves compuestas)
- **unique_indexes**: Define estrategias de indexación para restricciones únicas
- **unique_index_json_properties**: Permite incluir propiedades JSON en índices únicos
- **foreign_key_references**: Define relaciones entre tablas con reglas de actualización/eliminación
- **check_constraints**: Almacena expresiones de validación para restricciones CHECK

### Aclaración importante sobre subsistemas:

**Los sistemas de Reportes y Migración NO se implementarán como tablas físicas adicionales en la base de datos**. En su lugar, se implementarán como tablas lógicas dentro del modelo EAV:

1. Se crearán registros en la tabla física `logical_tables` para representar conceptos como:
   - `REPORT_DEFINITIONS`
   - `REPORT_QUERIES`
   - `ReportResponseContracts`
   - `REPORT_PARAMETERS`
   - `ReportPermissions`
   - `ReportVersions`
   - `MIGRATION_CONFIGURATIONS`
   - `MIGRATION_FIELD_MAPPINGS`
   - etc.

2. Los atributos de estas "tablas lógicas" se definirán como registros en `column_definitions` y se asociarán mediante `table_column_properties`.

3. Los datos concretos (registros) se almacenarán en `table_rows` y `column_values`, manteniendo la integridad del modelo EAV.

4. Las relaciones entre estas entidades lógicas (por ejemplo, entre un reporte y sus parámetros) se manejarán mediante las tablas de integridad referencial (`table_constraints`, etc.).

Esta arquitectura flexible permite modelar cualquier tipo de entidad sin modificar el esquema, lo que es el propósito fundamental del patrón EAV, mientras mantiene la integridad referencial propia de los sistemas relacionales tradicionales.

## Sesiones de Implementación

### Sesión 1: Estructura Base y Modelo de Dominio EAV
**Alcance detallado:**
- Configuración completa de la estructura del proyecto según Clean Architecture
- Implementación de las entidades base del modelo EAV Híbrido:
  - **LogicalTable**: Representa `logical_tables`
  - **ColumnDefinition**: Representa `column_definitions`
  - **TableColumnProperty**: Representa `table_column_properties`
  - **TableRow**: Representa `table_rows` con soporte para extended_properties (JSONB)
  - **ColumnValue**: Representa `column_values`
- Implementación de entidades para autenticación:
  - **User**: Representa `app_users`
  - **Session**: Para gestión de sesiones
  - **Passkey**: Representa `user_passkeys`
  - **PasskeyCredential**: Para gestión de credenciales WebAuthn
- Entidades para RBAC:
  - **Role**: Representa `roles`
  - **Permission**: Representa `permissions`
  - **UserRole**: Representa `user_roles`
  - **RolePermission**: Representa `role_permissions`
- Entidades para integridad referencial:
  - **TableConstraint**: Representa `table_constraints`
  - **ConstraintColumn**: Representa `constraint_columns`
  - **UniqueIndex**: Representa `unique_indexes`
  - **UniqueIndexJsonProperty**: Representa `unique_index_json_properties`
  - **ForeignKeyReference**: Representa `foreign_key_references`
  - **CheckConstraint**: Representa `check_constraints`
- Entidad **DomainEvent** para el patrón Outbox (representa `domain_events`)
- Interfaces de repositorio para todas las entidades:
  - LogicalTableRepository
  - ColumnDefinitionRepository
  - TableRowRepository
  - ColumnValueRepository
  - UserRepository
  - PasskeyRepository
  - SessionRepository
  - RoleRepository
  - PermissionRepository
  - UserRoleRepository
  - RolePermissionRepository
  - TableConstraintRepository
  - ConstraintRepository (para gestión de todas las restricciones)
  - DomainEventRepository
  - RolePermissionRepository
  - ForeignKeyReferenceRepository
  - UniqueIndexRepository
  - UniqueIndexJsonPropertyRepository
  - CheckConstraintRepository
- Contratos de servicios del dominio
- Errores de dominio estandarizados
- Definición de enumeraciones y tipos valor del dominio
- Configuración de módulos Go y dependencias base

**Entregable:** Modelo de dominio EAV completo con todas las entidades e interfaces.

### Sesión 2: Infraestructura Base y Patrones Core
**Alcance detallado:**
- Implementación de la conexión a PostgreSQL con pooling y monitoreo
- Implementación de la conexión a Redis con configuración óptima
- Configuración completa de logging estructurado con Zap
- Implementación completa del patrón Circuit Breaker con ejemplos funcionales
- Implementación completa del patrón Bulkhead para aislamiento de recursos
- Implementación del patrón Outbox para publicación confiable de eventos
- Implementación de la estrategia de caché multinivel (memoria + Redis)
- Funciones auxiliares para observabilidad (trazas, métricas, logs)
- Configuración base de métricas con Prometheus
- Mecanismos de carga de configuración y gestión de secretos
- Implementación completa del Unit of Work para transacciones atómicas
- Implementación o integración de un sistema de Feature Flags (utilizando `open-feature/go-sdk` o similar) para gestión dinámica de funcionalidades.
- Definición de la estrategia para la gestión y actualización de los feature flags.

**Entregable:** Infraestructura base con todos los patrones implementados.

### Sesión 3: Implementación de Repositorios EAV
**Alcance detallado:**
- Implementación completa de LogicalTableRepository
- Implementación completa de ColumnDefinitionRepository
- Implementación completa de TableColumnPropertyRepository
- Implementación completa de TableRowRepository con soporte para campos JSON
- Implementación completa de ColumnValueRepository
- Implementación completa de TableConstraintRepository
- Implementación completa de ConstraintColumnRepository
- Implementación completa de UniqueIndexRepository
- Implementación completa de UniqueIndexJsonPropertyRepository
- Implementación completa de ForeignKeyReferenceRepository
- Implementación completa de CheckConstraintRepository
- Implementación del repositorio para DomainEvents (Outbox)

- Sistema de migraciones de esquema de base de datos
- Implementación de consultas optimizadas para el modelo EAV
- Consultas específicas para búsquedas en campos JSONB
- Integración de los patrones Circuit Breaker y caché en repositorios
- Índices y optimizaciones de consulta para PostgreSQL
- Configuración y verificación del mantenimiento de estadísticas avanzadas (histogramas) en PostgreSQL para columnas clave del modelo EAV, asegurando un plan de ejecución óptimo.
- Particionamiento para tablas grandes (ColumnValue)
- Implementación del repositorio para DomainEvents (Outbox)
- Estrategias de caché para entidades EAV con invalidación
- Implementación de la lógica de control de concurrencia optimista (manejo del campo `version`) en las operaciones de actualización de los repositorios EAV.

**Entregable:** Capa de infraestructura para el modelo EAV completa y optimizada.

### Sesión 4: Repositorios para Autenticación y RBAC
**Alcance detallado:**
- Implementación completa de UserRepository (`app_users`)
- Implementación completa de SessionRepository
- Implementación completa de PasskeyRepository (`user_passkeys`)
- Implementación completa de RoleRepository (`roles`)
- Implementación completa de PermissionRepository (`permissions`)
- Implementación completa de UserRoleRepository (`user_roles`)
- Implementación completa de RolePermissionRepository (`role_permissions`)
- Integración con Redis para almacenamiento de sesiones
- Consultas optimizadas para autenticación y autorización
- Estrategias de caché para permisos y roles con invalidación
- Seguridad para almacenamiento de datos sensibles
- Implementación del repositorio de procesamiento de Outbox

**Entregable:** Capa de infraestructura para autenticación y RBAC completa.

### Sesión 5: Implementación del Patrón Mediator para CQRS
**Alcance detallado:**
- Implementación del patrón Mediator para orquestar comandos y queries
- Configuración del bus de comandos y queries
- Implementación del pipeline de procesamiento con:
  - Validación de comandos
  - Logging
  - Medición de rendimiento
  - Manejo de errores
  - Control de transacciones
  - Trazas distribuidas
- Implementación de handlers base para comandos y queries
- Implementación del sistema de validación con validator/v10, asegurando la configuración con `WithRequiredStructEnabled()` para un manejo robusto de structs nil.
- Implementación de resolución de dependencias para handlers
- Configuración del sistema de eventos a través del mediator
- Integración con Unit of Work

**Entregable:** Framework completo de CQRS con Mediator para comandos y queries.

### Sesión 6: Implementación de Comandos CQRS para EAV
**Alcance detallado:**
- Implementación de comandos para LogicalTable:
  - CreateLogicalTableCommand
  - UpdateLogicalTableCommand
  - DeleteLogicalTableCommand
- Implementación de comandos para ColumnDefinition:
  - CreateColumnDefinitionCommand
  - UpdateColumnDefinitionCommand
  - DeleteColumnDefinitionCommand
- Implementación de comandos para TableRow:
  - CreateTableRowCommand (con soporte para extended_properties)
  - UpdateTableRowCommand
  - DeleteTableRowCommand
  - BatchTableRowsCommand
- Implementación de comandos para relaciones:
  - AddColumnToTableCommand
  - RemoveColumnFromTableCommand
- Implementación de comandos para gestión de restricciones:
  - CreateTableConstraintCommand
  - UpdateTableConstraintCommand
  - DeleteTableConstraintCommand
  - AddConstraintColumnCommand
  - CreateUniqueIndexCommand
  - AddJsonPropertyToUniqueIndexCommand
  - CreateForeignKeyReferenceCommand
  - CreateCheckConstraintCommand
- Implementación de validadores para cada comando
- Implementación del sistema de validación de restricciones:
  - Validador de unicidad (simple y compuesta)
  - Validador de llave foránea
  - Validador de expresiones check
- Integración con el sistema de eventos vía Outbox para cambios
- Integración del control de concurrencia optimista en los handlers de comandos de actualización, incluyendo el manejo de `ErrConcurrentModification`.

**Entregable:** Comandos CQRS para el modelo EAV con validaciones completas.

### Sesión 7: Implementación de Queries CQRS para EAV
**Alcance detallado:**
- Implementación de queries para LogicalTable:
  - GetLogicalTableByIDQuery
  - GetLogicalTableByNameQuery
  - ListLogicalTablesQuery
- Implementación de queries para ColumnDefinition:
  - GetColumnDefinitionByIDQuery
  - GetColumnDefinitionByNameQuery
  - GetColumnsByTableQuery
- Implementación de queries para TableRow:
  - GetTableRowByIDQuery
  - QueryTableRowsQuery (con filtros dinámicos)
  - QueryTableRowsByJSONPropertiesQuery
- Implementación de queries para restricciones:
  - GetTableConstraintsQuery
  - GetUniqueConstraintsQuery
  - GetUniqueIndexJsonPropertiesQuery
  - GetForeignKeyConstraintsQuery
  - GetCheckConstraintsQuery
  - GetConstraintColumnsQuery
  - ValidateConstraintQuery
- Optimizaciones de rendimiento para consultas EAV
- Sistema de caché con invalidación basada en eventos
- Implementación de paginación keyset-based para consultas
- Ordenamiento dinámico para consultas
- Proyección selectiva de campos para reducir tamaño de respuesta
- Diseño e implementación de Vistas Materializadas para consultas EAV frecuentes y agregaciones, incluyendo scripts para su creación y estrategias de refresco.

**Entregable:** Queries CQRS para el modelo EAV con optimizaciones y caché.

### Sesión 8: Comandos y Queries para Autenticación y RBAC
**Alcance detallado:**
- Comandos para usuarios:
  - CreateUserCommand
  - UpdateUserCommand
  - DeleteUserCommand
  - ChangePasswordCommand
- Comandos para passkeys:
  - RegisterPasskeyCommand
  - DeletePasskeyCommand
- Comandos para gestión de sesiones:
  - CreateSessionCommand
  - RevokeSessionCommand
  - RefreshTokenCommand
- Comandos para gestión RBAC:
  - CreateRoleCommand
  - UpdateRoleCommand
  - DeleteRoleCommand
  - AssignRoleToUserCommand
  - RemoveRoleFromUserCommand
  - AddPermissionToRoleCommand
  - RemovePermissionFromRoleCommand
- Queries para usuarios y autenticación:
  - GetUserByIDQuery
  - GetUserByUsernameQuery
  - GetUserByEmailQuery
  - GetUserPasskeysQuery
  - GetActiveSessionsQuery
  - ValidateSessionQuery
- Queries para RBAC:
  - GetUserRolesQuery
  - GetUserPermissionsQuery
  - CheckPermissionQuery
  - GetRolesQuery
  - GetPermissionsQuery

**Entregable:** Comandos y Queries para autenticación y RBAC completos.

### Sesión 9: Comandos y Queries para Reportes y Migración (Implementados como Tablas Lógicas)
**Alcance detallado:**
- Implementación de las tablas lógicas para reportes:
  - Creación de registros en `logical_tables` para:
    - `REPORT_DEFINITIONS`
    - `REPORT_QUERIES`
    - `REPORT_RESPONSE_CONTRACTS`
    - `REPORT_PARAMETERS`
    - `REPORT_PERMISSIONS`
    - `REPORT_VERSIONS`
  - Definición de columnas para cada tabla lógica
  - Configuración de restricciones entre tablas lógicas
- Implementación de las tablas lógicas para migración:
  - Creación de registros en `logical_tables` para:
    - `MIGRATION_CONFIGURATIONS`
    - `MIGRATION_FIELD_MAPPINGS`
    - `MIGRATION_JOBS`
    - `MIGRATION_ERRORS`
  - Definición de columnas para cada tabla lógica
  - Configuración de restricciones entre tablas lógicas
- Comandos para el sistema de reportes (utilizando el modelo EAV):
  - CreateReportDefinitionCommand
  - UpdateReportDefinitionCommand
  - DeleteReportDefinitionCommand
  - SetReportQueryCommand
  - AddReportParameterCommand
  - RemoveReportParameterCommand
  - SetReportPermissionCommand
  - CreateReportVersionCommand
- Queries para el sistema de reportes:
  - GetReportDefinitionQuery
  - ListReportsQuery
  - GetReportParametersQuery
  - GetReportQueryQuery
  - ExecuteReportQuery
- Comandos para el sistema de migración (utilizando el modelo EAV):
  - CreateMigrationConfigurationCommand
  - UpdateMigrationConfigurationCommand
  - DeleteMigrationConfigurationCommand
  - AddMigrationFieldMappingCommand
  - RemoveMigrationFieldMappingCommand
  - StartMigrationJobCommand
  - CancelMigrationJobCommand
- Queries para el sistema de migración:
  - GetMigrationConfigurationQuery
  - ListMigrationConfigurationsQuery
  - GetMigrationJobStatusQuery
  - GetMigrationErrorsQuery
  - GetMigrationStatisticsQuery

**Entregable:** Comandos y Queries para reportes y migración implementados como tablas lógicas en el modelo EAV.

### Sesión 10: Autenticación con Passkeys
**Alcance detallado:**
- Implementación completa del servicio de Passkeys, no se genera autenticacion contra microsoft, pero se deja el desarrollo lo mas estable posible
- Integración con WebAuthn mediante go-webauthn/webauthn
- Implementación del flujo completo de registro de Passkeys:
  - Generación de opciones y desafío
  - Almacenamiento de estado de registro
  - Verificación y almacenamiento de credenciales
- Implementación del flujo completo de autenticación con Passkeys:
  - Generación de opciones y desafío
  - Verificación de aserción
  - Creación de sesión
- Implementación del Factory Method para diferentes proveedores
- Adaptadores para Microsoft (estructura, sin autenticación real)
- Implementación del Strategy Pattern para credenciales
- Verificación y validación de credenciales
- Gestión de errores específicos para WebAuthn
- Integración con el sistema de métricas y trazas

**Entregable:** Sistema completo de autenticación con Passkeys.

### Sesión 11: Autenticación Tradicional y Gestión de Sesiones
**Alcance detallado:**
- Implementación completa del sistema de autenticación con contraseña:
  - Servicio de hashing con Argon2id
  - Validación segura de contraseñas
  - Rotación de algoritmos de hash
  - Políticas configurables de contraseñas
  - Restablecimiento de contraseñas
- Sistema completo de limitación de intentos y bloqueo temporal
- Implementación de la gestión de sesiones:
  - Generación de JWT con firma
  - Almacenamiento de sesiones activas en Redis
  - Revocación global de tokens
  - Rotación de claves de firma
  - Refresh tokens
  - Control de expiración
- Implementación del sistema RBAC para autorización:
  - Verificación de permisos por acción y recurso
  - Caché optimizada de permisos
  - Comprobaciones jerárquicas de roles
  - Políticas de autorización

**Entregable:** Sistema completo de autenticación tradicional, sesiones y RBAC.

### Sesión 12: Motor de Ejecución de Reportes (Utilizando Tablas Lógicas)
**Alcance detallado:**
- Implementación completa del motor de ejecución de reportes utilizando las tablas lógicas EAV:
  - Procesamiento de SQL con parámetros
  - Validación de seguridad para consultas dinámicas
  - Sustitución de parámetros en queries
  - Validación de parámetros de entrada
  - Transformación de resultados según contrato
  - Aplicación de formato y post-procesamiento
- Sistema de paginación para resultados grandes
- Caché de resultados de reportes con invalidación
- Implementación de seguridad para ejecución de reportes:
  - Validación de permisos
  - Limitación de recursos
  - Protección contra inyección SQL
- Implementación del sistema de auditoría para ejecución de reportes, incluyendo un middleware de auditoría para capturar eventos de ejecución a nivel de API si es aplicable.
- Soporte para diferentes formatos de salida

**Entregable:** Motor completo de ejecución de reportes dinámicos, implementado sobre las tablas lógicas EAV.

### Sesión 13: Motor de Migración de Datos (Utilizando Tablas Lógicas)
**Alcance detallado:**
- Implementación completa del motor de migración utilizando las tablas lógicas EAV:
  - Worker pool para procesamiento paralelo
  - Control de progreso en tiempo real
  - Gestión de errores y recuperación
  - Procesador de eventos batch
  - Transformadores de datos para diferentes tipos
- Sistema de validación de datos durante migración
- Implementación de transformaciones:
  - Conversión de tipos
  - Formateo de datos
  - Aplicación de reglas de negocio
  - Validaciones personalizadas
- Backoff exponencial para reintentos
- Estrategias para manejo de errores:
  - Continuar con errores
  - Detener en error
  - Umbrales configurables
- Sistema de reportes de progreso en tiempo real

**Entregable:** Motor completo de migración de datos, implementado sobre las tablas lógicas EAV.

### Sesión 14: API HTTP Base y Middleware
**Alcance detallado:**
- Configuración del servidor HTTP con chi router
- Implementación de funcionalidades de API Gateway básicas (enrutamiento, composición simple si es necesaria) utilizando chi router y middleware.
- Implementación del middleware base:
  - Logger middleware para registro estructurado
  - Recuperación de pánicos
  - Trazas y correlación de solicitudes
  - Métricas de solicitudes HTTP
  - Timeouts configurables
- Middleware de seguridad:
  - Autenticación JWT
  - Control de sesiones
  - Rate limiting para protección contra ataques
  - Protección CSRF
  - Cabeceras de seguridad HTTP
  - Validación de contenido
- Middleware de autorización RBAC
- Configuración de CORS
- Gestión centralizada de errores HTTP
- Base para documentación OpenAPI
- Utilidades para serialización/deserialización JSON
- Configurar la base para documentación de API con Swagger/OpenAPI
- Implementar estructura inicial para anotaciones Swagger en código

**Entregable:** Configuración base de API HTTP con middleware completo.

### Sesión 15: API HTTP para EAV y Autenticación
**Alcance detallado:**
- Handlers para EAV:
  - LogicalTableHandler (CRUD)
  - ColumnDefinitionHandler (CRUD)
  - TableRowHandler (CRUD completo con soporte JSONB)
- Handlers para autenticación:
  - PasskeyRegistrationHandler
  - PasskeyAuthenticationHandler
  - PasswordAuthHandler
  - SessionHandler
- Handlers para RBAC:
  - RoleHandler
  - PermissionHandler
  - UserRoleHandler
- Validación de inputs con validator/v10
- Implementación de ruteo con chi router
- Versionado de API
- Manejo de respuestas HTTP
- Transformación de errores de dominio a HTTP
- Todos los handlers deben incluir anotaciones Swagger completas siguiendo el estándar swaggo/swag
- Documentar parámetros, respuestas, códigos de error y ejemplos para cada endpoint

**Entregable:** API HTTP para EAV y autenticación completa.

### Sesión 16: API HTTP para Reportes y Migración (Como Tablas Lógicas)
**Alcance detallado:**
- Handlers para el sistema de reportes (utilizando tablas lógicas EAV):
  - ReportDefinitionHandler (CRUD sobre la tabla lógica `REPORT_DEFINITIONS`)
  - ReportParameterHandler (CRUD sobre la tabla lógica `REPORT_PARAMETERS`)
  - ReportQueryHandler (CRUD sobre la tabla lógica `REPORT_QUERIES`)
  - ReportExecutionHandler (Ejecución de reportes)
  - ReportPermissionHandler (CRUD sobre la tabla lógica `REPORT_PERMISSIONS`)
- Handlers para el sistema de migración (utilizando tablas lógicas EAV):
  - MigrationConfigurationHandler (CRUD sobre la tabla lógica `MIGRATION_CONFIGURATIONS`)
  - MigrationFieldMappingHandler (CRUD sobre la tabla lógica `MIGRATION_FIELD_MAPPINGS`)
  - MigrationJobHandler (CRUD sobre la tabla lógica `MIGRATION_JOBS`)
  - MigrationStatusHandler (Control y monitoreo de trabajos de migración)
  - MigrationErrorHandler (Gestión de errores de migración)
- Endpoints para estadísticas y monitoreo
- Soporte para streaming de resultados grandes
- Endpoints para descargas de datos
- Handlers para uploads de archivos de migración
- Validación avanzada de inputs
- Tests de integración para endpoints
- Continuar con las anotaciones Swagger completas para todos los endpoints
- Mantener consistencia con el estilo de documentación establecido en la sesión anterior


**Entregable:** API HTTP para reportes y migración completa, operando sobre las tablas lógicas EAV.

### Sesión 17: Integración y Refinamiento de Documentación API
** Alcance detallado:**

- Configuración completa del generador swaggo/swag:
	- Instalación y configuración óptima del generador
	- Integración con el servidor HTTP existente
	- Configuración de la generación automatizada

- Implementación de la UI personalizada de Swagger para exploración interactiva:
	- Interfaz customizada con tema del proyecto
	- Agrupación lógica de endpoints
	- Funcionalidad Try-it-out completamente funcional
	- Autenticación en la UI para endpoints protegidos

- Validación y refinamiento de la documentación existente:
	- Revisión de coherencia entre endpoints
	- Verificación de completitud en todos los parámetros
	- Mejora de ejemplos y descripciones
	- Corrección de inconsistencias

- Generación de especificaciones OpenAPI finales:
	- Exportación de documentación en formatos JSON y YAML
	- Creación de versiones estáticas para distribución


- Integración de CI/CD para documentación:
	- Scripts de generación y validación automatizada
	- Comprobación de errores en la documentación
	- Publicación automática de nuevas versiones


- Creación de endpoints para acceder a diferentes versiones de documentación
- Pruebas automatizadas para verificar la integridad de la documentación:
	- Validación de estructura OpenAPI
	- Comprobación de consistencia con implementación real
	- Verificación de enlaces y referencias internas


**Entregable:** Sistema completo de documentación API integrado, validado y refinado con interfaz de exploración interactiva.

### Sesión 18: Observabilidad y Monitoreo
**Alcance detallado:**
- Configuración avanzada de logging estructurado con Zap:
  - Logging contextual
  - Rotación de logs
  - Niveles de log configurables
  - Formateo para entornos de desarrollo y producción
  - Integración con sistemas de log centralizados
- Implementación completa de métricas con Prometheus:
  - Métricas HTTP (latencia, tasa de error, solicitudes)
  - Métricas de bases de datos (queries, conexiones)
  - Métricas de caché (hit ratio, evictions)
  - Métricas de negocio (operaciones EAV, autenticaciones)
  - Métricas personalizadas para el modelo EAV
  - Métricas para circuit breaker y bulkhead
- Configuración de OpenTelemetry para trazas distribuidas:
  - Propagación de contexto entre servicios
  - Sampling configurable
  - Integración con backends de observabilidad
  - Correlación entre logs, métricas y trazas
- Health checks y readiness probes:
  - Verificación de conexiones de bases de datos
  - Verificación de servicios dependientes
  - Estado del sistema
  - Endpoint de health avanzado
- Dashboards Grafana para visualización
- Configuración de alertas

**Entregable:** Sistema completo de observabilidad y monitoreo.

### Sesión 19: Testing - Pruebas Unitarias y de Integración
**Alcance detallado:**
- Configuración del framework de testing
- Implementación de mocks para todas las interfaces:
  - Repositorios mock
  - Servicios mock
  - Elementos de infraestructura mock
- Pruebas unitarias para todas las entidades de dominio:
  - Entidades EAV (LogicalTable, ColumnDefinition, TableRow, ColumnValue)
  - Entidades de autenticación (User, Passkey, Session)
  - Entidades RBAC (Role, Permission)
  - DomainEvent para Outbox
- Pruebas unitarias para todos los comandos
- Pruebas unitarias para todas las queries
- Pruebas unitarias para servicios de aplicación
- Pruebas unitarias para validadores
- Configuración de testcontainers para pruebas de integración:
  - Contenedor PostgreSQL
  - Contenedor Redis
- Pruebas de integración para repositorios
- Pruebas de integración para servicios que requieren BD
- Configuración de cobertura de código
- Pruebas para flujos de autenticación
- Herramientas para generación de datos de prueba

**Entregable:** Suite completa de pruebas unitarias y de integración.

### Sesión 20: Testing - E2E y Rendimiento
**Alcance detallado:**
- Configuración de entorno para pruebas E2E
- Pruebas E2E para flujos principales:
  - Registro y autenticación completa
  - Gestión de modelo EAV
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
- Verificación de Memory leaks

**Entregable:** Suite completa de pruebas E2E y de rendimiento.

### Sesión 21: Containerización y Configuración Local
**Alcance detallado:**
- Dockerfile optimizado para servicios con multi-stage building:
  - Etapa de compilación
  - Etapa de ejecución mínima
  - Optimizaciones de tamaño y seguridad
  - Validaciones de seguridad
- Docker Compose para entorno de desarrollo completo:
  - Servicio API
  - PostgreSQL
  - Redis
  - Prometheus
  - Grafana
  - Jaeger para trazas
  - Loki para logs
- Scripts de inicialización para bases de datos:
  - Esquema inicial
  - Datos de prueba
  - Migraciones
- Configuración de variables de entorno
- Gestión de secretos para desarrollo
- Scripts de conveniencia para desarrollo:
  - Comandos make
  - Scripts de inicio rápido
  - Scripts de reinicio
- Documentación completa para desarrollo local
- Prueba de integración del entorno completo

**Entregable:** Configuración completa de containerización y entorno local.

### Sesión 22: Despliegue Kubernetes
**Alcance detallado:**
- Manifiestos Kubernetes completos:
  - Deployments para todos los servicios
  - StatefulSets para bases de datos
  - ConfigMaps para configuración
  - Secrets para información sensible
  - Services para comunicación interna
  - Ingress para exposición externa
- Helm charts para despliegue simplificado:
  - Plantillas para todos los recursos
  - Valores por defecto
  - Opciones de configuración
- Configuración de autoscaling horizontal (HPA)
- Implementación de sondas de readiness y liveness
- Estrategias de rolling update
- Configuración de recursos (CPU/memoria)
- Configuración de PersistentVolumes para datos
- Configuración de NetworkPolicies para seguridad
- Scripts de despliegue
- Pruebas de verificación post-despliegue

**Entregable:** Configuración completa para despliegue en Kubernetes.

### Sesión 23: CI/CD y Automatización
**Alcance detallado:**
- Configuración de GitHub Actions para CI/CD:
  - Build y test automatizados
  - Análisis estático de código
  - Verificación de seguridad
  - Construcción y publicación de imágenes Docker
  - Despliegue automatizado en entornos
- Configuración de ArgoCD para GitOps:
  - Configuración de aplicaciones
  - Sincronización automática
  - Políticas de rollback
- Scripts de migración de base de datos para despliegues
- Estrategias de canary deployment y blue/green
- Configuración de monitoreo post-despliegue
- Automatización de rollbacks
- Pipelines separados para diferentes entornos
- Gestión de secretos en CI/CD
- Integración con un sistema de gestión de secretos centralizado (como HashiCorp Vault) para el pipeline de CI/CD y, opcionalmente, para la inyección de secretos en Kubernetes si se decide utilizarlo en lugar de Kubernetes Secrets nativos.
- Verificaciones de seguridad automatizadas

**Entregable:** Pipeline completo de CI/CD y automatización de despliegue.

## Resumen del Plan

Este plan revisado cubre exhaustivamente todos los aspectos del proyecto EAV Híbrido con Passkeys, asegurando que:

1. **Modelo EAV Híbrido como base**: Implementa todas las tablas físicas (`logical_tables`, `column_definitions`, `table_rows`, etc.) y usa este modelo para todos los subsistemas
2. **Integridad Referencial Completa**: Implementa tablas y lógica para restricciones (`table_constraints`, `constraint_columns`, etc.) permitiendo llaves únicas, compuestas y foráneas
3. **Autenticación y RBAC completos**: Implementa todas las tablas para usuarios, passkeys, roles y permisos (`app_users`, `user_passkeys`, `roles`, etc.)
4. **Patrones robustos**: Implementa Circuit Breaker, Bulkhead, Outbox, Unit of Work
5. **CQRS completo**: Mediator, comandos y queries para todas las funcionalidades
6. **Autenticación exhaustiva**: Passkeys y tradicional con todas las medidas de seguridad
7. **Sistemas avanzados**: Reportes dinámicos y migración de datos implementados como tablas lógicas dentro del modelo EAV
8. **API HTTP completa**: Con seguridad, middleware y documentación exhaustiva
9. **Documentación Swagger/OpenAPI**: Completa e interactiva para toda la API
10. **Observabilidad total**: Logs estructurados, métricas, trazas y health checks
11. **Testing completo**: Unitario, integración, E2E y rendimiento
12. **Despliegue completo**: Containerización, Kubernetes y CI/CD

Cada sesión está diseñada para producir entregables completos y verificables que sirven como base para las siguientes sesiones, garantizando un avance constante, coherente y acumulativo hacia la implementación total del sistema.
