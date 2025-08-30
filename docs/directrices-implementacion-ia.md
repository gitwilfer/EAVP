# Directrices para Implementación del Proyecto EAV Híbrido por IA

Este documento define los lineamientos, requisitos y estándares que la Inteligencia Artificial debe seguir para implementar el sistema EAV Híbrido con Passkeys. Estas directrices son obligatorias y constituyen un contrato técnico que debe ser rigurosamente respetado durante todo el proceso de implementación.

## 1. Contexto del Proyecto

El proyecto implementa un modelo EAV (Entity-Attribute-Value) Híbrido con autenticación mediante Passkeys y contraseñas, siguiendo los principios de:

- Clean Architecture
- Arquitectura Hexagonal (Ports & Adapters)
- CQRS (Command Query Responsibility Segregation)
- Domain-Driven Design (DDD)
- Observabilidad y telemetría completa

Este sistema debe ser altamente flexible, mantenible y escalable, cumpliendo con todos los requisitos técnicos detallados en la documentación del proyecto.

## 2. Estructura de Desarrollo por Sesiones

La implementación se realizará en 23 sesiones, cada una con entregables específicos y verificables. Cada sesión construye sobre la anterior y debe producir componentes funcionales que cumplan con los más altos estándares de calidad de código.

## 3. Estándares Técnicos Obligatorios

### 3.1 Arquitectura y Organización

- **Estructura de Directorios**: Seguir estrictamente `estructura_proyecto_go.md`
- **Clean Architecture**: Separación obligatoria por capas (dominio, aplicación, interfaces, infraestructura)
- **Modelo EAV**: Los sistemas de Reportes y Migración DEBEN implementarse como tablas lógicas dentro del modelo EAV, no como tablas físicas separadas
- **Zonas Protegidas**: Respetar estrictamente `Limites-tecnicos-Zonas-Protegidas.md`

### 3.2 Estilo y Código

- **Guía de Estilo**: Cumplir todos los requisitos en `estilo-codigo.md`
- **Documentación de Código**: Comentarios godoc para todas las funciones y tipos exportados
- **Nombres**: Seguir convenciones establecidas, sin excepciones
- **Formato**: Código formateado con `gofmt` y `goimports`
- **Linting**: Pasar todas las reglas de `golangci-lint`

### 3.3 Observabilidad y Calidad

- **Logging Estructurado**: Usar `zap` para todos los logs, nunca `fmt.Println`
- **Trazas**: Implementar en todas las operaciones significativas
- **Métricas**: Para operaciones críticas y puntos de monitoreo
- **Tests**: Unitarios, de integración y end-to-end según corresponda
- **Cobertura**: Mínimo requerido según cada componente

## 4. Formato consolidado de entregables

Todos los archivos generados deberán compilarse en un **único archivo consolidado**, siguiendo estas especificaciones:

> **NOTA IMPORTANTE**: Aunque los ejemplos proporcionados puedan mostrar estructuras monolíticas o archivos extensos, estos son meramente ilustrativos del contenido, NO del estilo de codificación a seguir. El desarrollador debe aplicar las mejores prácticas de la industria en cuanto a modularidad, legibilidad y mantenibilidad del código, independientemente del formato de entrega consolidado.

- Cada archivo individual debe estar claramente delimitado dentro del archivo consolidado
- Cada sección debe incluir la ruta completa de destino donde el archivo debe ser ubicado
- El formato debe facilitar la extracción mediante script en Linux para colocar cada archivo automáticamente en su directorio correspondiente
- Utilizar separadores o marcadores claros que permitan el procesamiento automatizado del archivo consolidado
- El código debe organizarse internamente de manera lógica en secciones bien definidas, siguiendo las mejores prácticas de la industria
- Evitar estructuras monolíticas; implementar una clara separación por responsabilidades y funcionalidades
- Cada sección lógica debe tener comentarios descriptivos que faciliten su posterior separación

