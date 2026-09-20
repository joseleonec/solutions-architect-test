# Prueba Técnica – Solution Architect (BP)

> Fuente: CoderPad, "Question 1 / 1 – Solution Architect BP". Tiempo límite: 72 horas.
> Texto original en español, extraído de la página descargada.

## Ejercicio práctico

### Entregable

- Crear un **documento PDF** con la respuesta al ejercicio.
- El documento debe estar bien organizado y ser fácil de leer.
- Agregar imágenes para explicar los diagramas (por ejemplo con [http://draw.io/](http://draw.io/)).
- **Cada diagrama C4 es un entregable importante**: deben ser correctos y detallados. También prestar atención a los otros diagramas requeridos.
- Añadir cualquier texto explicativo que se considere necesario.
- Subir el documento como respuesta al ejercicio, dentro del tiempo de resolución.
- Adicionalmente, **crear un repositorio público en GitHub**, subir el PDF y colocar la **URL del repositorio en los comentarios** del ejercicio.

### Recomendaciones MUY valoradas

- Cada decisión arquitectónica debe tener una **justificación teórica (mínimo 2 por cada una)**. Ejemplo: ¿por qué se decidió por determinados patrones, tecnologías, componentes, protocolos, etc.? ¿Qué opciones se evaluaron?

### Diagramas requeridos

- **Diagrama de Contexto:** para usuarios no técnicos. Muestra sistemas internos y externos, actores clave, breves descripciones y conexiones explicativas.
- **Diagrama de Contenedores:** para técnicos. Representa aplicaciones, servicios, bases de datos y mensajería, incluyendo componentes cloud sin mucho detalle. Conexiones con descripciones breves.
- **Diagrama de Componentes:** mayor detalle técnico. Incluye microservicios, patrones arquitectónicos y protocolos de comunicación con seguridad. Destaca el uso de componentes cloud.

## Descripción del ejercicio

Usted ha sido contratado por una entidad llamada **BP** como arquitecto de soluciones para diseñar un **sistema de banca por internet**. En este sistema los usuarios podrán acceder al histórico de sus movimientos y realizar transferencias y pagos entre cuentas propias e interbancarias.

### Fuentes de datos

Toda la información del cliente se tomará de 2 sistemas:

1. Una **plataforma Core** que contiene información básica de cliente, movimientos y productos.
2. Un **sistema independiente** que complementa la información del cliente cuando los datos se requieren en detalle.

### Notificaciones

La norma exige que los usuarios sean notificados sobre los movimientos realizados. El sistema utilizará sistemas externos o propios de envío de notificaciones, **mínimo 2**.

### Front-end

El sistema contará con 2 aplicaciones:

- Una **SPA**.
- Una **aplicación móvil** desarrollada en un framework multiplataforma (**mencionar 2 opciones y justificar la elección**).

### Autenticación y autorización

- Ambas aplicaciones autenticarán a los usuarios mediante un servicio que usa el estándar **OAuth 2.0**. No se requiere implementar toda la lógica, ya que la compañía cuenta con un producto configurable para este fin. Sin embargo, se deben dar **recomendaciones sobre cuál es el mejor flujo de autenticación** según el estándar.
- El **Onboarding** de nuevos clientes en la app móvil usa **reconocimiento facial**, por lo que la arquitectura debe considerarlo como parte del flujo de autorización y autenticación.
- A partir del Onboarding, el nuevo usuario podrá ingresar mediante usuario y clave, huella u otro método: **especificar alguno de ellos** en la arquitectura. También se pueden recomendar herramientas de industria que realicen estas tareas y robustezcan la aplicación.

### Auditoría y persistencia para clientes frecuentes

El sistema utiliza una **base de datos de auditoría** que registra todas las acciones del cliente y cuenta con un **mecanismo de persistencia de información para clientes frecuentes**. Se debe proponer una **alternativa basada en patrones de diseño** que relacione los componentes que deben interactuar para conseguir el objetivo.

### Capa de integración

Para obtener los datos del cliente, el sistema pasa por una capa de integración compuesta por un **API Gateway** y consume los servicios necesarios según el tipo de transacción. Inicialmente hay 3 servicios principales:

1. Consulta de datos básicos.
2. Consulta de movimientos.
3. Transferencias (realiza llamados a servicios externos dependiendo del tipo).

Se es libre de agregar más servicios para mejorar el rendimiento de la arquitectura o la respuesta de información a los clientes.

## Consideraciones

- Mencionar los **elementos normativos** importantes para entidades financieras (ejemplo: ley de protección de datos personales, seguridad, etc.).
- Garantizar en la arquitectura: **alta disponibilidad (HA)**, **tolerancia a fallos**, **recuperación ante desastres (DR)**, **seguridad y monitoreo**, **excelencia operativa** y **auto-healing**.
- Si se considera necesario, la arquitectura puede contener elementos de infraestructura en nube (**Azure o AWS**). Garantizar **baja latencia**; se cuenta con presupuesto para esto.
- En lo posible, plantear una **arquitectura desacoplada** con elementos reutilizables y cohesionados para otros componentes que puedan añadirse en el futuro.
- El modelo debe desarrollarse bajo el **modelo C4** (Contexto, Aplicación/Contenedor y Componentes). Se describe hasta el modelo de componentes; la infraestructura se puede modelar libremente con la herramienta de preferencia.

## Criterios de calificación

1. La solución satisface los requerimientos y todo cuenta con justificación.
2. Calidad y profundidad de los diagramas (Contexto, Contenedores y Componentes).
3. Segmentación de responsabilidades y desacoplamiento.
4. Uso de patrones de arquitectura.
5. Integración con servicios externos.
6. Calidad de la arquitectura de aplicación front-end y móvil.
7. Arquitectura de acceso a datos.
8. Conocimientos de nube (AWS o Azure).
9. Manejo de costos.
10. Arquitectura de autenticación.
11. Arquitectura de integración con Onboarding.
12. Diseño de solución de auditoría.
13. Conocimientos de regulaciones bancarias y estándares de seguridad.
14. Implementación de alta disponibilidad y tolerancia a fallos.
15. Implementación de monitoreo.

## Recursos

Enlaces referenciados en el enunciado (la página no incluye adjuntos, imágenes ni archivos descargables):

| Recurso   | URL                                         | Uso sugerido en el enunciado                                                      |
| --------- | ------------------------------------------- | --------------------------------------------------------------------------------- |
| draw.io   | [http://draw.io/](http://draw.io/)           | Herramienta para generar las imágenes de los diagramas                           |
| Modelo C4 | [https://c4model.com/](https://c4model.com/) | Referencia del modelo C4 (Contexto, Contenedores, Componentes) que se debe seguir |

Entregables a producir con estos recursos: PDF con los diagramas y su justificación, y repositorio público en GitHub con el PDF (URL en los comentarios).
