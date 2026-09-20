# PDF outline: BP Sistema de Banca por Internet

**Hard limit: 15 pages in total** (cover and diagrams included). Final language: **Spanish** (this outline uses Spanish section titles; working files are English and get translated in Phase 3, keeping terms such as API Gateway, BFF, CQRS in English).
Order follows the project owner's plan: business processes first, then the cross-cutting views, then delivery plan and discarded alternatives.

## Decisions taken with the project owner

1. **Notifications stay as designed:** the Notification Service reads the existing events (transfer, onboarding) through its own Service Bus subscription, which works as its queue; channels are SMS (two providers) and email.
2. **Internal transfers:** covered as a short variant inside the transfers chapter, no separate dynamic diagram.
3. **Roadmap diagram:** not now; kept in the backlog for later (see the end of this file).
4. **Length:** maximum 15 pages.

## 1. Page budget (15 pages)

| Pág. | Contenido | Tipo | Fuente |
| --- | --- | --- | --- |
| 1 | **Portada y resumen ejecutivo**: problema, solución en pocas líneas, decisiones clave, metas (99,95 %, RPO 5 min, RTO 1 h), costo y plan en una línea | Texto | solution-plan.md, D1, D18, D20 |
| 2 | **Alcance, procesos de negocio, requisitos y supuestos**: tabla de procesos (ver sección 2), qué queda fuera (tarjetas), supuestos (Ecuador, banco mediano, Core REST) | Texto y tabla | PROBLEM_STATEMENT.md, plan |
| 3 | **C4 nivel 1: Contexto** (para no técnicos) con guía de lectura de 3 líneas | Diagrama (horizontal) | 01-context |
| 4 | **C4 nivel 2: Contenedores**: apps, gateway, BFF, servicios, datos, temas de mensajería; decisiones D2, D3, D7 a D9 y D13 en una tabla corta bajo el diagrama | Diagrama (horizontal) | 02-container |
| 5 | **C4 nivel 3: Componentes de Transferencias y Pagos** (saga, idempotencia, outbox, adaptadores, seguridad) | Diagrama (horizontal) | 03a |
| 6 | **C4 nivel 3: Componentes de Onboarding** (atestación, webhook autenticado, decisión, provisión en el IdP) | Diagrama (horizontal) | 03d |
| 7 | **C4 nivel 3: Componentes de Movimientos y Cliente frecuente** (Cache-Aside, modelo de lectura, clasificación por eventos), dos diagramas lado a lado | Diagramas (horizontal) | 03b, 03c |
| 8 | **Despliegue e infraestructura**: borde, región primaria con zonas, región de recuperación, replicación, enlace privado, conmutación | Diagrama (horizontal) | 04-deployment |
| 9 | **Procesos: Onboarding y Autenticación** (Web SPA y Mobile App): flujo, decisiones, seguridad, flujo OAuth recomendado y flujos descartados | Texto y tabla | D2 a D6 |
| 10 | **Procesos: Consulta de datos y movimientos, Transferencias internas e interbancarias**: flujo con el diagrama dinámico de la transferencia (mitad de página), variante corta para transferencias entre cuentas propias | Texto y diagrama | D10, D12, 05c |
| 11 | **Procesos: Auditoría y Notificaciones** (suscripción de Service Bus como cola de solicitudes, SMS, correo) y **Seguridad** (OWASP, WAF, identidad, datos, móvil) | Texto y tabla | D9, D11, D14, D16 |
| 12 | **Regulaciones** (Ley Orgánica de Protección de Datos Personales y otras, tabla norma a control), **Resiliencia y monitoreo**, **Costos** | Texto y tablas | D15 a D19 |
| 13 | **Plan de entregas y evolución**: cuatro etapas, pretotipado, hipótesis y puertas, umbral de resiliencia, deuda técnica, curva de costos | Texto y tabla | D20 |
| 14 | **Alternativas evaluadas y descartadas** (parte 1): por decisión, la opción elegida, al menos dos justificaciones y las alternativas descartadas (D1 a D10) | Tabla densa | decisions.md |
| 15 | **Alternativas evaluadas y descartadas** (parte 2, D11 a D20), **riesgos, supuestos por validar y enlace al repositorio** | Tabla densa y texto | decisions.md |