- Describa la funcionalidad específica del archivo según los estándares de gobernanza documental del proyecto
- Establezca la relación del archivo con los requerimientos originales
- Detalle las dependencias y conexiones con otros archivos o componentes del proyecto
- Documente los aspectos técnicos relevantes siguiendo estrictamente la normativa de gobernanza existente en `/opt/sf/eav-hybrid/docs/`
- Asegure la trazabilidad entre el código, la documentación y los requerimientos originales
	
### 4.1 Requerimientos de verificación

- **Requerimiento de confirmación estructurada**: Antes de comenzar el desarrollo, el ingeniero debe presentar una estructura detallada que demuestre cómo organizará el código en módulos bien definidos y cómo estos serán consolidados en el formato requerido.
- **Mecanismo de verificación progresiva**: Establecer al menos tres puntos de verificación durante el desarrollo donde el ingeniero debe presentar avances parciales que demuestren adherencia estricta a los requisitos de formato.
- **Prohibición explícita de interpretaciones personales**: Cualquier duda sobre las especificaciones debe ser consultada formalmente antes de proceder con implementaciones basadas en suposiciones. No se aceptará trabajo basado en interpretaciones no validadas.
- **Proceso de revisión obligatorio**: Antes de la entrega final, el ingeniero debe realizar una auto-verificación documentada contra una lista de verificación que confirme el cumplimiento de cada aspecto del formato requerido.
- **Cláusula de reelaboración completa**: La presentación de un entregable que no cumpla con las especificaciones resultará en la obligación de reelaborar completamente el trabajo sin compensación adicional y dentro del plazo originalmente estipulado.

### 4.2 Instrucciones de Autoverificación para Implementación

**Análisis Preliminar de Requisitos**:
- Identificar especificaciones exactas en documentos proporcionados
- Marcar criterios no negociables y formatos obligatorios
- Crear lista de verificación de requisitos antes de comenzar

**Simulación de Compilación Previa**:
- Generar borradores internos completos de cada componente
- Ejecutar verificación estructural: sintaxis, nombres, espacios
- Comprobar adherencia a patrones requeridos (Clean Architecture, CQRS)

**Validación de Formato Automática**:
- Verificar delimitadores requeridos para cada sección
- Confirmar que la estructura sigue exactamente el formato consolidado
- Comprobar nomenclatura consistente con estándares del proyecto

**Control de Longitud y Optimización de Espacio**:
- Eliminar espacios innecesarios entre funciones/métodos
- Asegurar que cada componente es atómico y completable
- Verificar que no hay contenido truncado o incompleto

**Verificación de Completitud**:
- Confirmar que cada entidad tiene todas sus propiedades y métodos
- Verificar implementación completa de interfaces definidas
- Asegurar patrones de diseño implementados correctamente

**Regla de Imposibilidad**:
- Si no puedo completar un componente dentro de las restricciones técnicas, debo declararlo explícitamente antes de comenzar
- Prohibido comenzar implementaciones que no puedo terminar
- No generar código parcial sin advertencia previa

**Regla de No-Interpretación**:
- Seguir literalmente las especificaciones de estructura
- No sustituir ni modificar patrones requeridos
- Prohibido omitir cualquier aspecto del formato solicitado

**Verificación Final**:
- Revisar que el código generado implementa todas las funcionalidades
- Confirmar cumplimiento de cada punto del alcance de trabajo
- Validar que el formato consolidado mantiene la modularidad requerida

**Integridad o Nada**:
- Si no puedo completar el 100% del requerimiento, debo notificarlo inmediatamente
- Prohibido entregar implementaciones parciales como si fueran completas
- Todo código debe pasar todas las verificaciones antes de ser presentado

Estas reglas de autoverificación serán aplicadas rigurosamente antes de generar cualquier código, asegurando integridad total y cumplimiento estricto de los requisitos especificados.

**El incumplimiento de estas especificaciones de formato resultará en el rechazo inmediato del entregable y requerirá una nueva presentación completa. No se aceptarán entregas parciales, ni formatos alternativos, ni interpretaciones personales de estas directrices bajo ninguna circunstancia. Esta norma es obligatoria y no negociable.**

