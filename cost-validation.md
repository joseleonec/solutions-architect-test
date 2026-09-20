# Validación de costos: BP Sistema de Banca por Internet

Documento de apoyo del PDF (pág. 12 y 13). Explica de dónde sale cada cifra y cómo comprobarla.

- **Fecha de consulta:** 20 de septiembre de 2026.
- **Fuente de precios unitarios:** API pública de precios de Azure (Azure Retail Prices API), moneda USD, precios de lista (pago por uso), región South Central US. No requiere inicio de sesión.
- **Cómo reproducir una consulta:** abrir en el navegador `https://prices.azure.com/api/retail/prices?$filter=<filtro>` con el filtro de la última columna de la tabla.
- **Convención:** 730 horas al mes. Los precios no incluyen acuerdos empresariales, reservas ni impuestos.

## Supuestos de dimensionamiento (banco mediano)

| Supuesto | Valor |
| --- | --- |
| Clientes | 1 000 000 |
| Usuarios activos mensuales | **300 000** |
| Usuarios activos por día | 60 000 |
| Transferencias por día | 40 000 (≈ 1,2 millones de movimientos al mes) |
| Altas nuevas al mes | ≈ 3 000 |
| Tráfico de salida | ≈ 3 TB al mes |
| Registros de observabilidad | 20 GB por día; de ellos 5 GB por día van al sistema de seguridad (SIEM) |

## Etapa 4 (diseño completo, región primaria y recuperación), línea por línea

Estado: **Verificado** = precio unitario obtenido de la API (la cantidad sigue siendo un supuesto). **Parcial** = precio unitario verificado, pero el volumen o parte del costo es supuesto. **Estimado** = no está en la API; requiere cotización.

| Componente | Configuración | Precio unitario | US$ por mes | Estado | Consulta (filtro) |
| --- | --- | --- | --- | --- | --- |
| API Management Premium | 3 unidades (2 primaria, 1 recuperación) | 3,829 por hora por unidad (2 795 al mes) | 8 385 | Verificado | `serviceName eq 'API Management' and armRegionName eq 'southcentralus' and skuName eq 'Premium'` |
| Nodos de contenedores (AKS) | 9 nodos D4s v5 Linux (6 primaria, 3 recuperación) + tarifa de 2 clústeres | 0,23 por hora por nodo (168); 0,10 por hora por clúster (73) | 1 657 | Verificado | `serviceName eq 'Virtual Machines' and armRegionName eq 'southcentralus' and armSkuName eq 'Standard_D4s_v5'` y `serviceName eq 'Azure Kubernetes Service' and armRegionName eq 'southcentralus'` |
| Azure SQL Business Critical | 8 vCore × 2 réplicas + 200 GB × 2 | 0,365322 por vCore-hora (2 133 por réplica); 0,30 por GB al mes | ≈ 4 390 | Verificado | `serviceName eq 'SQL Database' and armRegionName eq 'southcentralus' and contains(productName,'Business Critical') and contains(meterName,'vCore')` |
| Cosmos DB (modelo de lectura) | 10 000 RU/s aprovisionados en 2 regiones + 100 GB × 2 | 0,008 por hora por 100 RU/s (584 al mes por región); 0,25 por GB | ≈ 1 220 (hasta 1 800 con autoescalado) | Verificado | `serviceName eq 'Azure Cosmos DB' and armRegionName eq 'southcentralus' and contains(meterName,'RU/s')` |
| Azure Cache for Redis Premium P1 | 1 por región (primaria y réplica) | 0,555 por hora (405) | 810 | Verificado | `serviceName eq 'Redis Cache' and armRegionName eq 'southcentralus' and contains(skuName,'P1')` |
| Service Bus Premium | 1 unidad por región | 0,9275 por hora (677) | 1 354 | Verificado | `serviceName eq 'Service Bus' and armRegionName eq 'southcentralus' and skuName eq 'Premium'` |
| Managed HSM | 1 pool por región | 3,20 por hora (2 336) | 4 672 | Verificado | `productName eq 'Key Vault HSM Pool' and (armRegionName eq 'southcentralus' or armRegionName eq 'northcentralus')` |
| Front Door Premium y tráfico | Tarifa base + ≈ 3 TB de salida | 330 al mes de base; 0,0825 a 0,14 por GB según la zona de destino | 580 a 1 000 | Parcial | `serviceName eq 'Azure Front Door Service' and skuName eq 'Premium'` |
| Observabilidad y seguridad | Log Analytics 20 GB/día (2,76 por GB ≈ 1 680); Sentinel 5 GB/día (5,16 por GB ≈ 780); Defender para SQL (0,015 por vCore-hora ≈ 175) y contenedores; Grafana y Prometheus | ver detalle | 3 200 a 4 300 | Parcial | `serviceName eq 'Log Analytics' and armRegionName eq 'southcentralus'`; `serviceName eq 'Sentinel' and armRegionName eq 'southcentralus'`; `contains(serviceName,'Defender') and armRegionName eq 'southcentralus'` |
| Almacenamiento, respaldos y tráfico entre regiones | Archivo inmutable (≈ 1 TB en Blob Hot ZRS a 0,023 por GB ≈ 23), respaldos, replicación y salida a internet | mezcla | 500 a 1 500 | Parcial (solo almacenamiento verificado) | `serviceName eq 'Storage' and armRegionName eq 'southcentralus' and contains(meterName,'ZRS Data Stored')` |
| Verificación de identidad (proveedor) | ≈ 3 000 altas al mes | US$ 1 a 2 por verificación (supuesto) | 3 000 a 6 000 | **Estimado** | No está en la API: cotizar con 2 o 3 proveedores |
| SMS y correo | ≈ 15 % de los movimientos por SMS (≈ 180 000 mensajes) | US$ 0,03 por SMS (supuesto) | 3 000 a 9 000 | **Estimado** | No está en la API de precios de Azure: pedir la tarifa por país (Ecuador) a Azure Communication Services o a un agregador local |
| **Total** | | | **≈ 33 000 a 45 000** (redondeado a 32 000 a 46 000 en el PDF) | | |