Notes on the budget:
- Every decision keeps its **two justifications** in the page 14 and 15 tables (compact, small type). Pages 9 to 13 explain the *processes* and refer to decision numbers instead of repeating the justification text.
- If the Spanish text grows beyond the budget, the first thing to move to the repository is page 7 (Movements and Frequent Client components), then the dynamic diagram of page 10.
- No page is spent on glossary or references; terms are explained in place and references go in one line in page 15.

## 2. Procesos de negocio cubiertos (contenido de la página 2)

| Proceso | Qué logra el cliente o el banco | Actores | Decisiones |
| --- | --- | --- | --- |
| **Onboarding** | Abrir una cuenta desde el móvil con cédula, selfie y huella dactilar | Cliente potencial, personal de revisión | D5, D6, D19 |
| **Autenticación** (Web SPA y Mobile App) | Ingresar de forma segura con huella o usuario y clave | Cliente, servidor de autorización de BP | D2, D3, D4, D6 |
| **Consulta de datos y movimientos** | Ver saldos e histórico de movimientos, rápido incluso para clientes frecuentes | Cliente | D8, D12, D13 |
| **Transferencias internas** (cuentas propias) | Mover dinero entre sus propias cuentas al instante | Cliente | D10 |
| **Transferencias interbancarias** | Enviar dinero a otros bancos con estados claros y sin duplicados | Cliente, red interbancaria | D10, D14 |
| **Auditoría** | Registro inmutable de cada acción, útil para reguladores y disputas | Personal de cumplimiento, reguladores | D11 |
| **Notificaciones** | Aviso de cada movimiento por SMS y correo | Cliente | D9, D14 |

Transversales: seguridad (OWASP, WAF), regulaciones (protección de datos), resiliencia y monitoreo, costos, plan de entregas.

## 3. Contenido mínimo de cada capítulo de proceso (páginas 9 a 11)

1. Una frase de valor para el negocio y el riesgo principal.
2. Flujo en pasos numerados (referencia al diagrama).
3. Decisiones aplicadas, con su número (justificaciones en las páginas 14 y 15).
4. Seguridad y cumplimiento que aplican.
5. Qué se entrega en cada etapa del plan.

## 4. Qué va al repositorio y no al PDF

El PDF es el resumen de 15 páginas. El repositorio público contiene además:

- **Registro completo de decisiones** (D1 a D20) con contexto, opciones y compromisos.
- **Todos los diagramas** en formato editable (`.drawio`) y en imagen: los 14 diagramas, incluidos los componentes de Notificaciones, Auditoría, Web BFF y API Gateway, y los diagramas dinámicos de inicio de sesión y onboarding.
- Plan de solución, enunciado extraído y este esquema.

Página 15 enlaza el repositorio.

## 5. Riesgos del límite de 15 páginas y mitigación

| Riesgo | Mitigación |
| --- | --- |
| Los diagramas C4 son grandes y el texto se ve pequeño al imprimir | Exportar en vectorial para poder acercar en pantalla; recortar márgenes; cada diagrama ocupa una página completa; los diagramas completos también van en el repositorio |
| El español ocupa más que el inglés | Frases cortas, tablas en lugar de párrafos, tipografía de 9 a 10 puntos en tablas densas; comprobar el conteo de páginas antes de entregar |
| Las justificaciones de 20 decisiones no caben en prosa | Tabla compacta (páginas 14 y 15): decisión, elegido, justificación 1, justificación 2, alternativas descartadas |
| Los lectores no técnicos se pierden en las páginas de diagramas | El resumen ejecutivo y el capítulo de alcance se entienden solos; cada diagrama lleva una guía de lectura de 2 a 3 líneas |

## 6. Pendiente para más adelante (backlog)

- **Diagrama de hoja de ruta** (cuatro etapas, puertas y costo) en draw.io, para el capítulo del plan de entregas o la presentación. Ahora el plan va como tabla.
- **Diagrama dinámico propio de transferencia interna**, si más adelante se quiere mostrar en el repositorio.
- **Canal push** (diferido, D14 y D20).
- **Herramienta de generación del PDF** (por decidir en la fase 3): se prevé HTML a PDF con páginas horizontales para los diagramas y verticales para el texto, con verificación del número de páginas.