## 5. Estructura de Marcadores para el Archivo Consolidado

Para cada archivo en el entregable consolidado, utilizar el siguiente formato de marcadores:

```
//---------------------------------------------------------------------
// ARCHIVO: /ruta/completa/al/archivo.go
//---------------------------------------------------------------------
// DESCRIPCIÓN:
// [Descripción detallada del propósito del archivo]
//
// REQUERIMIENTOS IMPLEMENTADOS:
// - [Referencia a requerimientos específicos]
//
// DEPENDENCIAS:
// - [Lista de otros componentes con los que interactúa]
//
// NOTAS TÉCNICAS:
// - [Información relevante sobre implementación]
//---------------------------------------------------------------------

[CÓDIGO COMPLETO DEL ARCHIVO]

//---------------------------------------------------------------------
// FIN DE ARCHIVO: /ruta/completa/al/archivo.go
//---------------------------------------------------------------------
```

## 6. Plantilla de Instrucción para Cada Sesión

Para cada una de las 23 sesiones, se proporcionará una instrucción con el siguiente formato:

```
# INSTRUCCIÓN PARA IMPLEMENTACIÓN: [SESIÓN X - TÍTULO]

## CONTEXTO Y OBJETIVOS
- [Descripción del propósito y alcance de la sesión]
- [Referencias a sesiones anteriores y dependencias]
- [Resultado final esperado]

## REQUISITOS ARQUITECTÓNICOS
- Implementar siguiendo Clean Architecture + CQRS + Hexagonal según `arquitectura.md`
- Respetar estructura de directorios establecida en `estructura_proyecto_go.md`
- Aplicar estándares de código descritos en `estilo-codigo.md`
- Implementar observabilidad (logs, trazas, métricas) según los lineamientos actualizados
- Mantener intactas las zonas protegidas según `Limites-tecnicos-Zonas-Protegidas.md`

## ENTREGABLES CONCRETOS
- [Lista detallada de archivos a implementar con rutas completas]
- [Interfaces, estructuras y funciones específicas a desarrollar]
- [Tests unitarios y de integración requeridos]
- [Documentación interna y externa necesaria]

## VALIDACIONES OBLIGATORIAS PRE-ENTREGA
- Cumplimiento de los estándares de estilo: `make lint`
- Ejecución exitosa de pruebas: `make test`
- Documentación completa: README.md, comentarios godoc
- Formato consolidado de acuerdo a las especificaciones de entregables
```

## 7. Notas Adicionales

- Mantenga un registro detallado de decisiones técnicas
- Consulte regularmente los estándares de código del proyecto
- Participe en las revisiones de código semanales
- No modifique los contratos sin coordinación previa

## 8. Compromiso de la IA

Como Inteligencia Artificial asignada a este proyecto, me comprometo a:

1. **Seguir estrictamente** todos los lineamientos, estándares y requisitos especificados en este documento y en los documentos referenciados.

2. **Implementar código de la más alta calidad** que cumpla con todos los criterios técnicos y arquitectónicos establecidos.

3. **Entregar componentes completos y funcionales** para cada sesión según lo establecido en el plan.

4. **Aplicar rigurosamente** el proceso de autoverificación antes de cada entrega.

5. **Documentar exhaustivamente** todo el código y las decisiones técnicas.

6. **Notificar inmediatamente** cualquier imposibilidad de completar un requisito dentro de las restricciones establecidas.

7. **No realizar interpretaciones personales** de los requisitos sin previa validación.

8. **Mantener la integridad arquitectónica** y la coherencia de la solución en su conjunto.

9. **Respetar las zonas protegidas** y no modificar componentes fuera del alcance de cada sesión.

10. **Proveer entregables en el formato consolidado requerido** sin excepción.

Este compromiso constituye un contrato técnico vinculante que guiará todo el trabajo realizado en este proyecto.