- Por usuario activo mensual: 32 800 ÷ 300 000 ≈ 0,11 y 45 100 ÷ 300 000 ≈ 0,15.
- **Parte verificada:** las filas de estado "Verificado" suman ≈ 22 500 de un total de 33 000 a 45 000, es decir, entre la mitad y dos tercios. El resto son estimaciones.

### Advertencia importante sobre los SMS

La estimación supone SMS para ≈ 15 % de los movimientos (por ejemplo, montos altos o clientes que lo pidan) y correo para el resto. **Si se enviara un SMS por cada movimiento** (≈ 1,2 millones al mes), a US$ 0,03 serían ≈ 36 000 al mes, unos **30 000 más** que lo estimado. La regla de canales (qué movimiento genera SMS) es por tanto la decisión de costo más sensible del proyecto y debe cerrarse con Producto y Cumplimiento (decisión D14). El precio de US$ 0,03 por SMS es un supuesto sin verificar.

## Etapas anteriores

| Etapa | Cómo se calcula | US$ por mes | Estado |
| --- | --- | --- | --- |
| 1. Pretotipo | API Management Basic 0,2016 por hora (147) + 2 nodos D2s v5 a 0,115 por hora (168) + Front Door Standard (35) + Log Analytics 2 GB/día (166) + registro de contenedores y otros | ≈ 520 como mínimo; 500 a 1 500 con entorno de prueba y herramientas | Verificado (mínimo) |
| 2. MVP de solo lectura | API Management Standard 0,9407 por hora (687) + 3 nodos D4s v5 (504) + clúster (73) + Front Door con tráfico (580) + Log Analytics 10 GB/día (840) + SQL de propósito general zona-redundante de 2 vCore (≈ 430) + Redis pequeño y Defender (≈ 150) | ≈ 3 300; 3 000 a 6 000 con margen | Verificado (unitarios) |
| 3. Transaccional | Infraestructura: dos API Management Standard (1 400), 5 nodos (1 000), SQL zona-redundante de propósito general de 4 vCore × 2 (1 700), Service Bus Premium × 2 (1 350), Redis × 2 (810), un pool de HSM (2 340), Front Door (580 a 760), observabilidad (1 500 a 2 500), otros (300 a 800) ≈ 11 000 a 13 000; más proveedores variables (verificación y SMS) ≈ 4 000 a 10 000 | **≈ 15 000 a 23 000** | Parcial |
| 4. Escala empresarial | Tabla anterior | ≈ 33 000 a 45 000 (infraestructura ≈ 27 000 a 30 000 + proveedores ≈ 6 000 a 15 000) | Parcial |

Nota: en la etapa 3 se usa API Management Standard (sin zonas de disponibilidad) para contener el costo; el paso a Premium (zona-redundante y multirregión) ocurre en la etapa 4.

## Alternativas de ahorro identificadas

- **SQL de propósito general zona-redundante** (0,18266 + 0,109596 = 0,292 por vCore-hora) en lugar de Business Critical (0,365 por vCore-hora): ≈ 0,9 mil menos al mes para 8 vCore × 2 réplicas, si la carga lo permite.
- **Reservas de 1 a 3 años** para cómputo, SQL y Redis, y acuerdos empresariales.
- **Regla de SMS** (ver advertencia): es la palanca más grande.
- **API Management Premium** solo cuando se necesite multirregión o zonas (etapa 4).

## Cómo validar con la Calculadora de precios de Azure

1. Abrir `azure.microsoft.com/pricing/calculator`, elegir región South Central US y moneda USD.
2. Agregar cada fila de la tabla con la configuración indicada.
3. Comparar el total con el rango: debe caer dentro de ±30 %.
4. Para las filas "Estimado": pedir cotización (SMS a Ecuador, proveedor de verificación) y reemplazar el supuesto.
5. Consultar con el representante de Azure el efecto de un acuerdo empresarial y de las reservas.
