# Arquitectura EAV Híbrida con Passkeys: Diseño e Implementación en Go

**Versión**: 3.0  
**Fecha**: Mayo 2025  
**Autor**: Departamento de Arquitectura con revisión por Junta de Arquitectos y Casa de Software

## Índice

1. [Introducción](#1-introducción)
2. [Visión General de la Arquitectura](#2-visión-general-de-la-arquitectura)
3. [Modelo de Datos EAV Híbrido](#3-modelo-de-datos-eav-híbrido)
4. [Stack Tecnológico](#4-stack-tecnológico)
5. [Arquitectura de Microservicios](#5-arquitectura-de-microservicios)
6. [Implementación del Patrón CQRS](#6-implementación-del-patrón-cqrs)
7. [Sistema de Autenticación con Passkeys](#7-sistema-de-autenticación-con-passkeys)
8. [Gestión de Transacciones y Consistencia](#8-gestión-de-transacciones-y-consistencia)
9. [Patrones de Diseño Implementados](#9-patrones-de-diseño-implementados)
10. [API y Contratos](#10-api-y-contratos)
11. [Escalabilidad y Rendimiento](#11-escalabilidad-y-rendimiento)
12. [Seguridad](#12-seguridad)
13. [Observabilidad y Monitoreo](#13-observabilidad-y-monitoreo)
14. [Estrategia de Pruebas](#14-estrategia-de-pruebas)
	15. [Implementación DevOps](#15-implementación-devops)
	16. [Riesgos y Mitigaciones](#16-riesgos-y-mitigaciones)
	17. [Conclusión](#17-conclusión)
18. [Sistema de Queries Dinámicos](#18-sistema-de-queries-dinámicos)
19. [Sistema de Migración de Datos](#19-sistema-de-migración-de-datos)
20. [Apéndices](#20-apéndices)

## 1. Introducción

### 1.1 Propósito del Documento

Este documento detalla la arquitectura completa para la implementación de un sistema EAV (Entity-Attribute-Value) Híbrido con capacidades de autenticación avanzada mediante Passkeys, utilizando Microsoft como proveedor principal. La arquitectura está diseñada siguiendo los principios de Clean Architecture, SOLID, Domain-Driven Design (DDD) y patrones de microservicios para garantizar flexibilidad, mantenibilidad y rendimiento.

Esta versión 3.0 incorpora recomendaciones críticas de la Casa de Software y la Junta de Arquitectos, incluyendo patrones adicionales para garantizar resiliencia, observabilidad y gestión avanzada del estado del sistema.

### 1.2 Objetivos del Sistema

1. Implementar un modelo de datos EAV Híbrido que balancee la flexibilidad y el rendimiento
2. Proporcionar un sistema de autenticación flexible mediante Passkeys y métodos tradicionales basados en contraseña
3. Garantizar la escalabilidad horizontal y el alto rendimiento del sistema
4. Mantener la integridad transaccional y la consistencia de datos
5. Facilitar la extensibilidad para adaptarse a requisitos futuros
6. Asegurar observabilidad completa mediante implementación avanzada de métricas, trazas y logs
7. Proporcionar resiliencia a fallos con patrones adecuados para sistemas distribuidos
8. Facilitar despliegues confiables en entornos de Kubernetes

### 1.3 Consideraciones Clave

La arquitectura ha sido diseñada considerando cuidadosamente las debilidades conocidas del patrón EAV tradicional e implementando optimizaciones sustanciales para mitigarlas. Este enfoque "EAV Híbrido" conserva las ventajas de flexibilidad mientras evita muchos de los problemas de rendimiento y complejidad asociados con implementaciones EAV puras.

Se ha prestado especial atención a la integridad de datos, la resiliencia del sistema, y la facilidad de operación y mantenimiento.

Se ha puesto especial énfasis en la seguridad de la gestión de contraseñas, utilizando técnicas modernas de hashing y protección contra ataques comunes

## 2. Visión General de la Arquitectura

### 2.1 Diagrama de Arquitectura de Alto Nivel

```
┌───────────────────────────────────────────────────────────────────┐
│                        Cliente (Frontend)                         │
└───────────────────────┬──────────────────────────────────────────z┘
                            │
┌───────────────────────────▼───────────────────────────────────────┐
│                           API Gateway                             │
└┬──────────────────────────┬──────────────────────────────────────n┘
 │                          │                                       │
┌▼──────────────────┐  ┌────▼─────────────┐  ┌────────────────────┐ │
│ Auth Service      │  │ Metadata Service │  │ Data Service       │ │
│ ┌───────────────┐ │  │ ┌─────────────┐  │  │ ┌────────────────┐ │ │
│ │ Passkey       │ │  │ │ Schema Mgmt │  │  │ │ Command API    │ │ │
│ │ Management    │ │  │ └─────────────┘  │  │ └────────────────┘ │ │
│ └───────────────┘ │  │ ┌─────────────┐  │  │ ┌────────────────┐ │ │
│ ┌───────────────┐ │  │ │ Table Def   │  │  │ │ Query API      │ │ │
│ │ Auth Provider │ │  │ └─────────────┘  │  │ └────────────────┘ │ │
│ │ Adaptors      │ │  │                  │  │                    │ │
│ └───────────────┘ │  └──────────────────┘  └────────────────────┘ │
└───────────────────┘                                               │
                                                                    │
┌──────────────────────────────────────────────────────────────────▼┐
│                    Infrastructure Services                        │
│  ┌─────────────┐  ┌───────────────┐  ┌──────────────┐  ┌─────────┐│
│  │ PostgreSQL  │  │ Redis Cache   │  │ Observability│  │ Secrets ││
│  └─────────────┘  └───────────────┘  └──────────────┘  └─────────┘│
└───────────────────────────────────────────────────────────────────┘
```

### 2.2 Principios Arquitectónicos

1. **Separación de Responsabilidades**: Cada microservicio tiene un único propósito y conjunto de responsabilidades bien definido.
2. **Independencia de Servicios**: Los servicios pueden evolucionar de forma independiente sin afectar a otros.
3. **Diseño Orientado al Dominio**: Las entidades y agregados del dominio guían la estructura del código.
4. **Arquitectura Hexagonal**: Separación clara entre dominio, aplicación e infraestructura.
5. **API First**: Las interfaces de API son contratos primarios y están versionadas.
6. **Infraestructura como Código**: Toda la infraestructura se define como código para garantizar la reproducibilidad.
7. **Observabilidad por Diseño**: La telemetría, logging y métricas se integran desde el principio.
8. **Defensa en Profundidad**: Se aplican múltiples capas de protecciones de seguridad.
9. **Resiliencia por Diseño**: Se implementan patrones para garantizar que el sistema pueda recuperarse de fallos parciales.
10. **Equilibrio entre Flexibilidad y Rendimiento**: Las decisiones de diseño balancean la necesidad de flexibilidad con requisitos de rendimiento.

## 3. Modelo de Datos EAV Híbrido

### 3.1 Fundamentos del Modelo EAV Híbrido

El EAV Híbrido que implementamos difiere significativamente del patrón EAV tradicional, abordando sus principales debilidades mientras conserva la flexibilidad que hace valioso este enfoque en ciertos contextos.

El enfoque EAV tradicional ha sido ampliamente criticado por problemas de rendimiento y complejidad. Sin embargo, nuestra implementación híbrida adopta una postura pragmática: utiliza EAV solo donde es realmente beneficioso y emplea enfoques relacionales convencionales para el resto.

#### 3.1.1 Diferencias con EAV Tradicional

| Aspecto | EAV Tradicional | Nuestro EAV Híbrido |
|---------|-----------------|---------------------|
| Estructura | Una tabla genérica con 3 columnas (entidad, atributo, valor) | Campos principales "promovidos" a columnas físicas + columnas EAV solo para atributos flexibles |
| Tipado | Tipado débil o conversiones necesarias | Tipado fuerte con columnas específicas por tipo de datos |
| Rendimiento de consultas | Pobre, requiere múltiples joins | Optimizado, consultas directas para atributos comunes |
| Indexación | Compleja y poco eficiente | Indexación directa de columnas principales + índices específicos para atributos EAV frecuentes |
| Validación | Difícil de implementar a nivel de BD | Esquema de validación definido en metadatos con aplicación en servicios |

### 3.2 Tablas Principales del Esquema

#### 3.2.1 Tablas Relacionales Convencionales

```
- logical_tables
  - id (UUID)
  - table_name (VARCHAR) [UNIQUE]
  - description (TEXT)
  - is_hierarchical (BOOLEAN)
  - cache_policy (VARCHAR)
  - version (INT) [Para control de versiones de esquema]
  - [campos de auditoría]

- column_definitions
  - id (UUID)
  - column_name (VARCHAR) [UNIQUE]
  - data_type (VARCHAR)
  - default_value (TEXT)
  - masking_rule (VARCHAR)
  - description (TEXT)
  - version (INT) [Para control de versiones de schema]
  - [campos de auditoría]

- table_column_properties
  - logical_table_id (UUID) [FK]
  - column_definition_id (UUID) [FK]
  - display_order (INT)
  - is_mandatory (BOOLEAN)
  - is_value_unique (BOOLEAN)
  - validation_rule_type (VARCHAR)
  - validation_parameters (JSONB)
  - [otros campos de configuración]
  - [campos de auditoría]
```

#### 3.2.2 Tablas Centrales EAV Híbridas

```
- table_rows
  - id (UUID)
  - logical_table_id (UUID) [FK]
  - code (VARCHAR)
  - display_value (VARCHAR)
  - sort_order (INT)
  - is_active (BOOLEAN)
  - parent_row_id (UUID) [FK, NULL]
  - extended_properties (JSONB) [NUEVO]
  - version (INT) [Para control de concurrencia optimista]
  - [campos de auditoría]

- column_values
  - id (UUID)
  - row_id (UUID) [FK]
  - column_definition_id (UUID) [FK]
  - string_value (TEXT)
  - int_value (BIGINT)
  - decimal_value (DECIMAL)
  - boolean_value (BOOLEAN)
  - date_value (TIMESTAMP)
  - version (INT) [Para control de concurrencia optimista]
  - [campos de auditoría]
```

#### 3.2.3 Tablas de Autenticación y Passkeys

```
- app_users
  - id (UUID)
  - username (VARCHAR) [UNIQUE]
  - email (VARCHAR) [UNIQUE]
  - password_hash (VARCHAR, 255)  [NUEVO: Hash de la contraseña]
  - password_salt (VARCHAR, opcional, dependiendo del algoritmo de hashing)
  - password_last_changed_at (TIMESTAMP, NULL) [NUEVO: Fecha último cambio]
  - failed_login_attempts (INT, DEFAULT 0) [NUEVO: Para bloqueo temporal]
  - locked_until (TIMESTAMP, NULL) [NUEVO: Para bloqueo temporal]
  - status (VARCHAR)
  - is_active (BOOLEAN)
  - user_profile_row_id (UUID) [FK, NULL]
  - [campos de auditoría]

- user_passkeys
  - id (UUID)
  - user_id (UUID) [FK]
  - credential_id (VARCHAR) [UNIQUE]
  - public_key (TEXT)
  - attestation_type (VARCHAR)
  - provider (VARCHAR)
  - device_info (JSONB)
  - aaguid (UUID)
  - last_used_at (TIMESTAMP)
  - is_active (BOOLEAN)
  - [campos de auditoría]
```

#### 3.2.4 Tabla de Eventos de Dominio para Patrón Outbox

```
- domain_events
  - id (UUID)
  - event_type (VARCHAR)
  - aggregate_id (UUID)
  - aggregate_type (VARCHAR)
  - event_data (JSONB)
  - created_at (TIMESTAMP)
  - processed_at (TIMESTAMP, NULL)
  - processing_attempts (INT)
  - version (INT)
```

##$ 3.2.5 Tablas para RBAC (Role-Based Access Control)
```
- roles
  - id (UUID)
  - name (VARCHAR) [UNIQUE]
  - description (TEXT)
  - is_system_role (BOOLEAN)
  - is_active (BOOLEAN)
  - version (INT)
  - [campos de auditoría]

- permissions
  - id (UUID)
  - permission_name (VARCHAR) [UNIQUE]
  - resource_type (VARCHAR)
  - action_type (VARCHAR)
  - description (TEXT)
  - is_system_permission (BOOLEAN)
  - version (INT)
  - [campos de auditoría]

- user_roles
  - id (UUID)
  - user_id (UUID) [FK → app_users.id]
  - role_id (UUID) [FK → roles.id]
  - is_active (BOOLEAN)
  - expires_at (TIMESTAMP, NULL)
  - [campos de auditoría]

- role_permissions
  - id (UUID)
  - role_id (UUID) [FK → roles.id]
  - permission_id (UUID) [FK → permissions.id]
  - [campos de auditoría]
```
### 3.2.6 Tablas para Integridad Referencial
```
- table_constraints
  - id (UUID)
  - logical_table_id (UUID) [FK → logical_tables.id]
  - constraint_name (VARCHAR)
  - constraint_type (VARCHAR) [CHECK ('PRIMARY KEY', 'UNIQUE', 'FOREIGN KEY', 'CHECK')]
  - is_active (BOOLEAN)
  - version (INT)
  - [campos de auditoría]

- constraint_columns
  - id (UUID)
  - constraint_id (UUID) [FK → table_constraints.id]
  - column_definition_id (UUID) [FK → column_definitions.id]
  - ordinal_position (INT)
  - [campos de auditoría]

- unique_indexes
  - id (UUID)
  - constraint_id (UUID) [FK → table_constraints.id]
  - is_case_sensitive (BOOLEAN)
  - index_type (VARCHAR)
  - [campos de auditoría]

- unique_index_json_properties
  - id (UUID)
  - unique_index_id (UUID) [FK → unique_indexes.id]
  - property_path (VARCHAR)
  - is_case_sensitive (BOOLEAN)
  - ordinal_position (INT)
  - [campos de auditoría]

- foreign_key_references
  - id (UUID)
  - constraint_id (UUID) [FK → table_constraints.id]
  - referenced_table_id (UUID) [FK → logical_tables.id]
  - on_update_action (VARCHAR) [CHECK ('CASCADE', 'RESTRICT', 'SET NULL', 'SET DEFAULT', 'NO ACTION')]
  - on_delete_action (VARCHAR) [CHECK ('CASCADE', 'RESTRICT', 'SET NULL', 'SET DEFAULT', 'NO ACTION')]
  - match_type (VARCHAR) [CHECK ('FULL', 'PARTIAL', 'SIMPLE')]
  - [campos de auditoría]

- check_constraints
  - id (UUID)
  - constraint_id (UUID) [FK → table_constraints.id]
  - check_expression (TEXT)
  - error_message (TEXT)
  - [campos de auditoría]
```

### 3.3 Optimizaciones del Modelo

1. **Campos Promovidos**: Los atributos más comunes y consultados se almacenan como columnas físicas en `table_rows`.
2. **Propiedades Extendidas en JSONB**: Propiedades poco consultadas, específicas de ciertos tipos de entidades o con estructura variable se almacenan en `extended_properties` (JSONB).
3. **Tipado por Columna**: Valores de diferentes tipos se almacenan en columnas específicas, evitando conversiones.
4. **Indexación Estratégica**: Índices compuestos optimizados para patrones de consulta comunes.
5. **Indexación JSONB**: Índices GIN para consulta eficiente de propiedades extensibles.
6. **Vistas Materializadas**: Para agregaciones y consultas frecuentes sobre datos EAV.
7. **Caché Multinivel**: Implementación de caché para entidades reconstruidas con estrategias de invalidación.
8. **Particionamiento**: Para tablas grandes como `column_values` basado en `logical_table_id`.
9. **Histogramas de Estadísticas**: Mantenimiento de estadísticas avanzadas para el optimizador de consultas de PostgreSQL.

### 3.4 Consideraciones de Integridad de Datos

1. **Validación Programática**: Reglas de validación definidas en metadatos y aplicadas en servicios.
2. **Constraints a Nivel de Base de Datos**: Para relaciones y validaciones básicas.
3. **Transacciones Atómicas**: Para mantener la coherencia en operaciones multi-tabla.
4. **Control de Versiones Optimista**: Para gestionar actualizaciones concurrentes.
5. **Eventos de Dominio**: Para operaciones que requieren consistencia eventual.

## 4. Stack Tecnológico

### 4.1 Lenguaje y Runtime

- **Go**: v1.22+ como lenguaje principal de desarrollo
- **Compilación**: Compilación estática con optimizaciones para reducir uso de recursos

### 4.2 Frameworks y Bibliotecas Core

- **API/Web Framework**: 
  - Go-Kit v0.15+ (orquestación de servicios)
  - Chi Router v5.1.1 (HTTP routing)
  - Validator: v10.25.0
		- **Validación**: github.com/go-playground/validator/v10 v10.25.0 (o la última patch v10.x)
		- **Nota de Configuración Importante:** Se utilizará la opción `WithRequiredStructEnabled()` al instanciar el validador para un manejo más robusto de la validación de structs anidados y campos nil.
  - GORM: v2.0
  - OpenTelemetry: v1.23.1
  - Zap: v1.27.0

- **Acceso a Datos**: 
  - SQLX v1.4.0 para acceso SQL directo 
  - PGX v5.5+ para funcionalidades avanzadas de PostgreSQL

- **Caché**: 
  - Redis v7.2+ cliente go-redis/v9
  - Implementación de caché local en memoria con dgraph-io/ristretto/v2 v2.2.0

### 4.3 Infraestructura

- **Base de Datos**: PostgreSQL 16+ (con soporte para JSONB, particionamiento)
- **Servicios de Caché**: Redis 7.2+
- **Observabilidad**: 
  - Prometheus v2.45+ para métricas
  - OpenTelemetry v1.28.0  para trazas distribuidas
  - Loki v2.9+ para logs
  - Grafana v10.2+ para dashboards
  - swaggo/http-swagger v1.3.4
  - swaggo/swag v1.16.2
- **Orquestación**: Kubernetes v1.28+

### 4.4 Bibliotecas Específicas para Passkeys

- **WebAuthn**: github.com/go-webauthn/webauthn v1.0+  (versión más reciente a mayo 2025)
- **Integración Microsoft**: ms-identity-go v1.0+ y extensiones personalizadas

### 4.5 Bibliotecas para Patrones Adicionales

- **Circuit Breaker**: github.com/sony/gobreaker v2.0.0
- **Rate Limiting**: golang.org/x/time/rate v0.5.0
- **Feature Flags**: github.com/open-feature/go-sdk v1.14.1 
- **Bulkhead**: Implementación personalizada para aislamiento de recursos
- **Validación**: github.com/go-playground/validator/v10 v10.17+
- **Logging Estructurado**: go.uber.org/zap v1.26+

## 5. Arquitectura de Microservicios

### 5.1 Servicios Principales

#### 5.1.1 Auth Service

Responsable de la autenticación, autorización y gestión de usuarios.

**Componentes**:
- **Passkey Manager**: Gestión del ciclo de vida de passkeys
- **Provider Adapters**: Adaptadores para diferentes proveedores (Microsoft, futuras integraciones)
- **Session Manager**: Gestión de sesiones de usuario
- **RBAC Manager**: Control de acceso basado en roles
- **Password Validator/Hasher". Gestión segura del hashing y verificación de contraseñas.
- **Login Flow Manager (Password)".

**APIs**:
- `/auth/passkey/register`: Inicio y finalización de registro de passkey
- `/auth/passkey/authenticate`: Autenticación mediante passkey
- `/auth/session`: Gestión de sesiones
- `/auth/users`: Gestión de usuarios
- `/auth/login`: Inicio y finalización de autenticación con usuario/contraseña.


**Patrones Aplicados**:
- Circuit Breaker para llamadas a servicios externos
- Rate Limiting para prevenir ataques de fuerza bruta
- Bulkhead para aislar recursos críticos
- Retry con backoff exponencial para operaciones transitorias

#### 5.1.2 Metadata Service

Responsable de la definición y gestión del esquema EAV.

**Componentes**:
- **Schema Manager**: Gestión de esquemas y tablas lógicas
- **Definition Manager**: Gestión de definiciones de columnas
- **Validation Manager**: Gestión de reglas de validación
- **Version Manager**: Control de versiones de esquema

**APIs**:
- `/metadata/tables`: Gestión de tablas lógicas
- `/metadata/columns`: Gestión de definiciones de columnas
- `/metadata/properties`: Gestión de propiedades de columnas en tablas
- `/metadata/schema`: Operaciones sobre el esquema completo

**Patrones Aplicados**:
- Cache Aside para almacenamiento de definiciones de esquema
- Versionado de Esquema para control de cambios
- Circuit Breaker para operaciones de base de datos
- Publisher-Subscriber para notificaciones de cambios de esquema

#### 5.1.3 Data Service

Responsable de las operaciones CRUD sobre los datos EAV.

**Componentes**:
- **Command API**: Operaciones de escritura (Create, Update, Delete)
- **Query API**: Operaciones de lectura optimizadas
- **Transaction Manager**: Gestión de transacciones
- **Cache Manager**: Gestión de caché de datos

**APIs**:
- `/data/{table_name}`: Operaciones CRUD estándar
- `/data/{table_name}/batch`: Operaciones por lotes
- `/data/{table_name}/query`: Consultas avanzadas

**Patrones Aplicados**:
- CQRS para separación de operaciones de lectura/escritura
- Unit of Work para integridad transaccional
- Circuit Breaker para operaciones de base de datos
- Bulkhead para aislamiento de recursos
- Outbox para publicación confiable de eventos

### 5.2 Servicios de Infraestructura

- **API Gateway**: Enrutamiento, autenticación, rate limiting
- **Observability Service**: Agregación de logs, métricas y trazas
- **Configuration Service**: Gestión centralizada de configuración
- **Feature Flag Service**: Gestión centralizada de Features Flags

### 5.3 Comunicación entre Servicios

- **Sincrónica**: REST/HTTP para comunicación directa con Circuit Breaker
- **Asincrónica**: Mensajería basada en eventos para operaciones no bloqueantes

### 5.4 Patrones de Despliegue

- **Containerización**: Docker para empaquetado
- **Orquestación**: Kubernetes para gestión de contenedores
- **Alta disponibilidad**: StatefulSets para PostgreSQL con replicación
- **Estrategia de despliegue**: Canary/Blue-Green para actualizaciones sin tiempo de inactividad

## 6. Implementación del Patrón CQRS

	6.1 Clarificación del Modelo CQRS con PostgreSQL y Redis

	En esta implementación, PostgreSQL funciona como almacenamiento principal para ambos modelos (comandos y consultas), manteniendo la fuente única de verdad para todos los datos. Redis se utiliza exclusivamente como capa de caché para optimizar las operaciones de lectura frecuentes, nunca como almacenamiento persistente.

	La arquitectura sigue este flujo:
	1. Las operaciones de escritura (comandos) se procesan directamente en PostgreSQL.
	2. Al completarse una escritura exitosa, se publica un evento mediante el patrón Outbox.
	3. Los procesadores de eventos reciben estas notificaciones y actualizan o invalidan las entradas correspondientes en la caché Redis.
	4. Las operaciones de lectura (consultas) verifican primero Redis y, en caso de fallo de caché, consultan PostgreSQL para luego actualizar la caché.

	Este enfoque garantiza que PostgreSQL mantenga la integridad y durabilidad de los datos, mientras Redis proporciona optimización de rendimiento para lecturas sin comprometer la consistencia. La estrategia de caché multinivel incluye tanto caché en memoria de la aplicación como Redis, con políticas de invalidación coordinadas basadas en eventos de dominio.

### 6.1 Separación de Modelos de Comando y Consulta

El sistema implementa CQRS (Command Query Responsibility Segregation) para separar las operaciones de lectura y escritura, optimizando cada una de forma independiente.

#### 6.1.1 Modelo de Comandos (Escritura)

```go
// Ejemplo de definición de comando con soporte para extended_properties
type CreateTableRowCommand struct {
    LogicalTableName string                 `json:"logical_table_name"`
    Code             string                 `json:"code"`
    DisplayValue     string                 `json:"display_value"`
    SortOrder        int                    `json:"sort_order"`
    IsActive         bool                   `json:"is_active"`
    ColumnValues     map[string]interface{} `json:"column_values"`
    ExtendedProps    map[string]interface{} `json:"extended_properties"` // Propiedades JSONB
}

// Handler de comando
func (h *CreateTableRowHandler) Handle(ctx context.Context, cmd CreateTableRowCommand) (TableRowDTO, error) {
    // Validación de comando
    if err := h.validator.Validate(cmd); err != nil {
        return TableRowDTO{}, err
    }
    
    // Inicio de span para trazabilidad
    ctx, span := h.tracer.Start(ctx, "CreateTableRowHandler.Handle")
    defer span.End()
    
    // Implementación con Unit of Work
    var result TableRowDTO
    err := h.uow.WithTransaction(ctx, func(ctx context.Context) error {
        // Lógica de creación para columnas y valores EAV...
        
        // Procesar extended_properties en JSONB
        jsonbProps := make(map[string]interface{})
        if len(cmd.ExtendedProps) > 0 {
            // Copiar propiedades extendidas al mapa JSONB
            for k, v := range cmd.ExtendedProps {
                jsonbProps[k] = v
            }
        }
        
        // Crear la entidad con propiedades en JSONB
        newRow := &TableRow{
            // Campos básicos...
            ExtendedProperties: jsonbProps,
        }
        
        // Guardar y publicar eventos...
        return nil
    })
    
    // Métricas y trazabilidad
    h.metrics.RecordCommandExecution("CreateTableRow", err == nil)
    
    return result, err
}
```

#### 6.1.2 Modelo de Consultas (Lectura)

Las consultas son optimizadas para rendimiento de lectura, utilizando modelos desnormalizados cuando sea necesario.

```go
// Ejemplo de definición de consulta
type GetTableRowsQuery struct {
    LogicalTableName string            `json:"logical_table_name"`
    Filters          []FilterCriteria  `json:"filters"`
    Sorting          []SortCriteria    `json:"sorting"`
    PageSize         int               `json:"page_size"`
    NextKeySet       string            `json:"next_key_set"`
}

// Handler de consulta
func (h *GetTableRowsHandler) Handle(ctx context.Context, query GetTableRowsQuery) (PagedResult[TableRowDTO], error) {
    // Validación de consulta
    if err := h.validator.Validate(query); err != nil {
        return PagedResult[TableRowDTO]{}, err
    }
    
    // Inicio de span para trazabilidad
    ctx, span := h.tracer.Start(ctx, "GetTableRowsHandler.Handle")
    defer span.End()
    
    // Intento obtener de caché
    cacheKey := h.buildCacheKey(query)
    var result PagedResult[TableRowDTO]
    if h.cacheManager.Get(cacheKey, &result) {
        span.SetAttribute("cache.hit", true)
        h.metrics.RecordCacheHit("TableRows")
        return result, nil
    }
    
    // Caché miss, obtener de base de datos
    span.SetAttribute("cache.hit", false)
    h.metrics.RecordCacheMiss("TableRows")
    
    // Implementación con protección de Circuit Breaker
    var err error
    result, err = h.breaker.Execute(func() (PagedResult[TableRowDTO], error) {
        // Consulta optimizada...
        return h.repository.QueryByFilters(ctx, query)
    })
	
    
    // Guardar en caché si fue exitoso
    if err == nil {
        h.cacheManager.Set(cacheKey, result, h.cacheTTL)
    }
    
    // Métricas y trazabilidad
    h.metrics.RecordQueryExecution("GetTableRows", err == nil)
    
    return result, err
}
```

### 6.2 Ejemplo de Implementación de Query Optimizada (Modificado)

```go
// Consulta SQL optimizada usando agregación JSON y JSONB
func (r *TableRowPostgresRepository) QueryByTableName(ctx context.Context, tableName string, filters []FilterCriteria, sorting []SortCriteria, pageSize int, keySet interface{}) ([]TableRow, interface{}, error) {
    // Inicio de span para trazabilidad
    ctx, span := r.tracer.Start(ctx, "TableRowRepository.QueryByTableName")
    defer span.End()
    
    // Métricas para el tamaño de consulta
    r.metrics.RecordQuerySize("TableRows", len(filters), len(sorting))
    
    // Construir consulta base
    const baseQuery = `
        SELECT 
            tr.id, 
            tr.code,
            tr.display_value,
            tr.sort_order,
            tr.is_active,
            tr.extended_properties,
            jsonb_object_agg(
                cd.column_name, 
                CASE 
                    WHEN cd.data_type = 'String' THEN to_jsonb(cv.string_value)
                    WHEN cd.data_type = 'Int' THEN to_jsonb(cv.int_value)
                    WHEN cd.data_type = 'Decimal' THEN to_jsonb(cv.decimal_value)
                    WHEN cd.data_type = 'Boolean' THEN to_jsonb(cv.boolean_value)
                    WHEN cd.data_type = 'Date' THEN to_jsonb(cv.date_value)
                    ELSE NULL
                END
            ) AS column_values_json
        FROM 
            table_rows tr
        JOIN 
            logical_tables lt ON tr.logical_table_id = lt.id
        LEFT JOIN 
            column_values cv ON tr.id = cv.row_id
        LEFT JOIN 
            column_definitions cd ON cv.column_definition_id = cd.id
        WHERE 
            lt.table_name = $1
            %s
        GROUP BY 
            tr.id, tr.code, tr.display_value, tr.sort_order, tr.is_active, tr.extended_properties
        %s
        LIMIT $2 %s
    `
    
    // Implementación de construcción dinámica de filtros, ordenación y paginación...
    
    // Ahora incluir filtros para propiedades JSONB
    jsonbFilters := make([]string, 0)
    
    for _, filter := range filters {
        // Verificar si es una propiedad extendida (en JSONB)
        if strings.HasPrefix(filter.Field, "ext.") {
            jsonbPath := strings.TrimPrefix(filter.Field, "ext.")
            
            // Construir condición JSONB según tipo de filtro
            switch filter.Operator {
            case "eq":
                jsonbFilters = append(jsonbFilters, 
                    fmt.Sprintf("tr.extended_properties->>'%s' = $%d", jsonbPath, paramIndex))
                params = append(params, filter.Value)
                paramIndex++
            case "contains":
                jsonbFilters = append(jsonbFilters, 
                    fmt.Sprintf("tr.extended_properties->>'%s' ILIKE $%d", jsonbPath, paramIndex))
                params = append(params, "%"+filter.Value.(string)+"%")
                paramIndex++
            // Más operadores JSONB...
            }
        }
    }
    
    // Combinar con otros filtros...
    
    // Ejecutar consulta con timeout y reintentos
    rows, err := r.executeWithTimeout(ctx, query, params...)
    // Procesar resultados...
}
```

### 6.3 Gestión de Eventos de Dominio

El sistema utiliza eventos de dominio para comunicar cambios entre diferentes partes del sistema, implementando el patrón Outbox para garantizar la entrega confiable.

```go
// Ejemplo de evento de dominio
type TableRowCreatedEvent struct {
    RowID           uuid.UUID              `json:"row_id"`
    LogicalTableID  uuid.UUID              `json:"logical_table_id"`
    Code            string                 `json:"code"`
    DisplayValue    string                 `json:"display_value"`
    ColumnValues    map[string]interface{} `json:"column_values"`
    Timestamp       time.Time              `json:"timestamp"`
    UserID          uuid.UUID              `json:"user_id"`
}

// Publicación de evento a través de outbox para garantizar confiabilidad
func (s *TableRowService) Create(ctx context.Context, cmd CreateTableRowCommand) (TableRowDTO, error) {
    // Lógica de creación...
    
    // Crear evento para outbox
    event := TableRowCreatedEvent{
        RowID:         newRow.ID,
        LogicalTableID: logicalTable.ID,
        // Otros campos...
    }
    
    // Usar outbox para garantizar entrega eventual
    outboxEntry := &OutboxEntry{
        ID:          uuid.New(),
        EventType:   "TableRowCreated",
        AggregateID: newRow.ID,
        EventData:   event,
        CreatedAt:   time.Now(),
    }
    
    // Guardar en tabla de outbox en la misma transacción
    if err := s.outboxRepository.Create(ctx, outboxEntry); err != nil {
        return TableRowDTO{}, fmt.Errorf("error guardando evento en outbox: %w", err)
    }
    
    return tableRowDTO, nil
}

// Procesador de outbox en background
func (p *OutboxProcessor) ProcessOutbox(ctx context.Context) error {
    for {
        select {
        case <-ctx.Done():
            return ctx.Err()
        case <-time.After(p.pollingInterval):
            entries, err := p.outboxRepository.GetUnprocessed(ctx, 100)
            if err != nil {
                p.logger.Error("Error obteniendo entradas de outbox", zap.Error(err))
                continue
            }
            
            for _, entry := range entries {
                // Publicar evento con reintentos y circuit breaker
                err := p.breaker.Execute(func() error {
                    return p.publishWithRetry(ctx, entry)
                })
                
                if err != nil {
                    p.logger.Error("Error publicando evento", 
                        zap.String("eventType", entry.EventType),
                        zap.Error(err))
                    p.metrics.RecordOutboxFailure(entry.EventType)
                    continue
                }
                
                // Marcar como procesado
                if err := p.outboxRepository.MarkAsProcessed(ctx, entry.ID); err != nil {
                    p.logger.Error("Error marcando entrada como procesada", zap.Error(err))
                }
                
                p.metrics.RecordOutboxSuccess(entry.p.metrics.RecordOutboxSuccess(entry.EventType)
            }
        }
    }
}
```

## 7. Sistema de Autenticación (Passkeys y Contraseña)

Este sistema soporta múltiples métodos de autenticación, incluyendo Passkeys y la autenticación tradicional basada en contraseña. La implementación de Passkeys se detalla a continuación, mientras que la autenticación con contraseña se aborda en otras secciones relevantes (como la definición del modelo de datos y los endpoints API).

### 7.1 Integración con Microsoft Passkeys

El sistema está diseñado para integrarse con el sistema de passkeys de Microsoft, permitiendo a los usuarios autenticarse sin contraseñas utilizando la tecnología FIDO2/WebAuthn.


#### 7.1.1 Arquitectura de Integración

```
┌───────────────────┐      ┌───────────────────┐      ┌─────────────────────┐
│                   │      │                   │      │                     │
│  Cliente (Web/App)│<────>│  Auth Service     │<────>│  Microsoft Identity │
│                   │      │                   │      │                     │
└───────────────────┘      └───────────────────┘      └─────────────────────┘
         |                         ^
         |                         |
         v                         |
┌───────────────────┐      ┌───────────────────┐
│                   │      │                   │
│  Local Device Auth│      │  User Repository  │
│  (FIDO2/WebAuthn) │      │                   │
└───────────────────┘      └───────────────────┘
```

#### 7.1.2 Implementación con go-webauthn/webauthn

La implementación utiliza la biblioteca go-webauthn/webauthn v1.0+ (la versión más reciente a mayo 2025) que proporciona una API completa para la gestión de WebAuthn/FIDO2:

```go
// Inicialización de WebAuthn
func InitWebAuthn(config *Config) (*webauthn.WebAuthn, error) {
    // Configurar opciones WebAuthn
    wconfig := &webauthn.Config{
        RPDisplayName: config.RelyingPartyName,
        RPID:          config.RelyingPartyID,
        RPOrigins:     config.AllowedOrigins,
        AuthenticatorSelection: webauthn.AuthenticatorSelection{
            RequireResidentKey: webauthn.PointerTo(true),
            ResidentKey:        webauthn.ResidentKeyRequirementRequired,
            UserVerification:   webauthn.UserVerificationRequired,
        },
        Attestation: webauthn.AttestationConveyancePreference(config.AttestationPreference),
        Debug:       config.Debug,
    }
    
    // Crear instancia WebAuthn
    wa, err := webauthn.New(wconfig)
    if err != nil {
        return nil, fmt.Errorf("error inicializando WebAuthn: %w", err)
    }
    
    return wa, nil
}
```

### 7.2 Proceso de Registro de Passkeys

El proceso de registro de passkeys se divide en dos fases:

1. **Inicio de Registro**: Configuración de opciones y generación de desafío
2. **Finalización de Registro**: Verificación de credenciales y almacenamiento

```go
// Inicio de registro
func (s *PasskeyService) StartRegistration(ctx context.Context, username string) (*RegistrationOptions, error) {
    // Inicio de span para trazabilidad
    ctx, span := s.tracer.Start(ctx, "PasskeyService.StartRegistration")
    defer span.End()
    
    // Aplicar límite de tasa para prevenir ataques
    if err := s.limiter.Wait(ctx); err != nil {
        return nil, fmt.Errorf("límite de tasa excedido: %w", err)
    }
    
    // Obtener usuario o crear uno nuevo
    user, err := s.userRepository.FindByUsername(ctx, username)
    if err != nil {
        if errors.Is(err, ErrUserNotFound) {
            // Crear usuario nuevo
            user = &User{
                ID:          uuid.New(),
                Username:    username,
                DisplayName: username,
                CreatedAt:   time.Now(),
            }
            if err := s.userRepository.Create(ctx, user); err != nil {
                s.logger.Error("Error creando usuario", zap.Error(err))
                return nil, fmt.Errorf("error creando usuario: %w", err)
            }
        } else {
            s.logger.Error("Error buscando usuario", zap.Error(err))
            return nil, fmt.Errorf("error buscando usuario: %w", err)
        }
    }
    
    // Crear opciones de WebAuthn para registro con Circuit Breaker
    var options *webauthn.CredentialCreation
    var session *webauthn.SessionData
    
    err = s.breaker.Execute(func() error {
        var innerErr error
        // WebAuthn BeginRegistration puede fallar por razones externas
        options, session, innerErr = s.webAuthn.BeginRegistration(
            webauthn.NewUser(
                user.ID.String(),
                user.Username,
                user.DisplayName,
            ),
        )
        return innerErr
    })
    
    if err != nil {
        s.logger.Error("Error iniciando registro WebAuthn", zap.Error(err))
        return nil, fmt.Errorf("error iniciando registro: %w", err)
    }
    
    // Guardar el desafío en sesión con TTL
    sessionKey := fmt.Sprintf("passkey:reg:%s", user.ID)
    if err := s.sessionManager.SaveWithTTL(ctx, sessionKey, session, 5*time.Minute); err != nil {
        s.logger.Error("Error guardando estado de sesión", zap.Error(err))
        return nil, fmt.Errorf("error guardando estado de sesión: %w", err)
    }
    
    // Métricas
    s.metrics.RecordPasskeyRegistrationStart(username)
    
    return &RegistrationOptions{
        Username:  username,
        Challenge: base64.RawURLEncoding.EncodeToString(options.Challenge),
        Options:   options,
    }, nil
}

// Finalización de registro
func (s *PasskeyService) FinishRegistration(ctx context.Context, username string, response *AttestationResponse) (*PasskeyInfo, error) {
    // Inicio de span para trazabilidad
    ctx, span := s.tracer.Start(ctx, "PasskeyService.FinishRegistration")
    defer span.End()
    
    // Obtener usuario y estado de registro
    user, err := s.userRepository.FindByUsername(ctx, username)
    if err != nil {
        return nil, fmt.Errorf("error buscando usuario: %w", err)
    }
    
    // Recuperar estado de sesión
    sessionKey := fmt.Sprintf("passkey:reg:%s", user.ID)
    var session webauthn.SessionData
    if err := s.sessionManager.Get(ctx, sessionKey, &session); err != nil {
        return nil, fmt.Errorf("sesión de registro no encontrada o expirada: %w", err)
    }
    
    // Verificar respuesta de atestación con Circuit Breaker y reintentos
    var credential *webauthn.Credential
    
    err = s.breaker.Execute(func() error {
        var innerErr error
        // WebAuthn puede fallar por razones externas
        credential, innerErr = s.webAuthn.FinishRegistration(
            webauthn.NewUser(
                user.ID.String(),
                user.Username,
                user.DisplayName,
            ),
            session,
            response,
        )
        return innerErr
    })
    
    if err != nil {
        s.logger.Error("Error en verificación de atestación", zap.Error(err))
        s.metrics.RecordPasskeyRegistrationFailure(username, "verification_error")
        return nil, fmt.Errorf("error en verificación de atestación: %w", err)
    }
    
    // Verificar proveedor y extraer información de atestación
    providerInfo, err := s.detectProvider(credential)
    if err != nil {
        s.logger.Warn("No se pudo detectar proveedor", zap.Error(err))
    }
    
    // Crear nuevo registro de passkey con Unit of Work
    var passkey *Passkey
    
    err = s.uow.WithTransaction(ctx, func(ctx context.Context) error {
        passkey = &Passkey{
            ID:              uuid.New(),
            UserID:          user.ID,
            CredentialID:    base64.RawURLEncoding.EncodeToString(credential.ID),
            PublicKey:       credential.PublicKey,
            AttestationType: credential.AttestationType,
            Provider:        providerInfo.Provider,
            DeviceInfo:      providerInfo.DeviceInfo,
            AAGUID:          credential.Authenticator.AAGUID,
            LastUsedAt:      time.Now(),
            IsActive:        true,
            CreatedAt:       time.Now(),
        }
        
        if err := s.passkeyRepository.Create(ctx, passkey); err != nil {
            return fmt.Errorf("error guardando passkey: %w", err)
        }
        
        // Publicar evento de dominio a través de outbox
        event := PasskeyRegisteredEvent{
            PasskeyID:   passkey.ID,
            UserID:      user.ID,
            Provider:    providerInfo.Provider,
            DeviceInfo:  providerInfo.DeviceInfo,
            RegisteredAt: passkey.CreatedAt,
        }
        
        if err := s.outboxRepository.CreateEntry(ctx, "PasskeyRegistered", passkey.ID, event); err != nil {
            return fmt.Errorf("error publicando evento: %w", err)
        }
        
        return nil
    })
    
    if err != nil {
        s.logger.Error("Error en transacción de finalización de registro", zap.Error(err))
        s.metrics.RecordPasskeyRegistrationFailure(username, "transaction_error")
        return nil, err
    }
    
    // Limpiar estado de sesión
    if err := s.sessionManager.Delete(ctx, sessionKey); err != nil {
        s.logger.Warn("Error eliminando estado de sesión", zap.Error(err))
    }
    
    // Métricas de éxito
    s.metrics.RecordPasskeyRegistrationSuccess(username, providerInfo.Provider)
    
    return &PasskeyInfo{
        ID:          passkey.ID,
        DeviceInfo:  passkey.DeviceInfo,
        Provider:    passkey.Provider,
        RegisteredAt: passkey.CreatedAt,
    }, nil
}
```

### 7.3 Proceso de Autenticación con Passkeys

El proceso de autenticación también se divide en dos fases:

1. **Inicio de Autenticación**: Configuración de opciones y generación de desafío
2. **Finalización de Autenticación**: Verificación de la firma y generación de sesión

```go
// Inicio de autenticación
func (s *PasskeyService) StartAuthentication(ctx context.Context, username string) (*AuthenticationOptions, error) {
    // Inicio de span para trazabilidad
    ctx, span := s.tracer.Start(ctx, "PasskeyService.StartAuthentication")
    defer span.End()
    
    // Aplicar límite de tasa para prevenir ataques
    if err := s.limiter.Wait(ctx); err != nil {
        return nil, fmt.Errorf("límite de tasa excedido: %w", err)
    }
    
    // Obtener usuario
    user, err := s.userRepository.FindByUsername(ctx, username)
    if err != nil {
        if errors.Is(err, ErrUserNotFound) {
            // Devolver error genérico por seguridad
            return nil, ErrInvalidCredentials
        }
        s.logger.Error("Error buscando usuario", zap.Error(err))
        return nil, fmt.Errorf("error buscando usuario: %w", err)
    }
    
    // Obtener passkeys del usuario con Circuit Breaker
    var passkeys []*Passkey
    
    err = s.breaker.Execute(func() error {
        var innerErr error
        passkeys, innerErr = s.passkeyRepository.FindByUserID(ctx, user.ID)
        return innerErr
    })
    
    if err != nil {
        s.logger.Error("Error obteniendo passkeys", zap.Error(err))
        return nil, fmt.Errorf("error obteniendo passkeys: %w", err)
    }
    
    if len(passkeys) == 0 {
        return nil, ErrNoPasskeysRegistered
    }
    
    // Crear credenciales permitidas
    allowedCredentials := make([]webauthn.CredentialDescriptor, len(passkeys))
    for i, pk := range passkeys {
        credID, err := base64.RawURLEncoding.DecodeString(pk.CredentialID)
        if err != nil {
            s.logger.Error("Error decodificando credencialID", 
                zap.String("credentialID", pk.CredentialID),
                zap.Error(err))
            continue
        }
        
        allowedCredentials[i] = webauthn.CredentialDescriptor{
            Type:       webauthn.CredentialTypePublicKey,
            ID:         credID,
            Transports: pk.DeviceInfo.Transports,
        }
    }
    
    // Crear opciones de WebAuthn para autenticación con Circuit Breaker
    var options *webauthn.CredentialAssertion
    var session *webauthn.SessionData
    
    err = s.breaker.Execute(func() error {
        var innerErr error
        options, session, innerErr = s.webAuthn.BeginLogin(
            webauthn.NewUser(
                user.ID.String(),
                user.Username,
                user.DisplayName,
            ),
            webauthn.WithAllowedCredentials(allowedCredentials),
        )
        return innerErr
    })
    
    if err != nil {
        s.logger.Error("Error iniciando autenticación WebAuthn", zap.Error(err))
        return nil, fmt.Errorf("error iniciando autenticación: %w", err)
    }
    
    // Guardar el desafío en sesión con TTL
    sessionKey := fmt.Sprintf("passkey:auth:%s", user.ID)
    if err := s.sessionManager.SaveWithTTL(ctx, sessionKey, session, 5*time.Minute); err != nil {
        s.logger.Error("Error guardando estado de sesión", zap.Error(err))
        return nil, fmt.Errorf("error guardando estado de sesión: %w", err)
    }
    
    // Métricas
    s.metrics.RecordPasskeyAuthenticationStart(username)
    
    return &AuthenticationOptions{
        Username:  username,
        Challenge: base64.RawURLEncoding.EncodeToString(options.Challenge),
        Options:   options,
    }, nil
}

// Finalización de autenticación
func (s *PasskeyService) FinishAuthentication(ctx context.Context, username string, response *AssertionResponse) (*SessionInfo, error) {
    // Inicio de span para trazabilidad
    ctx, span := s.tracer.Start(ctx, "PasskeyService.FinishAuthentication")
    defer span.End()
    
    // Obtener usuario y estado de autenticación
    user, err := s.userRepository.FindByUsername(ctx, username)
    if err != nil {
        if errors.Is(err, ErrUserNotFound) {
            // Devolver error genérico por seguridad
            return nil, ErrInvalidCredentials
        }
        return nil, fmt.Errorf("error buscando usuario: %w", err)
    }
    
    // Recuperar estado de sesión
    sessionKey := fmt.Sprintf("passkey:auth:%s", user.ID)
    var session webauthn.SessionData
    if err := s.sessionManager.Get(ctx, sessionKey, &session); err != nil {
        return nil, fmt.Errorf("sesión de autenticación no encontrada o expirada: %w", err)
    }
    
    // Verificar respuesta de aserción con Circuit Breaker
    var credential *webauthn.Credential
    
    err = s.breaker.Execute(func() error {
        var innerErr error
        credential, innerErr = s.webAuthn.FinishLogin(
            webauthn.NewUser(
                user.ID.String(),
                user.Username,
                user.DisplayName,
            ),
            session,
            response,
        )
        return innerErr
    })
    
    if err != nil {
        s.logger.Error("Error en verificación de aserción", zap.Error(err))
        s.metrics.RecordPasskeyAuthenticationFailure(username, "verification_error")
        return nil, fmt.Errorf("error en verificación de aserción: %w", err)
    }
    
    // Actualizar información del passkey (último uso)
    credentialIDBase64 := base64.RawURLEncoding.EncodeToString(credential.RawID)
    
    err = s.uow.WithTransaction(ctx, func(ctx context.Context) error {
        // Obtener passkey
        passkey, err := s.passkeyRepository.FindByCredentialID(ctx, credentialIDBase64)
        if err != nil {
            return fmt.Errorf("passkey no encontrado: %w", err)
        }
        
        // Actualizar último uso
        passkey.LastUsedAt = time.Now()
        if err := s.passkeyRepository.Update(ctx, passkey); err != nil {
            return fmt.Errorf("error actualizando passkey: %w", err)
        }
        
        // Publicar evento de autenticación a través de outbox
        event := PasskeyAuthenticatedEvent{
            PasskeyID:      passkey.ID,
            UserID:         user.ID,
            AuthenticatedAt: passkey.LastUsedAt,
        }
        
        if err := s.outboxRepository.CreateEntry(ctx, "PasskeyAuthenticated", passkey.ID, event); err != nil {
            // No crítico, registrar y continuar
            s.logger.Warn("Error publicando evento", zap.Error(err))
        }
        
        return nil
    })
    
    if err != nil {
        s.logger.Error("Error actualizando última fecha de uso del passkey", zap.Error(err))
        // No es crítico, continuamos con la autenticación
    }
    
    // Generar token de sesión con Circuit Breaker
    var session *Session
    
    err = s.breaker.Execute(func() error {
        var innerErr error
        session, innerErr = s.sessionManager.CreateSession(ctx, user.ID, SessionOptions{
            Duration:       24 * time.Hour,
            AuthMethod:     "passkey",
            AuthProviderID: credentialIDBase64,
            DeviceInfo:     extractDeviceInfo(ctx),
        })
        return innerErr
    })
    
    if err != nil {
        s.logger.Error("Error creando sesión", zap.Error(err))
        s.metrics.RecordPasskeyAuthenticationFailure(username, "session_error")
        return nil, fmt.Errorf("error creando sesión: %w", err)
    }
    
    // Limpiar estado de autenticación
    if err := s.sessionManager.Delete(ctx, sessionKey); err != nil {
        s.logger.Warn("Error eliminando estado de sesión", zap.Error(err))
    }
    
    // Métricas de éxito
    s.metrics.RecordPasskeyAuthenticationSuccess(username)
    
    return &SessionInfo{
        UserID:      user.ID,
        Username:    user.Username,
        SessionID:   session.ID,
        Token:       session.Token,
        ExpiresAt:   session.ExpiresAt,
        Permissions: user.Permissions,
    }, nil
}
```

### 7.4 Arquitectura de Adaptador para Proveedores

El sistema está diseñado para soportar múltiples proveedores de passkeys a través de una interfaz de adaptador común. Inicialmente se implementa el adaptador de Microsoft, pero la arquitectura permite añadir fácilmente otros proveedores en el futuro.

```go
// Interfaz del adaptador de proveedor
type PasskeyProviderAdapter interface {
    // Valida si un AAGUID pertenece a este proveedor
    CanHandle(aaguid uuid.UUID) bool
    
    // Valida la atestación específica del proveedor
    ValidateAttestation(ctx context.Context, attestation *protocol.AttestationObject) error
    
    // Extrae información específica del proveedor
    ExtractProviderInfo(attestation *protocol.AttestationObject) (ProviderInfo, error)
    
    // Nombre del proveedor
    ProviderName() string
}

// Implementación para Microsoft
type MicrosoftPasskeyAdapter struct {
    msIdentityClient *msidentity.Client
    aaguidPrefixes   map[string]bool
    config           MicrosoftAdapterConfig
    metrics          metrics.Reporter
    tracer           trace.Tracer
    breaker          CircuitBreaker
}

func NewMicrosoftPasskeyAdapter(config MicrosoftAdapterConfig, metrics metrics.Reporter, tracer trace.Tracer) *MicrosoftPasskeyAdapter {
    // Crear circuit breaker para llamadas a Microsoft Identity
    cb := gobreaker.NewCircuitBreaker(gobreaker.Settings{
        Name:        "microsoft-identity",
        MaxRequests: 5,
        Interval:    30 * time.Second,
        Timeout:     60 * time.Second,
        ReadyToTrip: func(counts gobreaker.Counts) bool {
            failureRatio := float64(counts.TotalFailures) / float64(counts.Requests)
            return counts.Requests >= 5 && failureRatio >= 0.6
        },
        OnStateChange: func(name string, from gobreaker.State, to gobreaker.State) {
            metrics.RecordCircuitBreakerStateChange("microsoft-identity", from.String(), to.String())
        },
    })
    
    return &MicrosoftPasskeyAdapter{
        msIdentityClient: msidentity.NewClient(config.TenantID, config.ClientID, config.ClientSecret),
        aaguidPrefixes: map[string]bool{
            "1f516392": true, // Ejemplo de prefijo de Microsoft Authenticator
            "2d0ec81e": true, // Ejemplo de prefijo de Windows Hello
        },
        config:  config,
        metrics: metrics,
        tracer:  tracer,
        breaker: NewGoBreaker(cb),
    }
}

func (a *MicrosoftPasskeyAdapter) CanHandle(aaguid uuid.UUID) bool {
    aaguidStr := aaguid.String()
    
    // Verificar prefijos conocidos de Microsoft
    for prefix := range a.aaguidPrefixes {
        if strings.HasPrefix(aaguidStr, prefix) {
            return true
        }
    }
    
    return false
}

func (a *MicrosoftPasskeyAdapter) ValidateAttestation(ctx context.Context, attestation *protocol.AttestationObject) error {
    ctx, span := a.tracer.Start(ctx, "MicrosoftPasskeyAdapter.ValidateAttestation")
    defer span.End()
    
    // Implementación específica para Microsoft con Circuit Breaker
    return a.breaker.Execute(func() error {
        // Validación específica de Microsoft
        // ...
        return nil
    })
}

func (a *MicrosoftPasskeyAdapter) ExtractProviderInfo(attestation *protocol.AttestationObject) (ProviderInfo, error) {
    // Extraer datos específicos de Microsoft
    // ...
    
    return ProviderInfo{
        Provider:   "Microsoft",
        DeviceType: determineDeviceType(attestation),
        TrustLevel: determineTrustLevel(attestation),
        DeviceInfo: DeviceInfo{
            Name:       determineName(attestation),
            Platform:   determinePlatform(attestation),
            Transports: determineTransports(attestation),
        },
    }, nil
}

func (a *MicrosoftPasskeyAdapter) ProviderName() string {
    return "microsoft"
}
```

### 7.5 Gestión de Sesiones

El sistema implementa una gestión de sesiones segura y escalable utilizando JWTs con rotación de claves y revocación de tokens.

```go
// Servicio de gestión de sesiones
type SessionManager struct {
    secretManager   SecretManager
    sessionRepo     SessionRepository
    cacheManager    CacheManager
    config          SessionConfig
    metrics         metrics.Reporter
    tracer          trace.Tracer
    logger          *zap.Logger
}

// Creación de sesión
func (m *SessionManager) CreateSession(ctx context.Context, userID uuid.UUID, opts SessionOptions) (*Session, error) {
    ctx, span := m.tracer.Start(ctx, "SessionManager.CreateSession")
    defer span.End()
    
    // Generar ID de sesión
    sessionID := uuid.New()
    
    // Crear sesión
    session := &Session{
        ID:           sessionID,
        UserID:       userID,
        CreatedAt:    time.Now(),
        ExpiresAt:    time.Now().Add(opts.Duration),
        AuthMethod:   opts.AuthMethod,
        AuthProvider: opts.AuthProviderID,
        DeviceInfo:   opts.DeviceInfo,
        IsActive:     true,
    }
    
    // Generar JWT con clave rotativa
    token, err := m.generateJWT(session)
    if err != nil {
        m.metrics.RecordSessionError("jwt_generation")
        return nil, fmt.Errorf("error generando JWT: %w", err)
    }
    
    session.Token = token
    
    // Persistir sesión
    if err := m.sessionRepo.Create(ctx, session); err != nil {
        m.metrics.RecordSessionError("session_creation")
        return nil, fmt.Errorf("error guardando sesión: %w", err)
    }
    
    // Almacenar en caché
    cacheKey := fmt.Sprintf("session:%s", sessionID.String())
    if err := m.cacheManager.Set(
        cacheKey, 
        session,
        time.Until(session.ExpiresAt),
    ); err != nil {
        // Registrar pero continuar (caché es secundario)
        m.logger.Warn("error cacheando sesión", zap.Error(err))
    }
    
    // Métricas
    m.metrics.RecordSessionCreation(opts.AuthMethod)
    
    return session, nil
}

// Validación de sesión
func (m *SessionManager) ValidateSession(ctx context.Context, token string) (*Session, error) {
    ctx, span := m.tracer.Start(ctx, "SessionManager.ValidateSession")
    defer span.End()
    
    // Extraer ID de sesión y validar firma del JWT
    claims, err := m.validateJWT(token)
    if err != nil {
        m.metrics.RecordSessionError("jwt_validation")
        return nil, fmt.Errorf("token de sesión inválido: %w", err)
    }
    
    // Verificar que la sesión existe y está activa
    sessionID, err := uuid.Parse(claims.SessionID)
    if err != nil {
        m.metrics.RecordSessionError("invalid_session_id")
        return nil, fmt.Errorf("ID de sesión inválido en token: %w", err)
    }
    
    // Intentar obtener la sesión utilizando un mecanismo de caché aside
    var session *Session
    cacheKey := fmt.Sprintf("session:%s", sessionID.String())
    
    // Buscar en caché primero
    found, err := m.cacheManager.Get(cacheKey, &session)
    if err != nil {
        // Error de caché, registrar y continuar
        m.logger.Warn("error obteniendo sesión de caché", zap.Error(err))
    }
    
    if !found {
        // Caché miss, buscar en base de datos
        span.SetAttribute("cache.hit", false)
        m.metrics.RecordCacheMiss("session")
        
        // Utilizar circuit breaker para acceso a base de datos
        var dbErr error
        session, dbErr = m.sessionRepo.FindByID(ctx, sessionID)
        if dbErr != nil {
            if errors.Is(dbErr, ErrSessionNotFound) {
                m.metrics.RecordSessionError("session_not_found")
                return nil, fmt.Errorf("sesión no encontrada: %w", dbErr)
            }
            m.metrics.RecordSessionError("session_db_error")
            return nil, fmt.Errorf("error buscando sesión: %w", dbErr)
        }
        
        // Actualizar caché
        if err := m.cacheManager.Set(cacheKey, session, time.Until(session.ExpiresAt)); err != nil {
            m.logger.Warn("error actualizando caché de sesión", zap.Error(err))
        }
    } else {
        span.SetAttribute("cache.hit", true)
        m.metrics.RecordCacheHit("session")
    }
    
    // Verificar si la sesión está activa
    if !session.IsActive {
        m.metrics.RecordSessionError("session_revoked")
        return nil, ErrSessionRevoked
    }
    
    // Verificar si la sesión no ha expirado
    if time.Now().After(session.ExpiresAt) {
        m.metrics.RecordSessionError("session_expired")
        
        // Marcar explícitamente como inactiva para evitar futuros accesos
        session.IsActive = false
        if err := m.sessionRepo.Update(ctx, session); err != nil {
            m.logger.Warn("error actualizando estado de sesión expirada", zap.Error(err))
        }
        
        // Eliminar de caché
        if err := m.cacheManager.Delete(cacheKey); err != nil {
            m.logger.Warn("error eliminando sesión expirada de caché", zap.Error(err))
        }
        
        return nil, ErrSessionExpired
    }
    
    // Métricas de éxito
    m.metrics.RecordSessionValidation(session.AuthMethod, true)
    
    return session, nil
}

// Revocación de sesión
func (m *SessionManager) RevokeSession(ctx context.Context, sessionID uuid.UUID) error {
    ctx, span := m.tracer.Start(ctx, "SessionManager.RevokeSession")
    defer span.End()
    
    // Obtener sesión
    session, err := m.sessionRepo.FindByID(ctx, sessionID)
    if err != nil {
        if errors.Is(err, ErrSessionNotFound) {
            // Ya no existe, considerar como éxito
            return nil
        }
        m.metrics.RecordSessionError("revocation_find_error")
        return fmt.Errorf("error buscando sesión: %w", err)
    }
    
    // Marcar como inactiva
    session.IsActive = false
    if err := m.sessionRepo.Update(ctx, session); err != nil {
        m.metrics.RecordSessionError("revocation_update_error")
        return fmt.Errorf("error actualizando sesión: %w", err)
    }
    
    // Eliminar de caché
    cacheKey := fmt.Sprintf("session:%s", sessionID.String())
    if err := m.cacheManager.Delete(cacheKey); err != nil {
        // No crítico, registrar y continuar
        m.logger.Warn("error eliminando sesión de caché", zap.Error(err))
    }
    
    // Publicar evento de revocación para notificar a otros componentes
    if m.eventBus != nil {
        event := SessionRevokedEvent{
            SessionID:  sessionID,
            UserID:     session.UserID,
            RevokedAt:  time.Now(),
            DeviceInfo: session.DeviceInfo,
        }
        
        if err := m.eventBus.PublishEvent(ctx, "SessionRevoked", sessionID, event); err != nil {
            // No crítico, solo registrar
            m.logger.Warn("error publicando evento de revocación de sesión", zap.Error(err))
        }
    }
    
    // Métricas
    m.metrics.RecordSessionRevocation(session.AuthMethod)
    
    return nil
}
```

## 8. Gestión de Transacciones y Consistencia

### 8.1 Patrón Unit of Work

El sistema implementa el patrón Unit of Work para garantizar la atomicidad de las operaciones que afectan a múltiples tablas y entidades EAV.

```go
// Interfaz Unit of Work
type UnitOfWork interface {
    // Repositorios
    LogicalTables() LogicalTableRepository
    ColumnDefinitions() ColumnDefinitionRepository
    TableRows() TableRowRepository
    ColumnValues() ColumnValueRepository
    OutboxEntries() OutboxRepository
    
    // Operaciones transaccionales
    BeginTransaction(ctx context.Context) (context.Context, error)
    Commit(ctx context.Context) error
    Rollback(ctx context.Context) error
    
    // Operaciones transaccionales con función
    WithTransaction(ctx context.Context, fn func(ctx context.Context) error) error
}

// Implementación básica
type unitOfWork struct {
    db *sqlx.DB
    logicalTableRepo LogicalTableRepository
    columnDefinitionRepo ColumnDefinitionRepository
    tableRowRepo TableRowRepository
    columnValueRepo ColumnValueRepository
    outboxRepo OutboxRepository
    txKey contextKey
    logger *zap.Logger
    metrics metrics.Reporter
    tracer trace.Tracer
}

func NewUnitOfWork(
    db *sqlx.DB, 
    logger *zap.Logger,
    metrics metrics.Reporter,
    tracer trace.Tracer,
) UnitOfWork {
    return &unitOfWork{
        db: db,
        logicalTableRepo: NewLogicalTableRepository(db),
        columnDefinitionRepo: NewColumnDefinitionRepository(db),
        tableRowRepo: NewTableRowRepository(db),
        columnValueRepo: NewColumnValueRepository(db),
        outboxRepo: NewOutboxRepository(db),
        txKey: contextKey("transaction"),
        logger: logger,
        metrics: metrics,
        tracer: tracer,
    }
}

func (u *unitOfWork) LogicalTables() LogicalTableRepository {
    return u.logicalTableRepo
}

func (u *unitOfWork) ColumnDefinitions() ColumnDefinitionRepository {
    return u.columnDefinitionRepo
}

func (u *unitOfWork) TableRows() TableRowRepository {
    return u.tableRowRepo
}

func (u *unitOfWork) ColumnValues() ColumnValueRepository {
    return u.columnValueRepo
}

func (u *unitOfWork) OutboxEntries() OutboxRepository {
    return u.outboxRepo
}

func (u *unitOfWork) BeginTransaction(ctx context.Context) (context.Context, error) {
    ctx, span := u.tracer.Start(ctx, "UnitOfWork.BeginTransaction")
    defer span.End()
    
    // Verificar si ya existe una transacción en el contexto
    if tx := ctx.Value(u.txKey); tx != nil {
        return ctx, nil
    }
    
    // Iniciar nueva transacción
    tx, err := u.db.BeginTxx(ctx, nil)
    if err != nil {
        u.metrics.RecordDatabaseError("begin_transaction")
        return ctx, fmt.Errorf("error iniciando transacción: %w", err)
    }
    
    // Añadir la transacción al contexto
    span.AddEvent("Transaction started")
    return context.WithValue(ctx, u.txKey, tx), nil
}

func (u *unitOfWork) Commit(ctx context.Context) error {
    ctx, span := u.tracer.Start(ctx, "UnitOfWork.Commit")
    defer span.End()
    
    tx, ok := ctx.Value(u.txKey).(*sqlx.Tx)
    if !ok || tx == nil {
        return errors.New("no hay transacción activa")
    }
    
    if err := tx.Commit(); err != nil {
        u.metrics.RecordDatabaseError("commit_transaction")
        span.RecordError(err)
        span.SetStatus(codes.Error, "Transaction commit failed")
        return fmt.Errorf("error en commit: %w", err)
    }
    
    span.AddEvent("Transaction committed")
    u.metrics.RecordDatabaseTransaction("commit")
    return nil
}

func (u *unitOfWork) Rollback(ctx context.Context) error {
    ctx, span := u.tracer.Start(ctx, "UnitOfWork.Rollback")
    defer span.End()
    
    tx, ok := ctx.Value(u.txKey).(*sqlx.Tx)
    if !ok || tx == nil {
        return errors.New("no hay transacción activa")
    }
    
    if err := tx.Rollback(); err != nil {
        u.metrics.RecordDatabaseError("rollback_transaction")
        span.RecordError(err)
        span.SetStatus(codes.Error, "Transaction rollback failed")
        return fmt.Errorf("error en rollback: %w", err)
    }
    
    span.AddEvent("Transaction rolled back")
    u.metrics.RecordDatabaseTransaction("rollback")
    return nil
}

func (u *unitOfWork) WithTransaction(ctx context.Context, fn func(ctx context.Context) error) error {
    ctx, span := u.tracer.Start(ctx, "UnitOfWork.WithTransaction")
    defer span.End()
    
    // Iniciar transacción
    ctx, err := u.BeginTransaction(ctx)
    if err != nil {
        return err
    }
    
    // Asegurar rollback en caso de pánico
    defer func() {
        if r := recover(); r != nil {
            u.logger.Error("Panic en transacción",
                zap.Any("panic", r),
                zap.String("stack", string(debug.Stack())))
                
            u.Rollback(ctx)
            span.RecordError(fmt.Errorf("panic en transacción: %v", r))
            span.SetStatus(codes.Error, "Transaction panic")
            
            // Re-panic después de rollback
            panic(r)
        }
    }()
    
    // Ejecutar función dentro de la transacción
    if err := fn(ctx); err != nil {
        // Rollback en caso de error
        rbErr := u.Rollback(ctx)
        if rbErr != nil {
            return fmt.Errorf("error en rollback después de: %v: %w", err, rbErr)
        }
        
        span.RecordError(err)
        span.SetStatus(codes.Error, "Transaction function failed")
        return err
    }
    
    // Commit si todo fue exitoso
    if err := u.Commit(ctx); err != nil {
        return fmt.Errorf("error en commit: %w", err)
    }
    
    span.SetStatus(codes.Ok, "Transaction completed successfully")
    return nil
}
```

### 8.2 Implementación Transaccional para Operaciones Batch

```go
// Servicio para operaciones batch con implementación de Circuit Breaker
type BatchOperationService struct {
    uow UnitOfWork
    eventBus EventBus
    logger *zap.Logger
    metrics metrics.Reporter
    tracer trace.Tracer
    breaker CircuitBreaker
}

func NewBatchOperationService(
    uow UnitOfWork,
    eventBus EventBus,
    logger *zap.Logger,
    metrics metrics.Reporter,
    tracer trace.Tracer,
) *BatchOperationService {
    // Configurar Circuit Breaker para operaciones de batch
    cb := gobreaker.NewCircuitBreaker(gobreaker.Settings{
        Name:        "batch-operations",
        MaxRequests: 5,
        Interval:    30 * time.Second,
        Timeout:     60 * time.Second,
        ReadyToTrip: func(counts gobreaker.Counts) bool {
            failureRatio := float64(counts.TotalFailures) / float64(counts.Requests)
            return counts.Requests >= 5 && failureRatio >= 0.6
        },
        OnStateChange: func(name string, from gobreaker.State, to gobreaker.State) {
            metrics.RecordCircuitBreakerStateChange("batch-operations", from.String(), to.String())
        },
    })
    
    return &BatchOperationService{
        uow:      uow,
        eventBus: eventBus,
        logger:   logger,
        metrics:  metrics,
        tracer:   tracer,
        breaker:  NewGoBreaker(cb),
    }
}

// Ejecutar operaciones en batch
// Servicio para operaciones batch con implementación de Circuit Breaker
func (s *BatchOperationService) processBatch(ctx context.Context, logicalTable *LogicalTable, operations []Operation) (BatchResult, error) {
    result := BatchResult{
        ProcessedCount: len(operations),
        SuccessCount:   0,
        ErrorCount:     0,
    }
    
    // Estrategia de errores
    continueOnError := logicalTable.ErrorHandlingStrategy == "CONTINUE"
    
    // Proceso de lote con Circuit Breaker
    err := s.breaker.Execute(func() error {
        for i, op := range operations {
            // Procesar cada operación...
            
            // Para operaciones de tipo create o update
            if op.Type == OperationTypeCreate || op.Type == OperationTypeUpdate {
                // Separar propiedades regulares y extendidas
                regularProps := make(map[string]interface{})
                extendedProps := make(map[string]interface{})
                
                // Analizar esquema para determinar qué propiedades van a JSONB
                for key, value := range op.Fields {
                    colDef, err := s.getColumnDefinition(ctx, logicalTable.ID, key)
                    if err != nil {
                        if errors.Is(err, ErrNotFound) {
                            // Propiedad no definida en el esquema, va a JSONB
                            extendedProps[key] = value
                        } else {
                            return err
                        }
                    } else {
                        // Propiedad definida en el esquema, va a EAV tradicional
                        regularProps[key] = value
                    }
                }
                
                // Procesar según tipo de operación
                if op.Type == OperationTypeCreate {
                    _, err = s.createRowWithExtended(ctx, logicalTable.ID, regularProps, extendedProps)
                } else {
                    err = s.updateRowWithExtended(ctx, logicalTable.ID, op.RowID, regularProps, extendedProps)
                }
            } else {
                // Operaciones de borrado siguen igual
                err = s.deleteRow(ctx, logicalTable.ID, op.RowID)
            }
            
            // Manejar resultado...
        }
        
        return nil
    })
    
    // Actualizar contadores...
    return result, err
}

func (s *BatchOperationService) ExecuteBatch(ctx context.Context, tableName string, operations []Operation) (*BatchResult, error) {
    ctx, span := s.tracer.Start(ctx, "BatchOperationService.ExecuteBatch",
        trace.WithAttributes(
            attribute.String("table_name", tableName),
            attribute.Int("operations_count", len(operations)),
        ))
    defer span.End()
    
    var result BatchResult
    
    // Ejecutar con Circuit Breaker
    err := s.breaker.Execute(func() error {
        // Métricas de inicio
        s.metrics.RecordBatchOperationStart(tableName, len(operations))
        
        // Iniciar transacción global
        return s.uow.WithTransaction(ctx, func(ctx context.Context) error {
            // Obtener la tabla lógica
            logicalTable, err := s.uow.LogicalTables().FindByName(ctx, tableName)
            if err != nil {
                return fmt.Errorf("tabla no encontrada: %w", err)
            }
            
            // Procesar cada operación
			batchResult, err := s.processBatch(ctx, logicalTable, operations)
			if err != nil {
				return err
			}
            
            return nil
        })
    })
    
    // Establecer éxito general del batch
    result.Success = (err == nil)
    if err != nil {
        result.ErrorMessage = err.Error()
        span.RecordError(err)
        span.SetStatus(codes.Error, "Batch operation failed")
        
        // Métricas de error
        s.metrics.RecordBatchOperationError(tableName, "overall")
    } else {
        // Métricas de éxito global
        s.metrics.RecordBatchOperationSuccess(tableName, "overall")
        span.SetStatus(codes.Ok, "Batch operation completed successfully")
    }
    
    return &result, err
}

// Implementaciones de operaciones individuales
func (s *BatchOperationService) createRow(ctx context.Context, tableID uuid.UUID, fields map[string]interface{}) (*TableRow, error) {
    ctx, span := s.tracer.Start(ctx, "BatchOperationService.createRow")
    defer span.End()
    
    // Implementación con Unit of Work
    // ...
    return newRow, nil
}

func (s *BatchOperationService) updateRow(ctx context.Context, tableID uuid.UUID, rowID uuid.UUID, fields map[string]interface{}) error {
    ctx, span := s.tracer.Start(ctx, "BatchOperationService.updateRow")
    defer span.End()
    
    // Implementación con Unit of Work
    // ...
    return nil
}

func (s *BatchOperationService) deleteRow(ctx context.Context, tableID uuid.UUID, rowID uuid.UUID) error {
    ctx, span := s.tracer.Start(ctx, "BatchOperationService.deleteRow")
    defer span.End()
    
    // Implementación con Unit of Work
    // ...
    return nil
}

// Creación con soporte para propiedades extendidas
func (s *BatchOperationService) createRowWithExtended(
    ctx context.Context, 
    tableID uuid.UUID, 
    regularProps map[string]interface{},
    extendedProps map[string]interface{},
) (*TableRow, error) {
    // Crear fila con propiedades básicas y JSONB
    row := &TableRow{
        ID:                uuid.New(),
        LogicalTableID:    tableID,
        Code:              regularProps["code"].(string),
        DisplayValue:      regularProps["display_value"].(string),
        SortOrder:         getIntValue(regularProps, "sort_order", 0),
        IsActive:          getBoolValue(regularProps, "is_active", true),
        ExtendedProperties: extendedProps,
        CreatedAt:         time.Now(),
        CreatedByUserID:   getCurrentUserID(ctx),
    }
    
    // Persistir fila
    if err := s.uow.TableRows().Create(ctx, row); err != nil {
        return nil, err
    }
    
    // Procesar valores de columna tradicionales...
    
    return row, nil
}

```

### 8.3 Gestión de Bloqueos y Concurrencia

```go
// Estrategia de bloqueo optimista con control de versiones
type TableRowWithVersion struct {
    TableRow
    Version int64 `db:"version"`
}

// Actualización con control de versión
func (r *TableRowRepository) UpdateWithVersion(ctx context.Context, row *TableRowWithVersion) error {
    ctx, span := r.tracer.Start(ctx, "TableRowRepository.UpdateWithVersion")
    defer span.End()
    
    // Métricas
    r.metrics.RecordRepositoryOperation("TableRow", "update_versioned")
    
    // Obtener transacción del contexto
    tx := getTxFromContext(ctx, r.db)
    
    // Actualizar con control de versión
    result, err := tx.ExecContext(ctx, `
        UPDATE table_rows
        SET 
            code = $1, 
            display_value = $2, 
            sort_order = $3, 
            is_active = $4, 
            updated_at = $5, 
            updated_by_user_id = $6, 
            version = version + 1
        WHERE id = $7 AND version = $8
    `, 
        row.Code, 
        row.DisplayValue, 
        row.SortOrder, 
        row.IsActive,
        time.Now(), 
        getCurrentUserID(ctx), 
        row.ID, 
        row.Version,
    )
    
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, "Update failed")
        r.metrics.RecordRepositoryError("TableRow", "update_versioned")
        return fmt.Errorf("error actualizando fila: %w", err)
    }
    
    // Verificar si se actualizó alguna fila
    rowsAffected, err := result.RowsAffected()
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, "Error getting rows affected")
        r.metrics.RecordRepositoryError("TableRow", "update_versioned_rows")
        return fmt.Errorf("error obteniendo filas afectadas: %w", err)
    }
    
    if rowsAffected == 0 {
        // No se actualizó ninguna fila, probablemente por conflicto de versión
        r.metrics.RecordRepositoryError("TableRow", "concurrent_modification")
        return ErrConcurrentModification
    }
    
    // Incrementar versión en objeto local
    row.Version++
    
    // Invalidar caché
    if err := r.invalidateRowCache(ctx, row.ID); err != nil {
        r.logger.Warn("Error invalidando caché", zap.Error(err))
    }
    
    // Publicar evento mediante outbox
    if err := r.publishRowUpdatedEvent(ctx, row.TableRow); err != nil {
        r.logger.Warn("Error publicando evento de actualización", zap.Error(err))
    }
    
    span.SetStatus(codes.Ok, "Update successful")
    return nil
}

// Resolver conflictos de concurrencia
func (s *TableRowService) ResolveConflict(ctx context.Context, rowID uuid.UUID) (*TableRow, error) {
    ctx, span := s.tracer.Start(ctx, "TableRowService.ResolveConflict")
    defer span.End()
    
    // Bloquear registro para lectura exclusiva
    var row *TableRow
    var err error
    
    err = s.uow.WithTransaction(ctx, func(ctx context.Context) error {
        // Obtener fila con bloqueo exclusivo
        row, err = s.uow.TableRows().FindByIDWithLock(ctx, rowID)
        if err != nil {
            return err
        }
        
        // Cargar última versión de valores de columnas
        if err := s.loadColumnValues(ctx, row); err != nil {
            return err
        }
        
        return nil
    })
    
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, "Conflict resolution failed")
        return nil, fmt.Errorf("error resolviendo conflicto: %w", err)
    }
    
    span.SetStatus(codes.Ok, "Conflict resolved")
    return row, nil
}

// Implementación de bloqueo pesimista cuando sea necesario
func (r *TableRowRepository) FindByIDWithLock(ctx context.Context, id uuid.UUID) (*TableRow, error) {
    ctx, span := r.tracer.Start(ctx, "TableRowRepository.FindByIDWithLock")
    defer span.End()
    
    tx := getTxFromContext(ctx, r.db)
    
    var row TableRow
    err := tx.GetContext(ctx, &row, `
        SELECT id, logical_table_id, code, display_value, sort_order, is_active, parent_row_id,
               created_at, created_by_user_id, updated_at, updated_by_user_id, version
        FROM table_rows
        WHERE id = $1
        FOR UPDATE
    `, id)
    
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, ErrNotFound
        }
        return nil, fmt.Errorf("error consultando fila: %w", err)
    }
    
    return &row, nil
}
```

### 8.4 Patrón Outbox para Consistencia de Eventos

```go
// Estructura para Outbox
type OutboxEntry struct {
    ID               uuid.UUID      `db:"id"`
    EventType        string         `db:"event_type"`
    AggregateID      uuid.UUID      `db:"aggregate_id"`
    AggregateType    string         `db:"aggregate_type"`
    EventData        interface{}    `db:"event_data"`
    CreatedAt        time.Time      `db:"created_at"`
    ProcessedAt      *time.Time     `db:"processed_at"`
    ProcessingAttempts int          `db:"processing_attempts"`
    Version          int            `db:"version"`
}

// Repositorio de Outbox
type OutboxRepository interface {
    CreateEntry(ctx context.Context, eventType string, aggregateID uuid.UUID, eventData interface{}) error
    GetUnprocessed(ctx context.Context, limit int) ([]*OutboxEntry, error)
    MarkAsProcessed(ctx context.Context, id uuid.UUID) error
    IncrementAttempts(ctx context.Context, id uuid.UUID) error
}

// Implementación del repositorio
type PostgresOutboxRepository struct {
    db      *sqlx.DB
    logger  *zap.Logger
    metrics metrics.Reporter
    tracer  trace.Tracer
}

func (r *PostgresOutboxRepository) CreateEntry(ctx context.Context, eventType string, aggregateID uuid.UUID, eventData interface{}) error {
    ctx, span := r.tracer.Start(ctx, "OutboxRepository.CreateEntry")
    defer span.End()
    
    // Determinar el tipo de agregado basado en evento
    aggregateType := determineAggregateType(eventType)
    
    // Serializar datos de evento a JSONB
    eventDataJSON, err := json.Marshal(eventData)
    if err != nil {
        span.RecordError(err)
        r.metrics.RecordRepositoryError("Outbox", "serialize_event")
        return fmt.Errorf("error serializando datos de evento: %w", err)
    }
    
    // Obtener transacción del contexto
    tx := getTxFromContext(ctx, r.db)
    
    // Insertar entrada en outbox
    _, err = tx.ExecContext(ctx, `
        INSERT INTO domain_events(
            id, event_type, aggregate_id, aggregate_type, event_data, 
            created_at, processed_at, processing_attempts, version
        ) VALUES (
            $1, $2, $3, $4, $5, $6, $7, $8, $9
        )
    `,
        uuid.New(),
        eventType,
        aggregateID,
        aggregateType,
        eventDataJSON,
        time.Now(),
        nil,
        0,
        1,
    )
    
    if err != nil {
        span.RecordError(err)
        r.metrics.RecordRepositoryError("Outbox", "create_entry")
        return fmt.Errorf("error creando entrada de outbox: %w", err)
    }
    
    span.SetStatus(codes.Ok, "Outbox entry created")
    r.metrics.RecordOutboxEntry(eventType)
    
    return nil
}

// Servicio de procesamiento de Outbox
type OutboxProcessor struct {
    outboxRepo OutboxRepository
    eventBus   EventBus
    logger     *zap.Logger
    metrics    metrics.Reporter
    tracer     trace.Tracer
    breaker    CircuitBreaker
    
    pollingInterval time.Duration
    maxAttempts     int
}

func (p *OutboxProcessor) Start(ctx context.Context) error {
    p.logger.Info("Iniciando procesador de outbox")
    
    for {
        select {
        case <-ctx.Done():
            p.logger.Info("Deteniendo procesador de outbox")
            return ctx.Err()
            
        case <-time.After(p.pollingInterval):
            if err := p.processEntries(ctx); err != nil {
                p.logger.Error("Error procesando entradas de outbox", zap.Error(err))
            }
        }
    }
}

func (p *OutboxProcessor) processEntries(ctx context.Context) error {
    // Crear span para todo el procesamiento
    ctx, span := p.tracer.Start(ctx, "OutboxProcessor.processEntries")
    defer span.End()
    
    // Obtener entradas no procesadas
    entries, err := p.outboxRepo.GetUnprocessed(ctx, 100)
    if err != nil {
        span.RecordError(err)
        p.metrics.RecordOutboxError("fetch")
        return fmt.Errorf("error obteniendo entradas no procesadas: %w", err)
    }
    
    if len(entries) == 0 {
        return nil
    }
    
    span.SetAttributes(attribute.Int("entries.count", len(entries)))
    p.logger.Info("Procesando entradas de outbox", zap.Int("count", len(entries)))
    
    // Procesar cada entrada
    for _, entry := range entries {
        entrySpan := trace.SpanFromContext(ctx)
        entrySpan.SetAttributes(
            attribute.String("entry.id", entry.ID.String()),
            attribute.String("entry.type", entry.EventType),
            attribute.Int("entry.attempts", entry.ProcessingAttempts),
        )
        
        // Verificar intentos máximos
        if entry.ProcessingAttempts >= p.maxAttempts {
            p.logger.Warn("Entrada de outbox excedió intentos máximos",
                zap.String("id", entry.ID.String()),
                zap.String("eventType", entry.EventType),
                zap.Int("attempts", entry.ProcessingAttempts),
            )
            
            // Marcar como procesada con error
            if err := p.outboxRepo.MarkAsProcessed(ctx, entry.ID); err != nil {
                p.logger.Error("Error marcando entrada como procesada",
                    zap.String("id", entry.ID.String()),
                    zap.Error(err),
                )
            }
            
            p.metrics.RecordOutboxError("max_attempts_exceeded")
            entrySpan.SetStatus(codes.Error, "Max attempts exceeded")
            continue
        }
        
        // Incrementar intentos
        if err := p.outboxRepo.IncrementAttempts(ctx, entry.ID); err != nil {
            p.logger.Error("Error incrementando intentos",
                zap.String("id", entry.ID.String()),
                zap.Error(err),
            )
            continue
        }
        
        // Publicar evento con Circuit Breaker
        err := p.breaker.Execute(func() error {
            return p.eventBus.PublishEvent(ctx, entry.EventType, entry.AggregateID, entry.EventData)
        })
        
        if err != nil {
            p.logger.Error("Error publicando evento",
                zap.String("id", entry.ID.String()),
                zap.String("eventType", entry.EventType),
                zap.Error(err),
            )
            
            p.metrics.RecordOutboxError("publish_failed")
            entrySpan.RecordError(err)
            entrySpan.SetStatus(codes.Error, "Failed to publish event")
            continue
        }
        
        // Marcar como procesada exitosamente
        if err := p.outboxRepo.MarkAsProcessed(ctx, entry.ID); err != nil {
            p.logger.Error("Error marcando entrada como procesada",
                zap.String("id", entry.ID.String()),
                zap.Error(err),
            )
            
            p.metrics.RecordOutboxError("mark_processed_failed")
            entrySpan.RecordError(err)
            continue
        }
        
        p.metrics.RecordOutboxSuccess(entry.EventType)
        entrySpan.SetStatus(codes.Ok, "Entry processed successfully")
    }
    
    span.SetStatus(codes.Ok, "Entries processed")
    return nil
}
```

## 9. Patrones de Diseño Implementados

### 9.1 Repository Pattern

El patrón Repository abstrae el acceso a datos y proporciona operaciones CRUD sobre las entidades del dominio.

```go
// Interfaz genérica de repositorio
type Repository[T any, ID comparable] interface {
    FindByID(ctx context.Context, id ID) (*T, error)
    FindAll(ctx context.Context) ([]*T, error)
    Create(ctx context.Context, entity *T) error
    Update(ctx context.Context, entity *T) error
    Delete(ctx context.Context, id ID) error
}

// Implementación ejemplo para TableRow
type TableRowRepository interface {
    Repository[TableRow, uuid.UUID]
    FindByLogicalTableID(ctx context.Context, tableID uuid.UUID) ([]*TableRow, error)
    FindByCode(ctx context.Context, tableID uuid.UUID, code string) (*TableRow, error)
    Query(ctx context.Context, tableID uuid.UUID, filters []FilterCriteria, sorting []SortCriteria, pageSize int, keySet interface{}) ([]*TableRow, interface{}, error)
    FindByIDWithLock(ctx context.Context, id uuid.UUID) (*TableRow, error)
}

// Implementación PostgreSQL con trazas y métricas
type postgresTableRowRepository struct {
    db *sqlx.DB
    logger *zap.Logger
    metrics metrics.Reporter
    tracer trace.Tracer
    cacheManager CacheManager
}

func NewTableRowRepository(
    db *sqlx.DB,
    logger *zap.Logger,
    metrics metrics.Reporter,
    tracer trace.Tracer,
    cacheManager CacheManager,
) TableRowRepository {
    return &postgresTableRowRepository{
        db: db,
        logger: logger,
        metrics: metrics,
        tracer: tracer,
        cacheManager: cacheManager,
    }
}

func (r *postgresTableRowRepository) FindByID(ctx context.Context, id uuid.UUID) (*TableRow, error) {
    ctx, span := r.tracer.Start(ctx, "TableRowRepository.FindByID",
        trace.WithAttributes(attribute.String("row_id", id.String())))
    defer span.End()
    
    // Registro de métricas
    r.metrics.RecordRepositoryOperation("TableRow", "find_by_id")
    
    // Intentar obtener de caché primero
    cacheKey := fmt.Sprintf("row:%s", id.String())
    var row TableRow
    
    found, err := r.cacheManager.Get(cacheKey, &row)
    if err != nil {
        r.logger.Warn("Error leyendo de caché", zap.Error(err))
    }
    
if found {
        span.SetAttribute("cache.hit", true)
        r.metrics.RecordCacheHit("TableRow")
        return &row, nil
    }
    
    span.SetAttribute("cache.hit", false)
    r.metrics.RecordCacheMiss("TableRow")
    
    // Obtener la transacción del contexto si existe
    tx := getTxFromContext(ctx, r.db)
    
    // Consultar en base de datos
    err = tx.GetContext(ctx, &row, `
        SELECT id, logical_table_id, code, display_value, sort_order, is_active, parent_row_id,
               created_at, created_by_user_id, updated_at, updated_by_user_id, version
        FROM table_rows
        WHERE id = $1
    `, id)
    
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            r.metrics.RecordRepositoryError("TableRow", "not_found")
            return nil, ErrNotFound
        }
        
        span.RecordError(err)
        span.SetStatus(codes.Error, "Database error")
        r.metrics.RecordRepositoryError("TableRow", "db_error")
        return nil, fmt.Errorf("error consultando fila: %w", err)
    }
    
    // Cargar valores de columnas
    if err := r.loadColumnValues(ctx, &row); err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, "Error loading column values")
        return nil, err
    }
    
    // Guardar en caché
    if err := r.cacheManager.Set(cacheKey, &row, 30*time.Minute); err != nil {
        r.logger.Warn("Error guardando en caché", zap.Error(err))
    }
    
    span.SetStatus(codes.Ok, "Row found")
    return &row, nil
}

// Implementación de otros métodos omitida para brevedad...
```

### 9.2 Factory Method Pattern

```go
// Fábrica para crear adaptadores de proveedores de passkeys
type PasskeyProviderFactory struct {
    providerConfigs map[string]ProviderConfig
    metrics         metrics.Reporter
    tracer          trace.Tracer
    logger          *zap.Logger
}

func NewPasskeyProviderFactory(
    configs map[string]ProviderConfig,
    metrics metrics.Reporter,
    tracer trace.Tracer,
    logger *zap.Logger,
) *PasskeyProviderFactory {
    return &PasskeyProviderFactory{
        providerConfigs: configs,
        metrics: metrics,
        tracer: tracer,
        logger: logger,
    }
}

func (f *PasskeyProviderFactory) CreateProvider(providerName string) (PasskeyProviderAdapter, error) {
    ctx, span := f.tracer.Start(context.Background(), "PasskeyProviderFactory.CreateProvider",
        trace.WithAttributes(attribute.String("provider", providerName)))
    defer span.End()
    
    config, exists := f.providerConfigs[providerName]
    if !exists {
        err := fmt.Errorf("configuración no encontrada para proveedor: %s", providerName)
        span.RecordError(err)
        span.SetStatus(codes.Error, "Provider config not found")
        f.metrics.RecordProviderFactoryError(providerName, "config_not_found")
        return nil, err
    }
    
    var provider PasskeyProviderAdapter
    var err error
    
    switch providerName {
    case "microsoft":
        provider = NewMicrosoftPasskeyAdapter(config.(MicrosoftAdapterConfig), f.metrics, f.tracer)
        
    case "google":
        provider = NewGooglePasskeyAdapter(config.(GoogleAdapterConfig), f.metrics, f.tracer)
        
    case "apple":
        provider = NewApplePasskeyAdapter(config.(AppleAdapterConfig), f.metrics, f.tracer)
        
    default:
        err = fmt.Errorf("proveedor no soportado: %s", providerName)
        span.RecordError(err)
        span.SetStatus(codes.Error, "Provider not supported")
        f.metrics.RecordProviderFactoryError(providerName, "not_supported")
        return nil, err
    }
    
    f.metrics.RecordProviderFactorySuccess(providerName)
    span.SetStatus(codes.Ok, "Provider created")
    return provider, nil
}

func (f *PasskeyProviderFactory) CreateProviderForAAGUID(aaguid uuid.UUID) (PasskeyProviderAdapter, error) {
    ctx, span := f.tracer.Start(context.Background(), "PasskeyProviderFactory.CreateProviderForAAGUID",
        trace.WithAttributes(attribute.String("aaguid", aaguid.String())))
    defer span.End()
    
    // Crear instancias de todos los adaptadores disponibles
    var providers []PasskeyProviderAdapter
    
    for name := range f.providerConfigs {
        provider, err := f.CreateProvider(name)
        if err != nil {
            f.logger.Warn("Error creando proveedor", 
                zap.String("provider", name), 
                zap.Error(err))
            continue
        }
        providers = append(providers, provider)
    }
    
    // Buscar el adaptador adecuado para el AAGUID
    for _, provider := range providers {
        if provider.CanHandle(aaguid) {
            span.SetAttributes(attribute.String("provider.selected", provider.ProviderName()))
            f.metrics.RecordProviderDetection(provider.ProviderName(), true)
            return provider, nil
        }
    }
    
    err := fmt.Errorf("no se encontró proveedor compatible para AAGUID: %s", aaguid)
    span.RecordError(err)
    span.SetStatus(codes.Error, "No compatible provider found")
    f.metrics.RecordProviderDetection("unknown", false)
    
    return nil, err
}
```

### 9.3 Strategy Pattern

```go
// Interfaz para estrategia de caché
type CacheStrategy interface {
    ShouldCache(tableName string) bool
    GetCacheKey(tableName string, id interface{}) string
    GetExpiration(tableName string) time.Duration
    InvalidationPattern(tableName string, id interface{}) string
}

// Implementación para distintas estrategias
type NoCacheStrategy struct{}

func (s *NoCacheStrategy) ShouldCache(tableName string) bool {
    return false
}

func (s *NoCacheStrategy) GetCacheKey(tableName string, id interface{}) string {
    return ""
}

func (s *NoCacheStrategy) GetExpiration(tableName string) time.Duration {
    return 0
}

func (s *NoCacheStrategy) InvalidationPattern(tableName string, id interface{}) string {
    return ""
}

type FullTableCacheStrategy struct {
    baseExpiration time.Duration
    tableExpirations map[string]time.Duration
    keyPrefix      string
}

func (s *FullTableCacheStrategy) ShouldCache(tableName string) bool {
    return true
}

func (s *FullTableCacheStrategy) GetCacheKey(tableName string, id interface{}) string {
    return fmt.Sprintf("%s:table:%s:id:%v", s.keyPrefix, tableName, id)
}

func (s *FullTableCacheStrategy) GetExpiration(tableName string) time.Duration {
    if exp, ok := s.tableExpirations[tableName]; ok {
        return exp
    }
    return s.baseExpiration
}

func (s *FullTableCacheStrategy) InvalidationPattern(tableName string, id interface{}) string {
    if id != nil {
        return fmt.Sprintf("%s:table:%s:id:%v", s.keyPrefix, tableName, id)
    }
    return fmt.Sprintf("%s:table:%s:*", s.keyPrefix, tableName)
}

// Estrategia con invalidación coordinada
type CoordinatedInvalidationStrategy struct {
    FullTableCacheStrategy
    eventBus EventBus
}

func NewCoordinatedInvalidationStrategy(
    baseExpiration time.Duration, 
    tableExpirations map[string]time.Duration,
    keyPrefix string,
    eventBus EventBus,
) *CoordinatedInvalidationStrategy {
    return &CoordinatedInvalidationStrategy{
        FullTableCacheStrategy: FullTableCacheStrategy{
            baseExpiration: baseExpiration,
            tableExpirations: tableExpirations,
            keyPrefix: keyPrefix,
        },
        eventBus: eventBus,
    }
}

// Invalidar caché de manera coordinada
func (s *CoordinatedInvalidationStrategy) InvalidateCache(ctx context.Context, tableName string, id interface{}) error {
    // Publicar evento de invalidación
    event := CacheInvalidationEvent{
        TableName:  tableName,
        EntityID:   id,
        Pattern:    s.InvalidationPattern(tableName, id),
        InvalidatedAt: time.Now(),
    }
    
    return s.eventBus.PublishEvent(ctx, "CacheInvalidated", uuid.New(), event)
}

// Fábrica de estrategias
func CreateCacheStrategyForTable(
    table *LogicalTable,
    eventBus EventBus,
) CacheStrategy {
    switch table.CachePolicy {
    case "None":
        return &NoCacheStrategy{}
        
    case "FullTable":
        return NewCoordinatedInvalidationStrategy(
            30 * time.Minute,
            map[string]time.Duration{},
            "eav",
            eventBus,
        )
        
    case "IndividualRows":
        return NewCoordinatedInvalidationStrategy(
            15 * time.Minute,
            map[string]time.Duration{},
            "eav",
            eventBus,
        )
        
    default:
        // Por defecto, no cachear
        return &NoCacheStrategy{}
    }
}
```

### 9.4 Decorator Pattern

```go
// Interfaz base
type TableRowService interface {
    Get(ctx context.Context, id uuid.UUID) (*TableRow, error)
    Create(ctx context.Context, cmd CreateTableRowCommand) (*TableRow, error)
    Update(ctx context.Context, cmd UpdateTableRowCommand) (*TableRow, error)
    Delete(ctx context.Context, id uuid.UUID) error
}

// Implementación base
type tableRowServiceImpl struct {
    uow UnitOfWork
}

func (s *tableRowServiceImpl) Get(ctx context.Context, id uuid.UUID) (*TableRow, error) {
    return s.uow.TableRows().FindByID(ctx, id)
}

// Otros métodos implementados...

// Decorador para logging
type loggingTableRowService struct {
    next   TableRowService
    logger *zap.Logger
}

func NewLoggingTableRowService(next TableRowService, logger *zap.Logger) TableRowService {
    return &loggingTableRowService{
        next:   next,
        logger: logger,
    }
}

func (s *loggingTableRowService) Get(ctx context.Context, id uuid.UUID) (*TableRow, error) {
    s.logger.Info("Obteniendo fila", zap.String("id", id.String()))
    
    result, err := s.next.Get(ctx, id)
    
    if err != nil {
        s.logger.Error("Error obteniendo fila", 
            zap.String("id", id.String()), 
            zap.Error(err))
    }
    
    return result, err
}

// Otros métodos implementados...

// Decorador para métricas
type metricTableRowService struct {
    next    TableRowService
    metrics metrics.Reporter
}

func NewMetricTableRowService(next TableRowService, metrics metrics.Reporter) TableRowService {
    return &metricTableRowService{
        next:    next,
        metrics: metrics,
    }
}

func (s *metricTableRowService) Get(ctx context.Context, id uuid.UUID) (*TableRow, error) {
    startTime := time.Now()
    
    result, err := s.next.Get(ctx, id)
    
    duration := time.Since(startTime)
    s.metrics.RecordServiceDuration("TableRowService", "Get", duration)
    
    if err != nil {
        s.metrics.RecordServiceError("TableRowService", "Get")
    } else {
        s.metrics.RecordServiceSuccess("TableRowService", "Get")
    }
    
    return result, err
}

// Otros métodos implementados...

// Decorador para trazas
type tracingTableRowService struct {
    next   TableRowService
    tracer trace.Tracer
}

func NewTracingTableRowService(next TableRowService, tracer trace.Tracer) TableRowService {
    return &tracingTableRowService{
        next:   next,
        tracer: tracer,
    }
}

func (s *tracingTableRowService) Get(ctx context.Context, id uuid.UUID) (*TableRow, error) {
    ctx, span := s.tracer.Start(ctx, "TableRowService.Get",
        trace.WithAttributes(attribute.String("row_id", id.String())))
    defer span.End()
    
    result, err := s.next.Get(ctx, id)
    
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, err.Error())
    } else {
        span.SetStatus(codes.Ok, "Success")
    }
    
    return result, err
}

// Otros métodos implementados...

// Decorador para circuit breaker
type circuitBreakerTableRowService struct {
    next    TableRowService
    breaker CircuitBreaker
}

func NewCircuitBreakerTableRowService(next TableRowService, breaker CircuitBreaker) TableRowService {
    return &circuitBreakerTableRowService{
        next:    next,
        breaker: breaker,
    }
}

func (s *circuitBreakerTableRowService) Get(ctx context.Context, id uuid.UUID) (*TableRow, error) {
    var result *TableRow
    var err error
    
    err = s.breaker.Execute(func() error {
        var execErr error
        result, execErr = s.next.Get(ctx, id)
        return execErr
    })
    
    return result, err
}

// Otros métodos implementados...

// Decorador para caché
type cachingTableRowService struct {
    next         TableRowService
    cacheManager CacheManager
    strategy     CacheStrategy
}

func NewCachingTableRowService(
    next TableRowService, 
    cacheManager CacheManager,
    strategy CacheStrategy,
) TableRowService {
    return &cachingTableRowService{
        next:         next,
        cacheManager: cacheManager,
        strategy:     strategy,
    }
}

func (s *cachingTableRowService) Get(ctx context.Context, id uuid.UUID) (*TableRow, error) {
    // Extraer tabla del contexto
    tableName := getTableNameFromContext(ctx)
    
    // Verificar si debemos cachear
    if !s.strategy.ShouldCache(tableName) {
        return s.next.Get(ctx, id)
    }
    
    // Construir clave de caché
    cacheKey := s.strategy.GetCacheKey(tableName, id)
    
    // Intentar obtener de caché
    var row *TableRow
    if s.cacheManager.Get(cacheKey, &row) {
        return row, nil
    }
    
    // Caché miss, obtener del siguiente servicio
    row, err := s.next.Get(ctx, id)
    if err != nil {
        return nil, err
    }
    
    // Guardar en caché
    s.cacheManager.Set(cacheKey, row, s.strategy.GetExpiration(tableName))
    
    return row, nil
}

// Función para crear servicio con decoradores
func NewTableRowService(
    uow UnitOfWork, 
    cacheManager CacheManager, 
    strategy CacheStrategy,
    logger *zap.Logger,
    metrics metrics.Reporter,
    tracer trace.Tracer,
) TableRowService {
    // Implementación base
    base := &tableRowServiceImpl{uow: uow}
    
    // Crear circuit breaker
    cb := gobreaker.NewCircuitBreaker(gobreaker.Settings{
        Name:        "tablerow-service",
        MaxRequests: 5,
        Interval:    30 * time.Second,
        Timeout:     60 * time.Second,
        ReadyToTrip: func(counts gobreaker.Counts) bool {
            failureRatio := float64(counts.TotalFailures) / float64(counts.Requests)
            return counts.Requests >= 5 && failureRatio >= 0.6
        },
        OnStateChange: func(name string, from gobreaker.State, to gobreaker.State) {
            metrics.RecordCircuitBreakerStateChange("tablerow-service", from.String(), to.String())
        },
    })
    
    // Aplicar decoradores
    service := base
    service = NewTracingTableRowService(service, tracer)
    service = NewMetricTableRowService(service, metrics)
    service = NewLoggingTableRowService(service, logger)
    service = NewCircuitBreakerTableRowService(service, NewGoBreaker(cb))
    service = NewCachingTableRowService(service, cacheManager, strategy)
    
    return service
}
```

### 9.5 Circuit Breaker Pattern

```go
// Interfaz para circuit breaker
type CircuitBreaker interface {
    Execute(func() error) error
    ExecuteWithFallback(func() error, func(error) error) error
    State() string
}

// Implementación con gobreaker
type GoBreakerAdapter struct {
    breaker *gobreaker.CircuitBreaker
}

func NewGoBreaker(breaker *gobreaker.CircuitBreaker) CircuitBreaker {
    return &GoBreakerAdapter{
        breaker: breaker,
    }
}

func (a *GoBreakerAdapter) Execute(fn func() error) error {
    _, err := a.breaker.Execute(func() (interface{}, error) {
        return nil, fn()
    })
    
    return err
}

func (a *GoBreakerAdapter) ExecuteWithFallback(fn func() error, fallback func(error) error) error {
    _, err := a.breaker.Execute(func() (interface{}, error) {
        return nil, fn()
    })
    
    if err != nil {
        return fallback(err)
    }
    
    return nil
}

func (a *GoBreakerAdapter) State() string {
    return a.breaker.State().String()
}
```

### 9.6 Bulkhead Pattern

```go
// Bulkhead para aislar recursos
type Bulkhead interface {
    Execute(func() error) error
    TryExecute(func() error) (bool, error)
    Available() int
}

// Implementación del patrón Bulkhead
type SimpleBulkhead struct {
    name        string
    semaphore   chan struct{}
    metrics     metrics.Reporter
    concurrency int
}

func NewBulkhead(name string, concurrency int, metrics metrics.Reporter) Bulkhead {
    return &SimpleBulkhead{
        name:        name,
        semaphore:   make(chan struct{}, concurrency),
        metrics:     metrics,
        concurrency: concurrency,
    }
}

func (b *SimpleBulkhead) Execute(fn func() error) error {
    // Adquirir un slot del semáforo
    b.semaphore <- struct{}{}
    defer func() {
        // Liberar slot al finalizar
        <-b.semaphore
    }()
    
    // Métricas
    available := b.Available()
    b.metrics.RecordBulkheadExecution(b.name, available)
    
    // Ejecutar función
    return fn()
}

func (b *SimpleBulkhead) TryExecute(fn func() error) (bool, error) {
    // Intentar adquirir un slot sin bloquear
    select {
    case b.semaphore <- struct{}{}:
        // Slot adquirido
        defer func() {
            // Liberar slot al finalizar
            <-b.semaphore
        }()
        
        // Métricas
        available := b.Available()
        b.metrics.RecordBulkheadExecution(b.name, available)
        
        // Ejecutar función
        return true, fn()
        
    default:
        // No hay slots disponibles
        b.metrics.RecordBulkheadRejection(b.name)
        return false, nil
    }
}

func (b *SimpleBulkhead) Available() int {
    return b.concurrency - len(b.semaphore)
}
```

### 9.7 Feature Flag Pattern

```go
// Interfaz para feature flags
type FeatureFlag interface {
    IsEnabled(ctx context.Context, feature string) bool
    IsEnabledForUser(ctx context.Context, feature string, userID string) bool
}

// Implementación básica
type SimpleFeatureFlag struct {
    features    map[string]bool
    userFlags   map[string]map[string]bool
    defaultFlag bool
    mu          sync.RWMutex
    metrics     metrics.Reporter
}

func NewSimpleFeatureFlag(defaultFlag bool, metrics metrics.Reporter) *SimpleFeatureFlag {
    return &SimpleFeatureFlag{
        features:    make(map[string]bool),
        userFlags:   make(map[string]map[string]bool),
        defaultFlag: defaultFlag,
        metrics:     metrics,
    }
}

func (f *SimpleFeatureFlag) IsEnabled(ctx context.Context, feature string) bool {
    f.mu.RLock()
    enabled, exists := f.features[feature]
    f.mu.RUnlock()
    
    if !exists {
        f.metrics.RecordFeatureFlagCheck(feature, "default", f.defaultFlag)
        return f.defaultFlag
    }
    
    f.metrics.RecordFeatureFlagCheck(feature, "global", enabled)
    return enabled
}

func (f *SimpleFeatureFlag) IsEnabledForUser(ctx context.Context, feature string, userID string) bool {
    // Primero verificar flag específico de usuario
    f.mu.RLock()
    userFeatures, exists := f.userFlags[userID]
    if exists {
        if enabled, hasFlag := userFeatures[feature]; hasFlag {
            f.mu.RUnlock()
            f.metrics.RecordFeatureFlagCheck(feature, "user", enabled)
            return enabled
        }
    }
    f.mu.RUnlock()
    
    // Si no hay flag específico, usar el global
    return f.IsEnabled(ctx, feature)
}

func (f *SimpleFeatureFlag) SetFeature(feature string, enabled bool) {
    f.mu.Lock()
    defer f.mu.Unlock()
    
    f.features[feature] = enabled
}

func (f *SimpleFeatureFlag) SetFeatureForUser(feature string, userID string, enabled bool) {
    f.mu.Lock()
    defer f.mu.Unlock()
    
    if _, exists := f.userFlags[userID]; !exists {
        f.userFlags[userID] = make(map[string]bool)
    }
    
    f.userFlags[userID][feature] = enabled
}
```

### 9.8 Mediator Pattern

```go
// Interfaz para el mediador
type Mediator interface {
    Send(ctx context.Context, request interface{}) (interface{}, error)
    Register(requestType reflect.Type, handler RequestHandler)
}

// Manejador de solicitudes
type RequestHandler func(ctx context.Context, request interface{}) (interface{}, error)

// Implementación del mediador
type SimpleMediator struct {
    handlers map[reflect.Type]RequestHandler
    mu       sync.RWMutex
    logger   *zap.Logger
    metrics  metrics.Reporter
    tracer   trace.Tracer
}

func NewMediator(logger *zap.Logger, metrics metrics.Reporter, tracer trace.Tracer) Mediator {
    return &SimpleMediator{
        handlers: make(map[reflect.Type]RequestHandler),
        logger:   logger,
        metrics:  metrics,
        tracer:   tracer,
    }
}

func (m *SimpleMediator) Register(requestType reflect.Type, handler RequestHandler) {
    m.mu.Lock()
    defer m.mu.Unlock()
    
    m.handlers[requestType] = handler
}

func (m *SimpleMediator) Send(ctx context.Context, request interface{}) (interface{}, error) {
    requestType := reflect.TypeOf(request)
    requestName := requestType.String()
    
    ctx, span := m.tracer.Start(ctx, "Mediator.Send",
        trace.WithAttributes(attribute.String("request.type", requestName)))
    defer span.End()
    
    // Buscar handler para este tipo de solicitud
    m.mu.RLock()
    handler, exists := m.handlers[requestType]
    m.mu.RUnlock()
    
    if !exists {
        err := fmt.Errorf("no hay manejador registrado para %s", requestName)
        span.RecordError(err)
        span.SetStatus(codes.Error, "No handler registered")
        m.metrics.RecordMediatorError(requestName, "no_handler")
        return nil, err
    }
    
    // Métricas
    start := time.Now()
    
    // Ejecutar handler
    result, err := handler(ctx, request)
    
    // Registrar resultado
    duration := time.Since(start)
    m.metrics.RecordMediatorDuration(requestName, duration)
    
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, err.Error())
        m.metrics.RecordMediatorError(requestName, "handler_error")
        m.logger.Error("Error procesando solicitud",
            zap.String("requestType", requestName),
            zap.Error(err))
        return nil, err
    }
    
    m.metrics.RecordMediatorSuccess(requestName)
    span.SetStatus(codes.Ok, "Success")
    return result, nil
}
```

### 9.9 Circuit Breaker para Caché

```go
// Interfaz para Circuit Breaker específico de caché
type CacheCircuitBreaker interface {
    Execute(key string, operation func() (interface{}, error), fallback func() (interface{}, error)) (interface{}, error)
    State(key string) string
    Reset(key string)
}

// Implementación de Circuit Breaker para caché con granularidad por clave
type CacheCircuitBreakerImpl struct {
    breakers    map[string]*gobreaker.CircuitBreaker
    mu          sync.RWMutex
    metrics     metrics.Reporter
    logger      *zap.Logger
    settings    CacheCircuitBreakerSettings
}

type CacheCircuitBreakerSettings struct {
    Timeout             time.Duration
    MaxRequests         uint32
    Interval            time.Duration
    FailureThreshold    float64
    FallbackTimeout     time.Duration
}

func NewCacheCircuitBreaker(
    settings CacheCircuitBreakerSettings,
    metrics metrics.Reporter,
    logger *zap.Logger,
) CacheCircuitBreaker {
    return &CacheCircuitBreakerImpl{
        breakers: make(map[string]*gobreaker.CircuitBreaker),
        metrics:  metrics,
        logger:   logger,
        settings: settings,
    }
}

func (cb *CacheCircuitBreakerImpl) getBreakerForKey(key string) *gobreaker.CircuitBreaker {
    cb.mu.RLock()
    breaker, exists := cb.breakers[key]
    cb.mu.RUnlock()
    
    if exists {
        return breaker
    }
    
    // Si no existe, crear un nuevo breaker para esta clave
    cb.mu.Lock()
    defer cb.mu.Unlock()
    
    // Verificar de nuevo por si otro goroutine lo creó mientras esperábamos el lock
    breaker, exists = cb.breakers[key]
    if exists {
        return breaker
    }
    
    breaker = gobreaker.NewCircuitBreaker(gobreaker.Settings{
        Name:        fmt.Sprintf("cache-breaker-%s", key),
        MaxRequests: cb.settings.MaxRequests,
        Interval:    cb.settings.Interval,
        Timeout:     cb.settings.Timeout,
        ReadyToTrip: func(counts gobreaker.Counts) bool {
            failureRatio := float64(counts.TotalFailures) / float64(counts.Requests)
            return counts.Requests >= 5 && failureRatio >= cb.settings.FailureThreshold
        },
        OnStateChange: func(name string, from gobreaker.State, to gobreaker.State) {
            cb.metrics.RecordCircuitBreakerStateChange("cache", from.String(), to.String())
            cb.logger.Info("Cambio de estado en circuit breaker de caché",
                zap.String("key", key),
                zap.String("from", from.String()),
                zap.String("to", to.String()),
            )
        },
    })
    
    cb.breakers[key] = breaker
    return breaker
}

func (cb *CacheCircuitBreakerImpl) Execute(
    key string,
    operation func() (interface{}, error),
    fallback func() (interface{}, error),
) (interface{}, error) {
    breaker := cb.getBreakerForKey(key)
    
    // Intentar ejecución protegida por circuit breaker
    result, err := breaker.Execute(operation)
    
    // Si el circuit breaker está abierto o hay error, usar fallback
    if err != nil {
        cb.metrics.RecordCacheError("circuit_breaker_fallback")
        
        // Configurar timeout para fallback
        ctx, cancel := context.WithTimeout(context.Background(), cb.settings.FallbackTimeout)
        defer cancel()
        
        // Canal para recibir resultado de fallback
        resultCh := make(chan struct {
            value interface{}
            err   error
        }, 1)
        
        go func() {
            val, err := fallback()
            resultCh <- struct {
                value interface{}
                err   error
            }{val, err}
        }()
        
        // Esperar resultado o timeout
        select {
        case res := <-resultCh:
            if res.err != nil {
                cb.metrics.RecordCacheError("fallback_failed")
                return nil, fmt.Errorf("fallo en operación principal y fallback: %w", res.err)
            }
            cb.metrics.RecordCacheSuccess("fallback")
            return res.value, nil
            
        case <-ctx.Done():
            cb.metrics.RecordCacheError("fallback_timeout")
            return nil, fmt.Errorf("timeout en fallback: %w", ctx.Err())
        }
    }
    
    return result, nil
}

func (cb *CacheCircuitBreakerImpl) State(key string) string {
    cb.mu.RLock()
    breaker, exists := cb.breakers[key]
    cb.mu.RUnlock()
    
    if !exists {
        return "not_initialized"
    }
    
    return breaker.State().String()
}

func (cb *CacheCircuitBreakerImpl) Reset(key string) {
    cb.mu.Lock()
    delete(cb.breakers, key)
    cb.mu.Unlock()
}
```

Este patrón de Circuit Breaker específico para caché proporciona:

1. **Degradación elegante**: Cuando Redis no está disponible o responde lentamente, el sistema automáticamente utiliza fallbacks.
2. **Granularidad por clave**: Permite aislar problemas en conjuntos específicos de datos sin afectar a todo el sistema.
3. **Recuperación automática**: Tras un período de enfriamiento, el sistema intenta reconectar a Redis.
4. **Observabilidad**: Registro detallado de cambios de estado y métricas para análisis.
5. **Protección contra cascada**: Evita que problemas en Redis impacten otros servicios o componentes.
```

## 10. API y Contratos

### 10.1 API RESTful para Datos EAV

```go
// Ejemplo de definición de rutas para el servicio de datos
func RegisterDataRoutes(router chi.Router, handler DataHandler) {
    router.Route("/data/{tableName}", func(r chi.Router) {
        // Operaciones CRUD estándar
        r.Get("/{rowID}", handler.GetRow)
        r.Post("/", handler.CreateRow)
        r.Patch("/{rowID}", handler.UpdateRow)
        r.Delete("/{rowID}", handler.DeleteRow)
        
        // Operaciones avanzadas
        r.Post("/query", handler.QueryRows)
        r.Post("/batch", handler.BatchOperations)
    })
}

// Ejemplo de handler con trazas, métricas y gestión de errores
type DataHandler struct {
    tableRowService TableRowService
    metadataService MetadataService
    uow             UnitOfWork
    logger          *zap.Logger
    metrics         metrics.Reporter
    tracer          trace.Tracer
    featureFlags    FeatureFlag
}

func (h *DataHandler) GetRow(w http.ResponseWriter, r *http.Request) {
    // Inicio de span para trazabilidad
    ctx, span := h.tracer.Start(r.Context(), "DataHandler.GetRow")
    defer span.End()
    
    // Extraer parámetros
    tableName := chi.URLParam(r, "tableName")
    rowIDStr := chi.URLParam(r, "rowID")
    
    span.SetAttributes(
        attribute.String("table_name", tableName),
        attribute.String("row_id", rowIDStr),
    )
    
    rowID, err := uuid.Parse(rowIDStr)
    if err != nil {
        RespondWithError(w, http.StatusBadRequest, "ID inválido", err)
        span.RecordError(err)
        span.SetStatus(codes.Error, "Invalid ID format")
        h.metrics.RecordHttpError("GetRow", http.StatusBadRequest)
        return
    }
    
    // Obtener tabla lógica para verificar permisos
    table, err := h.metadataService.GetLogicalTableByName(ctx, tableName)
    if err != nil {
        if errors.Is(err, ErrTableNotFound) {
            RespondWithError(w, http.StatusNotFound, "Tabla no encontrada", err)
            span.SetStatus(codes.Error, "Table not found")
        } else {
            RespondWithError(w, http.StatusInternalServerError, "Error obteniendo tabla", err)
            span.RecordError(err)
            span.SetStatus(codes.Error, "Error getting table")
        }
        h.metrics.RecordHttpError("GetRow", http.StatusInternalServerError)
        return
    }
    
    // Verificar permisos (omitido para brevedad)
    
    // Crear contexto con información de tabla
    ctx = context.WithValue(ctx, contextKeyTableName, tableName)
    ctx = context.WithValue(ctx, contextKeyTableID, table.ID)
    
    // Verificar feature flag para comportamiento extendido
    useExtendedBehavior := h.featureFlags.IsEnabled(ctx, "extended_row_data")
    if useExtendedBehavior {
        span.SetAttributes(attribute.Bool("feature.extended_row_data", true))
    }
    
    // Obtener fila con bulkhead para proteger recursos
    bulkhead := NewBulkhead("row-retrieval", 50, h.metrics)
    var row *TableRow
    
    err = bulkhead.Execute(func() error {
        var err error
        row, err = h.tableRowService.Get(ctx, rowID)
        return err
    })
    
    if err != nil {
        if errors.Is(err, ErrNotFound) {
            RespondWithError(w, http.StatusNotFound, "Fila no encontrada", nil)
            span.SetStatus(codes.Error, "Row not found")
            h.metrics.RecordHttpError("GetRow", http.StatusNotFound)
        } else {
            RespondWithError(w, http.StatusInternalServerError, "Error obteniendo fila", err)
            span.RecordError(err)
            span.SetStatus(codes.Error, "Error getting row")
            h.metrics.RecordHttpError("GetRow", http.StatusInternalServerError)
        }
        return
    }
    
// Convertir a DTO
    dto := ConvertToDTO(row, table)
    
    // Aplicar comportamiento extendido si está habilitado
    if useExtendedBehavior {
        if err := h.enrichRowData(ctx, &dto); err != nil {
            h.logger.Warn("Error enriqueciendo datos de fila", zap.Error(err))
        }
    }
    
    // Responder exitosamente
    RespondWithJSON(w, http.StatusOK, dto)
    span.SetStatus(codes.Ok, "Success")
    h.metrics.RecordHttpSuccess("GetRow")
}

// Implementaciones de otros handlers omitidas para brevedad...
```

### 10.2 API para Autenticación (Passkeys y Contraseña)

```go
// Ejemplo de definición de rutas para el servicio de autenticación
func RegisterAuthRoutes(router chi.Router, handler AuthHandler) {
    // Rutas de passkeys
    router.Route("/auth/passkey", func(r chi.Router) {
        r.Post("/register/start", handler.StartPasskeyRegistration)
        r.Post("/register/finish", handler.FinishPasskeyRegistration)
        r.Post("/authenticate/start", handler.StartPasskeyAuthentication)
        r.Post("/authenticate/finish", handler.FinishPasskeyAuthentication)
    })
    
    // Rutas de sesión
    router.Route("/auth/session", func(r chi.Router) {
        r.Get("/", AuthMiddleware(handler.GetCurrentSession))
        r.Delete("/", AuthMiddleware(handler.RevokeSession))
        r.Get("/list", AuthMiddleware(handler.ListUserSessions))
    })

    // Rutas de gestión de passkeys
    router.Route("/auth/passkeys", func(r chi.Router) {
        r.Get("/", AuthMiddleware(handler.ListUserPasskeys))
        r.Delete("/{passkeyID}", AuthMiddleware(handler.RevokePasskey))
    })
}
```

#### 10.2.1 Definición de Contratos para API de Passkeys

Los contratos de la API están claramente definidos para garantizar la coherencia entre el cliente y el servidor:

```go
// Solicitud para iniciar registro de passkey
type StartPasskeyRegistrationRequest struct {
    Username string `json:"username" validate:"required,email"`
}

// Respuesta al iniciar registro de passkey
type StartPasskeyRegistrationResponse struct {
    Challenge   string                       `json:"challenge"`
    Options     webauthn.PublicKeyCredentialCreationOptions `json:"options"`
    SessionData string                       `json:"sessionData"`
}

// Solicitud para finalizar registro de passkey
type FinishPasskeyRegistrationRequest struct {
    Username    string          `json:"username" validate:"required,email"`
    Attestation json.RawMessage `json:"attestation" validate:"required"`
    SessionData string          `json:"sessionData" validate:"required"`
}

// Respuesta al finalizar registro de passkey
type FinishPasskeyRegistrationResponse struct {
    Registered bool   `json:"registered"`
    UserID     string `json:"userId"`
    PasskeyID  string `json:"passkeyId"`
    DeviceInfo struct {
        Name     string `json:"name"`
        Platform string `json:"platform"`
    } `json:"deviceInfo"`
}

// Solicitud para iniciar autenticación con passkey
type StartPasskeyAuthenticationRequest struct {
    Username string `json:"username" validate:"required,email"`
}

// Respuesta al iniciar autenticación con passkey
type StartPasskeyAuthenticationResponse struct {
    Challenge   string                        `json:"challenge"`
    Options     webauthn.PublicKeyCredentialRequestOptions `json:"options"`
    SessionData string                        `json:"sessionData"`
}

// Solicitud para finalizar autenticación con passkey
type FinishPasskeyAuthenticationRequest struct {
    Username   string          `json:"username" validate:"required,email"`
    Assertion  json.RawMessage `json:"assertion" validate:"required"`
    SessionData string         `json:"sessionData" validate:"required"`
}

// Respuesta al finalizar autenticación con passkey
type FinishPasskeyAuthenticationResponse struct {
    Authenticated bool   `json:"authenticated"`
    UserID        string `json:"userId"`
    Username      string `json:"username"`
    SessionToken  string `json:"sessionToken"`
    ExpiresAt     string `json:"expiresAt"`
}

// Endpoint para autenticación tradicional
type LoginRequest struct {
    Username string `json:"username" validate:"required"`
    Password string `json:"password" validate:"required"`
}

type LoginResponse struct {
    Authenticated bool   `json:"authenticated"`
    UserID        string `json:"userId"`
    Username      string `json:"username"`
    SessionToken  string `json:"sessionToken"`
    ExpiresAt     string `json:"expiresAt"`
}

```

### 10.3 Documentación de API con OpenAPI/Swagger

La API está documentada utilizando OpenAPI 3.0, lo que facilita la comprensión y el uso por parte de los clientes.

```go
// Configuración de Swagger
func SetupSwagger(router chi.Router) {
    swaggerUI := http.FileServer(http.Dir("./swagger-ui"))
    router.Handle("/swagger/*", http.StripPrefix("/swagger", swaggerUI))
    
    // Servir archivo de especificación OpenAPI
    router.Get("/api/spec", func(w http.ResponseWriter, r *http.Request) {
        http.ServeFile(w, r, "./api/openapi.yaml")
    })
}

// Implementación de tags de documentación en handlers
// @Summary Obtiene una fila por ID
// @Description Obtiene una fila específica por su ID de una tabla lógica
// @Tags datos
// @Accept json
// @Produce json
// @Param tableName path string true "Nombre de la tabla lógica"
// @Param rowID path string true "ID de la fila a obtener"
// @Success 200 {object} TableRowDTO
// @Failure 400 {object} ErrorResponse
// @Failure 404 {object} ErrorResponse
// @Failure 500 {object} ErrorResponse
// @Router /data/{tableName}/{rowID} [get]
func (h *DataHandler) GetRow(w http.ResponseWriter, r *http.Request) {
    // Implementación (como se mostró anteriormente)
}
```

### 10.4 Gestión de Versiones de API

```go
// Configuración de versiones de API
func SetupAPIVersioning(router chi.Router) {
    // API v1 (actual)
    router.Route("/api/v1", func(r chi.Router) {
        RegisterDataRoutes(r, dataHandlerV1)
        RegisterAuthRoutes(r, authHandlerV1)
        RegisterMetadataRoutes(r, metadataHandlerV1)
    })
    
    // API v2 (experimental)
    router.Route("/api/v2", func(r chi.Router) {
        RegisterDataRoutesV2(r, dataHandlerV2)
        RegisterAuthRoutesV2(r, authHandlerV2)
        RegisterMetadataRoutesV2(r, metadataHandlerV2)
    })
    
    // Redirección de versión por defecto a v1
    router.Get("/api", func(w http.ResponseWriter, r *http.Request) {
        http.Redirect(w, r, "/api/v1", http.StatusMovedPermanently)
    })
}

// Middleware para comprobaciones de versión
func VersionHeaderMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        version := r.Header.Get("API-Version")
        
        if version != "" {
            // Adición de información de versión al contexto
            ctx := context.WithValue(r.Context(), contextKeyAPIVersion, version)
            r = r.WithContext(ctx)
        }
        
        next.ServeHTTP(w, r)
    })
}
```

## 11. Escalabilidad y Rendimiento

### 11.1 Estrategias de Caché Multinivel

Para optimizar el rendimiento, se implementa un sistema de caché multinivel con invalidación coordinada:

```go
// Interfaz de caché
type CacheManager interface {
    Get(key string, dest interface{}) bool
    Set(key string, value interface{}, expiration time.Duration) error
    Delete(key string) error
    DeleteByPattern(pattern string) error
}

// Implementación de caché multinivel
type MultiLevelCacheManager struct {
    localCache  *ristretto.Cache
    redisClient *redis.Client
    logger      *zap.Logger
    metrics     metrics.Reporter
    tracer      trace.Tracer
	breaker     CacheCircuitBreaker  // Agregar esta línea
}

	func NewMultiLevelCacheManager(
		localCacheSize int64, 
		redisClient *redis.Client, 
		logger *zap.Logger,
		metrics metrics.Reporter,
		tracer trace.Tracer,
	) (CacheManager, error) {
		// Configurar caché local (Ristretto)
		localCache, err := ristretto.NewCache(&ristretto.Config{
			NumCounters: localCacheSize * 10, // Recomienda 10x el tamaño del caché
			MaxCost:     localCacheSize,
			BufferItems: 64,
		})
		
		if err != nil {
			return nil, fmt.Errorf("error inicializando caché local: %w", err)
		}
		
		// Configurar circuit breaker para caché
		cacheBreaker := NewCacheCircuitBreaker(
			CacheCircuitBreakerSettings{
				Timeout:          5 * time.Second,
				MaxRequests:      50,
				Interval:         30 * time.Second,
				FailureThreshold: 0.5,
				FallbackTimeout:  100 * time.Millisecond,
			},
			metrics,
			logger,
		)
		
		return &MultiLevelCacheManager{
			localCache:  localCache,
			redisClient: redisClient,
			logger:      logger,
			metrics:     metrics,
			tracer:      tracer,
			breaker:     cacheBreaker,  // Agregar esta línea
		}, nil
	}


func (c *MultiLevelCacheManager) Get(key string, dest interface{}) bool {
    ctx, span := c.tracer.Start(context.Background(), "CacheManager.Get", 
        trace.WithAttributes(attribute.String("cache.key", key)))
    defer span.End()
    
    // Métricas
    c.metrics.RecordCacheOperation("get", key)
    
    // Implementar con Circuit Breaker
    result, err := c.breaker.Execute(
        key,
        func() (interface{}, error) {
            // Intentar obtener de caché local primero
            if value, found := c.localCache.Get(key); found {
                span.SetAttributes(attribute.Bool("cache.level", true))
                c.metrics.RecordCacheHit("local")
                
                if err := unmarshalCacheValue(value, dest); err != nil {
                    span.RecordError(err)
                    c.logger.Warn("Error unmarshalling valor de caché local", 
                        zap.String("key", key), 
                        zap.Error(err))
                    return nil, err
                }
                
                return true, nil
            }
            
            // Si no está en caché local, intentar Redis
            ctx, cancel := context.WithTimeout(context.Background(), 500*time.Millisecond)
            defer cancel()
            
            value, err := c.redisClient.Get(ctx, key).Bytes()
            if err != nil {
                if err != redis.Nil {
                    span.RecordError(err)
                    c.logger.Warn("Error obteniendo clave de Redis", 
                        zap.String("key", key), 
                        zap.Error(err))
                    c.metrics.RecordCacheError("redis_get")
                    return nil, err
                } else {
                    c.metrics.RecordCacheMiss("redis")
                    return false, nil
                }
            }
            
            span.SetAttributes(attribute.Bool("cache.level", false))
            c.metrics.RecordCacheHit("redis")
            
            // Deserializar valor
            if err := unmarshalCacheValue(value, dest); err != nil {
                span.RecordError(err)
                c.logger.Warn("Error unmarshalling valor de Redis", 
                    zap.String("key", key), 
                    zap.Error(err))
                return nil, err
            }
            
            // Promover a caché local
            c.localCache.Set(key, value, 1)
            
            return true, nil
        },
        func() (interface{}, error) {
            // Fallback cuando Redis no está disponible:
            // 1. Devolver false (cache miss) - obligará a ir a base de datos
            // 2. No propagar el error - protege sistema contra fallos de caché
            c.logger.Info("Usando fallback para Get de caché", 
                zap.String("key", key))
            c.metrics.RecordCacheFallback("get")
            return false, nil
        },
    )
    
    if err != nil {
        // Si hay error incluso después de fallback, reportar cache miss
        return false
    }
    
    return result.(bool)
}

func (c *MultiLevelCacheManager) Set(key string, value interface{}, expiration time.Duration) error {
    ctx, span := c.tracer.Start(context.Background(), "CacheManager.Set", 
        trace.WithAttributes(
            attribute.String("cache.key", key),
            attribute.Int64("cache.ttl_ms", expiration.Milliseconds()),
        ))
    defer span.End()
    
    // Métricas
    c.metrics.RecordCacheOperation("set", key)
    
    // Serializar valor
    serialized, err := json.Marshal(value)
    if err != nil {
        span.RecordError(err)
        c.metrics.RecordCacheError("serialize")
        return fmt.Errorf("error serializando valor para caché: %w", err)
    }
    
    // Guardar en caché local
    c.localCache.SetWithTTL(key, serialized, 1, expiration)
    
    // Guardar en Redis con Circuit Breaker
    cb := gobreaker.NewCircuitBreaker(gobreaker.Settings{
        Name:    "redis-set",
        Timeout: 5 * time.Second,
    })
    
    _, err = cb.Execute(func() (interface{}, error) {
        ctx, cancel := context.WithTimeout(context.Background(), 500*time.Millisecond)
        defer cancel()
        return nil, c.redisClient.Set(ctx, key, serialized, expiration).Err()
    })
    
    if err != nil {
        span.RecordError(err)
        c.metrics.RecordCacheError("redis_set")
        c.logger.Warn("Error guardando en Redis", 
            zap.String("key", key), 
            zap.Error(err))
        return fmt.Errorf("error guardando en Redis: %w", err)
    }
    
    return nil
}

func (c *MultiLevelCacheManager) Delete(key string) error {
    ctx, span := c.tracer.Start(context.Background(), "CacheManager.Delete", 
        trace.WithAttributes(attribute.String("cache.key", key)))
    defer span.End()
    
    // Métricas
    c.metrics.RecordCacheOperation("delete", key)
    
    // Eliminar de caché local
    c.localCache.Del(key)
    
    // Eliminar de Redis con Circuit Breaker
    cb := gobreaker.NewCircuitBreaker(gobreaker.Settings{
        Name:    "redis-del",
        Timeout: 5 * time.Second,
    })
    
    _, err := cb.Execute(func() (interface{}, error) {
        ctx, cancel := context.WithTimeout(context.Background(), 500*time.Millisecond)
        defer cancel()
        return nil, c.redisClient.Del(ctx, key).Err()
    })
    
    if err != nil {
        span.RecordError(err)
        c.metrics.RecordCacheError("redis_delete")
        c.logger.Warn("Error eliminando clave de Redis", 
            zap.String("key", key), 
            zap.Error(err))
        return fmt.Errorf("error eliminando de Redis: %w", err)
    }
    
    return nil
}

func (c *MultiLevelCacheManager) DeleteByPattern(pattern string) error {
    ctx, span := c.tracer.Start(context.Background(), "CacheManager.DeleteByPattern", 
        trace.WithAttributes(attribute.String("cache.pattern", pattern)))
    defer span.End()
    
    // Métricas
    c.metrics.RecordCacheOperation("delete_pattern", pattern)
    
    // Redis soporta eliminación por patrón (SCAN + DEL)
    ctx = context.Background()
    
    // Usar SCAN para obtener claves que coincidan con el patrón
    iter := c.redisClient.Scan(ctx, 0, pattern, 100).Iterator()
    
    var keys []string
    for iter.Next(ctx) {
        key := iter.Val()
        keys = append(keys, key)
        
        // Eliminar también de caché local
        c.localCache.Del(key)
    }
    
    if err := iter.Err(); err != nil {
        span.RecordError(err)
        c.metrics.RecordCacheError("redis_scan")
        c.logger.Warn("Error escaneando claves en Redis", 
            zap.String("pattern", pattern), 
            zap.Error(err))
        return fmt.Errorf("error escaneando claves en Redis: %w", err)
    }
    
    // Eliminar claves en lotes usando pipeline
    if len(keys) > 0 {
        // Usar Circuit Breaker
        cb := gobreaker.NewCircuitBreaker(gobreaker.Settings{
            Name:    "redis-multi-del",
            Timeout: 10 * time.Second,
        })
        
        _, err := cb.Execute(func() (interface{}, error) {
            pipe := c.redisClient.Pipeline()
            for _, key := range keys {
                pipe.Del(ctx, key)
            }
            return pipe.Exec(ctx)
        })
        
        if err != nil {
            span.RecordError(err)
            c.metrics.RecordCacheError("redis_multi_delete")
            c.logger.Warn("Error eliminando claves de Redis", 
                zap.Int("count", len(keys)), 
                zap.Error(err))
            return fmt.Errorf("error eliminando claves de Redis: %w", err)
        }
        
        span.SetAttributes(attribute.Int("keys.deleted", len(keys)))
    }
    
    return nil
}

// Función para deserializar valores de caché
func unmarshalCacheValue(data interface{}, dest interface{}) error {
    var bytes []byte
    
    switch v := data.(type) {
    case []byte:
        bytes = v
    case string:
        bytes = []byte(v)
    default:
        // Intentar serializar si no es []byte ni string
        var err error
        bytes, err = json.Marshal(data)
        if err != nil {
            return err
        }
    }
    
    if err := json.Unmarshal(bytes, dest); err != nil {
        return err
    }
    
    return nil
}

// Servicio de escucha para invalidación coordinada de caché
type CacheInvalidationListener struct {
    eventBus      EventBus
    cacheManager  CacheManager
    logger        *zap.Logger
    metrics       metrics.Reporter
    tracer        trace.Tracer
}

func (l *CacheInvalidationListener) Start(ctx context.Context) error {
    // Suscribirse a eventos de invalidación de caché
    return l.eventBus.Subscribe("CacheInvalidated", func(ctx context.Context, event interface{}) error {
        ctx, span := l.tracer.Start(ctx, "CacheInvalidationListener.Handle")
        defer span.End()
        
        // Convertir evento
        invalidationEvent, ok := event.(CacheInvalidationEvent)
        if !ok {
            err := errors.New("tipo de evento incorrecto")
            span.RecordError(err)
            l.metrics.RecordCacheError("invalid_event_type")
            return err
        }
        
        span.SetAttributes(
            attribute.String("table_name", invalidationEvent.TableName),
            attribute.String("pattern", invalidationEvent.Pattern),
        )
        
        // Invalidar caché utilizando el patrón
        if err := l.cacheManager.DeleteByPattern(invalidationEvent.Pattern); err != nil {
            span.RecordError(err)
            l.metrics.RecordCacheError("invalidation_failed")
            l.logger.Error("Error invalidando caché",
                zap.String("pattern", invalidationEvent.Pattern),
                zap.Error(err))
            return err
        }
        
        l.metrics.RecordCacheInvalidation(invalidationEvent.TableName)
        l.logger.Info("Caché invalidada",
            zap.String("table", invalidationEvent.TableName),
            zap.Any("entity_id", invalidationEvent.EntityID),
            zap.String("pattern", invalidationEvent.Pattern))
        
        return nil
    })
}
```

### 11.2 Optimizaciones para PostgreSQL

```go
// Configuración de la conexión a PostgreSQL optimizada
func SetupPostgreSQL(connStr string, metrics metrics.Reporter) (*sqlx.DB, error) {
    db, err := sqlx.Connect("postgres", connStr)
    if err != nil {
        return nil, fmt.Errorf("error conectando a PostgreSQL: %w", err)
    }
    
    // Configurar pool de conexiones
    db.SetMaxOpenConns(50)                // Máximo de conexiones abiertas
    db.SetMaxIdleConns(10)                // Máximo de conexiones inactivas
    db.SetConnMaxLifetime(30 * time.Minute) // Tiempo máximo de vida de una conexión
    db.SetConnMaxIdleTime(5 * time.Minute)  // Tiempo máximo de inactividad
    
    // Monitoreo continuo de estado de conexiones
    go monitorDBPool(db, metrics)
    
    return db, nil
}

// Monitoreo del pool de conexiones
func monitorDBPool(db *sqlx.DB, metrics metrics.Reporter) {
    ticker := time.NewTicker(15 * time.Second)
    defer ticker.Stop()
    
    for range ticker.C {
        stats := db.Stats()
        
        metrics.RecordDBPoolStats("open_connections", stats.OpenConnections)
        metrics.RecordDBPoolStats("in_use", stats.InUse)
        metrics.RecordDBPoolStats("idle", stats.Idle)
        metrics.RecordDBPoolStats("wait_count", stats.WaitCount)
        metrics.RecordDBPoolStats("wait_duration_ms", stats.WaitDuration.Milliseconds())
        metrics.RecordDBPoolStats("max_idle_closed", stats.MaxIdleClosed)
        metrics.RecordDBPoolStats("max_lifetime_closed", stats.MaxLifetimeClosed)
    }
}

// Consulta optimizada EAV usando índices adecuados
func (r *TableRowRepository) QueryOptimized(
    ctx context.Context, 
    tableID uuid.UUID, 
    filters []FilterCriteria,
    sorting []SortCriteria,
    pageSize int,
    keySet interface{},
) ([]*TableRow, interface{}, error) {
    ctx, span := r.tracer.Start(ctx, "TableRowRepository.QueryOptimized",
        trace.WithAttributes(
            attribute.String("table_id", tableID.String()),
            attribute.Int("filters_count", len(filters)),
            attribute.Int("sorting_count", len(sorting)),
            attribute.Int("page_size", pageSize),
        ))
    defer span.End()
    
    r.metrics.RecordRepositoryOperation("TableRow", "query_optimized")
    
    // Consulta primero índices para obtener IDs que cumplen los filtros
    // Esto evita los JOINs múltiples en la primera fase
    
    // Fase 1: Filtrado usando índices
    idsQuery, idsParams, err := r.buildFilteredIDsQuery(tableID, filters)
    if err != nil {
        span.RecordError(err)
        r.metrics.RecordRepositoryError("TableRow", "build_filtered_query")
        return nil, nil, fmt.Errorf("error construyendo consulta filtrada: %w", err)
    }
    
    span.AddEvent("Phase 1: Querying filtered IDs")
    
    // Ejecutar consulta de IDs
    tx := getTxFromContext(ctx, r.db)
    var rowIDs []uuid.UUID
    
    if err := tx.SelectContext(ctx, &rowIDs, idsQuery, idsParams...); err != nil {
        span.RecordError(err)
        r.metrics.RecordRepositoryError("TableRow", "query_ids")
        return nil, nil, fmt.Errorf("error consultando IDs: %w", err)
    }
    
    if len(rowIDs) == 0 {
        // No hay resultados, devolver lista vacía
        return []*TableRow{}, nil, nil
    }
    
    span.SetAttributes(attribute.Int("filtered_ids_count", len(rowIDs)))
    
    // Fase 2: Obtener datos completos de las filas filtradas
    span.AddEvent("Phase 2: Loading full rows")
    
    // Construir consulta IN para IDs
    placeholders := make([]string, len(rowIDs))
    params := make([]interface{}, len(rowIDs))
    for i, id := range rowIDs {
        placeholders[i] = fmt.Sprintf("$%d", i+1)
        params[i] = id
    }
    
    // Consulta para obtener filas
    rowsQuery := fmt.Sprintf(`
        SELECT id, logical_table_id, code, display_value, sort_order, is_active, parent_row_id,
               created_at, created_by_user_id, updated_at, updated_by_user_id, version
        FROM table_rows
        WHERE id IN (%s)
        ORDER BY %s
        LIMIT %d
    `, strings.Join(placeholders, ","), buildOrderByClause(sorting), pageSize)
    
    var rows []*TableRow
    if err := tx.SelectContext(ctx, &rows, rowsQuery, params...); err != nil {
        span.RecordError(err)
        r.metrics.RecordRepositoryError("TableRow", "query_rows")
        return nil, nil, fmt.Errorf("error consultando filas: %w", err)
    }
    
    // Fase 3: Cargar valores de columnas para las filas
    span.AddEvent("Phase 3: Loading column values")
    
    for _, row := range rows {
        if err := r.loadColumnValues(ctx, row); err != nil {
            span.RecordError(err)
            return nil, nil, err
        }
    }
    
    // Calcular siguiente keyset para paginación
    var nextKeySet interface{}
    if len(rows) == pageSize {
        nextKeySet = buildNextKeySet(rows[len(rows)-1], sorting)
    }
    
    span.SetStatus(codes.Ok, "Query successful")
    return rows, nextKeySet, nil
}
```

### 11.3 Índices Optimizados para EAV

```sql
-- Índices para table_rows (campos promovidos)
CREATE INDEX idx_table_rows_logical_table ON table_rows(logical_table_id);
CREATE INDEX idx_table_rows_code ON table_rows(logical_table_id, code) WHERE code IS NOT NULL;
CREATE INDEX idx_table_rows_active ON table_rows(logical_table_id, is_active);
CREATE INDEX idx_table_rows_sort ON table_rows(logical_table_id, sort_order);
CREATE INDEX idx_table_rows_parent ON table_rows(parent_row_id) WHERE parent_row_id IS NOT NULL;

-- Índices para JSONB
CREATE INDEX idx_table_rows_jsonb ON table_rows USING GIN (extended_properties);
CREATE INDEX idx_table_rows_jsonb_path_ops ON table_rows USING GIN (extended_properties jsonb_path_ops);

-- Índices para valores específicos en JSONB (ejemplo para campos comúnmente consultados)
CREATE INDEX idx_table_rows_jsonb_status ON table_rows ((extended_properties->>'status'));
CREATE INDEX idx_table_rows_jsonb_category ON table_rows ((extended_properties->>'category'));

-- Índices para valores de columnas (por tipo)
CREATE INDEX idx_column_values_row ON column_values(row_id);
CREATE INDEX idx_column_values_definition ON column_values(column_definition_id);
CREATE INDEX idx_column_values_row_definition ON column_values(row_id, column_definition_id);

-- Índices para búsqueda por valor según tipo
CREATE INDEX idx_column_values_string ON column_values USING btree(column_definition_id, string_value) 
    WHERE string_value IS NOT NULL;
CREATE INDEX idx_column_values_int ON column_values USING btree(column_definition_id, int_value) 
    WHERE int_value IS NOT NULL;
CREATE INDEX idx_column_values_decimal ON column_values USING btree(column_definition_id, decimal_value) 
    WHERE decimal_value IS NOT NULL;
CREATE INDEX idx_column_values_boolean ON column_values USING btree(column_definition_id, boolean_value) 
    WHERE boolean_value IS NOT NULL;
CREATE INDEX idx_column_values_date ON column_values USING btree(column_definition_id, date_value) 
    WHERE date_value IS NOT NULL;

-- Índice de texto para búsqueda de texto parcial con GIN
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX idx_column_values_string_trgm ON column_values USING gin(string_value gin_trgm_ops) 
    WHERE string_value IS NOT NULL;

-- Índice para particionamiento
CREATE INDEX idx_column_values_partition_logical_table ON column_values USING btree(
    logical_table_id
) WHERE logical_table_id IS NOT NULL;
```

### 11.4 Vistas Materializadas para Consultas Frecuentes

```sql
-- Vista materializada para una tabla lógica específica (ejemplo)
CREATE MATERIALIZED VIEW mv_products AS
SELECT 
    tr.id, 
    tr.code,
    tr.display_value AS name,
    tr.is_active,
    tr.sort_order,
    (SELECT cv.string_value FROM column_values cv 
     JOIN column_definitions cd ON cv.column_definition_id = cd.id
     WHERE cv.row_id = tr.id AND cd.column_name = 'description') AS description,
    (SELECT cv.decimal_value FROM column_values cv 
     JOIN column_definitions cd ON cv.column_definition_id = cd.id
     WHERE cv.row_id = tr.id AND cd.column_name = 'price') AS price,
    (SELECT cv.string_value FROM column_values cv 
     JOIN column_definitions cd ON cv.column_definition_id = cd.id
     WHERE cv.row_id = tr.id AND cd.column_name = 'category') AS category
FROM 
    table_rows tr
WHERE 
    tr.logical_table_id = (SELECT id FROM logical_tables WHERE table_name = 'PRODUCTS');

-- Índices para la vista materializada
CREATE UNIQUE INDEX idx_mv_products_id ON mv_products(id);
CREATE INDEX idx_mv_products_category ON mv_products(category);
CREATE INDEX idx_mv_products_price ON mv_products(price);
		
-- Función para refrescar la vista materializada
CREATE OR REPLACE FUNCTION refresh_mv_products()
RETURNS TRIGGER AS $$
BEGIN
    REFRESH MATERIALIZED VIEW CONCURRENTLY mv_products;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Trigger para actualizar la vista materializada cuando cambian los datos
CREATE TRIGGER trigger_refresh_mv_products
AFTER INSERT OR UPDATE OR DELETE ON table_rows
FOR EACH STATEMENT
WHEN (EXISTS (
    SELECT 1 FROM table_rows tr
    JOIN logical_tables lt ON tr.logical_table_id = lt.id
    WHERE lt.table_name = 'PRODUCTS' AND 
          tr.id = NEW.id
))
EXECUTE FUNCTION refresh_mv_products();

CREATE TRIGGER trigger_refresh_mv_products_values
AFTER INSERT OR UPDATE OR DELETE ON column_values
FOR EACH STATEMENT
WHEN (EXISTS (
    SELECT 1 FROM column_definitions cd
    JOIN column_values cv ON cd.id = cv.column_definition_id
	JOIN table_rows tr ON cv.row_id = tr.id
    JOIN logical_tables lt ON tr.logical_table_id = lt.id
    WHERE lt.table_name = 'PRODUCTS' AND 
          (cd.column_name IN ('description', 'price', 'category')) AND
          cv.id = NEW.id
))
EXECUTE FUNCTION refresh_mv_products();
```

### 11.5 Particionamiento de Tablas para Escalabilidad

Para manejar grandes volúmenes de datos, se implementa el particionamiento de tablas:

```sql
-- Particionamiento de column_values por logical_table_id
CREATE TABLE column_values (
    id UUID PRIMARY KEY,
    row_id UUID NOT NULL,
    column_definition_id UUID NOT NULL,
    logical_table_id UUID NOT NULL,  -- Campo adicional para particionamiento
    string_value TEXT,
    int_value BIGINT,
    decimal_value DECIMAL,
    boolean_value BOOLEAN,
    date_value TIMESTAMP,
    version INT NOT NULL DEFAULT 1,
    created_at TIMESTAMP NOT NULL,
    created_by_user_id UUID NOT NULL,
    updated_at TIMESTAMP,
    updated_by_user_id UUID
) PARTITION BY LIST (logical_table_id);

-- Crear particiones para diferentes tablas lógicas
CREATE TABLE column_values_products PARTITION OF column_values
    FOR VALUES IN ('11111111-1111-1111-1111-111111111111');  -- ID de tabla PRODUCTS

CREATE TABLE column_values_customers PARTITION OF column_values
    FOR VALUES IN ('22222222-2222-2222-2222-222222222222');  -- ID de tabla CUSTOMERS

CREATE TABLE column_values_orders PARTITION OF column_values
    FOR VALUES IN ('33333333-3333-3333-3333-333333333333');  -- ID de tabla ORDERS

-- Partición por defecto para nuevas tablas lógicas
CREATE TABLE column_values_default PARTITION OF column_values DEFAULT;
```

### 11.6 Implementación de Lotes (Batching) para Operaciones Masivas

```go
// Servicio para operaciones de lotes con rendimiento optimizado
type BatchImportService struct {
    uow             UnitOfWork
    eventBus        EventBus
    logger          *zap.Logger
    metrics         metrics.Reporter
    tracer          trace.Tracer
    bulkhead        Bulkhead
    batchSize       int
    cacheManager    CacheManager
    cacheStrategy   CacheStrategy
}

// Importación por lotes con optimizaciones
func (s *BatchImportService) ImportData(ctx context.Context, tableName string, records []map[string]interface{}) (*ImportResult, error) {
    ctx, span := s.tracer.Start(ctx, "BatchImportService.ImportData", 
        trace.WithAttributes(
            attribute.String("table_name", tableName),
            attribute.Int("records_count", len(records)),
        ))
    defer span.End()
    
    // Limitar el procesamiento concurrente con Bulkhead
    executed, err := s.bulkhead.TryExecute(func() error {
        return s.processImport(ctx, tableName, records)
    })
    
    if !executed {
        s.metrics.RecordBatchRejection("import", tableName)
        return nil, ErrTooManyRequests
    }
    
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, "Import failed")
        return nil, err
    }
    
    // Resultado de importación
    result := &ImportResult{
        TableName:     tableName,
        TotalRecords:  len(records),
        SuccessCount:  len(records), // Ajustar si hay errores parciales
        ErrorCount:    0,            // Ajustar si hay errores parciales
        ProcessingTime: time.Since(span.StartTime()),
    }
    
    span.SetStatus(codes.Ok, "Import successful")
    return result, nil
}

// Procesamiento de importación optimizado
func (s *BatchImportService) processImport(ctx context.Context, tableName string, records []map[string]interface{}) error {
    // Obtener tabla lógica
    var logicalTable *LogicalTable
    
    err := s.uow.WithTransaction(ctx, func(ctx context.Context) error {
        var err error
        logicalTable, err = s.uow.LogicalTables().FindByName(ctx, tableName)
        return err
    })
    
    if err != nil {
        return fmt.Errorf("error obteniendo tabla lógica: %w", err)
    }
    
    // Procesar en lotes para optimizar rendimiento
    batches := splitIntoBatches(records, s.batchSize)
    
    for i, batch := range batches {
        batchCtx, batchSpan := s.tracer.Start(ctx, "BatchImportService.processBatch",
            trace.WithAttributes(
                attribute.Int("batch_number", i+1),
                attribute.Int("batch_size", len(batch)),
            ))
        
        // Procesar lote en una transacción
        if err := s.processBatch(batchCtx, logicalTable, batch); err != nil {
            batchSpan.RecordError(err)
            batchSpan.SetStatus(codes.Error, "Batch processing failed")
            batchSpan.End()
            return fmt.Errorf("error procesando lote %d: %w", i+1, err)
        }
        
        batchSpan.SetStatus(codes.Ok, "Batch processed successfully")
        batchSpan.End()
    }
    
    // Invalida caché después de importación
    if s.cacheStrategy.ShouldCache(tableName) {
        cachePattern := fmt.Sprintf("eav:table:%s:*", tableName)
        if err := s.cacheManager.DeleteByPattern(cachePattern); err != nil {
            s.logger.Warn("Error invalidando caché después de importación", 
                zap.String("table", tableName),
                zap.Error(err))
        }
    }
    
    return nil
}

// Procesamiento de un lote individual
func (s *BatchImportService) processBatch(ctx context.Context, logicalTable *LogicalTable, batch []map[string]interface{}) error {
    return s.uow.WithTransaction(ctx, func(ctx context.Context) error {
        // Preparar sentencias preparadas para mayor eficiencia
        tx := getTxFromContext(ctx, s.uow.(*unitOfWork).db)
        
        // 1. Preparar sentencia para inserción de filas
        rowStmt, err := tx.PreparexContext(ctx, `
            INSERT INTO table_rows (
                id, logical_table_id, code, display_value, sort_order, is_active,
                parent_row_id, version, created_at, created_by_user_id
            ) VALUES (
                $1, $2, $3, $4, $5, $6, $7, $8, $9, $10
            )
        `)
        if err != nil {
            return fmt.Errorf("error preparando sentencia para filas: %w", err)
        }
        defer rowStmt.Close()
        
        // 2. Preparar sentencia para inserción de valores de columna
        valStmt, err := tx.PreparexContext(ctx, `
            INSERT INTO column_values (
                id, row_id, column_definition_id, logical_table_id,
                string_value, int_value, decimal_value, boolean_value, date_value,
                version, created_at, created_by_user_id
            ) VALUES (
                $1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11, $12
            )
        `)
        if err != nil {
            return fmt.Errorf("error preparando sentencia para valores: %w", err)
        }
        defer valStmt.Close()
        
        // Cargar mapeo de nombres de columna a definiciones
        columnDefs, err := s.loadColumnDefinitions(ctx, logicalTable.ID)
        if err != nil {
            return err
        }
        
        // Procesar cada registro en el lote
        currentUserID := getCurrentUserID(ctx)
        now := time.Now()
        
        for _, record := range batch {
            // 1. Insertar fila base
            rowID := uuid.New()
            
            code := getStringValue(record, "code", "")
            displayValue := getStringValue(record, "display_value", "")
            sortOrder := getIntValue(record, "sort_order", 0)
            isActive := getBoolValue(record, "is_active", true)
            var parentRowID *uuid.UUID
            if parentID, ok := record["parent_row_id"]; ok {
                if parentIDStr, ok := parentID.(string); ok && parentIDStr != "" {
                    id, err := uuid.Parse(parentIDStr)
                    if err == nil {
                        parentRowID = &id
                    }
                }
            }
            
            _, err = rowStmt.ExecContext(ctx,
                rowID, logicalTable.ID, code, displayValue, sortOrder, isActive,
                parentRowID, 1, now, currentUserID,
            )
            if err != nil {
                return fmt.Errorf("error insertando fila: %w", err)
            }
            
            // 2. Insertar valores de columnas
            for colName, colValue := range record {
                // Saltar columnas promovidas ya procesadas
                if isPromotedColumn(colName) {
                    continue
                }
                
                // Buscar definición de columna
                colDef, ok := columnDefs[colName]
                if !ok {
                    s.logger.Warn("Columna no encontrada, omitiendo",
                        zap.String("column", colName),
                        zap.String("table", logicalTable.TableName))
                    continue
                }
                
                // Convertir y asignar valor según tipo
                var strVal, dateVal *string
                var intVal *int64
                var decVal *decimal.Decimal
                var boolVal *bool
                
                switch colDef.DataType {
                case "String":
                    if v, ok := colValue.(string); ok {
                        strVal = &v
                    }
                case "Int":
                    if v, ok := getIntPointer(colValue); ok {
                        intVal = v
                    }
                case "Decimal":
                    if v, ok := getDecimalPointer(colValue); ok {
                        decVal = v
                    }
                case "Boolean":
                    if v, ok := getBoolPointer(colValue); ok {
                        boolVal = v
                    }
                case "Date":
                    if v, ok := getDatePointer(colValue); ok {
                        d := v.Format(time.RFC3339)
                        dateVal = &d
                    }
                }
                
                // Insertar valor de columna
                _, err = valStmt.ExecContext(ctx,
                    uuid.New(), rowID, colDef.ID, logicalTable.ID,
                    strVal, intVal, decVal, boolVal, dateVal,
                    1, now, currentUserID,
                )
                if err != nil {
                    return fmt.Errorf("error insertando valor de columna: %w", err)
                }
            }
            
            // 3. Publicar evento mediante outbox para cada fila
            event := TableRowCreatedEvent{
                RowID:         rowID,
                LogicalTableID: logicalTable.ID,
                Code:          code,
                DisplayValue:  displayValue,
                TableName:     logicalTable.TableName,
                CreatedAt:     now,
            }
            
            if err := s.uow.OutboxEntries().CreateEntry(ctx, "TableRowCreated", rowID, event); err != nil {
                return fmt.Errorf("error creando entrada de outbox: %w", err)
            }
        }
        
        return nil
    })
}
```

## 12. Seguridad

### 12.0 Seguridad de Contraseñas Tradicionales

Para la autenticación basada en contraseña, se implementarán las siguientes medidas de seguridad:

*   **Hashing Seguro:** Las contraseñas nunca se almacenarán en texto plano. Se utilizarán algoritmos de hashing modernos y resistentes a la fuerza bruta como Argon2, scrypt o bcrypt, con un salt único por usuario.
*   **Políticas de Contraseña:** Se aplicarán políticas de complejidad (longitud mínima, combinación de caracteres) y, opcionalmente, expiración o prohibición de contraseñas comunes.
*   **Protección contra Fuerza Bruta:** Implementación de limitación de intentos de login fallidos por usuario o IP, con bloqueo temporal progresivo.
*   **Mitigación de Credential Stuffing:** Monitoreo de patrones de login sospechosos y posible integración con servicios de verificación de credenciales comprometidas.
*   **Almacenamiento Seguro de Hashes:** La columna `password_hash` y `password_salt` en la base de datos estarán protegidas con cifrado a nivel de disco y acceso restringido.

### 12.1 Implementación de RBAC (Role-Based Access Control)

```go
// Interfaz para verificación de permisos
type PermissionVerifier interface {
    HasPermission(ctx context.Context, userID uuid.UUID, action string, resource string) (bool, error)
    GetUserPermissions(ctx context.Context, userID uuid.UUID) ([]string, error)
}

// Implementación de RBAC
type RBACPermissionVerifier struct {
    db *sqlx.DB
    cacheManager CacheManager
    logger *zap.Logger
    metrics metrics.Reporter
    tracer trace.Tracer
    breaker CircuitBreaker
}

func NewRBACPermissionVerifier(
    db *sqlx.DB,
    cacheManager CacheManager,
    logger *zap.Logger,
    metrics metrics.Reporter,
    tracer trace.Tracer,
) PermissionVerifier {
    // Configurar circuit breaker
    cb := gobreaker.NewCircuitBreaker(gobreaker.Settings{
        Name:        "rbac-permissions",
        MaxRequests: 5,
        Interval:    30 * time.Second,
        Timeout:     60 * time.Second,
        ReadyToTrip: func(counts gobreaker.Counts) bool {
            failureRatio := float64(counts.TotalFailures) / float64(counts.Requests)
            return counts.Requests >= 5 && failureRatio >= 0.6
        },
    })
    
    return &RBACPermissionVerifier{
        db: db,
        cacheManager: cacheManager,
        logger: logger,
        metrics: metrics,
        tracer: tracer,
        breaker: NewGoBreaker(cb),
    }
}

func (v *RBACPermissionVerifier) HasPermission(ctx context.Context, userID uuid.UUID, action string, resource string) (bool, error) {
    ctx, span := v.tracer.Start(ctx, "RBACPermissionVerifier.HasPermission",
        trace.WithAttributes(
            attribute.String("user_id", userID.String()),
            attribute.String("action", action),
            attribute.String("resource", resource),
        ))
    defer span.End()
    
    // Métricas
    v.metrics.RecordPermissionCheck(action, resource)
    
    // Construir clave de caché
    cacheKey := fmt.Sprintf("permission:%s:%s:%s", userID, action, resource)
    
    // Intentar obtener de caché
    var hasPermission bool
    found, err := v.cacheManager.Get(cacheKey, &hasPermission)
    if err != nil {
        v.logger.Warn("Error leyendo de caché", zap.Error(err))
    }
    
    if found {
        span.SetAttributes(attribute.Bool("cache.hit", true))
        v.metrics.RecordCacheHit("permission")
        return hasPermission, nil
    }
    
    span.SetAttributes(attribute.Bool("cache.hit", false))
    v.metrics.RecordCacheMiss("permission")
    
    // Verificar permiso con circuit breaker
    var result bool
    err = v.breaker.Execute(func() error {
        // Construir combinación de permiso
        permission := action + "_" + resource
        
        // Consultar en BD
        query := `
            SELECT EXISTS(
                SELECT 1
                FROM app_users u
                JOIN user_roles ur ON u.id = ur.user_id
                JOIN role_permissions rp ON ur.role_id = rp.role_id
                JOIN permissions p ON rp.permission_id = p.id
                WHERE u.id = $1 AND p.permission_name = $2
            ) AS has_permission
        `
        
        if err := v.db.GetContext(ctx, &result, query, userID, permission); err != nil {
            return fmt.Errorf("error verificando permiso: %w", err)
        }
        
        return nil
    })
    
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, "Permission check failed")
        v.metrics.RecordPermissionError(action, resource)
        return false, fmt.Errorf("error verificando permiso: %w", err)
    }
    
    // Guardar en caché (TTL corto para permisos)
    if err := v.cacheManager.Set(cacheKey, result, 5*time.Minute); err != nil {
        v.logger.Warn("Error guardando en caché", zap.Error(err))
    }
    
    v.metrics.RecordPermissionResult(action, resource, result)
    span.SetAttributes(attribute.Bool("permission.granted", result))
    span.SetStatus(codes.Ok, "Permission check completed")
    
    return result, nil
}

func (v *RBACPermissionVerifier) GetUserPermissions(ctx context.Context, userID uuid.UUID) ([]string, error) {
    ctx, span := v.tracer.Start(ctx, "RBACPermissionVerifier.GetUserPermissions",
        trace.WithAttributes(attribute.String("user_id", userID.String())))
    defer span.End()
    
    // Métricas
    v.metrics.RecordPermissionOperation("get_user_permissions")
    
    // Construir clave de caché
    cacheKey := fmt.Sprintf("permissions:%s", userID)
    
    // Intentar obtener de caché
    var permissions []string
    found, err := v.cacheManager.Get(cacheKey, &permissions)
    if err != nil {
        v.logger.Warn("Error leyendo de caché", zap.Error(err))
    }
    
    if found {
        span.SetAttributes(attribute.Bool("cache.hit", true))
        v.metrics.RecordCacheHit("permissions")
        return permissions, nil
    }
    
    span.SetAttributes(attribute.Bool("cache.hit", false))
    v.metrics.RecordCacheMiss("permissions")
    
    // Consultar permisos con circuit breaker
    err = v.breaker.Execute(func() error {
        // Consultar en BD
        query := `
            SELECT p.permission_name
            FROM app_users u
            JOIN user_roles ur ON u.id = ur.user_id
            JOIN role_permissions rp ON ur.role_id = rp.role_id
            JOIN permissions p ON rp.permission_id = p.id
            WHERE u.id = $1
        `
        
        if err := v.db.SelectContext(ctx, &permissions, query, userID); err != nil {
            return fmt.Errorf("error obteniendo permisos: %w", err)
        }
        
        return nil
    })
    
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, "Getting permissions failed")
        v.metrics.RecordPermissionError("get_all", "user")
        return nil, fmt.Errorf("error obteniendo permisos: %w", err)
    }
    
    // Guardar en caché
    if err := v.cacheManager.Set(cacheKey, permissions, 5*time.Minute); err != nil {
        v.logger.Warn("Error guardando en caché", zap.Error(err))
    }
    
    span.SetAttributes(attribute.Int("permissions.count", len(permissions)))
    span.SetStatus(codes.Ok, "Got permissions successfully")
    return permissions, nil
}
```

### 12.2 Middleware de Autenticación

```go
// Middleware de autenticación con JWT y métricas
func AuthMiddleware(
    sessionManager *SessionManager,
    metrics metrics.Reporter,
    tracer trace.Tracer,
    logger *zap.Logger,
) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            ctx, span := tracer.Start(r.Context(), "AuthMiddleware")
            defer span.End()
            
            // Inicio de métricas
            startTime := time.Now()
            
            // Obtener token de header Authorization
            authHeader := r.Header.Get("Authorization")
            if authHeader == "" {
                metrics.RecordAuthError("missing_header")
                RespondWithError(w, http.StatusUnauthorized, "Token de autorización requerido", nil)
                span.SetStatus(codes.Error, "Missing auth header")
                return
            }
            
            // Extraer token Bearer
            tokenString := strings.TrimPrefix(authHeader, "Bearer ")
            if tokenString == authHeader {
                metrics.RecordAuthError("invalid_format")
                RespondWithError(w, http.StatusUnauthorized, "Formato de token inválido", nil)
                span.SetStatus(codes.Error, "Invalid token format")
                return
            }
            
            // Validar sesión con Circuit Breaker
            var session *Session
            var err error
            
            cb := gobreaker.NewCircuitBreaker(gobreaker.Settings{
                Name:        "session-validation",
                MaxRequests: 5,
                Interval:    30 * time.Second,
                Timeout:     60 * time.Second,
                ReadyToTrip: func(counts gobreaker.Counts) bool {
                    failureRatio := float64(counts.TotalFailures) / float64(counts.Requests)
                    return counts.Requests >= 5 && failureRatio >= 0.6
                },
            })
            
            _, cbErr := cb.Execute(func() (interface{}, error) {
                session, err = sessionManager.ValidateSession(ctx, tokenString)
                return nil, err
            })
            
            if cbErr != nil {
                statusCode := http.StatusUnauthorized
                message := "Sesión inválida"
                
                if errors.Is(err, ErrSessionExpired) {
                    message = "Sesión expirada"
                    metrics.RecordAuthError("session_expired")
                } else if errors.Is(err, ErrSessionRevoked) {
                    message = "Sesión revocada"
                    metrics.RecordAuthError("session_revoked")
                } else {
                    metrics.RecordAuthError("validation_error")
                }
                
                RespondWithError(w, statusCode, message, nil)
                span.RecordError(err)
                span.SetStatus(codes.Error, message)
                return
            }
            
            // Añadir información de usuario y sesión al contexto
            ctx = context.WithValue(ctx, contextKeyUserID, session.UserID)
            ctx = context.WithValue(ctx, contextKeySessionID, session.ID)
            ctx = context.WithValue(ctx, contextKeyUsername, session.Username)
            
            // Registrar métricas de éxito
            metrics.RecordAuthSuccess(session.AuthMethod)
            metrics.RecordRequestDuration("auth_middleware", time.Since(startTime))
            
            span.SetAttributes(
                attribute.String("user.id", session.UserID.String()),
                attribute.String("session.id", session.ID.String()),
                attribute.String("auth.method", session.AuthMethod),
            )
            span.SetStatus(codes.Ok, "Authentication successful")
            
            // Continuar con el siguiente handler con el contexto actualizado
            next.ServeHTTP(w, r.WithContext(ctx))
        })
    }
}
```

### 12.3 Protección contra Ataques Comunes

```go
// Middleware para protección contra CSRF
func CSRFProtection(
    allowedOrigins []string,
    logger *zap.Logger,
    metrics metrics.Reporter,
) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            // No aplicar en OPTIONS (preflight CORS)
            if r.Method == http.MethodOptions {
                next.ServeHTTP(w, r)
                return
            }
            
            // Solo aplicar en métodos no idempotentes
            if r.Method != http.MethodGet && r.Method != http.MethodHead && r.Method != http.MethodOptions {
                // Verificar origen
                origin := r.Header.Get("Origin")
                referer := r.Header.Get("Referer")
                
                // Añadir orígenes de desarrollo si es necesario
                if os.Getenv("ENVIRONMENT") == "development" {
                    allowedOrigins = append(allowedOrigins, 
                        "http://localhost:3000",
                        "http://127.0.0.1:3000")
                }
                
                originValid := false
                
                if origin != "" {
                    for _, allowed := range allowedOrigins {
                        if origin == allowed {
                            originValid = true
                            break
                        }
                    }
                } else if referer != "" {
                    for _, allowed := range allowedOrigins {
                        if strings.HasPrefix(referer, allowed) {
                            originValid = true
                            break
                        }
                    }
                }
                
                if !originValid {
                    metrics.RecordSecurityError("csrf_failure")
                    RespondWithError(w, http.StatusForbidden, "CSRF verificación fallida", nil)
                    logger.Warn("CSRF verificación fallida",
                        zap.String("origin", origin),
                        zap.String("referer", referer),
                        zap.String("ip", GetClientIP(r)),
                        zap.String("method", r.Method),
                        zap.String("path", r.URL.Path))
                    return
                }
            }
            
            next.ServeHTTP(w, r)
        })
    }
}

// Middleware para rate limiting
func RateLimitMiddleware(metrics metrics.Reporter) func(http.Handler) http.Handler {
    // Implementación usando token bucket para limitar por IP
    limiters := &sync.Map{}
    
    // Función para obtener limitador por IP
    getLimiter := func(ip string) *rate.Limiter {
        // Intentar obtener limitador existente
        if limiter, ok := limiters.Load(ip); ok {
            return limiter.(*rate.Limiter)
        }
        
        // Crear nuevo limitador: 100 req/min con burst de 50
        limiter := rate.NewLimiter(rate.Limit(100.0/60.0), 50)
        limiters.Store(ip, limiter)
        return limiter
    }
    
    // Limpieza periódica de limitadores no utilizados
    go func() {
        ticker := time.NewTicker(10 * time.Minute)
        for range ticker.C {
            // Eliminar limitadores no utilizados
            now := time.Now()
            limiters.Range(func(key, value interface{}) bool {
                limiter := value.(*rate.Limiter)
                // Aquí usaríamos un campo lastUsed que tendríamos que añadir
                // a una implementación personalizada de limitador
                return true
            })
        }
    }()
    
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            // Obtener IP del cliente
            ip := GetClientIP(r)
            
            // Obtener limitador para esta IP
            limiter := getLimiter(ip)

            // Verificar si hay tokens disponibles
            if !limiter.Allow() {
                metrics.RecordSecurityError("rate_limit_exceeded")
                RespondWithError(w, http.StatusTooManyRequests, "Demasiadas peticiones. Intente más tarde.", nil)
                return
            }
            
            next.ServeHTTP(w, r)
        })
    }
}

// Middleware para seguridad de cabeceras HTTP
func SecurityHeadersMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Protección contra XSS
        w.Header().Set("X-XSS-Protection", "1; mode=block")
        
        // Políticas estrictas de transporte seguro
        w.Header().Set("Strict-Transport-Security", "max-age=31536000; includeSubDomains")
        
        // Evitar que el navegador haga MIME sniffing
        w.Header().Set("X-Content-Type-Options", "nosniff")
        
        // Políticas de marcos (prevenir clickjacking)
        w.Header().Set("X-Frame-Options", "DENY")
        
        // Política de referrer
        w.Header().Set("Referrer-Policy", "strict-origin-when-cross-origin")
        
        // Content Security Policy (ajustar según necesidades)
        w.Header().Set("Content-Security-Policy", "default-src 'self'; script-src 'self'; object-src 'none'; img-src 'self' data:; style-src 'self' 'unsafe-inline';")
        
        next.ServeHTTP(w, r)
    })
}

// Middleware para prevenir ataques de inyección SQL
func SQLInjectionProtectionMiddleware(logger *zap.Logger) func(http.Handler) http.Handler {
    // Patrones comunes de inyección SQL
    sqlInjectionPatterns := []string{
        "(?i)\\b(SELECT|INSERT|UPDATE|DELETE|DROP|ALTER)\\b.*\\b(FROM|INTO|WHERE)\\b",
        "(?i)\\b(UNION|JOIN)\\b.*\\b(SELECT)\\b",
        "(?i)['\"].*--",
        "(?i)/\\*.*\\*/",
    }
    
    // Compilar patrones
    var regexPatterns []*regexp.Regexp
    for _, pattern := range sqlInjectionPatterns {
        regex, err := regexp.Compile(pattern)
        if err != nil {
            logger.Error("Error compilando patrón de inyección SQL", zap.Error(err))
            continue
        }
        regexPatterns = append(regexPatterns, regex)
    }
    
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            // Verificar parámetros de consulta
            for key, values := range r.URL.Query() {
                for _, value := range values {
                    for _, regex := range regexPatterns {
                        if regex.MatchString(value) {
                            logger.Warn("Posible intento de inyección SQL en parámetro de consulta",
                                zap.String("ip", GetClientIP(r)),
                                zap.String("param", key),
                                zap.String("value", value))
                            RespondWithError(w, http.StatusBadRequest, "Parámetro de consulta inválido", nil)
                            return
                        }
                    }
                }
            }

            // Si es un formulario o JSON, verificar el body
            if r.Method == http.MethodPost || r.Method == http.MethodPut || r.Method == http.MethodPatch {
                // Para formularios
                if err := r.ParseForm(); err == nil {
                    for key, values := range r.PostForm {
                        for _, value := range values {
                            for _, regex := range regexPatterns {
                                if regex.MatchString(value) {
                                    logger.Warn("Posible intento de inyección SQL en formulario",
                                        zap.String("ip", GetClientIP(r)),
                                        zap.String("param", key),
                                        zap.String("value", value))
                                    RespondWithError(w, http.StatusBadRequest, "Parámetro de formulario inválido", nil)
                                    return
                                }
                            }
                        }
                    }
                }
                
                // Para JSON, usamos un ResponseWriter personalizado para leer el body
                bodyBytes, err := io.ReadAll(r.Body)
                if err != nil {
                    logger.Error("Error leyendo body", zap.Error(err))
                    RespondWithError(w, http.StatusBadRequest, "Error leyendo body", nil)
                    return
                }
                
                // Restaurar el body para que pueda ser leído por los siguientes handlers
                r.Body = io.NopCloser(bytes.NewBuffer(bodyBytes))
                
                // Verificar contenido JSON
                bodyStr := string(bodyBytes)
                for _, regex := range regexPatterns {
                    if regex.MatchString(bodyStr) {
                        logger.Warn("Posible intento de inyección SQL en JSON",
                            zap.String("ip", GetClientIP(r)),
                            zap.String("body", bodyStr))
                        RespondWithError(w, http.StatusBadRequest, "Body inválido", nil)
                        return
                    }
                }
            }
            
            next.ServeHTTP(w, r)
        })
    }
}
```

## 13. Observabilidad y Monitoreo

### 13.1 Configuración de Logging Estructurado

```go
// Configuración de logging con Zap
func SetupLogger(config LoggerConfig) (*zap.Logger, error) {
    // Configuración según entorno
    var zapConfig zap.Config
    
    if config.Environment == "production" {
        zapConfig = zap.NewProductionConfig()
        zapConfig.EncoderConfig.TimeKey = "timestamp"
        zapConfig.EncoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder
        
        // Ajustar nivel de logs según configuración
        level, err := zapcore.ParseLevel(config.Level)
        if err == nil {
            zapConfig.Level = zap.NewAtomicLevelAt(level)
        }
        
        // Configurar salida a stdout/stderr
        if config.OutputPaths != nil && len(config.OutputPaths) > 0 {
            zapConfig.OutputPaths = config.OutputPaths
        }
        
        // Configurar salida de errores
        if config.ErrorOutputPaths != nil && len(config.ErrorOutputPaths) > 0 {
            zapConfig.ErrorOutputPaths = config.ErrorOutputPaths
        }
    } else {
        zapConfig = zap.NewDevelopmentConfig()
        zapConfig.EncoderConfig.EncodeLevel = zapcore.CapitalColorLevelEncoder
        
        // En desarrollo, usar nivel debug por defecto
        zapConfig.Level = zap.NewAtomicLevelAt(zapcore.DebugLevel)
    }
    
    // Agregar campos globales
    zapConfig.InitialFields = map[string]interface{}{
        "service": config.ServiceName,
        "version": config.ServiceVersion,
    }
    
    // Crear logger
    logger, err := zapConfig.Build()
    if err != nil {
        return nil, fmt.Errorf("error configurando logger: %w", err)
    }
    
    // Reemplazar logger global
    zap.ReplaceGlobals(logger)
    
    return logger, nil
}

// Middleware para logging de requests HTTP
func RequestLoggingMiddleware(logger *zap.Logger, metrics metrics.Reporter) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()
            
            // Wrapper para capturar código de estado
            ww := middleware.NewWrapResponseWriter(w, r.ProtoMajor)
            
            // ID de correlación para seguimiento de peticiones
            correlationID := r.Header.Get("X-Correlation-ID")
            if correlationID == "" {
                correlationID = uuid.New().String()
            }
            
            // Añadir al contexto y headers
            ctx := context.WithValue(r.Context(), contextKeyCorrelationID, correlationID)
            r = r.WithContext(ctx)
            ww.Header().Set("X-Correlation-ID", correlationID)
            
            // Detalles básicos pre-petición
            logger.Info("Solicitud HTTP recibida",
                zap.String("method", r.Method),
                zap.String("path", r.URL.Path),
                zap.String("query", r.URL.RawQuery),
                zap.String("remote_addr", GetClientIP(r)),
                zap.String("user_agent", r.UserAgent()),
                zap.String("correlation_id", correlationID),
            )
            
            // Ejecutar el siguiente handler
            next.ServeHTTP(ww, r)
            
            // Calcular duración
            duration := time.Since(start)
            
            // Registrar métricas
            metrics.RecordHTTPResponse(r.Method, r.URL.Path, ww.Status(), duration)
            
            // Nivel de log según código de respuesta
            logFunc := logger.Info
            if ww.Status() >= 400 && ww.Status() < 500 {
                logFunc = logger.Warn
            } else if ww.Status() >= 500 {
                logFunc = logger.Error
            }
            
            // Log post-petición
            logFunc("Solicitud HTTP completada",
                zap.String("method", r.Method),
                zap.String("path", r.URL.Path),
                zap.Int("status", ww.Status()),
                zap.Duration("duration", duration),
                zap.Int("bytes_written", ww.BytesWritten()),
                zap.String("correlation_id", correlationID),
            )
        })
    }
}

// Log de errores con contexto
func LogError(ctx context.Context, logger *zap.Logger, msg string, err error) {
    // Extraer información del contexto
    correlationID := ""
    if ctx != nil {
        if id := ctx.Value(contextKeyCorrelationID); id != nil {
            correlationID = id.(string)
        }
    }
    
    // Construir campos de log
    fields := []zap.Field{
        zap.Error(err),
    }
    
    // Añadir correlación si existe
    if correlationID != "" {
        fields = append(fields, zap.String("correlation_id", correlationID))
    }
    
    // Añadir stack trace si es posible
    if stackErr, ok := err.(stackTracer); ok {
        fields = append(fields, zap.String("stack", fmt.Sprintf("%+v", stackErr.StackTrace())))
    }
    
    // Registrar error
    logger.Error(msg, fields...)
}

// Integración con OpenTelemetry para logs
func SetupOTelLogging(ctx context.Context, logger *zap.Logger) (*zap.Logger, func(), error) {
    // Configurar proveedor de logs
    exporter, err := stdoutlog.New(stdoutlog.WithPrettyPrint())
    if err != nil {
        return nil, nil, fmt.Errorf("error creando exportador de logs: %w", err)
    }
    
    // Crear proveedor de logs
    loggerProvider, err := log.NewProvider(
        log.WithExporter(exporter),
        log.WithResource(resource.NewWithAttributes(
            semconv.SchemaURL,
            semconv.ServiceNameKey.String(os.Getenv("SERVICE_NAME")),
            semconv.ServiceVersionKey.String(os.Getenv("SERVICE_VERSION")),
        )),
    )
    if err != nil {
        return nil, nil, fmt.Errorf("error creando proveedor de logs: %w", err)
    }
    
    // Establecer como proveedor global
    global.SetLoggerProvider(loggerProvider)
    
    // Crear puente entre Zap y OpenTelemetry
    otelLogger := otelslog.NewFactory(
        loggerProvider.Logger("eav-service"),
    )
    
    // Crear logger que envía tanto a Zap como a OpenTelemetry
    combinedLogger := logger.WithOptions(
        zap.WrapCore(func(zapcore.Core) zapcore.Core {
            return zapcore.NewTee(
                logger.Core(),
                zapcore.NewCore(
                    zapcore.NewJSONEncoder(logger.TeeConfig().EncoderConfig),
                    zapcore.AddSync(otelLogger),
                    logger.Level(),
                ),
            )
        }),
    )
    
    // Función de limpieza
    cleanup := func() {
        if err := loggerProvider.Shutdown(context.Background()); err != nil {
            logger.Error("Error cerrando proveedor de logs", zap.Error(err))
        }
    }
    
    return combinedLogger, cleanup, nil
}
```

### 13.2 Integración con OpenTelemetry para Trazas Distribuidas

```go
// Configuración de OpenTelemetry para trazas distribuidas
func SetupOpenTelemetry(ctx context.Context, serviceName, serviceVersion string) (*sdktrace.TracerProvider, error) {
    // Configurar exportador
    var exporter sdktrace.SpanExporter
    var err error
    
    // Utilizar exportador OTLP para enviar a colector de OpenTelemetry
    if endpoint := os.Getenv("OTEL_EXPORTER_OTLP_ENDPOINT"); endpoint != "" {
        exporter, err = otlptracegrpc.New(ctx,
            otlptracegrpc.WithEndpoint(endpoint),
            otlptracegrpc.WithInsecure(),
        )
    } else {
        // Fallback a consola para desarrollo
        exporter, err = stdouttrace.New(stdouttrace.WithPrettyPrint())
    }
    
    if err != nil {
        return nil, fmt.Errorf("error configurando exportador de trazas: %w", err)
    }
    
    // Configurar recursos
    res, err := resource.New(ctx,
        resource.WithAttributes(
            semconv.ServiceNameKey.String(serviceName),
            semconv.ServiceVersionKey.String(serviceVersion),
            attribute.String("environment", os.Getenv("ENVIRONMENT")),
        ),
        resource.WithHost(),
        resource.WithOS(),
        resource.WithProcess(),
    )
    if err != nil {
        return nil, fmt.Errorf("error configurando recursos: %w", err)
    }
    
    // Configurar proveedor de trazas con muestreo basado en parámetros
    sampler := sdktrace.ParentBased(
        sdktrace.TraceIDRatioBased(getSamplingRate()),
        sdktrace.WithRemoteParentSampled(sdktrace.AlwaysSample()),
    )
    
    tracerProvider := sdktrace.NewTracerProvider(
        sdktrace.WithSampler(sampler),
        sdktrace.WithBatcher(exporter),
        sdktrace.WithResource(res),
    )
    
    // Configurar propagadores (W3C Trace Context y Baggage)
    otel.SetTextMapPropagator(propagation.NewCompositeTextMapPropagator(
        propagation.TraceContext{},
        propagation.Baggage{},
    ))
    
    // Configurar como proveedor global
    otel.SetTracerProvider(tracerProvider)
    
    return tracerProvider, nil
}

// Determinar tasa de muestreo basada en entorno
func getSamplingRate() float64 {
    env := os.Getenv("ENVIRONMENT")
    samplingRateStr := os.Getenv("OTEL_TRACE_SAMPLER_ARG")
    
    var defaultRate float64
    switch env {
    case "production":
        defaultRate = 0.1 // 10% en producción
    case "staging":
        defaultRate = 0.3 // 30% en staging
    default:
        defaultRate = 1.0 // 100% en desarrollo
    }
    
    if samplingRateStr != "" {
        if rate, err := strconv.ParseFloat(samplingRateStr, 64); err == nil {
            return rate
        }
    }
    
    return defaultRate
}

// Middleware para trazas HTTP
func TracingMiddleware(serviceName string) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            // Obtener tracer
            tracer := otel.Tracer(serviceName + ".http")
            
            // Extraer contexto de propagación desde headers de la petición
            ctx := r.Context()
            propagator := otel.GetTextMapPropagator()
            ctx = propagator.Extract(ctx, propagation.HeaderCarrier(r.Header))
            
            // Normalizar path para evitar cardinalidad excesiva
            path := NormalizePath(r.URL.Path)
            
            // Crear span para la solicitud
            spanName := fmt.Sprintf("%s %s", r.Method, path)
            
            ctx, span := tracer.Start(ctx, spanName,
                trace.WithAttributes(
                    semconv.HTTPMethodKey.String(r.Method),
                    semconv.HTTPURLKey.String(r.URL.String()),
                    semconv.HTTPTargetKey.String(r.URL.Path),
                    semconv.HTTPRouteKey.String(path),
                    semconv.HTTPUserAgentKey.String(r.UserAgent()),
                    semconv.HTTPClientIPKey.String(GetClientIP(r)),
                    attribute.String("http.correlation_id", getCorrelationID(ctx)),
                ),
                trace.WithSpanKind(trace.SpanKindServer),
            )
            defer span.End()
            
            // Wrapper para capturar código de estado
            ww := middleware.NewWrapResponseWriter(w, r.ProtoMajor)
            
            // Ejecutar el siguiente handler con contexto de traza
            next.ServeHTTP(ww, r.WithContext(ctx))
            
            // Actualizar span con información de respuesta
            span.SetAttributes(
                semconv.HTTPStatusCodeKey.Int(ww.Status()),
                attribute.Int("http.response_size", ww.BytesWritten()),
            )
            
            // Marcar span como error si es respuesta 5xx
            if ww.Status() >= 500 {
                span.SetStatus(codes.Error, http.StatusText(ww.Status()))
            } else {
                span.SetStatus(codes.Ok, "")
            }
        })
    }
}

// Normalizar path para reducir cardinalidad en trazas
func NormalizePath(path string) string {
    // Detectar y reemplazar IDs en la ruta (ej: /users/123 -> /users/:id)
    re := regexp.MustCompile(`/([a-f0-9]{8}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{12})(/|$)`)
    path = re.ReplaceAllString(path, "/:id$2")
    
    // Patrones numéricos
    re = regexp.MustCompile(`/(\d+)(/|$)`)
    path = re.ReplaceAllString(path, "/:id$2")
    
    return path
}

// Instrumentación de cliente HTTP
func InstrumentHTTPClient(client *http.Client, serviceName string) *http.Client {
    if client == nil {
        client = http.DefaultClient
    }
    
    // Crear cliente instrumentado
    return &http.Client{
        Transport: otelhttp.NewTransport(
            getHTTPTransport(client),
            otelhttp.WithTracerProvider(otel.TracerProvider()),
            otelhttp.WithPropagators(otel.GetTextMapPropagator()),
            otelhttp.WithSpanNameFormatter(func(operation string, r *http.Request) string {
                return fmt.Sprintf("%s %s", r.Method, r.URL.Path)
            }),
        ),
        Timeout:   client.Timeout,
        Jar:       client.Jar,
        CheckRedirect: client.CheckRedirect,
    }
}

// Obtener transporte HTTP desde cliente, con fallback
func getHTTPTransport(client *http.Client) http.RoundTripper {
    if client.Transport != nil {
        return client.Transport
    }
    return http.DefaultTransport
}

// Iniciar span de OpenTelemetry para base de datos
func StartDBSpan(ctx context.Context, tracer trace.Tracer, operation, query string, args ...interface{}) (context.Context, trace.Span) {
    ctx, span := tracer.Start(ctx, "db."+operation,
        trace.WithAttributes(
            attribute.String("db.system", "postgresql"),
            attribute.String("db.operation", operation),
            attribute.String("db.statement", sanitizeQuery(query)),
        ),
        trace.WithSpanKind(trace.SpanKindClient),
    )
    
    // Solo incluir args en desarrollo
    if os.Getenv("ENVIRONMENT") != "production" && len(args) > 0 {
        span.SetAttributes(attribute.String("db.args", fmt.Sprintf("%v", args)))
    }
    
    return ctx, span
}

// Sanitizar query para trazas (eliminar información sensible)
func sanitizeQuery(query string) string {
    // Limitar longitud
    if len(query) > 1000 {
        query = query[:1000] + "..."
    }
    
    // Remover contraseñas, tokens, etc.
    passwordPattern := regexp.MustCompile(`(?i)(password|secret|token|key)([^\w]*)=['"]?[^\s,);]*['"]?`)
    return passwordPattern.ReplaceAllString(query, "$1$2=[REDACTED]")
}
```

### 13.3 Monitoreo con Prometheus

```go
// Configuración de métricas Prometheus
func SetupPrometheus(serviceName, serviceVersion string) (*prometheus.Registry, *promhttp.Handler, error) {
    // Registro de métricas
    reg := prometheus.NewRegistry()
    
    // Registrar colectores estándar
    reg.MustRegister(
        collectors.NewGoCollector(),
        collectors.NewProcessCollector(collectors.ProcessCollectorOpts{}),
    )
    
    // Registrar información de build
    buildInfo := prometheus.NewGaugeVec(
        prometheus.GaugeOpts{
            Namespace: sanitizeMetricName(serviceName),
            Name:      "build_info",
            Help:      "Información de build del servicio",
        },
        []string{"version", "commit", "build_date"},
    )
    reg.MustRegister(buildInfo)
    buildInfo.WithLabelValues(serviceVersion, GetGitCommit(), GetBuildDate()).Set(1)
    
    // Métricas específicas para modelo EAV
    setupEAVMetrics(reg, serviceName)
    
    // Métricas HTTP
    httpRequestsTotal := prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Namespace: sanitizeMetricName(serviceName),
            Name:      "http_requests_total",
            Help:      "Total de solicitudes HTTP recibidas",
        },
        []string{"method", "path", "status"},
    )
    
    httpRequestDuration := prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Namespace: sanitizeMetricName(serviceName),
            Name:      "http_request_duration_seconds",
            Help:      "Duración de solicitudes HTTP en segundos",
            Buckets:   []float64{.005, .01, .025, .05, .1, .25, .5, 1, 2.5, 5, 10},
        },
        []string{"method", "path", "status"},
    )
    
    // Métricas de base de datos
    databaseQueryDuration := prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Namespace: sanitizeMetricName(serviceName),
            Name:      "database_query_duration_seconds",
            Help:      "Duración de consultas de base de datos en segundos",
            Buckets:   []float64{.001, .005, .01, .025, .05, .1, .25, .5, 1, 2.5, 5},
        },
        []string{"operation", "table"},
    )
    
    // Métricas de caché
    cacheHitTotal := prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Namespace: sanitizeMetricName(serviceName),
            Name:      "cache_hit_total",
            Help:      "Total de aciertos de caché",
        },
        []string{"cache", "key_pattern"},
    )
    
    cacheMissTotal := prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Namespace: sanitizeMetricName(serviceName),
            Name:      "cache_miss_total",
            Help:      "Total de fallos de caché",
        },
        []string{"cache", "key_pattern"},
    )
    
	// Métricas para Circuit Breaker de caché
	cacheCircuitBreakerState := prometheus.NewGaugeVec(
		prometheus.GaugeOpts{
			Namespace: sanitizeMetricName(serviceName),
			Name:      "cache_circuit_breaker_state",
			Help:      "Estado del circuit breaker de caché (0=closed, 1=half-open, 2=open)",
		},
		[]string{"cache_key"},
	)

	cacheCircuitBreakerTrips := prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Namespace: sanitizeMetricName(serviceName),
			Name:      "cache_circuit_breaker_trips_total",
			Help:      "Número total de veces que se ha abierto el circuit breaker de caché",
		},
		[]string{"cache_key"},
	)

	cacheFallbacksTotal := prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Namespace: sanitizeMetricName(serviceName),
			Name:      "cache_fallbacks_total",
			Help:      "Número total de veces que se ha utilizado el fallback de caché",
		},
		[]string{"operation"},
	)

	cacheDegradationLevel := prometheus.NewGaugeVec(
		prometheus.GaugeOpts{
			Namespace: sanitizeMetricName(serviceName),
			Name:      "cache_degradation_level",
			Help:      "Nivel de degradación de la caché (0=normal, 1=degraded, 2=severe)",
		},
		[]string{},
	)


    // Registrar métricas
    reg.MustRegister(
        httpRequestsTotal,
        httpRequestDuration,
        databaseQueryDuration,
        cacheHitTotal,
        cacheMissTotal,
		cacheCircuitBreakerState,
		cacheCircuitBreakerTrips,
		cacheFallbacksTotal,
		cacheDegradationLevel
	)
  
    // Crear handler HTTP para métricas
    handler := promhttp.HandlerFor(reg, promhttp.HandlerOpts{
        Registry:          reg,
        EnableOpenMetrics: true,
    })
    
    return reg, &handler, nil
}

// Configurar métricas específicas para el modelo EAV
func setupEAVMetrics(reg *prometheus.Registry, serviceName string) {
    // Métricas para rendimiento EAV
    eavEntityReconstructionDuration := prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Namespace: sanitizeMetricName(serviceName),
            Name:      "eav_entity_reconstruction_duration_seconds",
            Help:      "Tiempo de reconstrucción de entidades EAV en segundos",
            Buckets:   []float64{.001, .005, .01, .025, .05, .1, .25, .5, 1},
        },
        []string{"table_name", "operation"},
    )
    
    eavTableRowsProcessed := prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Namespace: sanitizeMetricName(serviceName),
            Name:      "eav_table_rows_processed_total",
            Help:      "Número total de filas EAV procesadas",
        },
        []string{"table_name", "operation"},
    )
    
    eavColumnValuesPerRow := prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Namespace: sanitizeMetricName(serviceName),
            Name:      "eav_column_values_per_row",
            Help:      "Distribución de valores de columna por fila",
            Buckets:   []float64{1, 5, 10, 20, 50, 100, 200},
        },
        []string{"table_name"},
    )
    
    eavQueryComplexity := prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Namespace: sanitizeMetricName(serviceName),
            Name:      "eav_query_complexity",
            Help:      "Complejidad de consultas EAV (número de filtros y ordenamientos)",
            Buckets:   []float64{0, 1, 2, 3, 5, 10, 20},
        },
        []string{"table_name", "query_type"},
    )
    
    reg.MustRegister(
        eavEntityReconstructionDuration,
        eavTableRowsProcessed,
        eavColumnValuesPerRow,
        eavQueryComplexity,
    )
}

// Sanitizar nombre para métricas Prometheus
func sanitizeMetricName(name string) string {
    // Prometheus solo permite caracteres alfanuméricos y guiones bajos
    re := regexp.MustCompile(`[^a-zA-Z0-9_]`)
    return re.ReplaceAllString(name, "_")
}

// Middleware para métricas HTTP
func MetricsMiddleware(requestsTotal *prometheus.CounterVec, requestDuration *prometheus.HistogramVec) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()
            
            // Wrapper para capturar código de estado
            ww := middleware.NewWrapResponseWriter(w, r.ProtoMajor)
            
            // Ejecutar el siguiente handler
            next.ServeHTTP(ww, r)
            
            // Normalizar path para evitar cardinalidad alta en métricas
            path := NormalizePath(r.URL.Path)
            
            // Registrar métricas
            status := strconv.Itoa(ww.Status())
            requestsTotal.WithLabelValues(r.Method, path, status).Inc()
            requestDuration.WithLabelValues(r.Method, path, status).Observe(time.Since(start).Seconds())
        })
    }
}

// Implementación de Reporter de métricas con Prometheus
type PrometheusMetricsReporter struct {
    httpRequestsTotal          *prometheus.CounterVec
    httpRequestDuration        *prometheus.HistogramVec
    databaseQueryDuration      *prometheus.HistogramVec
    cacheHitTotal              *prometheus.CounterVec
    cacheMissTotal             *prometheus.CounterVec
    circuitBreakerState        *prometheus.GaugeVec
    eavEntityReconstructionDuration *prometheus.HistogramVec
    eavTableRowsProcessed      *prometheus.CounterVec
    eavColumnValuesPerRow      *prometheus.HistogramVec
    eavQueryComplexity         *prometheus.HistogramVec
}

// Registrar duración de reconstrucción EAV
func (r *PrometheusMetricsReporter) RecordEAVReconstructionDuration(tableName, operation string, duration time.Duration) {
    r.eavEntityReconstructionDuration.WithLabelValues(tableName, operation).Observe(duration.Seconds())
}

// Registrar filas EAV procesadas
func (r *PrometheusMetricsReporter) RecordEAVRowsProcessed(tableName, operation string, count int) {
    r.eavTableRowsProcessed.WithLabelValues(tableName, operation).Add(float64(count))
}

// Registrar número de valores de columna por fila
func (r *PrometheusMetricsReporter) RecordEAVColumnValuesPerRow(tableName string, count int) {
    r.eavColumnValuesPerRow.WithLabelValues(tableName).Observe(float64(count))
}

// Registrar complejidad de consulta EAV
func (r *PrometheusMetricsReporter) RecordEAVQueryComplexity(tableName, queryType string, filtersCount, sortingsCount int) {
    complexity := float64(filtersCount + sortingsCount)
    r.eavQueryComplexity.WithLabelValues(tableName, queryType).Observe(complexity)
}

// Registrar estado de circuit breaker
func (r *PrometheusMetricsReporter) RecordCircuitBreakerStateChange(name, from, to string) {
    // Mapeando estados a valores numéricos
    states := map[string]float64{
        "closed":    0,
        "half-open": 0.5,
        "open":      1,
    }
    
    if value, ok := states[to]; ok {
        r.circuitBreakerState.WithLabelValues(name).Set(value)
    }
}
```

### 13.4 Configuración de Health Checks

```go
// Servicio de health checks
type HealthService struct {
    db          *sqlx.DB
    redisClient *redis.Client
    checkers    map[string]HealthChecker
    logger      *zap.Logger
    metrics     metrics.Reporter
    tracer      trace.Tracer
}

// Interfaz para comprobaciones de salud
type HealthChecker interface {
    Check(ctx context.Context) error
    Name() string
    Type() string // "liveness" o "readiness"
}

// Resultado de health check
type HealthStatus struct {
    Status     string                 `json:"status"`
    Components map[string]ComponentStatus `json:"components"`
    Timestamp  time.Time              `json:"timestamp"`
}

// Estado de componente individual
type ComponentStatus struct {
    Status  string `json:"status"`
    Message string `json:"message,omitempty"`
    Type    string `json:"type"`
}

// Crear servicio de health check
func NewHealthService(
    db *sqlx.DB,
    redisClient *redis.Client,
    logger *zap.Logger,
    metrics metrics.Reporter,
    tracer trace.Tracer,
) *HealthService {
    service := &HealthService{
        db:          db,
        redisClient: redisClient,
        checkers:    make(map[string]HealthChecker),
        logger:      logger,
        metrics:     metrics,
        tracer:      tracer,
    }
    
    // Registrar checkers por defecto
    service.RegisterChecker(NewPostgreSQLHealthChecker(db))
    service.RegisterChecker(NewRedisHealthChecker(redisClient))
    service.RegisterChecker(NewDiskSpaceHealthChecker("/", 10)) // 10% mínimo libre
    
    return service
}

// Registrar checker personalizado
func (s *HealthService) RegisterChecker(checker HealthChecker) {
    s.checkers[checker.Name()] = checker
}

// Implementación para PostgreSQL
type PostgreSQLHealthChecker struct {
    db *sqlx.DB
}

func NewPostgreSQLHealthChecker(db *sqlx.DB) *PostgreSQLHealthChecker {
    return &PostgreSQLHealthChecker{db: db}
}

func (c *PostgreSQLHealthChecker) Check(ctx context.Context) error {
    // Verificar con timeout
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()
    
    // Consulta simple para verificar conexión
    var result int
    err := c.db.GetContext(ctx, &result, "SELECT 1")
    if err != nil {
        return fmt.Errorf("error conectando a PostgreSQL: %w", err)
    }
    
    // Verificar también estadísticas de conexiones
    stats := c.db.Stats()
    if stats.Idle == 0 && stats.OpenConnections >= int(float64(stats.MaxOpenConnections)*0.9) {
        return fmt.Errorf("pool de conexiones PostgreSQL casi agotado: %d/%d", 
            stats.OpenConnections, stats.MaxOpenConnections)
    }
    
    return nil
}

func (c *PostgreSQLHealthChecker) Name() string {
    return "postgresql"
}

func (c *PostgreSQLHealthChecker) Type() string {
    return "readiness"
}

// Implementación para Redis
type RedisHealthChecker struct {
    client *redis.Client
}

func NewRedisHealthChecker(client *redis.Client) *RedisHealthChecker {
    return &RedisHealthChecker{client: client}
}

func (c *RedisHealthChecker) Check(ctx context.Context) error {
    // Verificar con timeout
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()
    
    // Ping simple
    status := c.client.Ping(ctx)
    if err := status.Err(); err != nil {
        return fmt.Errorf("error conectando a Redis: %w", err)
    }
    
    return nil
}

func (c *RedisHealthChecker) Name() string {
    return "redis"
}

func (c *RedisHealthChecker) Type() string {
    return "readiness"
}

// Implementación para espacio en disco
type DiskSpaceHealthChecker struct {
    path       string
    minPercent float64
}

func NewDiskSpaceHealthChecker(path string, minPercentFree float64) *DiskSpaceHealthChecker {
    return &DiskSpaceHealthChecker{
        path:       path,
        minPercent: minPercentFree,
    }
}

func (c *DiskSpaceHealthChecker) Check(ctx context.Context) error {
    var stat syscall.Statfs_t
    if err := syscall.Statfs(c.path, &stat); err != nil {
        return fmt.Errorf("error obteniendo estadísticas de disco: %w", err)
    }
    
    // Calcular porcentaje libre
    total := float64(stat.Blocks) * float64(stat.Bsize)
    free := float64(stat.Bavail) * float64(stat.Bsize)
    percentFree := (free / total) * 100
    
    if percentFree < c.minPercent {
        return fmt.Errorf("espacio en disco bajo: %.2f%% libre, mínimo requerido: %.2f%%", 
            percentFree, c.minPercent)
    }
    
    return nil
}

func (c *DiskSpaceHealthChecker) Name() string {
    return "disk-space"
}

func (c *DiskSpaceHealthChecker) Type() string {
    return "liveness"
}

// Handler HTTP para health checks
func (s *HealthService) HealthCheckHandler(w http.ResponseWriter, r *http.Request) {
    ctx, cancel := context.WithTimeout(r.Context(), 5*time.Second)
    defer cancel()
    
    // Iniciar span para trazabilidad
    ctx, span := s.tracer.Start(ctx, "HealthService.HealthCheck")
    defer span.End()
    
    // Verificar si es liveness o readiness
    isLiveness := strings.Contains(r.URL.Path, "/health/live")
    
    // Registrar métricas
    s.metrics.RecordHealthCheck(isLiveness)
    
    // Para liveness, solo verificar que el servicio esté respondiendo
    if isLiveness {
        // Verificar solo checkers de tipo "liveness"
        status := s.performChecks(ctx, func(checker HealthChecker) bool {
            return checker.Type() == "liveness"
        })
        
        respondWithHealthStatus(w, status)
        return
    }
    
    // Para readiness, verificar todas las dependencias
    status := s.performChecks(ctx, func(checker HealthChecker) bool {
        return true // verificar todos
    })
    
    // Determinar código HTTP
    httpStatus := http.StatusOK
    if status.Status != "UP" {
        httpStatus = http.StatusServiceUnavailable
    }
    
    // Registrar resultado
    s.metrics.RecordHealthStatus(status.Status == "UP")
    
    respondWithHealthStatus(w, status, httpStatus)
}

// Realizar comprobaciones de salud
func (s *HealthService) performChecks(ctx context.Context, filter func(HealthChecker) bool) HealthStatus {
    status := HealthStatus{
        Status:     "UP",
        Components: make(map[string]ComponentStatus),
        Timestamp:  time.Now(),
    }
    
    // Verificar cada componente
    for name, checker := range s.checkers {
        // Aplicar filtro
        if !filter(checker) {
            continue
        }
        
        // Verificar componente
        err := checker.Check(ctx)
        componentStatus := ComponentStatus{
            Type: checker.Type(),
        }
        
        if err != nil {
            componentStatus.Status = "DOWN"
            componentStatus.Message = err.Error()
            status.Status = "DOWN"
            
            s.logger.Warn("Componente no saludable", 
                zap.String("component", name),
                zap.Error(err))
        } else {
            componentStatus.Status = "UP"
        }
        
        status.Components[name] = componentStatus
    }
    
    return status
}

// Responder con estado de salud
func respondWithHealthStatus(w http.ResponseWriter, status HealthStatus, httpStatus ...int) {
    code := http.StatusOK
    if len(httpStatus) > 0 {
        code = httpStatus[0]
    }
    
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(code)
    
    if err := json.NewEncoder(w).Encode(status); err != nil {
        w.WriteHeader(http.StatusInternalServerError)
        w.Write([]byte(`{"status":"ERROR","message":"Error encoding health status"}`))
    }
}
```

### 13.5 Métricas Específicas para Modelo EAV

```go
// Configuración de métricas específicas para EAV
type EAVMetrics struct {
    // Rendimiento de reconstrucción
    EntityReconstructionDuration *prometheus.HistogramVec
    
    // Estadísticas de entidades
    ColumnValuesPerRow *prometheus.HistogramVec
    TableRowsProcessed *prometheus.CounterVec
    
    // Complejidad de consultas
    QueryFilterCount *prometheus.HistogramVec
    QuerySortCount *prometheus.HistogramVec
    QueryExecutionTime *prometheus.HistogramVec
    
    // Caché
    CacheHitRate *prometheus.GaugeVec
    CacheInvalidations *prometheus.CounterVec
    
    // Operaciones por tabla
    TableOperations *prometheus.CounterVec
}

// Crear e inicializar métricas EAV
func NewEAVMetrics(reg *prometheus.Registry, namespace string) *EAVMetrics {
    metrics := &EAVMetrics{
        EntityReconstructionDuration: prometheus.NewHistogramVec(
            prometheus.HistogramOpts{
                Namespace: namespace,
                Name:      "eav_entity_reconstruction_seconds",
                Help:      "Tiempo de reconstrucción de entidades EAV",
                Buckets:   []float64{0.001, 0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5},
            },
            []string{"table_name", "operation_type"},
        ),
        
        ColumnValuesPerRow: prometheus.NewHistogramVec(
            prometheus.HistogramOpts{
                Namespace: namespace,
                Name:      "eav_column_values_per_row",
                Help:      "Distribución de valores de columna por fila",
                Buckets:   []float64{1, 5, 10, 20, 50, 100},
            },
            []string{"table_name"},
        ),
        
        TableRowsProcessed: prometheus.NewCounterVec(
            prometheus.CounterOpts{
                Namespace: namespace,
                Name:      "eav_table_rows_processed_total",
                Help:      "Número total de filas EAV procesadas",
            },
            []string{"table_name", "operation"},
        ),
        
        QueryFilterCount: prometheus.NewHistogramVec(
            prometheus.HistogramOpts{
                Namespace: namespace,
                Name:      "eav_query_filter_count",
                Help:      "Número de filtros en consultas EAV",
                Buckets:   []float64{0, 1, 2, 3, 5, 10, 20},
            },
            []string{"table_name"},
        ),
        
        QuerySortCount: prometheus.NewHistogramVec(
            prometheus.HistogramOpts{
                Namespace: namespace,
                Name:      "eav_query_sort_count",
                Help:      "Número de ordenamientos en consultas EAV",
                Buckets:   []float64{0, 1, 2, 3, 5},
            },
            []string{"table_name"},
        ),
        
        QueryExecutionTime: prometheus.NewHistogramVec(
            prometheus.HistogramOpts{
                Namespace: namespace,
                Name:      "eav_query_execution_seconds",
                Help:      "Tiempo de ejecución de consultas EAV",
                Buckets:   []float64{0.001, 0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10},
            },
            []string{"table_name", "query_type", "complexity"},
        ),
        
        CacheHitRate: prometheus.NewGaugeVec(
            prometheus.GaugeOpts{
                Namespace: namespace,
                Name:      "eav_cache_hit_rate",
                Help:      "Tasa de aciertos de caché para entidades EAV",
            },
            []string{"table_name", "cache_level"},
        ),
        
        CacheInvalidations: prometheus.NewCounterVec(
            prometheus.CounterOpts{
                Namespace: namespace,
                Name:      "eav_cache_invalidations_total",
                Help:      "Número total de invalidaciones de caché para entidades EAV",
            },
            []string{"table_name", "reason"},
        ),
        
        TableOperations: prometheus.NewCounterVec(
            prometheus.CounterOpts{
                Namespace: namespace,
                Name:      "eav_table_operations_total",
                Help:      "Número total de operaciones por tabla lógica",
            },
            []string{"table_name", "operation_type"},
        ),
    }
    
    // Registrar todas las métricas
    reg.MustRegister(
        metrics.EntityReconstructionDuration,
        metrics.ColumnValuesPerRow,
        metrics.TableRowsProcessed,
        metrics.QueryFilterCount,
        metrics.QuerySortCount,
        metrics.QueryExecutionTime,
        metrics.CacheHitRate,
        metrics.CacheInvalidations,
        metrics.TableOperations,
    )
    
    return metrics
}
```

## 14. Estrategia de Pruebas

Añadir a la descripción de las pruebas unitarias, de integración y E2E:
*   **Pruebas de Autenticación con Contraseña:** Se incluirán casos de prueba específicos para:
    *   Validación y hashing de contraseñas (unitarias).
    *   Flujo completo de login con usuario/contraseña (integración, E2E).
    *   Manejo de intentos fallidos y bloqueo de cuenta (integración).
    *   Pruebas de seguridad contra fuerza bruta y otros ataques comunes (pruebas de seguridad/penetración).
	
### 14.1 Pruebas Unitarias

```go
// Ejemplo de prueba unitaria para un servicio
func TestTableRowService_Create(t *testing.T) {
    // Configurar mocks
    mockUow := mocks.NewMockUnitOfWork(t)
    mockTableRowRepo := mocks.NewMockTableRowRepository(t)
    mockLogicalTableRepo := mocks.NewMockLogicalTableRepository(t)
    mockColumnDefRepo := mocks.NewMockColumnDefinitionRepository(t)
    mockEventBus := mocks.NewMockEventBus(t)
    mockOutboxRepo := mocks.NewMockOutboxRepository(t)
    mockMetrics := mocks.NewMockMetricsReporter(t)
    mockTracer := mocks.NewMockTracer(t)
    mockSpan := mocks.NewMockSpan(t)
    
    // Configurar expectativas
    mockUow.EXPECT().TableRows().Return(mockTableRowRepo).AnyTimes()
    mockUow.EXPECT().LogicalTables().Return(mockLogicalTableRepo).AnyTimes()
    mockUow.EXPECT().ColumnDefinitions().Return(mockColumnDefRepo).AnyTimes()
    mockUow.EXPECT().OutboxEntries().Return(mockOutboxRepo).AnyTimes()
    mockTracer.EXPECT().Start(mock.Anything, "TableRowService.Create", mock.Anything).Return(context.Background(), mockSpan)
    mockSpan.EXPECT().End().Once()
    
    // Crear ID de tabla lógica dummy
    tableID := uuid.New()
    
    // Configurar LogicalTable mock para devolver datos dummy
    logicalTable := &domain.LogicalTable{
        ID:          tableID,
        TableName:   "TEST_TABLE",
        Description: "Test Table",
    }
    mockLogicalTableRepo.EXPECT().FindByName(mock.Anything, "TEST_TABLE").Return(logicalTable, nil)
    
    // Configurar ColumnDefinition mock
    colDef := &domain.ColumnDefinition{
        ID:         uuid.New(),
        ColumnName: "test_column",
        DataType:   "String",
    }
    mockColumnDefRepo.EXPECT().FindByName(mock.Anything, "test_column").Return(colDef, nil)
    
    // Configurar mock para creación de TableRow
    rowID := uuid.New()
    mockTableRowRepo.EXPECT().Create(mock.Anything, mock.AnythingOfType("*domain.TableRow")).Run(func(ctx context.Context, row *domain.TableRow) {
        // Asignar ID simulando la persistencia
        row.ID = rowID
    }).Return(nil)
    
    // Configurar expectativa para UnitOfWork.WithTransaction
    mockUow.EXPECT().WithTransaction(mock.Anything, mock.AnythingOfType("func(context.Context) error")).
        Run(func(ctx context.Context, fn func(context.Context) error) {
            // Ejecutar función en contexto de transacción
            _ = fn(ctx)
        }).Return(nil)
    
    // Configurar expectativa para OutboxRepository.CreateEntry
    mockOutboxRepo.EXPECT().CreateEntry(
        mock.Anything, 
        "TableRowCreated", 
        mock.AnythingOfType("uuid.UUID"), 
        mock.AnythingOfType("domain.TableRowCreatedEvent"),
    ).Return(nil)
    
    // Configurar expectativas para métricas
    mockMetrics.EXPECT().RecordRepositoryOperation("TableRow", "create").Once()
    mockMetrics.EXPECT().RecordServiceDuration("TableRowService", "Create", mock.Anything).Once()
    
    // Crear servicio con dependencias mockeadas
    service := NewTableRowService(mockUow, mockEventBus, mockMetrics, mockTracer, nil)
    
    // Crear comando de prueba
    cmd := &application.CreateTableRowCommand{
        LogicalTableName: "TEST_TABLE",
        Code:             "TEST-01",
        DisplayValue:     "Test Row",
        IsActive:         true,
        ColumnValues: map[string]interface{}{
            "test_column": "test value",
        },
    }
    
    // Ejecutar método bajo prueba
    row, err := service.Create(context.Background(), cmd)
    
    // Verificar resultado
    assert.NoError(t, err)
    assert.NotNil(t, row)
    assert.Equal(t, rowID, row.ID)
    assert.Equal(t, "TEST-01", row.Code)
    assert.Equal(t, "Test Row", row.DisplayValue)
    assert.True(t, row.IsActive)
    assert.Equal(t, "test value", row.Fields["test_column"])
}
```

### 14.2 Pruebas de Integración

```go
// Ejemplo de prueba de integración usando contenedores
func TestTableRowRepositoryIntegration(t *testing.T) {
    if testing.Short() {
        t.Skip("Saltando prueba de integración en modo corto")
    }
    
    // Iniciar contenedor PostgreSQL
    ctx := context.Background()
    postgres, err := testcontainers.GenericContainer(ctx, testcontainers.GenericContainerRequest{
        ContainerRequest: testcontainers.ContainerRequest{
            Image:        "postgres:16-alpine",
            ExposedPorts: []string{"5432/tcp"},
            Env: map[string]string{
                "POSTGRES_PASSWORD": "testpassword",
                "POSTGRES_USER":     "testuser",
                "POSTGRES_DB":       "testdb",
            },
            WaitingFor: wait.ForLog("database system is ready to accept connections"),
        },
        Started: true,
    })
    if err != nil {
        t.Fatalf("No se pudo iniciar contenedor PostgreSQL: %v", err)
    }
    defer postgres.Terminate(ctx)
    
    // Obtener host y puerto mapeados
    host, err := postgres.Host(ctx)
    if err != nil {
        t.Fatalf("Error obteniendo host: %v", err)
    }
    
    port, err := postgres.MappedPort(ctx, "5432")
    if err != nil {
        t.Fatalf("Error obteniendo puerto: %v", err)
    }
    
    // Construir DSN
    dsn := fmt.Sprintf("postgres://testuser:testpassword@%s:%s/testdb?sslmode=disable", host, port.Port())
    
    // Conectar a la BD
    db, err := sqlx.Connect("postgres", dsn)
    if err != nil {
        t.Fatalf("Error conectando a PostgreSQL: %v", err)
    }
    defer db.Close()
    
    // Ejecutar migraciones
    migrator := &PostgreSQLMigrator{
        DB:         db,
        MigrationDir: "./migrations",
    }
    err = migrator.MigrateUp()
    if err != nil {
        t.Fatalf("Error aplicando migraciones: %v", err)
    }
    
    // Crear datos de prueba
    logicalTableID := uuid.New()
    
    // Insertar tabla lógica
    _, err = db.Exec(`
        INSERT INTO logical_tables (id, table_name, description, is_hierarchical)
        VALUES ($1, $2, $3, $4)
    `, logicalTableID, "TEST_TABLE", "Test Table", false)
    if err != nil {
        t.Fatalf("Error insertando tabla lógica: %v", err)
    }
    
    // Insertar definición de columna
    columnDefID := uuid.New()
    _, err = db.Exec(`
        INSERT INTO column_definitions (id, column_name, data_type, description)
        VALUES ($1, $2, $3, $4)
    `, columnDefID, "test_column", "String", "Test Column")
    if err != nil {
        t.Fatalf("Error insertando definición de columna: %v", err)
    }
    
    // Relacionar columna con tabla
    _, err = db.Exec(`
        INSERT INTO table_column_properties 
        (logical_table_id, column_definition_id, display_order, is_mandatory)
        VALUES ($1, $2, $3, $4)
    `, logicalTableID, columnDefID, 1, false)
    if err != nil {
        t.Fatalf("Error insertando propiedad de columna: %v", err)
    }
    
    // Inicializar tracer y métricas para pruebas
    tracer := trace.NewNoopTracerProvider().Tracer("test")
    metrics := NewNoopMetricsReporter()
    
    // Configurar logger para pruebas
    logger, _ := zap.NewDevelopment()
    
    // Crear repositorio a probar
    repo := NewTableRowRepository(db, logger, metrics, tracer)
    
    // Crear fila de prueba
    rowID := uuid.New()
    row := &domain.TableRow{
        ID:             rowID,
        LogicalTableID: logicalTableID,
        Code:           "TEST-01",
        DisplayValue:   "Test Row",
        SortOrder:      1,
        IsActive:       true,
        ColumnValues: []domain.ColumnValue{
            {
                RowID:             rowID,
                ColumnDefinitionID: columnDefID,
                StringValue:       "test value",
            },
        },
    }
    
    // Probar método Create
    err = repo.Create(ctx, row)
    if err != nil {
        t.Fatalf("Error creando fila: %v", err)
    }
    
    // Probar método FindByID
    foundRow, err := repo.FindByID(ctx, rowID)
    if err != nil {
        t.Fatalf("Error buscando fila: %v", err)
    }
    
    // Verificar datos
    assert.Equal(t, rowID, foundRow.ID)
    assert.Equal(t, "TEST-01", foundRow.Code)
    assert.Equal(t, "Test Row", foundRow.DisplayValue)
    assert.Equal(t, 1, foundRow.SortOrder)
    assert.True(t, foundRow.IsActive)
    
    // Verificar valores de columna
    assert.Len(t, foundRow.ColumnValues, 1)
    assert.Equal(t, columnDefID, foundRow.ColumnValues[0].ColumnDefinitionID)
    assert.Equal(t, "test value", foundRow.ColumnValues[0].StringValue)
}
```

### 14.3 Pruebas E2E para API

```go
// Ejemplo de prueba end-to-end para API de datos
func TestDataAPI_CreateRow_E2E(t *testing.T) {
    if testing.Short() {
        t.Skip("Saltando prueba E2E en modo corto")
    }
    
    // Iniciar infraestructura de prueba (containers)
    infra, err := setupTestInfrastructure(t)
    if err != nil {
        t.Fatalf("Error configurando infraestructura de prueba: %v", err)
    }
    defer infra.Cleanup()
    
    // Iniciar la aplicación con la infraestructura de prueba
    app, err := NewTestApplication(infra)
    if err != nil {
        t.Fatalf("Error creando aplicación de prueba: %v", err)
    }
    
    // Iniciar servidor HTTP
    server := app.StartTestServer()
    defer server.Close()
    
    // Configurar cliente HTTP
    client := resty.New().
        SetBaseURL(server.URL).
        SetTimeout(5 * time.Second)
    
    // Crear tabla lógica de prueba
    tableResp, err := client.R().
        SetBody(map[string]interface{}{
            "table_name":     "PRODUCTS",
            "description":    "Test Products Table",
            "is_hierarchical": false,
        }).
        SetAuthToken(infra.AdminToken).
        Post("/api/v1/metadata/tables")
    
    if err != nil || tableResp.StatusCode() != http.StatusCreated {
        t.Fatalf("Error creando tabla lógica: %v, status: %d", err, tableResp.StatusCode())
    }
    
    var tableResult map[string]interface{}
    if err := json.Unmarshal(tableResp.Body(), &tableResult); err != nil {
        t.Fatalf("Error decodificando respuesta: %v", err)
    }
    
    tableID := tableResult["id"].(string)
    
    // Crear definición de columna
    colResp, err := client.R().
        SetBody(map[string]interface{}{
            "column_name":  "price",
            "data_type":    "Decimal",
            "description":  "Product price",
        }).
        SetAuthToken(infra.AdminToken).
        Post("/api/v1/metadata/columns")
        
    if err != nil || colResp.StatusCode() != http.StatusCreated {
        t.Fatalf("Error creando definición de columna: %v, status: %d", err, colResp.StatusCode())
    }
    
    var colResult map[string]interface{}
    if err := json.Unmarshal(colResp.Body(), &colResult); err != nil {
        t.Fatalf("Error decodificando respuesta: %v", err)
    }
    
    columnID := colResult["id"].(string)
    
    // Asociar columna a tabla
    propResp, err := client.R().
        SetBody(map[string]interface{}{
            "logical_table_id":      tableID,
            "column_definition_id":  columnID,
            "display_order":         1,
            "is_mandatory":          true,
        }).
        SetAuthToken(infra.AdminToken).
        Post("/api/v1/metadata/properties")
        
    if err != nil || propResp.StatusCode() != http.StatusCreated {
        t.Fatalf("Error asociando columna a tabla: %v, status: %d", err, propResp.StatusCode())
    }
    
    // Crear una nueva fila
    rowResp, err := client.R().
        SetBody(map[string]interface{}{
            "code":          "PROD-001",
            "display_value": "Test Product",
            "is_active":     true,
            "column_values": map[string]interface{}{
                "price": 29.99,
            },
        }).
        SetAuthToken(infra.AdminToken).
        Post("/api/v1/data/PRODUCTS")
        
    if err != nil || rowResp.StatusCode() != http.StatusCreated {
        t.Fatalf("Error creando fila: %v, status: %d", err, rowResp.StatusCode())
    }
    
    var rowResult map[string]interface{}
    if err := json.Unmarshal(rowResp.Body(), &rowResult); err != nil {
        t.Fatalf("Error decodificando respuesta: %v", err)
    }
    
    rowID := rowResult["id"].(string)
    
    // Verificar que la fila se creó correctamente
    getResp, err := client.R().
        SetAuthToken(infra.AdminToken).
        Get(fmt.Sprintf("/api/v1/data/PRODUCTS/%s", rowID))
        
    if err != nil || getResp.StatusCode() != http.StatusOK {
        t.Fatalf("Error obteniendo fila: %v, status: %d", err, getResp.StatusCode())
    }
    
    var getResult map[string]interface{}
    if err := json.Unmarshal(getResp.Body(), &getResult); err != nil {
        t.Fatalf("Error decodificando respuesta: %v", err)
    }
    
    // Verificar datos
    assert.Equal(t, "PROD-001", getResult["code"])
    assert.Equal(t, "Test Product", getResult["display_value"])
    assert.Equal(t, true, getResult["is_active"])
    
    // Verificar valores de columna
    columnValues := getResult["column_values"].(map[string]interface{})
    assert.Equal(t, 29.99, columnValues["price"])
}

// Estructura para infraestructura de prueba
type TestInfrastructure struct {
    PostgresContainer testcontainers.Container
    RedisContainer    testcontainers.Container
    AdminToken        string
    PostgresURI       string
    RedisURI          string
    Cleanup           func()
}

// Configurar infraestructura de prueba con contenedores
// Configurar infraestructura de prueba con contenedores
func setupTestInfrastructure(t *testing.T) (*TestInfrastructure, error) {
    // Iniciar contenedor PostgreSQL
    ctx := context.Background()
    postgres, err := testcontainers.GenericContainer(ctx, testcontainers.GenericContainerRequest{
        ContainerRequest: testcontainers.ContainerRequest{
            Image:        "postgres:16-alpine",
            ExposedPorts: []string{"5432/tcp"},
            Env: map[string]string{
                "POSTGRES_PASSWORD": "testpassword",
                "POSTGRES_USER":     "testuser",
                "POSTGRES_DB":       "testdb",
            },
            WaitingFor: wait.ForLog("database system is ready to accept connections"),
        },
        Started: true,
    })
    if err != nil {
        return nil, fmt.Errorf("error iniciando PostgreSQL: %w", err)
    }
    
    // Iniciar contenedor Redis
    redis, err := testcontainers.GenericContainer(ctx, testcontainers.GenericContainerRequest{
        ContainerRequest: testcontainers.ContainerRequest{
            Image:        "redis:7-alpine",
            ExposedPorts: []string{"6379/tcp"},
            WaitingFor:   wait.ForLog("Ready to accept connections"),
        },
        Started: true,
    })
    if err != nil {
        postgres.Terminate(ctx)
        return nil, fmt.Errorf("error iniciando Redis: %w", err)
    }
    
    // Obtener conexiones
    pgHost, _ := postgres.Host(ctx)
    pgPort, _ := postgres.MappedPort(ctx, "5432")
    pgURI := fmt.Sprintf("postgres://testuser:testpassword@%s:%s/testdb?sslmode=disable", pgHost, pgPort.Port())
    
    redisHost, _ := redis.Host(ctx)
    redisPort, _ := redis.MappedPort(ctx, "6379")
    redisURI := fmt.Sprintf("redis://%s:%s/0", redisHost, redisPort.Port())
    
    // Crear token de admin para pruebas
    adminToken := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
        "sub":  "admin-test-user",
        "role": "admin",
        "exp":  time.Now().Add(time.Hour).Unix(),
    })
    
    adminTokenString, err := adminToken.SignedString([]byte("test-secret-key"))
    if err != nil {
        postgres.Terminate(ctx)
        redis.Terminate(ctx)
        return nil, fmt.Errorf("error generando token admin: %w", err)
    }
    
    cleanup := func() {
        postgres.Terminate(ctx)
        redis.Terminate(ctx)
    }
    
    return &TestInfrastructure{
        PostgresContainer: postgres,
        RedisContainer:    redis,
        AdminToken:        adminTokenString,
        PostgresURI:       pgURI,
        RedisURI:          redisURI,
        Cleanup:           cleanup,
    }, nil
}

// Aplicación de prueba para E2E
type TestApplication struct {
    DB           *sqlx.DB
    RedisClient  *redis.Client
    Router       chi.Router
    Services     *ServiceContainer
    AuthManager  *PasskeyService
    Config       *TestConfig
}

// Configuración de prueba
type TestConfig struct {
    LogLevel        string
    DisableMetrics  bool
    DisableTracing  bool
    MockExternals   bool
}

// Crear aplicación de prueba
func NewTestApplication(infra *TestInfrastructure) (*TestApplication, error) {
    // Conectar a la base de datos
    db, err := sqlx.Connect("postgres", infra.PostgresURI)
    if err != nil {
        return nil, fmt.Errorf("error conectando a PostgreSQL: %w", err)
    }
    
    // Configurar pool de conexiones
    db.SetMaxOpenConns(25)
    db.SetMaxIdleConns(5)
    db.SetConnMaxLifetime(5 * time.Minute)
    
    // Conectar a Redis
    opt, err := redis.ParseURL(infra.RedisURI)
    if err != nil {
        return nil, fmt.Errorf("error parseando URI de Redis: %w", err)
    }
    
    redisClient := redis.NewClient(opt)
    
    // Verificar conexión a Redis
    if _, err := redisClient.Ping(context.Background()).Result(); err != nil {
        return nil, fmt.Errorf("error conectando a Redis: %w", err)
    }
    
    // Configurar logger de prueba
    logger, _ := zap.NewDevelopment()
    
    // Crear router
    router := chi.NewRouter()
    
    // Middleware de prueba
    router.Use(
        middleware.RequestID,
        middleware.RealIP,
        middleware.Logger,
        middleware.Recoverer,
    )
    
    // Métricas para pruebas
    metrics := NewNoopMetricsReporter()
    
    // Tracer para pruebas
    tracer := trace.NewNoopTracerProvider().Tracer("test")
    
    // Caché para pruebas
    cacheManager, err := NewMultiLevelCacheManager(10000, redisClient, logger, metrics, tracer)
    if err != nil {
        return nil, fmt.Errorf("error creando cache manager: %w", err)
    }
    
    // Servicios para pruebas
    services := &ServiceContainer{
        // ... inicialización de servicios
    }
    
    config := &TestConfig{
        LogLevel:       "debug",
        DisableMetrics: true,
        DisableTracing: true,
        MockExternals:  true,
    }
    
    return &TestApplication{
        DB:           db,
        RedisClient:  redisClient,
        Router:       router,
        Services:     services,
        Config:       config,
    }, nil
}

// Iniciar servidor HTTP de prueba
func (app *TestApplication) StartTestServer() *httptest.Server {
    RegisterAPIRoutes(app.Router, app.Services)
    
    server := httptest.NewServer(app.Router)
    
    return server
}

// Migrador de esquema para pruebas
type PostgreSQLMigrator struct {
    DB          *sqlx.DB
    MigrationDir string
}

// Aplicar migraciones hacia arriba
func (m *PostgreSQLMigrator) MigrateUp() error {
    driver, err := postgres.WithInstance(m.DB.DB, &postgres.Config{})
    if err != nil {
        return fmt.Errorf("error creando driver de migración: %w", err)
    }
    
    migrator, err := migrate.NewWithDatabaseInstance(
        fmt.Sprintf("file://%s", m.MigrationDir),
        "postgres", driver)
    if err != nil {
        return fmt.Errorf("error creando instancia de migración: %w", err)
    }
    
    if err := migrator.Up(); err != nil && err != migrate.ErrNoChange {
        return fmt.Errorf("error aplicando migraciones: %w", err)
    }
    
    return nil
}

// Reporter de métricas para pruebas
type NoopMetricsReporter struct{}

func NewNoopMetricsReporter() *NoopMetricsReporter {
    return &NoopMetricsReporter{}
}

// Implementación de métodos sin operación
func (r *NoopMetricsReporter) RecordRepositoryOperation(repo string, operation string) {}
func (r *NoopMetricsReporter) RecordRepositoryError(repo string, errorType string) {}
func (r *NoopMetricsReporter) RecordServiceDuration(service string, method string, duration time.Duration) {}
func (r *NoopMetricsReporter) RecordCacheHit(cacheType string) {}
func (r *NoopMetricsReporter) RecordCacheMiss(cacheType string) {}
func (r *NoopMetricsReporter) RecordCacheError(errorType string) {}
func (r *NoopMetricsReporter) RecordPasskeyRegistrationStart(username string) {}
func (r *NoopMetricsReporter) RecordPasskeyRegistrationSuccess(username string, provider string) {}
func (r *NoopMetricsReporter) RecordPasskeyRegistrationFailure(username string, reason string) {}
// ... otros métodos de métricas

14.4 Pruebas de Rendimiento
Las pruebas de rendimiento son fundamentales para validar que el sistema EAV Híbrido cumple con los requisitos de escalabilidad y velocidad. Se implementan varios niveles de pruebas de rendimiento:
14.4.1 Pruebas de Carga
Se utilizan herramientas como k6 para simular cargas de usuario realistas:
// Ejemplo de script k6 para pruebas de carga
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
    stages: [
        { duration: '2m', target: 100 }, // Rampa hasta 100 usuarios
        { duration: '5m', target: 100 }, // Mantener 100 usuarios
        { duration: '2m', target: 200 }, // Rampa hasta 200 usuarios
        { duration: '5m', target: 200 }, // Mantener 200 usuarios
        { duration: '2m', target: 0 },   // Rampa a 0 usuarios
    ],
    thresholds: {
        'http_req_duration': ['p(95)<500'], // 95% de las peticiones deben completarse en menos de 500ms
        'http_req_failed': ['rate<0.01'],    // Menos del 1% de fallos
    },
};

const BASE_URL = __ENV.BASE_URL || 'http://localhost:8080';
const AUTH_TOKEN = __ENV.AUTH_TOKEN || 'test-token';

export default function() {
    // Consulta por tabla lógica (caso común)
    let tableResponse = http.get(`${BASE_URL}/api/v1/data/PRODUCTS?pageSize=20`, {
        headers: {
            'Authorization': `Bearer ${AUTH_TOKEN}`,
        },
    });
    
    check(tableResponse, {
        'table query status is 200': (r) => r.status === 200,
        'table query response time < 200ms': (r) => r.timings.duration < 200,
    });
    
    sleep(1);
    
    // Consulta por registro individual (caso común)
    let rowResponse = http.get(`${BASE_URL}/api/v1/data/PRODUCTS/e7f88c5d-9f14-4e77-a568-49a3843ab4f0`, {
        headers: {
            'Authorization': `Bearer ${AUTH_TOKEN}`,
        },
    });
    
    check(rowResponse, {
        'row query status is 200': (r) => r.status === 200,
        'row query response time < 100ms': (r) => r.timings.duration < 100,
    });
    
    sleep(1);
    
    // Creación de registros (menos frecuente)
    let createPayload = JSON.stringify({
        code: `PROD-${Math.floor(Math.random() * 100000)}`,
        display_value: `Test Product ${Math.floor(Math.random() * 10000)}`,
        is_active: true,
        column_values: {
            price: 29.99 + Math.random() * 50,
            description: `This is a test product created during load testing`,
            category: "TEST",
            in_stock: Math.random() > 0.2
        }
    });
    
    let createResponse = http.post(`${BASE_URL}/api/v1/data/PRODUCTS`, createPayload, {
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${AUTH_TOKEN}`,
        },
    });
    
    check(createResponse, {
        'create status is 201': (r) => r.status === 201,
        'create response time < 300ms': (r) => r.timings.duration < 300,
    });
    
    sleep(3);
}

14.4.2 Pruebas de Estrés
Diseñadas para encontrar los límites del sistema:
// Ejemplo de configuración de pruebas de estrés
export let options = {
    scenarios: {
        stress: {
            executor: 'ramping-arrival-rate',
            startRate: 10,
            timeUnit: '1s',
            preAllocatedVUs: 100,
            maxVUs: 1000,
            stages: [
                { duration: '1m', target: 50 },   // 50 RPS
                { duration: '3m', target: 50 },   // Mantener 50 RPS
                { duration: '1m', target: 100 },  // 100 RPS
                { duration: '3m', target: 100 },  // Mantener 100 RPS
                { duration: '1m', target: 200 },  // 200 RPS
                { duration: '3m', target: 200 },  // Mantener 200 RPS
                { duration: '1m', target: 300 },  // 300 RPS
                { duration: '3m', target: 300 },  // Mantener 300 RPS
                { duration: '1m', target: 0 },    // Rampa a 0 RPS
            ],
        },
    },
    thresholds: {
        'http_req_duration': ['p(95)<1000'], // 95% de las peticiones deben completarse en menos de 1s
        'http_req_failed': ['rate<0.05'],    // Menos del 5% de fallos
    },
};


14.4.3 Pruebas de Resistencia
Para verificar el comportamiento del sistema bajo carga sostenida:

// Ejemplo de configuración de pruebas de resistencia
export let options = {
    scenarios: {
        soak: {
            executor: 'constant-vus',
            vus: 50,
            duration: '12h',
        },
    },
    thresholds: {
        'http_req_duration': ['p(95)<500'],
        'http_req_failed': ['rate<0.01'],
    },
};

14.4.4 Monitoreo Continuo de Rendimiento
Para detectar regresiones de rendimiento, se implementa un sistema de pruebas automáticas que se ejecutan periódicamente y comparan con líneas base establecidas:

// Estructura para almacenar referencias de rendimiento (benchmarks)
type PerformanceBenchmark struct {
    Endpoint            string    `json:"endpoint"`
    Method              string    `json:"method"`
    P50ResponseTimeMs   float64   `json:"p50_response_time_ms"`
    P95ResponseTimeMs   float64   `json:"p95_response_time_ms"`
    P99ResponseTimeMs   float64   `json:"p99_response_time_ms"`
    MaxRPS              float64   `json:"max_rps"`
    LastUpdated         time.Time `json:"last_updated"`
}

// Servicio para gestionar benchmarks de rendimiento
type PerformanceBenchmarkService struct {
    db      *sqlx.DB
    logger  *zap.Logger
    metrics metrics.Reporter
}

// Guardar nuevo benchmark
func (s *PerformanceBenchmarkService) SaveBenchmark(ctx context.Context, benchmark *PerformanceBenchmark) error {
    query := `
        INSERT INTO performance_benchmarks (
            endpoint, method, p50_response_time_ms, p95_response_time_ms, 
            p99_response_time_ms, max_rps, last_updated
        ) VALUES (
            $1, $2, $3, $4, $5, $6, $7
        )
        ON CONFLICT (endpoint, method) DO UPDATE SET
            p50_response_time_ms = $3,
            p95_response_time_ms = $4,
            p99_response_time_ms = $5,
            max_rps = $6,
            last_updated = $7
    `
    
    _, err := s.db.ExecContext(ctx, query,
        benchmark.Endpoint,
        benchmark.Method,
        benchmark.P50ResponseTimeMs,
        benchmark.P95ResponseTimeMs,
        benchmark.P99ResponseTimeMs,
        benchmark.MaxRPS,
        benchmark.LastUpdated,
    )
    
    if err != nil {
        return fmt.Errorf("error guardando benchmark: %w", err)
    }
    
    return nil
}

// Comparar rendimiento actual con benchmark
func (s *PerformanceBenchmarkService) CompareBenchmark(ctx context.Context, endpoint, method string, current *PerformanceBenchmark) (*BenchmarkComparison, error) {
    var baseline PerformanceBenchmark
    
    query := `
        SELECT 
            endpoint, method, p50_response_time_ms, p95_response_time_ms, 
            p99_response_time_ms, max_rps, last_updated
        FROM performance_benchmarks
        WHERE endpoint = $1 AND method = $2
    `
    
    err := s.db.GetContext(ctx, &baseline, query, endpoint, method)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, ErrBenchmarkNotFound
        }
        return nil, fmt.Errorf("error consultando benchmark: %w", err)
    }
    
    // Calcular diferencias en porcentaje
    p50Diff := calculatePercentChange(baseline.P50ResponseTimeMs, current.P50ResponseTimeMs)
    p95Diff := calculatePercentChange(baseline.P95ResponseTimeMs, current.P95ResponseTimeMs)
    p99Diff := calculatePercentChange(baseline.P99ResponseTimeMs, current.P99ResponseTimeMs)
    rpsDiff := calculatePercentChange(current.MaxRPS, baseline.MaxRPS) // Nota: invertido para RPS (mayor es mejor)
    
    // Determinar si hay regresión de rendimiento
    hasRegression := p50Diff > 10 || p95Diff > 15 || p99Diff > 20 || rpsDiff < -10
    
    return &BenchmarkComparison{
        Baseline:           baseline,
        Current:            *current,
        P50PercentChange:   p50Diff,
        P95PercentChange:   p95Diff,
        P99PercentChange:   p99Diff,
        RPSPercentChange:   rpsDiff,
        HasRegression:      hasRegression,
        ComparisonDate:     time.Now(),
    }, nil
}

// Calcular cambio porcentual
func calculatePercentChange(baseline, current float64) float64 {
    if baseline == 0 {
        return 0
    }
    return ((current - baseline) / baseline) * 100
}

14.5 Gestión de Calidad Continua
Se implementa un sistema de gestión de calidad continua que incluye:
14.5.1 Análisis Estático de Código

// Configuración para análisis estático con golangci-lint
// .golangci.yml
linters:
  enable:
    - goimports
    - govet
    - errcheck
    - staticcheck
    - gosec
    - golint
    - gocyclo
    - maligned
    - unparam
    - misspell
    - prealloc
    - gosimple
    - stylecheck
    - unconvert
    - whitespace

linters-settings:
  gocyclo:
    min-complexity: 15
  maligned:
    suggest-new: true
  golint:
    min-confidence: 0.8
  govet:
    check-shadowing: true
  gosec:
    excludes:
      - G107 # HTTPS insecure
      - G104 # Errors unchecked

issues:
  exclude-rules:
    - path: _test\.go
      linters:
        - gocyclo
        - dupl
        - gosec
        - errcheck
		
14.5.2 Métricas de Cobertura de Código

// Script para generar y reportar métricas de cobertura
#!/bin/bash

set -e

echo "Generando reporte de cobertura..."
go test -coverprofile=coverage.out ./...

# Convertir a HTML para visualización
go tool cover -html=coverage.out -o coverage.html

# Extraer porcentaje total de cobertura
COVERAGE=$(go tool cover -func=coverage.out | grep total | awk '{print $3}')
echo "Cobertura total: $COVERAGE"

# Verificar umbral mínimo
THRESHOLD="80.0%"
if (( $(echo "$COVERAGE < $THRESHOLD" | bc -l) )); then
    echo "ERROR: La cobertura está por debajo del umbral mínimo de $THRESHOLD"
    exit 1
fi

echo "La cobertura cumple con el umbral mínimo de $THRESHOLD"

14.5.3 Revisiones de Código Automatizadas

// Configuración para Reviewdog en CI/CD
// .github/workflows/code-review.yml
name: Code Review

on:
  pull_request:
    branches: [ main, develop ]

jobs:
  reviewdog:
    name: Reviewdog
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Set up Go
        uses: actions/setup-go@v2
        with:
          go-version: 1.22

      - name: Install golangci-lint
        run: curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin v1.51.1

      - name: Run golangci-lint with reviewdog
        uses: reviewdog/action-golangci-lint@v2
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          reporter: github-pr-review
          level: error
          fail_on_error: true
          filter_mode: nofilter
		  
14.5.4 Automatización de Pruebas en CI/CD

// Configuración para ejecución de pruebas en CI
// .github/workflows/tests.yml
name: Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_USER: postgres
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      
      redis:
        image: redis:7
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v2

      - name: Set up Go
        uses: actions/setup-go@v2
        with:
          go-version: 1.22

      - name: Run migrations
        run: |
          go install -tags 'postgres' github.com/golang-migrate/migrate/v4/cmd/migrate@latest
          migrate -path ./db/migrations -database "postgres://postgres:postgres@localhost:5432/testdb?sslmode=disable" up

      - name: Run unit tests
        run: go test -v -race -coverprofile=unit.out -tags=unit ./...

      - name: Run integration tests
        run: go test -v -race -coverprofile=integration.out -tags=integration ./...

      - name: Upload coverage reports
        uses: codecov/codecov-action@v2
        with:
          files: ./unit.out,./integration.out
          flags: unit,integration
          fail_ci_if_error: true
          verbose: true
		  
		  


```go
## 15. Implementación DevOps

### 15.1 Estrategia de CI/CD

La implementación del sistema EAV Híbrido con Passkeys sigue una estrategia de integración continua y despliegue continuo (CI/CD) basada en los siguientes principios:

1. **Automatización Completa**: Todo el proceso de build, testing, y despliegue debe estar completamente automatizado.
2. **Control de Versiones**: Uso de Git con un modelo de ramificación basado en GitFlow.
3. **Entornos Independientes**: Desarrollo, QA, Staging y Producción completamente aislados.
4. **Promoción Gradual**: Las versiones se promueven progresivamente entre entornos.
5. **Infraestructura como Código**: Todos los recursos de infraestructura definidos mediante Terraform.

#### 15.1.1 Pipeline de CI/CD

```
┌───────────────┐     ┌────────────┐     ┌────────────┐     ┌────────────┐     ┌────────────┐
│   Commit &    │     │            │     │            │     │            │     │            │
│  Pull Request ├────►│   Build    ├────►│    Test    ├────►│   Deploy   ├────►│   Verify   │
│               │     │            │     │            │     │            │     │            │
└───────────────┘     └────────────┘     └────────────┘     └────────────┘     └────────────┘
                           │                  │                  │                  │
                           ▼                  ▼                  ▼                  ▼
                     ┌────────────┐     ┌────────────┐     ┌────────────┐     ┌────────────┐
                     │  Compile   │     │   Unit     │     │  Deploy to │     │ Automated  │
                     │  Services  │     │   Tests    │     │Environment │     │ Smoke Tests│
                     └────────────┘     └────────────┘     └────────────┘     └────────────┘
                     ┌────────────┐     ┌────────────┐     ┌────────────┐     ┌────────────┐
                     │  Static    │     │Integration │     │  Database  │     │ Performance│
                     │  Analysis  │     │   Tests    │     │ Migrations │     │    Tests   │
                     └────────────┘     └────────────┘     └────────────┘     └────────────┘
                     ┌────────────┐     ┌────────────┐     ┌────────────┐     ┌────────────┐
                     │  Security  │     │    E2E     │     │Config & IaC│     │ Canary     │
                     │   Scan     │     │   Tests    │     │ Deployment │     │ Analysis   │
                     └────────────┘     └────────────┘     └────────────┘     └────────────┘
```

### 15.2 Infraestructura como Código

Todo el entorno de infraestructura se gestiona mediante Terraform, lo que permite:

1. **Reproducibilidad**: Garantiza que todos los entornos sean idénticos.
2. **Versionado**: Los cambios en la infraestructura se versionan como código.
3. **Auditabilidad**: Cada cambio queda registrado en el historial de Git.
4. **Automatización**: Despliegue automatizado de infraestructura.

Los manifiestos de Terraform definen:

- Clusters de Kubernetes en cada entorno
- Bases de datos PostgreSQL
- Instancias de Redis para caché
- Servicios de observabilidad (Prometheus, Grafana, Loki)
- Políticas de seguridad y networking

### 15.3 Containerización y Orquestación

#### 15.3.1 Estrategia de Contenedores

Todos los servicios se empaquetan en contenedores Docker siguiendo estas prácticas:

1. **Imágenes Base Mínimas**: Uso de imágenes distroless o alpine para minimizar la superficie de ataque.
2. **Multi-stage Builds**: Separación de compilación y ejecución para reducir tamaño de imágenes.
3. **Inmutabilidad**: Las imágenes se versionan con tags específicos, nunca "latest".
4. **Escaneo de Vulnerabilidades**: Todas las imágenes se analizan con Trivy y Snyk.
5. **Firma de Imágenes**: Implementación de Cosign para garantizar la integridad.

Ejemplo de Dockerfile multi-stage para servicios Go:

```dockerfile
# Builder stage
FROM golang:1.22-alpine AS builder

WORKDIR /app

# Copiar archivos necesarios
COPY go.mod go.sum ./
RUN go mod download

COPY . .

# Compilar con flags de optimización
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o service .

# Runtime stage
FROM gcr.io/distroless/static:nonroot

WORKDIR /app

# Copiar binario compilado
COPY --from=builder /app/service .
COPY --from=builder /app/config/default.yaml /app/config/

# Metadatos para auditoría y observabilidad
LABEL org.opencontainers.image.source="https://github.com/organization/repository"
LABEL org.opencontainers.image.created="${BUILD_DATE}"
LABEL org.opencontainers.image.revision="${GIT_COMMIT}"

USER nonroot:nonroot

ENTRYPOINT ["/app/service"]
```

#### 15.3.2 Estrategia de Kubernetes

La arquitectura se despliega en Kubernetes utilizando:

1. **Helm Charts**: Abstracción para despliegue consistente en todos los entornos.
2. **GitOps**: Uso de ArgoCD para despliegues declarativos y sincronización automática.
3. **Políticas de Seguridad**: Implementación de PodSecurityPolicies y OPA Gatekeeper.
4. **Autoscaling**: HPA (Horizontal Pod Autoscaler) para escalado automático basado en métricas.
5. **Afinidad y Anti-afinidad**: Distribución óptima de pods para alta disponibilidad.

### 15.4 Monitoreo y Alertas

Se implementa un sistema completo de monitoreo que incluye:

1. **Dashboards de Grafana**: Visualización de métricas y KPIs críticos.
2. **Alertas Proactivas**: Configuración de alertas basadas en umbrales y patrones anómalos.
3. **On-Call Rotation**: Integración con PagerDuty para gestión de guardias.
4. **SLOs/SLIs**: Definición y seguimiento de objetivos de nivel de servicio.
5. **Postmortems**: Proceso estructurado para análisis de incidentes.

### 15.5 Gestión de Secretos

La gestión de secretos se realiza mediante:

1. **Vault de HashiCorp**: Almacenamiento centralizado de secretos.
2. **Rotación Automática**: Renovación periódica de credenciales.
3. **Injección de Secretos**: Integración con Kubernetes mediante el operador de Vault.
4. **Auditoría de Acceso**: Registro detallado de cada acceso a secretos.

## 16. Riesgos y Mitigaciones

### 16.1 Riesgos Técnicos

| Riesgo | Impacto | Probabilidad | Estrategia de Mitigación |
|--------|---------|--------------|--------------------------|
| Degradación de rendimiento por uso intensivo del modelo EAV | Alto | Media | Implementación de caché multinivel, índices optimizados y monitoreo proactivo de consultas lentas |
| Escalabilidad limitada en acceso concurrente | Alto | Media | Implementación de patrones Bulkhead, sharding de datos y límites de conexiones por cliente |
| Consistencia de datos en actualizaciones concurrentes | Medio | Alta | Control de concurrencia optimista con versiones, patrones CQRS y Outbox para eventos |
| Fallo en integración con proveedores de Passkeys | Alto | Baja | Implementación de Circuit Breakers, fallbacks a métodos alternativos y monitoreo de disponibilidad |
| Vulnerabilidades de seguridad | Crítico | Baja | Revisiones de código, pruebas de penetración regulares, actualizaciones continuas de dependencias |
| Pérdida de datos en migraciones | Crítico | Baja | Backups automáticos, estrategia blue-green para migraciones, validación previa con datos reales |
| Exposición de hashes de contraseña por brecha en BD | Crítico | Baja | Uso de algoritmos de hashing modernos (Argon2/scrypt/bcrypt), salting único, cifrado a nivel de disco, acceso restringido a la tabla de usuarios. |
| Ataques de fuerza bruta o credential stuffing | Alto | Media | Limitación de intentos de login, bloqueo temporal de cuentas, monitoreo de patrones de ataque, políticas de contraseñas robustas. | 

### 16.2 Riesgos Operativos

| Riesgo | Impacto | Probabilidad | Estrategia de Mitigación |
|--------|---------|--------------|--------------------------|
| Indisponibilidad de servicio | Alto | Baja | Arquitectura multi-AZ, recuperación automatizada de fallos, monitoreo 24/7 |
| Errores humanos en despliegues | Medio | Media | Automatización de despliegues, pruebas de canary, capacidad de rollback |
| Pérdida de conocimiento por rotación de equipo | Medio | Media | Documentación exhaustiva, prácticas de pair programming, conocimiento compartido |
| Aumento inesperado de carga | Alto | Baja | Pruebas de carga regulares, autoscaling, throttling de API como último recurso |
| Degradación gradual del rendimiento | Medio | Alta | Monitoreo continuo de tendencias, alertas tempranas, mantenimiento preventivo |

### 16.3 Riesgos de Negocio

| Riesgo | Impacto | Probabilidad | Estrategia de Mitigación |
|--------|---------|--------------|--------------------------|
| Cambios en requisitos de negocios | Medio | Alta | Arquitectura modular, desarrollo ágil, feature flags |
| Adopción insuficiente por usuarios | Alto | Media | Pruebas con usuarios reales, métricas de adopción, mejora continua |
| Cambios regulatorios | Alto | Media | Revisiones periódicas de cumplimiento, diseño flexible para adaptaciones |
| Costos operativos superiores a lo previsto | Medio | Media | Monitoreo de costos, optimización continua, uso de instancias reservadas |

### 16.4 Plan de Contingencia

Se ha desarrollado un plan de contingencia completo que incluye:

1. **Procedimientos de Failover**: Pasos detallados para activar sistemas redundantes.
2. **Recovery Point Objective (RPO)**: Objetivo de 5 minutos para minimizar pérdida de datos.
3. **Recovery Time Objective (RTO)**: Objetivo de recuperación de 15 minutos.
4. **Pruebas de Disaster Recovery**: Simulacros trimestrales de recuperación.
5. **Comunicación de Crisis**: Canales y procedimientos claramente definidos.

## 17. Conclusión

### 17.1 Resumen de la Arquitectura

La arquitectura EAV Híbrida con Passkeys representa una solución innovadora que balancea efectivamente flexibilidad y rendimiento para sistemas con requisitos de datos altamente dinámicos. Las principales fortalezas de esta arquitectura son:

1. **Modelo EAV Híbrido Optimizado**: Superando las limitaciones tradicionales del patrón EAV mediante estrategias avanzadas de indexación, particionamiento y caché.

2. **Autenticación Moderna con Passkeys**: Implementación de estándares FIDO2/WebAuthn que elimina dependencias de contraseñas y mejora significativamente la seguridad y experiencia de usuario.

3. **Arquitectura de Microservicios Resiliente**: Uso de patrones avanzados como Circuit Breaker, Bulkhead y CQRS para garantizar disponibilidad y escalabilidad.

4. **Observabilidad Completa**: Instrumentación integral que permite visibilidad profunda del comportamiento del sistema y facilita la resolución proactiva de problemas.

5. **Stack Tecnológico Moderno**: Selección cuidadosa de tecnologías y herramientas que potencian desarrollo rápido, mantenibilidad y escalabilidad.

### 17.2 Beneficios Clave

La implementación de esta arquitectura proporciona los siguientes beneficios:

1. **Flexibilidad Extrema**: Capacidad para evolucionar esquemas sin tiempo de inactividad o migraciones complejas.
2. **Rendimiento Optimizado**: Estrategias de caché multinivel, indexación selectiva y procesamiento asíncrono.
3. **Seguridad Mejorada**: Eliminación de riesgos asociados a contraseñas mediante autenticación moderna.
4. **Escalabilidad Horizontal**: Capacidad de crecer añadiendo nodos sin cambios arquitectónicos.
5. **Operabilidad Superior**: Visibilidad completa del comportamiento del sistema facilitando diagnósticos.
6. **Time-to-Market Reducido**: Capacidad para evolucionar rápidamente sin bloqueos por restricciones de esquema.
7. **Seguridad y Flexibilidad en Autenticación:** Ofrece métodos de autenticación modernos y seguros (Passkeys) 
	junto con opciones tradicionales (contraseña) para adaptarse a diversas necesidades de usuario y casos de uso, con robustas medidas de seguridad para ambos.

### 17.3 Consideraciones Futuras

Para el desarrollo continuo de esta arquitectura, se recomienda considerar:

1. **Expansión a Múltiples Proveedores de Passkeys**: Incorporar más proveedores además de Microsoft para ampliar compatibilidad.
2. **Optimizaciones Adicionales para PostgreSQL**: Explorar funcionalidades específicas de PostgreSQL 16+ para el modelo EAV.
3. **Implementación de Machine Learning**: Para detección de anomalías y optimización predictiva.
4. **Evolución hacia Edge Computing**: Distribución estratégica de caché y procesamiento en edge para reducir latencia.
5. **Adopción de WebAssembly**: Para cargas de trabajo específicas que requieren alto rendimiento.

### 17.4 Lecciones Aprendidas

Durante el diseño e implementación inicial, se identificaron las siguientes lecciones clave:

1. **Equilibrio entre Flexibilidad y Complejidad**: El modelo EAV Híbrido requiere inversión inicial en diseño e implementación, pero proporciona flexibilidad a largo plazo.
2. **Importancia de la Instrumentación Temprana**: La observabilidad debe ser una consideración primaria desde las etapas iniciales.
3. **Necesidad de Automatización Exhaustiva**: La complejidad del sistema justifica inversión significativa en automatización de pruebas y despliegues.
4. **Valor de la Experimentación**: Las pruebas de rendimiento y estrés revelaron puntos de optimización no evidentes en el diseño teórico.

Este diseño establece una base sólida para sistemas de datos dinámicos con requisitos estrictos de seguridad, rendimiento y flexibilidad, proporcionando una ventaja competitiva significativa en términos de agilidad y capacidad de evolución.


## 18. Sistema de Queries Dinámicos

### 18.1 Arquitectura de Reportes Dinámicos

El sistema de queries dinámicos se implementa como una capa sobre el modelo EAV que permite definir, gestionar y ejecutar reportes de negocio de forma flexible y segura.

```
┌──────────────────────────────────────────────────────────────────────┐
│                       Interfaz de Usuario                            │
└───────────────────────┬──────────────────────────────────────────────┘
                        │
┌───────────────────────▼──────────────────────────────────────────────┐
│                 API de Reportes Dinámicos                            │
│  ┌───────────────┐  ┌────────────────┐  ┌──────────────────────────┐ │
│  │ Endpoint único│  │ Motor de       │  │Consulta directa sin caché│ │
│  │ parametrizado │  │ Ejecución      │  │ para datos operacionales │ │
│  └───────────────┘  └────────────────┘  └──────────────────────────┘ │
└┬──────────────────────────┬──────────────────────────────────────────┘
 │                          │
┌▼──────────────────┐  ┌────▼─────────────────────────────────────┐
│ Modelo EAV para   │  │ Servicios EAV Existentes                 │
│ Metadatos de      │  │                                          │
│ Reportes          │  │                                          │
└───────────────────┘  └──────────────────────────────────────────┘
```

### 18.2 Modelo de Datos para Reportes

El sistema utiliza tablas lógicas EAV para almacenar toda la información necesaria para los reportes:

```
- ReportDefinitions
  - id (UUID)
  - name (VARCHAR)
  - description (TEXT)
  - category (VARCHAR)
  - is_active (BOOLEAN)
  - version (INT)
  - [campos de auditoría]

- ReportQueries
  - id (UUID)
  - report_definition_id (UUID) [FK]
  - sql_query (TEXT)
  - version (INT)
  - [campos de auditoría]

- ReportResponseContracts
  - id (UUID)
  - report_definition_id (UUID) [FK]
  - field_name (VARCHAR)
  - field_type (VARCHAR)
  - field_alias (VARCHAR)
  - is_required (BOOLEAN)
  - display_order (INT)
  - version (INT)
  - [campos de auditoría]

- ReportParameters
  - id (UUID)
  - report_definition_id (UUID) [FK]
  - parameter_name (VARCHAR)
  - parameter_type (VARCHAR)
  - is_required (BOOLEAN)
  - default_value (TEXT)
  - validation_rule_type (VARCHAR)
  - validation_parameters (JSONB)
  - [campos de auditoría]

- ReportPermissions
  - id (UUID)
  - report_definition_id (UUID) [FK]
  - role_id (UUID) [FK]
  - permission_type (VARCHAR)
  - [campos de auditoría]

- ReportVersions
  - id (UUID)
  - report_definition_id (UUID) [FK]
  - version_number (INT)
  - change_description (TEXT)
  - is_active (BOOLEAN)
  - [campos de auditoría]
```

### 18.3 API para Reportes Dinámicos

La implementación se basa en un único endpoint flexible que maneja toda la funcionalidad de reportes:

```go
// Endpoint principal para ejecución de reportes
// POST /api/v1/reports/execute
type ExecuteReportRequest struct {
    ReportID      uuid.UUID               `json:"report_id" validate:"required"`
    Version       *int                    `json:"version,omitempty"`
    Parameters    map[string]interface{}  `json:"parameters,omitempty"`
    Fields        []string                `json:"fields,omitempty"`
    Pagination    PaginationOptions       `json:"pagination"`
    Sorting       []SortOption            `json:"sorting,omitempty"`
}

type PaginationOptions struct {
    PageSize    int     `json:"page_size" validate:"required,min=1,max=1000"`
    PageNumber  *int    `json:"page_number,omitempty"`
    Cursor      *string `json:"cursor,omitempty"`
}

type SortOption struct {
    Field     string `json:"field" validate:"required"`
    Direction string `json:"direction" validate:"required,oneof=ASC DESC"`
}

type ExecuteReportResponse struct {
    ReportID    uuid.UUID                `json:"report_id"`
    ReportName  string                   `json:"report_name"`
    Data        []map[string]interface{} `json:"data"`
    Metadata    ReportMetadata           `json:"metadata"`
    Pagination  PaginationResult         `json:"pagination"`
}

type ReportMetadata struct {
    Version     int                      `json:"version"`
    ExecutedAt  time.Time                `json:"executed_at"`
    Duration    int64                    `json:"duration_ms"`
    TotalRows   int                      `json:"total_rows,omitempty"`
    Fields      []FieldMetadata          `json:"fields"`
}

type FieldMetadata struct {
    Name      string `json:"name"`
    Type      string `json:"type"`
    Alias     string `json:"alias"`
}

type PaginationResult struct {
    PageSize    int     `json:"page_size"`
    PageNumber  int     `json:"page_number,omitempty"`
    TotalPages  int     `json:"total_pages,omitempty"`
    HasMore     bool    `json:"has_more"`
    NextCursor  string  `json:"next_cursor,omitempty"`
}

// Handler para ejecutar reportes
func (h *ReportHandler) ExecuteReport(w http.ResponseWriter, r *http.Request) {
    // Iniciar span para trazabilidad
    ctx, span := h.tracer.Start(r.Context(), "ReportHandler.ExecuteReport")
    defer span.End()

    // Registrar la ejecución para auditoría
    auditInfo := ReportAuditEvent{
        Timestamp:  time.Now(),
        UserID:     getCurrentUserID(ctx),
        RemoteIP:   GetClientIP(r),
        UserAgent:  r.UserAgent(),
    }
    
    // Decodificar solicitud
    var req ExecuteReportRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        h.metrics.RecordReportError("request_decode")
        RespondWithError(w, http.StatusBadRequest, "Formato de solicitud inválido", err)
        span.RecordError(err)
        return
    }

    // Actualizar auditoría con ID del reporte
    auditInfo.ReportID = req.ReportID.String()
    auditInfo.Parameters = req.Parameters
    
    // Verificar permisos
    hasPermission, err := h.permissionVerifier.HasPermission(ctx, 
        getCurrentUserID(ctx), 
        "EXECUTE", 
        fmt.Sprintf("REPORT:%s", req.ReportID))
    
    if err != nil {
        h.metrics.RecordReportError("permission_check")
        RespondWithError(w, http.StatusInternalServerError, "Error verificando permisos", err)
        span.RecordError(err)
        return
    }
    
    if !hasPermission {
        h.metrics.RecordReportError("permission_denied")
        RespondWithError(w, http.StatusForbidden, "No tiene permisos para ejecutar este reporte", nil)
        return
    }

    // Ejecutar reporte con bulkhead para protección de recursos
    bulkhead := NewBulkhead("report-execution", 50, h.metrics)
    var result *ExecuteReportResponse
    err = bulkhead.Execute(func() error {
        var execErr error
        result, execErr = h.reportService.ExecuteReport(ctx, req)
        return execErr
    })

    if err != nil {
        code := http.StatusInternalServerError
        msg := "Error ejecutando reporte"
        
        if errors.Is(err, ErrReportNotFound) {
            code = http.StatusNotFound
            msg = "Reporte no encontrado"
        } else if errors.Is(err, ErrInvalidParameters) {
            code = http.StatusBadRequest
            msg = "Parámetros inválidos"
        } else if errors.Is(err, ErrInvalidVersion) {
            code = http.StatusBadRequest
            msg = "Versión de reporte inválida"
        }
        
        h.metrics.RecordReportError(strings.ToLower(strings.Replace(msg, " ", "_", -1)))
        RespondWithError(w, code, msg, err)
        span.RecordError(err)
        return
    }
    
    // Registrar auditoría de forma asíncrona
    auditInfo.ResponseRows = len(result.Data)
    auditInfo.ResponseTime = time.Since(auditInfo.Timestamp).Milliseconds()
    go h.auditService.LogReportExecution(context.Background(), auditInfo)
    
    // Responder exitosamente
    h.metrics.RecordReportSuccess(result.ReportID.String(), len(result.Data))
    RespondWithJSON(w, http.StatusOK, result)
    span.SetStatus(codes.Ok, "Report executed successfully")
	
}
```

### 18.4 Motor de Ejecución de Reportes

```go
// Servicio para ejecución de reportes
type ReportService struct {
    uow             UnitOfWork
    cacheManager    CacheManager
    metrics         metrics.Reporter
    tracer          trace.Tracer
    logger          *zap.Logger
    breaker         CircuitBreaker
    featureFlags    FeatureFlag
}

// Ejecutar reporte
func (s *ReportService) ExecuteReport(ctx context.Context, req ExecuteReportRequest) (*ExecuteReportResponse, error) {
    // Inicio de span para trazabilidad
    ctx, span := s.tracer.Start(ctx, "ReportService.ExecuteReport",
        trace.WithAttributes(attribute.String("report_id", req.ReportID.String())))
    defer span.End()
    
    startTime := time.Now()
    
    // Intentar obtener de caché si aplica
    if s.shouldUseCache(ctx, req) {
        cacheKey := s.buildCacheKey(req)
        var cachedResult ExecuteReportResponse
        if found, _ := s.cacheManager.Get(cacheKey, &cachedResult); found {
            span.SetAttribute("cache.hit", true)
            s.metrics.RecordCacheHit("report_execution")
            
            // Actualizar timestamp de ejecución para reflejar el caché hit
            cachedResult.Metadata.ExecutedAt = time.Now()
            return &cachedResult, nil
        }
        span.SetAttribute("cache.hit", false)
        s.metrics.RecordCacheMiss("report_execution")
    }
    
    // Obtener definición del reporte
    reportDef, err := s.getReportDefinition(ctx, req.ReportID, req.Version)
    if err != nil {
        span.RecordError(err)
        return nil, err
    }
    
    // Obtener contrato de respuesta
    responseContract, err := s.getResponseContract(ctx, reportDef.ID, reportDef.Version)
    if err != nil {
        span.RecordError(err)
        return nil, err
    }
    
    // Validar y preparar parámetros
    params, err := s.validateAndPrepareParameters(ctx, reportDef.ID, reportDef.Version, req.Parameters)
    if err != nil {
        span.RecordError(err)
        return nil, fmt.Errorf("error en parámetros: %w", err)
    }
    
    // Obtener y procesar SQL con parámetros
    processedSQL, err := s.processReportSQL(ctx, reportDef.ID, reportDef.Version, params, req.Sorting, req.Pagination)
    if err != nil {
        span.RecordError(err)
        return nil, fmt.Errorf("error procesando SQL: %w", err)
    }
    
    // Ejecutar consulta con Circuit Breaker
    var rawData []map[string]interface{}
    var paginationResult PaginationResult
    
    err = s.breaker.Execute(func() error {
        var execErr error
        rawData, paginationResult, execErr = s.executeSQL(ctx, processedSQL, params)
        return execErr
    })
    
    if err != nil {
        span.RecordError(err)
        return nil, fmt.Errorf("error ejecutando consulta: %w", err)
    }
    
    // Aplicar filtro de campos si se especificaron
    filteredData := s.filterFields(rawData, req.Fields, responseContract)
    
    // Construir resultado
    result := &ExecuteReportResponse{
        ReportID:   reportDef.ID,
        ReportName: reportDef.Name,
        Data:       filteredData,
        Metadata: ReportMetadata{
            Version:    reportDef.Version,
            ExecutedAt: time.Now(),
            Duration:   time.Since(startTime).Milliseconds(),
            TotalRows:  paginationResult.TotalRows,
            Fields:     s.buildFieldsMetadata(responseContract, req.Fields),
        },
        Pagination: paginationResult,
    }
    
    // Almacenar en caché si aplica
    if s.shouldUseCache(ctx, req) {
        cacheKey := s.buildCacheKey(req)
        cacheTTL := s.determineCacheTTL(reportDef)
        if err := s.cacheManager.Set(cacheKey, result, cacheTTL); err != nil {
            s.logger.Warn("Error cacheando resultado de reporte", 
                zap.String("report_id", req.ReportID.String()), 
                zap.Error(err))
        }
    }
    
    // Métricas de éxito
    s.metrics.RecordReportExecution(reportDef.ID.String(), time.Since(startTime), len(filteredData))
    span.SetStatus(codes.Ok, "Report executed successfully")
    
    return result, nil
}

// Procesar SQL con parámetros
func (s *ReportService) processReportSQL(ctx context.Context, reportID uuid.UUID, version int, params map[string]interface{}, sorting []SortOption, pagination PaginationOptions) (string, error) {
    ctx, span := s.tracer.Start(ctx, "ReportService.processReportSQL")
    defer span.End()

    // Obtener SQL base del reporte
    var baseSQL string
    err := s.uow.WithTransaction(ctx, func(ctx context.Context) error {
        query, err := s.uow.ReportQueries().FindByReportIDAndVersion(ctx, reportID, version)
        if err != nil {
            return err
        }
        baseSQL = query.SQLQuery
        return nil
    })

    if err != nil {
        return "", fmt.Errorf("error obteniendo SQL del reporte: %w", err)
    }

    // Sustituir parámetros en SQL
    processedSQL, err := s.substituteParameters(baseSQL, params)
    if err != nil {
        return "", fmt.Errorf("error sustituyendo parámetros: %w", err)
    }

	// Procesar campos JSONB
	processedSQL = strings.ReplaceAll(
		processedSQL, 
		"[[JSONB_FIELDS_PLACEHOLDER]]", 
		"tr.extended_properties"
	)

	// Si hay filtros por propiedades JSONB, procesarlos
	for key, value := range params {
		if strings.HasPrefix(key, "jsonb.") {
			jsonbPath := strings.TrimPrefix(key, "jsonb.")
			processedSQL = strings.ReplaceAll(
				processedSQL,
				fmt.Sprintf("[[JSONB_FILTER:%s]]", jsonbPath),
				fmt.Sprintf("tr.extended_properties->>'%s' = '%v'", jsonbPath, value)
			)
		}
	}

    // Aplicar ordenamiento
    processedSQL, err = s.applySorting(processedSQL, sorting)
    if err != nil {
        return "", fmt.Errorf("error aplicando ordenamiento: %w", err)
    }
    
    // Aplicar paginación
    processedSQL, err = s.applyPagination(processedSQL, pagination)
    if err != nil {
        return "", fmt.Errorf("error aplicando paginación: %w", err)
    }
    
    return processedSQL, nil
}

// Ejecutar SQL y obtener resultados
func (s *ReportService) executeSQL(ctx context.Context, sql string, params map[string]interface{}) ([]map[string]interface{}, PaginationResult, error) {
    ctx, span := s.tracer.Start(ctx, "ReportService.executeSQL")
    defer span.End()
    
    // Inicio de métricas
    start := time.Now()
    
    // Obtener transacción
    tx := getTxFromContext(ctx, s.uow.DB())
    
    // Preparar parámetros para query
    namedParams := s.prepareNamedParams(params)
    
    // Ejecutar query
    rows, err := tx.NamedQueryContext(ctx, sql, namedParams)
    if err != nil {
        span.RecordError(err)
        s.metrics.RecordDatabaseError("report_query_execution")
        return nil, PaginationResult{}, fmt.Errorf("error ejecutando query: %w", err)
    }
    defer rows.Close()
    
    // Mapear resultados
    var results []map[string]interface{}
    for rows.Next() {
        // Escanear fila a un mapa
        row := make(map[string]interface{})
        if err := rows.MapScan(row); err != nil {
            span.RecordError(err)
            s.metrics.RecordDatabaseError("report_row_scan")
            return nil, PaginationResult{}, fmt.Errorf("error escaneando fila: %w", err)
        }
        
        // Procesar tipos especiales (decimal, fechas, etc)
        s.processSpecialTypes(row)
        
        results = append(results, row)
    }
    
    // Verificar errores en iteración
    if err := rows.Err(); err != nil {
        span.RecordError(err)
        s.metrics.RecordDatabaseError("report_rows_iteration")
        return nil, PaginationResult{}, fmt.Errorf("error iterando filas: %w", err)
    }
    
    // Calcular información de paginación
    paginationResult := s.calculatePaginationResult(results, sql, params)
    
    // Métricas de duración
    duration := time.Since(start)
    s.metrics.RecordDatabaseQueryDuration("report_execution", duration)
    span.SetAttributes(
        attribute.Int("result_rows", len(results)),
        attribute.Int64("duration_ms", duration.Milliseconds()),
    )
    
    return results, paginationResult, nil
}
```

### 18.5 Middleware de Auditoría

```go
// Middleware para auditoría de ejecución de reportes
func ReportAuditMiddleware(auditService AuditService) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            // Solo aplicar al endpoint de ejecución de reportes
            if !strings.HasSuffix(r.URL.Path, "/reports/execute") {
                next.ServeHTTP(w, r)
                return
            }

            // Extraer información relevante
            userID := getCurrentUserID(r.Context())
            reportData, _ := io.ReadAll(r.Body)
            r.Body = io.NopCloser(bytes.NewBuffer(reportData)) // Restaurar body

            // Crear evento de auditoría
            auditEvent := ReportAuditEvent{
                Timestamp:  time.Now(),
                UserID:     userID,
                RemoteIP:   GetClientIP(r),
                ReportID:   extractReportID(reportData),
                Parameters: extractParameters(reportData),
                UserAgent:  r.UserAgent(),
            }

            // Enviar al servicio de auditoría (asíncrono)
            go func() {
                err := auditService.LogReportExecution(context.Background(), auditEvent)
                if err != nil {
                    // Solo log, no afectar la ejecución principal
                    log.Printf("Error logging audit event: %v", err)
                }
            }()

            // Continuar con el procesamiento normal
            next.ServeHTTP(w, r)
        })
    }
}

// Interfaz del servicio de auditoría
type AuditService interface {
    LogReportExecution(ctx context.Context, event ReportAuditEvent) error
}

// Estructura del evento de auditoría
type ReportAuditEvent struct {
    Timestamp    time.Time             `json:"timestamp"`
    UserID       uuid.UUID             `json:"user_id"`
    RemoteIP     string                `json:"remote_ip"`
    ReportID     string                `json:"report_id"`
    Parameters   map[string]interface{} `json:"parameters"`
    UserAgent    string                `json:"user_agent"`
    ResponseRows int                   `json:"response_rows,omitempty"`
    ResponseTime int64                 `json:"response_time_ms,omitempty"`
}
```

## 19. Sistema de Migración de Datos

### 19.1 Arquitectura de Migración

El sistema de migración proporciona herramientas flexibles para importar datos desde sistemas externos hacia el modelo EAV híbrido, con capacidad para transformaciones y validaciones complejas.

```
┌─────────────────────────────────────────────────────────────────┐
│                     Clientes de Migración                        │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────────┐ │
│  │ Batch Import   │  │ ETL Workflows  │  │ Integration Tools  │ │
│  └────────────────┘  └────────────────┘  └────────────────────┘ │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                   API de Migración                               │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────────┐ │
│  │ Batch Import   │  │ Mapping Engine │  │ Validation Engine  │ │
│  └────────────────┘  └────────────────┘  └────────────────────┘ │
│  ┌────────────────┐  ┌────────────────┐                         │
│  │ Progress       │  │ Error Handling │                         │
│  │ Tracking       │  │ & Recovery     │                         │
│  └────────────────┘  └────────────────┘                         │
└┬──────────────────────────┬──────────────────────────────────────┘
 │                          │
┌▼─────────────────┐   ┌────▼─────────────────────────────────────┐
│ Servicio de      │   │ Core EAV                                 │
│ Almacenamiento   │   │ ┌─────────────┐  ┌────────────────────┐  │
│ de Mapeos        │   │ │ Metadata    │  │ Data Service       │  │
└──────────────────┘   │ │ Service     │  │                    │  │
                       │ └─────────────┘  └────────────────────┘  │
                       └───────────────────────────────────────────┘
```

### 19.2 Modelo de Datos para Migración

```
- MigrationConfigurations
  - id (UUID)
  - name (VARCHAR)
  - description (TEXT)
  - source_type (VARCHAR)
  - target_logical_table_id (UUID) [FK]
  - validation_level (VARCHAR)
  - error_handling_strategy (VARCHAR)
  - version (INT)
  - [campos de auditoría]

- MigrationFieldMappings
  - id (UUID)
  - migration_configuration_id (UUID) [FK]
  - source_field (VARCHAR)
  - target_field (VARCHAR)
  - transformation_type (VARCHAR)
  - transformation_parameters (JSONB)
  - is_required (BOOLEAN)
  - [campos de auditoría]

- MigrationJobs
  - id (UUID)
  - migration_configuration_id (UUID) [FK]
  - status (VARCHAR)
  - total_records (INT)
  - processed_records (INT)
  - success_records (INT)
  - error_records (INT)
  - started_at (TIMESTAMP)
  - completed_at (TIMESTAMP)
  - created_by_user_id (UUID)
  - [campos de auditoría]

- MigrationErrors
  - id (UUID)
  - migration_job_id (UUID) [FK]
  - record_identifier (VARCHAR)
  - error_type (VARCHAR)
  - error_message (TEXT)
  - record_data (JSONB)
  - created_at (TIMESTAMP)
```

### 19.3 API para Migración de Datos

#### 19.3.1 API para Migración por Lotes

```go
// Endpoint para iniciar una migración por lotes
// POST /api/v1/migration/batch
type StartBatchMigrationRequest struct {
    ConfigurationID uuid.UUID          `json:"configuration_id" validate:"required"`
    SourceData      []json.RawMessage  `json:"source_data,omitempty"`
    SourceFileID    string             `json:"source_file_id,omitempty"`
    BatchSize       int                `json:"batch_size,omitempty"`
}

type StartBatchMigrationResponse struct {
    JobID           uuid.UUID  `json:"job_id"`
    Status          string     `json:"status"`
    TotalRecords    int        `json:"total_records"`
    EstimatedTime   int        `json:"estimated_time_seconds,omitempty"`
}

// Endpoint para verificar el estado de una migración
// GET /api/v1/migration/batch/{job_id}
type MigrationJobStatusResponse struct {
    JobID             uuid.UUID  `json:"job_id"`
    Status            string     `json:"status"`
    Progress          float64    `json:"progress"`
    TotalRecords      int        `json:"total_records"`
    ProcessedRecords  int        `json:"processed_records"`
    SuccessRecords    int        `json:"success_records"`
    ErrorRecords      int        `json:"error_records"`
    StartedAt         time.Time  `json:"started_at"`
    CompletedAt       *time.Time `json:"completed_at,omitempty"`
    EstimatedTimeLeft int        `json:"estimated_time_left_seconds,omitempty"`
    ElapsedTime       int        `json:"elapsed_time_seconds"`
}

// Handler para iniciar migración por lotes
func (h *MigrationHandler) StartBatchMigration(w http.ResponseWriter, r *http.Request) {
    // Inicio de span para trazabilidad
    ctx, span := h.tracer.Start(r.Context(), "MigrationHandler.StartBatchMigration")
    defer span.End()
    
    // Decodificar solicitud
    var req StartBatchMigrationRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        h.metrics.RecordMigrationError("request_decode")
        RespondWithError(w, http.StatusBadRequest, "Formato de solicitud inválido", err)
        span.RecordError(err)
        return
    }
    
    // Validar la solicitud
    if err := h.validator.Validate(req); err != nil {
        h.metrics.RecordMigrationError("validation")
        RespondWithError(w, http.StatusBadRequest, "Validación fallida", err)
        span.RecordError(err)
        return
    }
    
    // Verificar permisos
    hasPermission, err := h.permissionVerifier.HasPermission(ctx, 
        getCurrentUserID(ctx), 
        "EXECUTE", 
        "MIGRATION")
    
    if err != nil {
        h.metrics.RecordMigrationError("permission_check")
        RespondWithError(w, http.StatusInternalServerError, "Error verificando permisos", err)
        span.RecordError(err)
        return
    }
    
    if !hasPermission {
        h.metrics.RecordMigrationError("permission_denied")
        RespondWithError(w, http.StatusForbidden, "No tiene permisos para ejecutar migraciones", nil)
        return
    }
    
    // Verificar que los datos fuente están disponibles
    if len(req.SourceData) == 0 && req.SourceFileID == "" {
        h.metrics.RecordMigrationError("missing_source_data")
        RespondWithError(w, http.StatusBadRequest, "Se requiere datos fuente o un ID de archivo", nil)
        return
    }
    
    // Iniciar migración con bulkhead para protección de recursos
    bulkhead := NewBulkhead("migration-execution", 10, h.metrics)
    var result *StartBatchMigrationResponse
    
    executed, err := bulkhead.TryExecute(func() error {
        var execErr error
        result, execErr = h.migrationService.StartBatchMigration(ctx, req)
        return execErr
    })
    
if !executed {
        h.metrics.RecordMigrationError("bulkhead_rejected")
        RespondWithError(w, http.StatusTooManyRequests, "Demasiadas migraciones en proceso", nil)
        return
    }
    
    if err != nil {
        code := http.StatusInternalServerError
        msg := "Error iniciando migración"
        
        if errors.Is(err, ErrConfigurationNotFound) {
            code = http.StatusNotFound
            msg = "Configuración de migración no encontrada"
        } else if errors.Is(err, ErrInvalidSourceData) {
            code = http.StatusBadRequest
            msg = "Datos fuente inválidos"
        }
        
        h.metrics.RecordMigrationError(strings.ToLower(strings.Replace(msg, " ", "_", -1)))
        RespondWithError(w, code, msg, err)
        span.RecordError(err)
        return
    }
    
    // Responder exitosamente
    h.metrics.RecordMigrationSuccess("batch_started")
    RespondWithJSON(w, http.StatusOK, result)
    span.SetStatus(codes.Ok, "Migration started successfully")
}

// Handler para verificar estado de migración
func (h *MigrationHandler) GetBatchMigrationStatus(w http.ResponseWriter, r *http.Request) {
    // Inicio de span para trazabilidad
    ctx, span := h.tracer.Start(r.Context(), "MigrationHandler.GetBatchMigrationStatus")
    defer span.End()
    
    // Obtener ID de job de migración
    jobIDStr := chi.URLParam(r, "job_id")
    jobID, err := uuid.Parse(jobIDStr)
    if err != nil {
        h.metrics.RecordMigrationError("invalid_job_id")
        RespondWithError(w, http.StatusBadRequest, "ID de job inválido", err)
        span.RecordError(err)
        return
    }
    
    // Obtener estado del job
    status, err := h.migrationService.GetBatchMigrationStatus(ctx, jobID)
    if err != nil {
        code := http.StatusInternalServerError
        msg := "Error obteniendo estado de migración"
        
        if errors.Is(err, ErrJobNotFound) {
            code = http.StatusNotFound
            msg = "Job de migración no encontrado"
        }
        
        h.metrics.RecordMigrationError(strings.ToLower(strings.Replace(msg, " ", "_", -1)))
        RespondWithError(w, code, msg, err)
        span.RecordError(err)
        return
    }
    
    // Responder exitosamente
    RespondWithJSON(w, http.StatusOK, status)
    span.SetStatus(codes.Ok, "Migration status retrieved successfully")
}
```

#### 19.3.2 API para Migración Individual de Registros

```go
// Endpoint para migrar un registro individual
// POST /api/v1/migration/record
type MigrateRecordRequest struct {
    ConfigurationID uuid.UUID          `json:"configuration_id" validate:"required"`
    RecordData      json.RawMessage    `json:"record_data" validate:"required"`
}

type MigrateRecordResponse struct {
    Success      bool                `json:"success"`
    TargetRowID  uuid.UUID           `json:"target_row_id,omitempty"`
    Errors       []ValidationError   `json:"errors,omitempty"`
}

type ValidationError struct {
    Field       string  `json:"field"`
    ErrorType   string  `json:"error_type"`
    Message     string  `json:"message"`
}

// Handler para migrar registro individual
func (h *MigrationHandler) MigrateRecord(w http.ResponseWriter, r *http.Request) {
    // Inicio de span para trazabilidad
    ctx, span := h.tracer.Start(r.Context(), "MigrationHandler.MigrateRecord")
    defer span.End()
    
    // Decodificar solicitud
    var req MigrateRecordRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        h.metrics.RecordMigrationError("request_decode")
        RespondWithError(w, http.StatusBadRequest, "Formato de solicitud inválido", err)
        span.RecordError(err)
        return
    }
    
    // Validar la solicitud
    if err := h.validator.Validate(req); err != nil {
        h.metrics.RecordMigrationError("validation")
        RespondWithError(w, http.StatusBadRequest, "Validación fallida", err)
        span.RecordError(err)
        return
    }
    
    // Verificar permisos
    hasPermission, err := h.permissionVerifier.HasPermission(ctx, 
        getCurrentUserID(ctx), 
        "EXECUTE", 
        "MIGRATION")
    
    if err != nil {
        h.metrics.RecordMigrationError("permission_check")
        RespondWithError(w, http.StatusInternalServerError, "Error verificando permisos", err)
        span.RecordError(err)
        return
    }
    
    if !hasPermission {
        h.metrics.RecordMigrationError("permission_denied")
        RespondWithError(w, http.StatusForbidden, "No tiene permisos para ejecutar migraciones", nil)
        return
    }
    
    // Ejecutar migración
    result, err := h.migrationService.MigrateRecord(ctx, req)
    if err != nil {
        code := http.StatusInternalServerError
        msg := "Error migrando registro"
        
        if errors.Is(err, ErrConfigurationNotFound) {
            code = http.StatusNotFound
            msg = "Configuración de migración no encontrada"
        } else if errors.Is(err, ErrInvalidSourceData) {
            code = http.StatusBadRequest
            msg = "Datos de registro inválidos"
        }
        
        h.metrics.RecordMigrationError(strings.ToLower(strings.Replace(msg, " ", "_", -1)))
        RespondWithError(w, code, msg, err)
        span.RecordError(err)
        return
    }
    
    // Responder exitosamente
    if result.Success {
        h.metrics.RecordMigrationSuccess("record")
    } else {
        h.metrics.RecordMigrationError("validation_errors")
    }
    
    RespondWithJSON(w, http.StatusOK, result)
    span.SetStatus(codes.Ok, "Record migration processed")
}
```

### 19.4 Implementación del Servicio de Migración por Lotes

```go
// Servicio para migración de datos
type MigrationService struct {
    uow                 UnitOfWork
    tableRowService     TableRowService
    metadataService     MetadataService
    jobRepository       MigrationJobRepository
    errorRepository     MigrationErrorRepository
    configRepository    MigrationConfigRepository
    metrics             metrics.Reporter
    tracer              trace.Tracer
    logger              *zap.Logger
    breaker             CircuitBreaker
    bulkhead            Bulkhead
    workers             int
    jobQueue            chan *MigrationJob
}

// Iniciar migración por lotes
func (s *MigrationService) StartBatchMigration(ctx context.Context, req StartBatchMigrationRequest) (*StartBatchMigrationResponse, error) {
    // Inicio de span para trazabilidad
    ctx, span := s.tracer.Start(ctx, "MigrationService.StartBatchMigration")
    defer span.End()
    
    // Obtener configuración de migración
    config, err := s.configRepository.FindByID(ctx, req.ConfigurationID)
    if err != nil {
        if errors.Is(err, ErrNotFound) {
            return nil, ErrConfigurationNotFound
        }
        span.RecordError(err)
        return nil, fmt.Errorf("error obteniendo configuración: %w", err)
    }
    
    // Cargar datos fuente
    var sourceData []json.RawMessage
    if len(req.SourceData) > 0 {
        sourceData = req.SourceData
    } else if req.SourceFileID != "" {
        sourceData, err = s.loadSourceDataFromFile(ctx, req.SourceFileID, config.SourceType)
        if err != nil {
            span.RecordError(err)
            return nil, fmt.Errorf("error cargando datos de archivo: %w", err)
        }
    } else {
        return nil, ErrInvalidSourceData
    }
    
    // Crear nuevo job de migración
    batchSize := req.BatchSize
    if batchSize <= 0 {
        batchSize = 100 // Por defecto
    }
    
    job := &MigrationJob{
        ID:                     uuid.New(),
        MigrationConfigurationID: config.ID,
        Status:                 "PENDING",
        TotalRecords:           len(sourceData),
        ProcessedRecords:       0,
        SuccessRecords:         0,
        ErrorRecords:           0,
        StartedAt:              time.Now(),
        CreatedByUserID:        getCurrentUserID(ctx),
    }
    
    // Persistir job
    if err := s.jobRepository.Create(ctx, job); err != nil {
        span.RecordError(err)
        return nil, fmt.Errorf("error creando job: %w", err)
    }
    
    // Calcular tiempo estimado basado en histórico o valor por defecto
    estimatedTimeSeconds := s.calculateEstimatedTime(len(sourceData), config.ID)
    
    // Ejecutar migración de forma asíncrona
    go s.processMigrationJob(job, sourceData, batchSize, config)
    
    return &StartBatchMigrationResponse{
        JobID:          job.ID,
        Status:         job.Status,
        TotalRecords:   job.TotalRecords,
        EstimatedTime:  estimatedTimeSeconds,
    }, nil
}

// Procesar job de migración
func (s *MigrationService) processMigrationJob(job *MigrationJob, sourceData []json.RawMessage, batchSize int, config *MigrationConfiguration) {
    // Crear contexto con timeout para toda la operación (8h máximo)
    ctx, cancel := context.WithTimeout(context.Background(), 8*time.Hour)
    defer cancel()
    
    // Crear span para trazabilidad
    ctx, span := s.tracer.Start(ctx, "MigrationService.processMigrationJob",
        trace.WithAttributes(
            attribute.String("job_id", job.ID.String()),
            attribute.Int("total_records", job.TotalRecords),
        ))
    defer span.End()
    
    // Actualizar estado del job
    job.Status = "PROCESSING"
    if err := s.jobRepository.Update(ctx, job); err != nil {
        s.logger.Error("Error actualizando estado de job",
            zap.String("job_id", job.ID.String()),
            zap.Error(err))
        return
    }
    
    // Preparar mapeos
    mappings, err := s.configRepository.GetMappings(ctx, config.ID)
    if err != nil {
        s.logger.Error("Error obteniendo mapeos", zap.Error(err))
        s.failJob(ctx, job, "Error obteniendo mapeos")
        return
    }
    
    // Crear tabla para procesamiento concurrente por lotes
    var wg sync.WaitGroup
    jobErrors := make(chan MigrationErrorRecord, job.TotalRecords)
    
    // Determinar número de workers basado en configuración
    numWorkers := s.workers
    if numWorkers > job.TotalRecords {
        numWorkers = job.TotalRecords
    }
    
    // Crear batches y encolar trabajo
    batches := s.createBatches(sourceData, batchSize)
    resultChan := make(chan BatchResult, len(batches))
    
    // Iniciar el pool de workers
    for w := 0; w < numWorkers; w++ {
        wg.Add(1)
        go func(workerID int) {
            defer wg.Done()
            s.migrationWorker(ctx, workerID, batches, resultChan, jobErrors, config, mappings)
        }(w)
    }
    
    // Iniciar goroutine para recopilar resultados
    go func() {
        var processedCount, successCount, errorCount int
        
        // Procesar resultados de batches
        for result := range resultChan {
            processedCount += result.ProcessedCount
            successCount += result.SuccessCount
            errorCount += result.ErrorCount
            
            // Actualizar estado del job periódicamente (cada 5 batches o cuando sea necesario)
            if processedCount%1000 == 0 || processedCount == job.TotalRecords {
                job.ProcessedRecords = processedCount
                job.SuccessRecords = successCount
                job.ErrorRecords = errorCount
                
                if err := s.jobRepository.Update(ctx, job); err != nil {
                    s.logger.Warn("Error actualizando progreso de job",
                        zap.String("job_id", job.ID.String()),
                        zap.Error(err))
                }
                
                // Métricas de progreso
                s.metrics.RecordMigrationProgress(job.ID.String(), float64(processedCount)/float64(job.TotalRecords))
            }
        }
        
        // Recopilar errores y almacenarlos
        var errors []MigrationErrorRecord
        errorCount := 0
        
        // Recopilar hasta 1000 errores máximo para evitar sobrecarga
        for err := range jobErrors {
            errors = append(errors, err)
            errorCount++
            if errorCount >= 1000 {
                break
            }
        }
        
        // Almacenar errores en lotes
        if len(errors) > 0 {
            s.bulkStoreErrors(ctx, job.ID, errors)
        }
        
        // Completar el job
        now := time.Now()
        job.Status = "COMPLETED"
        job.ProcessedRecords = processedCount
        job.SuccessRecords = successCount
        job.ErrorRecords = errorCount
        job.CompletedAt = &now
        
        if err := s.jobRepository.Update(ctx, job); err != nil {
            s.logger.Error("Error actualizando estado final de job",
                zap.String("job_id", job.ID.String()),
                zap.Error(err))
        }
        
        // Métricas finales
        s.metrics.RecordMigrationCompletion(job.ID.String(), 
            job.SuccessRecords, job.ErrorRecords, 
            time.Since(job.StartedAt).Seconds())
    }()
    
    // Esperar a que terminen todos los workers
    wg.Wait()
    close(resultChan)
    close(jobErrors)
}

// Worker para procesamiento de migración
func (s *MigrationService) migrationWorker(
    ctx context.Context,
    workerID int,
    batches <-chan []json.RawMessage,
    results chan<- BatchResult,
    errors chan<- MigrationErrorRecord,
    config *MigrationConfiguration,
    mappings []*MigrationFieldMapping,
) {
    for batch := range batches {
        // Iniciar span para batch
        batchCtx, batchSpan := s.tracer.Start(ctx, "MigrationService.processBatch",
            trace.WithAttributes(
                attribute.Int("worker_id", workerID),
                attribute.Int("batch_size", len(batch)),
            ))
        
        // Aplicar bulkhead para controlar recursos
        executed, batchResult := s.bulkhead.TryExecute(func() (BatchResult, error) {
            return s.processBatch(batchCtx, batch, config, mappings, errors)
        })
        
        if !executed {
            // Si el bulkhead rechaza, retrasar e intentar nuevamente
            s.logger.Warn("Bulkhead rechazó procesamiento, reintentando", 
                zap.Int("worker_id", workerID),
                zap.Int("batch_size", len(batch)))
            
            // Reencolar el batch (con backoff exponencial podría implementarse aquí)
            time.Sleep(time.Second)
            continue
        }
        
        // Enviar resultado
        results <- batchResult
        batchSpan.End()
    }
}

// Procesar un batch de registros
func (s *MigrationService) processBatch(
    ctx context.Context,
    batch []json.RawMessage,
    config *MigrationConfiguration,
    mappings []*MigrationFieldMapping,
    errors chan<- MigrationErrorRecord,
) (BatchResult, error) {
    result := BatchResult{
        ProcessedCount: len(batch),
        SuccessCount:   0,
        ErrorCount:     0,
    }
    
    // Estrategia de errores
    continueOnError := config.ErrorHandlingStrategy == "CONTINUE"
    
    // Proceso de lote con Circuit Breaker
    err := s.breaker.Execute(func() error {
        for i, recordData := range batch {
            // Intentar mapear y migrar registro
            req := MigrateRecordRequest{
                ConfigurationID: config.ID,
                RecordData:      recordData,
            }
            
            // Obtener identificador del registro para logs
            recordIdentifier := s.extractRecordIdentifier(recordData, mappings)
            
            // Intentar migrar registro
            recordResult, err := s.migrateRecordInternal(ctx, req, mappings)
            
            if err != nil {
                result.ErrorCount++
                
                // Registrar error
                errors <- MigrationErrorRecord{
                    RecordIdentifier: recordIdentifier,
                    ErrorType:        "SYSTEM_ERROR",
                    ErrorMessage:     err.Error(),
                    RecordData:       recordData,
                }
                
                if !continueOnError {
                    // Abortar el batch si la estrategia es detener en error
                    return fmt.Errorf("error en registro %s: %w", recordIdentifier, err)
                }
                
                continue
            }
            
            if !recordResult.Success {
                result.ErrorCount++
                
                // Registrar errores de validación
                errorMsg := "Errores de validación: "
                if len(recordResult.Errors) > 0 {
                    validationMessages := make([]string, 0, len(recordResult.Errors))
                    for _, valErr := range recordResult.Errors {
                        validationMessages = append(validationMessages, 
                            fmt.Sprintf("%s: %s", valErr.Field, valErr.Message))
                    }
                    errorMsg += strings.Join(validationMessages, "; ")
                }
                
                errors <- MigrationErrorRecord{
                    RecordIdentifier: recordIdentifier,
                    ErrorType:        "VALIDATION_ERROR",
                    ErrorMessage:     errorMsg,
                    RecordData:       recordData,
                }
                
                if !continueOnError {
                    return fmt.Errorf("error de validación en registro %s", recordIdentifier)
                }
                
                continue
            }
            
            // Incrementar contador de éxitos
            result.SuccessCount++
        }
        
        return nil
    })
    
    // Actualizar contadores finales si hay error global del batch
    if err != nil && !continueOnError {
        result.ErrorCount = len(batch) - result.SuccessCount
    }
    
    return result, err
}

// Migrar un registro individual (implementación interna)
func (s *MigrationService) migrateRecordInternal(
    ctx context.Context,
    req MigrateRecordRequest,
    mappings []*MigrationFieldMapping,
) (*MigrateRecordResponse, error) {
    // Iniciar span para trazabilidad
    ctx, span := s.tracer.Start(ctx, "MigrationService.migrateRecordInternal")
    defer span.End()
    
    // Decodificar datos fuente
    var sourceData map[string]interface{}
    if err := json.Unmarshal(req.RecordData, &sourceData); err != nil {
        span.RecordError(err)
        return nil, fmt.Errorf("error decodificando datos fuente: %w", err)
    }
    
    // Obtener configuración
    config, err := s.configRepository.FindByID(ctx, req.ConfigurationID)
    if err != nil {
        if errors.Is(err, ErrNotFound) {
            return nil, ErrConfigurationNotFound
        }
        span.RecordError(err)
        return nil, fmt.Errorf("error obteniendo configuración: %w", err)
    }
    
	// Detectar campos para JSONB vs EAV tradicional basado en configuración
    targetData, extendedData, validationErrors := s.applyMappingsWithJsonb(ctx, sourceData, mappings)
	
    if len(validationErrors) > 0 && config.ValidationLevel == "STRICT" {
        // Modo estricto: cualquier error de validación falla
        return &MigrateRecordResponse{
            Success: false,
            Errors:  validationErrors,
        }, nil
    }
    
    // Si hay errores pero en modo lenient, continuar solo con campos válidos
    if len(validationErrors) > 0 {
        s.logger.Warn("Continuando con errores de validación en modo lenient",
            zap.Int("error_count", len(validationErrors)))
    }

    var targetRowID uuid.UUID
    err = s.uow.WithTransaction(ctx, func(ctx context.Context) error {
        // Obtener tabla lógica
        var innerErr error
        targetRowID, innerErr = s.createTargetRowWithExtended(ctx, config.TargetLogicalTableID, targetData, extendedData)
        return innerErr
    })
	
    if err != nil {
        span.RecordError(err)
        return nil, fmt.Errorf("error creando registro destino: %w", err)
    }
    
    return &MigrateRecordResponse{
        Success:     len(validationErrors) == 0,
        TargetRowID: targetRowID,
        Errors:      validationErrors,
    }, nil
}

// Aplicar mapeos de campo
func (s *MigrationService) applyMappings(
    ctx context.Context,
    sourceData map[string]interface{},
    mappings []*MigrationFieldMapping,
) (map[string]interface{}, []ValidationError) {
    targetData := make(map[string]interface{})
    var validationErrors []ValidationError
    
    for _, mapping := range mappings {
        // Obtener valor fuente
        sourceValue, exists := sourceData[mapping.SourceField]
        if !exists {
            if mapping.IsRequired {
                validationErrors = append(validationErrors, ValidationError{
                    Field:     mapping.TargetField,
                    ErrorType: "MISSING_REQUIRED_FIELD",
                    Message:   "Campo requerido no encontrado en datos fuente",
                })
            }
            continue
        }
        
        // Aplicar transformación si es necesario
        transformedValue, err := s.applyTransformation(sourceValue, mapping)
        if err != nil {
            validationErrors = append(validationErrors, ValidationError{
                Field:     mapping.TargetField,
                ErrorType: "TRANSFORMATION_ERROR",
                Message:   err.Error(),
            })
            continue
        }
        
        // Asignar al resultado
        targetData[mapping.TargetField] = transformedValue
    }
    
    return targetData, validationErrors
}

// Clasificación de campos entre EAV y JSONB
func (s *MigrationService) applyMappingsWithJsonb(
    ctx context.Context,
    sourceData map[string]interface{},
    mappings []*MigrationFieldMapping,
) (map[string]interface{}, map[string]interface{}, []ValidationError) {
    targetData := make(map[string]interface{})
    extendedData := make(map[string]interface{})
    var validationErrors []ValidationError
    
    for _, mapping := range mappings {
        // Procesar mapping...
        
        // Verificar destino del campo
        if mapping.StorageType == "JSONB" {
            // Este campo va a las propiedades extendidas
            extendedData[mapping.TargetField] = transformedValue
        } else {
            // Campo normal para EAV
            targetData[mapping.TargetField] = transformedValue
        }
    }
    
    return targetData, extendedData, validationErrors
}

// Aplicar transformación según tipo
func (s *MigrationService) applyTransformation(
    value interface{},
    mapping *MigrationFieldMapping,
) (interface{}, error) {
    switch mapping.TransformationType {
    case "NONE":
        return value, nil
        
    case "TO_STRING":
        return fmt.Sprintf("%v", value), nil
        
    case "TO_NUMBER":
        switch v := value.(type) {
        case string:
            num, err := strconv.ParseFloat(v, 64)
            if err != nil {
                return nil, fmt.Errorf("error convirtiendo a número: %w", err)
            }
            return num, nil
        case float64, int, int64:
            return v, nil
        default:
            return nil, fmt.Errorf("tipo no convertible a número: %T", value)
        }
        
    case "TO_BOOLEAN":
        switch v := value.(type) {
        case string:
            v = strings.ToLower(v)
            return v == "true" || v == "yes" || v == "1" || v == "y", nil
        case bool:
            return v, nil
        case float64, int, int64:
            return v != 0, nil
        default:
            return nil, fmt.Errorf("tipo no convertible a boolean: %T", value)
        }
        
    case "TO_DATE":
        switch v := value.(type) {
        case string:
            // Obtener formato de fecha de parámetros
            var format string
            if mapping.TransformationParameters != nil {
                if f, ok := mapping.TransformationParameters["format"].(string); ok {
                    format = f
                }
            }
            
            if format == "" {
                format = time.RFC3339 // Formato por defecto
            }
            
            date, err := time.Parse(format, v)
            if err != nil {
                return nil, fmt.Errorf("error parseando fecha: %w", err)
            }
            return date, nil
        default:
            return nil, fmt.Errorf("tipo no convertible a fecha: %T", value)
        }
        
    case "CUSTOM":
        // Implementación de transformaciones personalizadas
        // Este es un punto de extensión que puede incluir funciones como:
        // - Conversión de unidades
        // - Normalización de textos
        // - Transformaciones con expresiones regulares
        // - Lookup de valores en tablas de referencia
        
        return s.applyCustomTransformation(value, mapping.TransformationParameters)
        
    default:
        return value, nil
    }
}
```

## 20. Apéndices

### 20.1 Glosario de Términos

- **EAV (Entity-Attribute-Value)**: Patrón de diseño de base de datos que permite almacenar atributos de entidades sin necesidad de modificar el esquema.
- **EAV Híbrido**: Variante optimizada del patrón EAV que combina columnas físicas para atributos comunes con almacenamiento EAV para atributos flexibles.
- **Passkeys**: Tecnología de autenticación basada en FIDO2/WebAuthn que elimina la necesidad de contraseñas.
- **CQRS (Command Query Responsibility Segregation)**: Patrón arquitectónico que separa las operaciones de lectura y escritura.
- **Circuit Breaker**: Patrón de diseño que previene fallos en cascada en sistemas distribuidos.
- **Bulkhead**: Patrón que aísla elementos de un sistema para que el fallo de uno no afecte a otros.
- **Outbox**: Patrón que garantiza la publicación confiable de eventos como parte de una transacción.
- **Unit of Work**: Patrón que gestiona un conjunto de operaciones que deben ejecutarse atómicamente.

### 20.2 Referencias

1. Fowler, M. (2003). Patterns of Enterprise Application Architecture. Addison-Wesley.
2. Newman, S. (2021). Building Microservices. O'Reilly Media.
3. Evans, E. (2004). Domain-Driven Design: Tackling Complexity in the Heart of Software. Addison-Wesley.
4. FIDO Alliance. (2019). Web Authentication: An API for accessing Public Key Credentials.
5. Richardson, C. (2018). Microservices Patterns. Manning Publications.
6. Kleppmann, M. (2017). Designing Data-Intensive Applications. O'Reilly Media.
7. Microsoft. (2023). Microsoft Entra ID Passkey Authentication.
8. Nygard, M. (2018). Release It! Design and Deploy Production-Ready Software. Pragmatic Bookshelf.

### 20.3 Diagramas Adicionales

#### 20.3.1 Diagrama de Secuencia de Autenticación con Passkeys

```
┌─────────┐         ┌─────────┐          ┌────────────┐         ┌──────────┐         ┌─────────────┐
│         │         │         │          │            │         │          │         │             │
│ Usuario │         │ Cliente │          │Auth Service│         │Web Authn │         │  Microsoft  │
│         │         │         │          │            │         │          │         │             │
└────┬────┘         └────┬────┘          └──────┬─────┘         └────┬─────┘         └──────┬──────┘
     │                    │                     │                     │                     │       
     │   Iniciar auth     │                     │                     │                     │       
     │───────────────────>│                     │                     │                     │       
     │                    │  StartAuthentication│                     │                     │       
     │                    │────────────────────>│                     │                     │       
     │                    │                     │     BeginLogin      │                     │       
     │                    │                     │────────────────────>│                     │       
     │                    │                     │                     │                     │       
     │                    │                     │<───────────────────┐│                     │       
     │                    │                     │   Opciones+Desafío  │                     │       
     │                    │<────────────────────│                     │                     │       
     │                    │                     │                     │                     │       
     │<───────────────────│                     │                     │                     │       
     │  Solicitar firma   │                     │                     │                     │       
     │                    │                     │                     │                     │       
     │───────────────────>│                     │                     │                     │       
     │                    │                     │                     │                     │       
     │                    │ FinishAuthentication│                     │                     │       
     │                    │────────────────────>│                     │                     │       
     │                    │                     │     FinishLogin     │                     │       
     │                    │                     │────────────────────>│                     │       
     │                    │                     │                     │                     │       
     │                    │                     │<───────────────────┐│                     │       
     │                    │                     │ Verificación firma  │                     │       
     │                    │                     │                     │                     │       
     │                    │                     │Validación Microsoft │                     │       
     │                    │                     │──────────────────── ┼────────────────────>│       
     │                    │                     │                     │                     │       
     │                    │                     │<─────────────────── ┼─────────────────────┤       
     │                    │                     │Validación completa  │                     │       
     │                    │                     │                     │                     │       
     │                    │<────────────────────│                     │                     │       
     │                    │  Token de sesión    │                     │                     │       
     │                    │                     │                     │                     │       
     │<───────────────────│                     │                     │                     │       
     │  Autenticación     │                     │                     │                     │       
     │  exitosa           │                     │                     │                     │       
     │                    │                     │                     │                     │       
└─────────────────────────┴─────────────────────┴─────────────────────┴─────────────────────┘
```

#### 20.3.2 Diagrama de Componentes del Sistema EAV Híbrido

```
┌───────────────────────────────────────────────────────────────────────────────────────┐
│                                  Frontend Application                                  │
└────────────────────────────────────────┬──────────────────────────────────────────────┘
                                         │
┌────────────────────────────────────────▼──────────────────────────────────────────────┐
│                                      API Gateway                                       │
└─────────┬─────────────────────────────┬─────────────────────────────┬─────────────────┘
          │                             │                             │
┌─────────▼─────────┐       ┌───────────▼──────────┐       ┌──────────▼─────────┐
│   Auth Service    │       │   Metadata Service   │       │    Data Service    │
├───────────────────┤       ├────────────────────┐ │       ├────────────────┐   │
│ Passkey Management│       │ Schema Management  │ │       │ Command API     │   │
├───────────────────┤       ├────────────────────┤ │       ├────────────────┤   │
│ Session Management│       │ Version Management │ │       │ Query API       │   │
├───────────────────┤       ├────────────────────┤ │       ├────────────────┤   │
│ Authorization     │       │ Validation Rules   │ │       │ Cache Manager   │   │
└──────────┬────────┘       └───────────┬────────┘ │       └────────┬───────┘   │
           │                            │          │                │           │
┌──────────▼────────┐       ┌───────────▼──────────┘       ┌────────▼───────┐   │
│ Reports Service   │       │   Migration Service  │       │ Outbox Service ├───┘
└──────────┬────────┘       └───────────┬──────────┘       └────────┬───────┘
           │                            │                           │
           │                            │                           │
┌──────────▼────────────────────────────▼──────────────────────────▼───────────────────┐
│                                Infrastructure Services                                │
├─────────────────┬───────────────────┬───────────────────┬────────────────────────────┤
│  PostgreSQL     │    Redis Cache    │    Prometheus     │      OpenTelemetry         │
└─────────────────┴───────────────────┴───────────────────┴────────────────────────────┘
```

### 20.4 Ejemplos de Código Adicionales

#### 20.4.1 Implementación del Repositorio de Definiciones de Columnas

```go
// Repositorio para definiciones de columnas
type ColumnDefinitionRepository interface {
    FindByID(ctx context.Context, id uuid.UUID) (*ColumnDefinition, error)
    FindByName(ctx context.Context, name string) (*ColumnDefinition, error)
    FindByLogicalTable(ctx context.Context, tableID uuid.UUID) ([]*ColumnDefinition, error)
    Create(ctx context.Context, def *ColumnDefinition) error
    Update(ctx context.Context, def *ColumnDefinition) error
    Delete(ctx context.Context, id uuid.UUID) error
}

// Implementación PostgreSQL del repositorio
type PostgresColumnDefinitionRepository struct {
    db         *sqlx.DB
    logger     *zap.Logger
    metrics    metrics.Reporter
    tracer     trace.Tracer
    cache      CacheManager
}

func NewPostgresColumnDefinitionRepository(
    db *sqlx.DB,
    logger *zap.Logger,
    metrics metrics.Reporter,
    tracer trace.Tracer,
    cache CacheManager,
) ColumnDefinitionRepository {
    return &PostgresColumnDefinitionRepository{
        db:      db,
        logger:  logger,
        metrics: metrics,
        tracer:  tracer,
        cache:   cache,
    }
}

func (r *PostgresColumnDefinitionRepository) FindByID(ctx context.Context, id uuid.UUID) (*ColumnDefinition, error) {
    ctx, span := r.tracer.Start(ctx, "ColumnDefinitionRepository.FindByID")
    defer span.End()
    
    // Registrar operación
    r.metrics.RecordRepositoryOperation("ColumnDefinition", "find_by_id")
    
    // Intentar obtener de caché
    cacheKey := fmt.Sprintf("col_def:%s", id)
    var def ColumnDefinition
    
    found, err := r.cache.Get(cacheKey, &def)
    if err != nil {
        r.logger.Warn("Error obteniendo definición de columna de caché", zap.Error(err))
    }
    
    if found {
        span.SetAttribute("cache.hit", true)
        r.metrics.RecordCacheHit("column_definition")
        return &def, nil
    }
    
    // Caché miss, obtener de base de datos
    span.SetAttribute("cache.hit", false)
    r.metrics.RecordCacheMiss("column_definition")
    
    // Obtener transacción del contexto o usar DB directamente
    tx := getTxFromContext(ctx, r.db)
    
    query := `
        SELECT 
            id, column_name, data_type, default_value, 
            masking_rule, description, version,
            created_at, created_by_user_id, 
            updated_at, updated_by_user_id
        FROM column_definitions
        WHERE id = $1
    `
    
    err = tx.GetContext(ctx, &def, query, id)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, ErrNotFound
        }
        
        span.RecordError(err)
        span.SetStatus(codes.Error, "Database error")
        r.metrics.RecordRepositoryError("ColumnDefinition", "db_error")
        
        return nil, fmt.Errorf("error consultando definición de columna: %w", err)
    }
    
    // Guardar en caché
    if err := r.cache.Set(cacheKey, &def, 30*time.Minute); err != nil {
        r.logger.Warn("Error guardando definición de columna en caché", zap.Error(err))
    }
    
    span.SetStatus(codes.Ok, "Definition found")
    return &def, nil
}

func (r *PostgresColumnDefinitionRepository) FindByName(ctx context.Context, name string) (*ColumnDefinition, error) {
    ctx, span := r.tracer.Start(ctx, "ColumnDefinitionRepository.FindByName")
    defer span.End()
    
    // Registrar operación
    r.metrics.RecordRepositoryOperation("ColumnDefinition", "find_by_name")
    
    // Intentar obtener de caché
    cacheKey := fmt.Sprintf("col_def_name:%s", name)
    var def ColumnDefinition
    
    found, err := r.cache.Get(cacheKey, &def)
    if err != nil {
        r.logger.Warn("Error obteniendo definición de columna de caché", zap.Error(err))
    }
    
    if found {
        span.SetAttribute("cache.hit", true)
        r.metrics.RecordCacheHit("column_definition")
        return &def, nil
    }
    
    // Caché miss, obtener de base de datos
    span.SetAttribute("cache.hit", false)
    r.metrics.RecordCacheMiss("column_definition")
    
    // Obtener transacción del contexto o usar DB directamente
    tx := getTxFromContext(ctx, r.db)
    
    query := `
        SELECT 
            id, column_name, data_type, default_value, 
            masking_rule, description, version,
            created_at, created_by_user_id, 
            updated_at, updated_by_user_id
        FROM column_definitions
        WHERE column_name = $1
    `
    
    err = tx.GetContext(ctx, &def, query, name)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, ErrNotFound
        }
        
        span.RecordError(err)
        span.SetStatus(codes.Error, "Database error")
        r.metrics.RecordRepositoryError("ColumnDefinition", "db_error")
        
        return nil, fmt.Errorf("error consultando definición de columna: %w", err)
    }
    
    // Guardar en caché
    if err := r.cache.Set(cacheKey, &def, 30*time.Minute); err != nil {
        r.logger.Warn("Error guardando definición de columna en caché", zap.Error(err))
    }
    
    // También guardar por ID para consultas futuras
    idCacheKey := fmt.Sprintf("col_def:%s", def.ID)
    if err := r.cache.Set(idCacheKey, &def, 30*time.Minute); err != nil {
        r.logger.Warn("Error guardando definición de columna en caché por ID", zap.Error(err))
    }
    
    span.SetStatus(codes.Ok, "Definition found")
    return &def, nil
}

func (r *PostgresColumnDefinitionRepository) FindByLogicalTable(ctx context.Context, tableID uuid.UUID) ([]*ColumnDefinition, error) {
    ctx, span := r.tracer.Start(ctx, "ColumnDefinitionRepository.FindByLogicalTable")
    defer span.End()
    
    // Registrar operación
    r.metrics.RecordRepositoryOperation("ColumnDefinition", "find_by_table")
    
    // Intentar obtener de caché
    cacheKey := fmt.Sprintf("col_defs_table:%s", tableID)
    var defs []*ColumnDefinition
    
    found, err := r.cache.Get(cacheKey, &defs)
    if err != nil {
        r.logger.Warn("Error obteniendo definiciones de columna de caché", zap.Error(err))
    }
    
    if found {
        span.SetAttribute("cache.hit", true)
        r.metrics.RecordCacheHit("column_definitions")
        return defs, nil
    }
    
    // Caché miss, obtener de base de datos
    span.SetAttribute("cache.hit", false)
    r.metrics.RecordCacheMiss("column_definitions")
    
    // Obtener transacción del contexto o usar DB directamente
    tx := getTxFromContext(ctx, r.db)
    
    query := `
        SELECT 
            cd.id, cd.column_name, cd.data_type, cd.default_value, 
            cd.masking_rule, cd.description, cd.version,
            cd.created_at, cd.created_by_user_id, 
            cd.updated_at, cd.updated_by_user_id
        FROM column_definitions cd
        JOIN table_column_properties tcp ON cd.id = tcp.column_definition_id
        WHERE tcp.logical_table_id = $1
        ORDER BY tcp.display_order
    `
    
    err = tx.SelectContext(ctx, &defs, query, tableID)
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, "Database error")
        r.metrics.RecordRepositoryError("ColumnDefinition", "db_error")
        
        return nil, fmt.Errorf("error consultando definiciones de columna: %w", err)
    }
    
    // Guardar en caché
    if err := r.cache.Set(cacheKey, defs, 30*time.Minute); err != nil {
        r.logger.Warn("Error guardando definiciones de columna en caché", zap.Error(err))
    }
    
    // También guardar cada definición individualmente
    for _, def := range defs {
        idCacheKey := fmt.Sprintf("col_def:%s", def.ID)
        if err := r.cache.Set(idCacheKey, def, 30*time.Minute); err != nil {
            r.logger.Warn("Error guardando definición de columna en caché por ID", zap.Error(err))
        }
        
        nameCacheKey := fmt.Sprintf("col_def_name:%s", def.ColumnName)
        if err := r.cache.Set(nameCacheKey, def, 30*time.Minute); err != nil {
            r.logger.Warn("Error guardando definición de columna en caché por nombre", zap.Error(err))
        }
    }
    
    span.SetStatus(codes.Ok, "Definitions found")
    return defs, nil
}

func (r *PostgresColumnDefinitionRepository) Create(ctx context.Context, def *ColumnDefinition) error {
    ctx, span := r.tracer.Start(ctx, "ColumnDefinitionRepository.Create")
    defer span.End()
    
    // Registrar operación
    r.metrics.RecordRepositoryOperation("ColumnDefinition", "create")
    
    // Obtener transacción del contexto o usar DB directamente
    tx := getTxFromContext(ctx, r.db)
    
    // Verificar si ya existe una columna con el mismo nombre
    var count int
    err := tx.GetContext(ctx, &count, "SELECT COUNT(*) FROM column_definitions WHERE column_name = $1", def.ColumnName)
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, "Database error")
        r.metrics.RecordRepositoryError("ColumnDefinition", "db_error")
        
        return fmt.Errorf("error verificando existencia de columna: %w", err)
    }
    
    if count > 0 {
        return ErrDuplicateName
    }
    
    // Si no existe ID, generar uno nuevo
    if def.ID == uuid.Nil {
        def.ID = uuid.New()
    }
    
    // Establecer versión inicial
    def.Version = 1
    
    // Establecer información de auditoría
    now := time.Now()
    userID := getCurrentUserID(ctx)
    
    def.CreatedAt = now
    def.CreatedByUserID = userID
    def.UpdatedAt = now
    def.UpdatedByUserID = userID
    
    // Insertar en base de datos
    query := `
        INSERT INTO column_definitions (
            id, column_name, data_type, default_value, 
            masking_rule, description, version,
            created_at, created_by_user_id, 
            updated_at, updated_by_user_id
        ) VALUES (
            $1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11
        )
    `
    
    _, err = tx.ExecContext(ctx, query,
        def.ID, def.ColumnName, def.DataType, def.DefaultValue,
        def.MaskingRule, def.Description, def.Version,
        def.CreatedAt, def.CreatedByUserID,
        def.UpdatedAt, def.UpdatedByUserID,
    )
    
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, "Database error")
        r.metrics.RecordRepositoryError("ColumnDefinition", "db_error")
        
        return fmt.Errorf("error insertando definición de columna: %w", err)
    }
    
    // Invalidar caché (patrón de invalidación Cache-Aside)
    cacheKeys := []string{
        fmt.Sprintf("col_def:%s", def.ID),
        fmt.Sprintf("col_def_name:%s", def.ColumnName),
    }
    
    for _, key := range cacheKeys {
        if err := r.cache.Delete(key); err != nil {
            r.logger.Warn("Error invalidando caché", zap.String("key", key), zap.Error(err))
        }
    }
    
    span.SetStatus(codes.Ok, "Definition created")
    return nil
}

func (r *PostgresColumnDefinitionRepository) Update(ctx context.Context, def *ColumnDefinition) error {
    ctx, span := r.tracer.Start(ctx, "ColumnDefinitionRepository.Update")
    defer span.End()
    
    // Registrar operación
    r.metrics.RecordRepositoryOperation("ColumnDefinition", "update")
    
    // Obtener transacción del contexto o usar DB directamente
    tx := getTxFromContext(ctx, r.db)
    
    // Verificar si existe la definición
    var existingDef ColumnDefinition
    err := tx.GetContext(ctx, &existingDef, "SELECT * FROM column_definitions WHERE id = $1", def.ID)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return ErrNotFound
        }
        
        span.RecordError(err)
        span.SetStatus(codes.Error, "Database error")
        r.metrics.RecordRepositoryError("ColumnDefinition", "db_error")
        
        return fmt.Errorf("error verificando existencia de definición: %w", err)
    }
    
    // Verificar si el nombre cambia y si ya existe otra definición con ese nombre
    if def.ColumnName != existingDef.ColumnName {
        var count int
        err := tx.GetContext(ctx, &count, "SELECT COUNT(*) FROM column_definitions WHERE column_name = $1 AND id != $2",
            def.ColumnName, def.ID)
        
        if err != nil {
            span.RecordError(err)
            span.SetStatus(codes.Error, "Database error")
            r.metrics.RecordRepositoryError("ColumnDefinition", "db_error")
            
            return fmt.Errorf("error verificando unicidad de nombre: %w", err)
        }
        
        if count > 0 {
            return ErrDuplicateName
        }
    }
    
    // Incrementar versión
    def.Version = existingDef.Version + 1
    
    // Actualizar información de auditoría
    def.CreatedAt = existingDef.CreatedAt
    def.CreatedByUserID = existingDef.CreatedByUserID
    def.UpdatedAt = time.Now()
    def.UpdatedByUserID = getCurrentUserID(ctx)
    
    // Actualizar en base de datos
    query := `
        UPDATE column_definitions SET
            column_name = $1,
            data_type = $2,
            default_value = $3,
            masking_rule = $4,
            description = $5,
            version = $6,
            updated_at = $7,
            updated_by_user_id = $8
        WHERE id = $9
    `
    
    _, err = tx.ExecContext(ctx, query,
        def.ColumnName, def.DataType, def.DefaultValue,
        def.MaskingRule, def.Description, def.Version,
        def.UpdatedAt, def.UpdatedByUserID, def.ID,
    )
    
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, "Database error")
        r.metrics.RecordRepositoryError("ColumnDefinition", "db_error")
        
        return fmt.Errorf("error actualizando definición de columna: %w", err)
    }
    
    // Invalidar caché
    cacheKeys := []string{
        fmt.Sprintf("col_def:%s", def.ID),
        fmt.Sprintf("col_def_name:%s", def.ColumnName),
    }
    
    if existingDef.ColumnName != def.ColumnName {
        cacheKeys = append(cacheKeys, fmt.Sprintf("col_def_name:%s", existingDef.ColumnName))
    }
    
    // Invalidar caché por tabla (patrón invalidación por agrupación)
    tableIDs, err := r.getRelatedTableIDs(ctx, def.ID)
    if err == nil {
        for _, tableID := range tableIDs {
            cacheKeys = append(cacheKeys, fmt.Sprintf("col_defs_table:%s", tableID))
        }
    }
    
    for _, key := range cacheKeys {
        if err := r.cache.Delete(key); err != nil {
            r.logger.Warn("Error invalidando caché", zap.String("key", key), zap.Error(err))
        }
    }
    
    span.SetStatus(codes.Ok, "Definition updated")
    return nil
}

func (r *PostgresColumnDefinitionRepository) Delete(ctx context.Context, id uuid.UUID) error {
    ctx, span := r.tracer.Start(ctx, "ColumnDefinitionRepository.Delete")
    defer span.End()
    
    // Registrar operación
    r.metrics.RecordRepositoryOperation("ColumnDefinition", "delete")
    
    // Obtener transacción del contexto o usar DB directamente
    tx := getTxFromContext(ctx, r.db)
    
    // Verificar si existen valores asociados a esta definición
    var valueCount int
    err := tx.GetContext(ctx, &valueCount, "SELECT COUNT(*) FROM column_values WHERE column_definition_id = $1", id)
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, "Database error")
        r.metrics.RecordRepositoryError("ColumnDefinition", "db_error")
        
        return fmt.Errorf("error verificando valores asociados: %w", err)
    }
    
    if valueCount > 0 {
        return ErrReferentialIntegrity
    }
    
    // Obtener información para invalidación de caché
    var columnName string
    err = tx.GetContext(ctx, &columnName, "SELECT column_name FROM column_definitions WHERE id = $1", id)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return ErrNotFound
        }
        
        span.RecordError(err)
        span.SetStatus(codes.Error, "Database error")
        r.metrics.RecordRepositoryError("ColumnDefinition", "db_error")
        
        return fmt.Errorf("error obteniendo información de columna: %w", err)
    }
    
    // Obtener tablas relacionadas para invalidación de caché
    tableIDs, err := r.getRelatedTableIDs(ctx, id)
    if err != nil {
        r.logger.Warn("Error obteniendo tablas relacionadas", zap.Error(err))
    }
    
    // Eliminar propiedades de columna asociadas
    _, err = tx.ExecContext(ctx, "DELETE FROM table_column_properties WHERE column_definition_id = $1", id)
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, "Database error")
        r.metrics.RecordRepositoryError("ColumnDefinition", "db_error")
        
        return fmt.Errorf("error eliminando propiedades de columna: %w", err)
    }
    
    // Eliminar definición
    _, err = tx.ExecContext(ctx, "DELETE FROM column_definitions WHERE id = $1", id)
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, "Database error")
        r.metrics.RecordRepositoryError("ColumnDefinition", "db_error")
        
        return fmt.Errorf("error eliminando definición de columna: %w", err)
    }
    
    // Invalidar caché
    cacheKeys := []string{
        fmt.Sprintf("col_def:%s", id),
        fmt.Sprintf("col_def_name:%s", columnName),
    }
    
    for _, tableID := range tableIDs {
        cacheKeys = append(cacheKeys, fmt.Sprintf("col_defs_table:%s", tableID))
    }
    
    for _, key := range cacheKeys {
        if err := r.cache.Delete(key); err != nil {
            r.logger.Warn("Error invalidando caché", zap.String("key", key), zap.Error(err))
        }
    }
    
    span.SetStatus(codes.Ok, "Definition deleted")
    return nil
}

// Método auxiliar para obtener IDs de tablas relacionadas
func (r *PostgresColumnDefinitionRepository) getRelatedTableIDs(ctx context.Context, columnID uuid.UUID) ([]uuid.UUID, error) {
    var tableIDs []uuid.UUID
    
    // Obtener transacción del contexto o usar DB directamente
    tx := getTxFromContext(ctx, r.db)
    
    err := tx.SelectContext(ctx, &tableIDs, 
        "SELECT DISTINCT logical_table_id FROM table_column_properties WHERE column_definition_id = $1", 
        columnID)
    
    if err != nil {
        return nil, fmt.Errorf("error obteniendo tablas relacionadas: %w", err)
    }
    
    return tableIDs, nil
}
```

### 20.5 Estrategia de Gestión de Versiones

Durante el ciclo de vida del sistema, se seguirá la siguiente estrategia de versionado:

1. **Versionado Semántico**: Se utilizará SemVer (X.Y.Z) donde:
   - X: Cambios incompatibles con versiones anteriores
   - Y: Nuevas funcionalidades compatibles con versiones anteriores
   - Z: Correcciones de errores

2. **Versionado de API**: Cada endpoint estará versionado mediante prefijo en la ruta (/api/v1/, /api/v2/), permitiendo la coexistencia de múltiples versiones.

3. **Versionado de Esquema**: Los cambios en el esquema EAV se controlarán mediante el campo 'version' en cada tabla principal.

4. **Migraciones de Base de Datos**: Implementadas con herramientas como golang-migrate, aplicadas automáticamente durante el despliegue.

5. **Compatibilidad hacia atrás**: Garantizada durante al menos dos versiones mayores para permitir actualizaciones graduales.

6. **Documentación de Cambios**: Cada versión incluirá un registro detallado de cambios, categorizados como:
   - Breaking Changes: Requieren acción del cliente
   - Features: Nuevas funcionalidades
   - Enhancements: Mejoras a funcionalidades existentes
   - Bugfixes: Correcciones de errores

7. **Ciclo de Deprecación**: Funcionalidades marcadas como obsoletas durante al menos una versión mayor antes de su eliminación.

La aplicación de esta estrategia asegurará que el sistema pueda evolucionar manteniendo compatibilidad con integraciones existentes y permitiendo a los clientes planificar sus actualizaciones adecuadamente.
	
