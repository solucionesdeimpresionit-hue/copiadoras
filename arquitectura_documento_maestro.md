# Documento Maestro de Arquitectura

**Estado:** Documento vivo de trabajo — no versionado. Se actualiza y completa conforme se toman decisiones. No hay una línea base "aprobada" todavía; este documento es la fuente única de verdad mientras eso ocurre.
**Repositorio:** mfp-rental
**Ubicación:** docs/arquitectura/documento_maestro.md

---

## 1. Propósito del Sistema

Plataforma integral para la renta, venta y servicio prepago anual de equipos multifuncionales de impresión (MFP), operada por SOLUCIONES DE IMPRESIÓN Y SERVICIOS IT, S.A. DE C.V. Atiende principalmente escuelas de educación básica en CDMX y área metropolitana, y clientes corporativos bajo modalidad de renta o venta. Gestiona el ciclo de vida completo: prospección, contratación, entregas, lecturas de contador, facturación, cobranza, servicio técnico, suministros, garantía, renovaciones y base de conocimiento técnico.

El sistema opera **tres modelos de negocio diferenciados**, cada uno con su propio contrato maestro y comportamiento operativo:

| Modelo | Contrato | Método fiscal | Propiedad del equipo | Semáforo financiero |
|---|---|---|---|---|
| **Renta** | Contrato Maestro de Arrendamiento, Servicio y Suministro de Materiales | PPD | COMPAÑÍA | Sí — por RFC individual |
| **Venta** | Contrato de Venta de Equipo Multifuncional con Garantía | PUE | CLIENTE, desde la entrega | Solo en post-garantía / modalidad PF |
| **Escuela prepago anual** | Contrato de Servicio y Suministro para Equipo Multifuncional Modelo Escolar — Prepago Anual | PUE | ESCUELA (el MFP es propiedad de la escuela; la COMPAÑÍA da servicio) | **No aplica** — el control es por vigencia y volumen anual, no por cobranza |

**Nota operativa explícita:** en el modelo Escuela prepago, el pago ya se recibió por adelantado. El contador **no sirve para facturar** (salvo adquisición de nuevo paquete) — sirve exclusivamente para vigilar consumo acumulado contra el volumen anual pactado y disparar el seguimiento de renovación a tiempo. No hay estado de morosidad que vigilar en este modelo.

---

## 2. Stakeholders

| Rol | Interés en el sistema |
|---|---|
| **Usuario principal (cliente/escuela)** | Solicitar servicio técnico, solicitar tóner de reserva, confirmar lecturas, ver bitácora del equipo, acceder a biblioteca de cliente, confirmar recibido de cotizaciones de renovación. |
| **Persona autorizada (registro operativo por visita)** | Autorizar el acceso del técnico, confirmar contadores, recibir materiales, firmar órdenes de servicio, cerrar visitas. No es necesariamente el mismo rol que "usuario principal"; se registra por nombre, puesto y celular antes de cada visita. |
| **Contabilidad (cliente)** | Ver facturas, estado de cuenta, historial de pagos, referencias bancarias. |
| **Representante legal (cliente)** | Recibir notificaciones de morosidad, bloqueo, avisos contractuales. |
| **Prospecto** | Recibir cotizaciones, comunicarse con Ventas a través del mismo canal de mensajería de la app, sin ser todavía cliente formal. |
| **Técnico** | Capturar lecturas mensuales, registrar personal autorizado antes de cada visita, atender órdenes de servicio, registrar la solución de campo de cada falla (obligatorio), registrar cambios de tóner y refacciones, consultar biblioteca técnica interna. |
| **Coordinadora** | Asignar tickets/órdenes de servicio, programar rutas de lectura y entrega de tóner, supervisar SLA, participar en autorización conjunta de excepciones de semáforo en ROJO, dar seguimiento a renovaciones sin confirmación de recibido. |
| **Supervisor** | Validar soluciones de campo antes de integrarlas a biblioteca interna, atender escalaciones, auditar lecturas, participar en autorización conjunta de excepciones de semáforo. |
| **Almacén** | Gestionar inventario de tóners, refacciones y equipos; controlar stock central y stock de técnicos; generar solicitudes de compra; bloquear salida de equipos de venta hasta confirmación de pago. |
| **Administrador** | Configurar contratos, usuarios, roles, parámetros del sistema, asignar equipos a clientes. |
| **Gerente** | Dashboards de rentabilidad, proyecciones, morosidad, rendimiento de técnicos, sugerencias de reemplazo de equipos. |
| **Ventas** | Dar de alta prospectos, llenar cuestionario de nuevo contrato, simular cotizaciones con pricing engine, negociar condiciones, convertir prospecto en cliente, curar qué contenido de `FallaCatalogo` se publica en biblioteca pública. |
| **SAT (indirecto)** | Recepción de CFDI timbrados correctamente según el método de pago correspondiente (PUE para Venta y Escuela prepago; PPD para Renta). |

---

## 3. Restricciones

### 3.1 Regulatorias
- Facturación electrónica obligatoria a través de PAC autorizado: **Facturama**.
- Referencias bancarias para pago vía **SPEI** confirmadas por **Conekta**.
- Método de pago ante el SAT: **PUE** para Venta y Escuela prepago; **PPD** para Renta.

### 3.2 Técnicas
- Backend: **Python 3.12 + FastAPI**, desplegado en **AWS Elastic Beanstalk** mediante **Docker**.
- Base de datos transaccional: **AWS RDS PostgreSQL**.
- Autenticación y notificaciones push: **Firebase Auth + Firebase Cloud Messaging**.
- Canal de mensajería unificado: **Firestore**, acceso directo desde app cliente con token generado por backend. Este canal **no es exclusivo de tickets** — también transporta confirmaciones de lectura, confirmaciones de recibido de cotizaciones de renovación, y comunicación con prospectos.
- Almacenamiento de archivos y facturas: **AWS S3 + CloudFront**. Los archivos (manuales, facturas, evidencias, contratos firmados) se descargan desde la nube del proveedor mediante URL firmada; la app nunca los duplica ni los sirve como copia propia.
- Motor de búsqueda semántica: **pgvector** en PostgreSQL inicial, migrable a OpenSearch si el volumen lo requiere.
- Correo transaccional: **AWS SES**.
- CI/CD y control de versiones: **GitHub**.

### 3.3 Operativas

**Meterclick (aplica a los tres modelos):**
- Captura dentro de los primeros 10 días hábiles del mes.
- Alerta interna a Coordinación y Supervisión 3 días hábiles antes de vencer el plazo, si falta la lectura.
- Si se captura fuera de plazo, se registra con la fecha real de visita, evidencia y responsable — no se pierde ni se fuerza a la fecha límite.
- **Registro de persona autorizada obligatorio antes de trasladarse al equipo.** El técnico registra en la app/web nombre, puesto y celular de quien autorizará el acceso, y confirma si coincide con el registro previo (la app puede precargar el dato). **Si no existe persona autorizada disponible, el servicio no se realiza, no se genera visita pendiente imputable a la COMPAÑÍA, y no corre responsabilidad de SLA.** El cliente debe solicitar nueva programación.
- **Procedimiento normal:** el técnico imprime reporte del MFP (fecha, hora, número de serie, contador), escribe a mano la leyenda aplicable ("Toma de lectura", "Recibo de tóner de reserva", o ambas), la persona autorizada verifica y confirma numéricamente en la app/web, ambos firman, el técnico fotografía el documento firmado desde la app/web. El papel original queda en poder del cliente; la evidencia digital queda en la bitácora del MFP (visible según jerarquía) y se envía por email como acuse.
- **Procedimiento de contingencia:** si el MFP no puede imprimir el reporte (falta de tóner, papel, falla técnica u otra causa), el técnico fotografía el contador visible en el display, la persona autorizada verifica y confirma numéricamente, firma digital o en formato físico de contingencia. Tiene la misma validez operativa y probatoria que el reporte impreso, siempre que incluya identificación del equipo, fecha, responsable, contador y fotografía.
- **Validación de consistencia:** el sistema compara el contador capturado por el técnico contra la confirmación numérica del cliente y contra el volumen aproximado esperado según histórico. Si hay inconsistencia, exige verificación y recaptura antes de permitir el cierre de la visita. **No se permite edición destructiva de lecturas** — cualquier corrección se registra como evento correctivo (fecha, responsable, motivo, evidencia, trazabilidad), nunca como sobrescritura del dato original.

**Continuidad del contador (aplica a Renta y Venta con garantía):**
- Un contador **menor al anterior** solo es válido por cambio de número de serie del MFP o por reemplazo de tarjeta principal, memoria, o componente lógico que afecte la continuidad del contador.
- La orden correspondiente debe capturar: contador final del equipo/componente saliente, contador inicial posterior, número de serie involucrado, parte reemplazada y evidencia.
- Durante los **3 meses posteriores** al ajuste, toda factura o reporte de servicio debe incluir una leyenda de justificación (cambio de tarjeta lógica, memoria, o reemplazo de equipo).
- Cualquier otro contador menor al anterior queda **bloqueado como inconsistente** y no permite facturación automática.

**Contadores facturables por modelo (Renta y Venta):**
- Equipos de negro requieren contador negro/totalizador; equipos de color requieren contador negro y color (o totalizadores correspondientes).
- Solo son facturables/computables para garantía las impresiones en negro y color — no escaneos u otros contadores, salvo pacto expreso en Carátula.
- La compañía trabaja con equipos Ricoh y marcas comerciales propiedad de Ricoh; la nomenclatura del modelo (ej. MP/IM para negro, MPC/IMC para color) ayuda a determinar el tipo de contador requerido, sin perjuicio del catálogo maestro vigente.

**Notificaciones de consumo (Venta con garantía y Escuela prepago):**
- Notificación automática al 80%, 90% y 100% del volumen/bolsa pactado.
- Al alcanzar 100%: en Venta con garantía se suspende el servicio de esa bolsa mensual (no acumulativa) hasta el mes siguiente o hasta adquirir cobertura adicional. En Escuela prepago, se suspende el servicio hasta que la escuela adquiera un nuevo paquete.

**Doble confirmación de lectura mensual (Renta y Venta):** técnico captura en sitio, cliente confirma. Si no hay respuesta en 5 días hábiles, el sistema aplica aceptación tácita.

**Bloqueo por morosidad en Renta:** aviso preventivo entre 6 y 15 días de atraso (semáforo amarillo); bloqueo total a partir de 15 días naturales de atraso posteriores al vencimiento (semáforo rojo). Durante la suspensión no se factura ni cobra renta mínima del periodo suspendido (sí lo ya devengado antes de suspender). Reanudación en no más de 48 horas hábiles tras regularizar el adeudo.

**Excepción operativa en semáforo ROJO (Renta y Venta post-garantía/PF):** requiere **autorización conjunta de Coordinadora y Supervisor**, considerando historial de pagos, cuidado de equipos, reincidencias, consumo de materiales y criticidad operativa. **El técnico nunca está facultado para autorizar excepciones** — en su pantalla solo se muestra "bloqueado" / "no bloqueado", nunca el detalle del semáforo ni la capacidad de decidir. La decisión y su fundamento quedan registrados en sistema y pueden mostrarse al cliente de forma resumida (transparencia).

**Bloqueo en Venta (antes de garantía):** el equipo no sale de almacén hasta que el pago PUE es confirmado por el banco o medio de pago autorizado.

**Rescisión en Escuela prepago:** si el pago anual no se realiza dentro de los 10 días hábiles siguientes a la fecha pactada, la COMPAÑÍA puede rescindir el contrato sin responsabilidad para ella.

**Renovación — condición exacta para rechazo tácito (Escuela prepago y Venta post-garantía/PF):**
El rechazo tácito **no se activa solo por vencer el plazo de aceptación**. Se activa **si y solo si existe confirmación de recibido de la cotización de renovación**, generada y enviada por el sistema a través del mismo canal de mensajería (app) usado para confirmaciones de lectura. La secuencia es:
1. El sistema genera y envía la cotización de renovación (60 días naturales antes del vencimiento en Escuela prepago; 30 días de anticipación en Venta post-garantía/PF).
2. El cliente confirma **recibido** en la app — evento independiente de aceptar o rechazar el contenido.
3. Solo a partir de esa confirmación de recibido empieza a correr la ventana de aceptación (30 días naturales en Escuela prepago; 15 días naturales en Venta PF).
4. Si vence la ventana sin decisión expresa, **entonces sí** se asume rechazo tácito.
5. **Si nunca hay confirmación de recibido, el sistema no asume nada.** El seguimiento queda en estado de espera indefinida y requiere escalamiento manual de Coordinadora — no hay vencimiento automático sin confirmación previa. *(Pendiente de decisión del equipo: si conviene una alerta de antigüedad, ej. "sin confirmar por más de N días", para evitar que un seguimiento se pierda; el sistema no debe inventar esa regla por sí mismo.)*

**Inventario de técnicos:** cada técnico tiene un stock asignado; el consumo se registra con idempotency key para prevenir doble registro en zonas de mala conectividad. Toda entrega de tóner, material o refacción queda ligada a número de serie del MFP, RFC, contrato, contador, persona que recibió, fecha, responsable y evidencia.

**Falla por causa imputable al cliente:** cuando una falla, daño o consumo anómalo deriva de mal uso, negligencia, consumibles no autorizados, daño eléctrico o causa imputable al cliente, la compañía puede cobrar partes, materiales o servicios no cubiertos, documentando causa y evidencia.

**Orden de servicio y solución de campo (obligatorio, ver también sección 5 y 7):** toda orden de servicio queda inscrita con la falla reportada. Al cierre, el técnico está obligado a registrar la solución exacta que resolvió la falla en campo — no es opcional. Este dato alimenta el catálogo interno de experiencia técnica (`SolucionCampo`), distinto de la solución de laboratorio del fabricante.

### 3.4 De Negocio
- Renta: contratos sin plazo forzoso; cancelación con 30 días naturales de anticipación. Renta mínima mensual: 5,000 o 10,000 impresiones. Excedente fijo o escalonado según contrato.
- Venta: sin derecho de devolución una vez entregado el equipo, salvo defectos de fábrica no subsanables. Garantía cancelable anticipadamente por el cliente con 30 días naturales de aviso, sin reembolso del periodo no disfrutado.
- Escuela prepago: pago único anual anticipado; sin plazo forzoso más allá del periodo de 12 meses pactado.
- Márgenes de utilidad no inferiores al 35%.
- Ajuste de precios anual solo por INPC si la inflación anual excede el 5%, salvo pacto distinto en Carátula.
- Garantía en Venta: 12 meses desde la entrega **o** hasta alcanzar el volumen total de impresiones pactado, lo que ocurra primero. Bolsa de impresiones mensual no acumulativa.
- Bonificación por no uso: en Venta, las impresiones no utilizadas en garantía se convierten en saldo bonificable para la renovación. En Escuela prepago, la renovación se calcula con base en consumo real, bonificando a favor de la escuela las impresiones contratadas y no utilizadas conforme a política comercial.
- Escuela prepago — renovación con consumo bajo: si el consumo del periodo fue cero o inferior al 70% del volumen mínimo operativo definido por la compañía, la renovación puede cotizarse con volumen mínimo ajustado, o sugerirse la suspensión del servicio si no resulta viable económicamente.
- SLA: CDMX y área metropolitana, 8 horas hábiles. Distancias mayores a 100 km, 24 horas hábiles. Horario hábil: lunes a viernes, 8:00–18:00h.
- Gastos de viaje aplican para equipos a más de 50 km del centro (umbral distinto al de SLA extendido de 100 km — son dos reglas independientes, no deben confundirse).
- Exclusiones en los tres modelos: daños eléctricos, consumibles de terceros, negligencia, mal uso, siniestros, modificaciones no autorizadas, caso fortuito o fuerza mayor.
- Renta: los equipos pueden estar cubiertos por póliza global contra robo o pérdida material conforme a Carátula. En caso de siniestro, el cliente debe levantar acta ante autoridad competente dentro de 24 horas y entregar documentación necesaria. Existe `deducible_seguro` a cargo del cliente.
- RFC como unidad legal, contable y operativa: cada RFC es cliente legal y fiscalmente independiente, aun compartiendo domicilio, corporativo, personal administrativo o representantes con otros RFC. `GrupoCliente` es solo para administración, relación comercial, consulta consolidada y análisis interno — **no implica suspensión automática cruzada entre RFC**. Morosidad, semáforo, suspensión, bloqueo, reactivación y facturación se evalúan por RFC individual y por contrato.
- Venta post-garantía / modalidad preferente (PF): el equipo sigue siendo propiedad del cliente, pero el servicio se comporta operativamente como una renta (servicio, consumibles, lecturas, semáforo financiero, facturación).

---

## 4. Principios Arquitectónicos

1. **Modularidad estricta:** cada dominio de negocio es un módulo independiente con puertos, casos de uso, repositorios y adaptadores propios. Comunicación entre módulos por eventos asíncronos o llamadas a puertos definidos. Ningún módulo comparte modelos de datos internos; cada módulo gestiona sus propias migraciones y tablas.

2. **Tolerancia a fallos:** los fallos en servicios externos (Facturama, Conekta, Firebase) no se propagan. Circuit Breaker en todos los adaptadores externos. Operaciones críticas usan patrón **Transactional Outbox**.

3. **Seguridad desde el diseño:** autenticación centralizada en Firebase Auth con JWT. Autorización basada en `grupo_cliente_id` como tenant y roles. `module_key` es para trazabilidad y rate limiting interno, no mecanismo de seguridad. La biblioteca interna (partes, costos, soluciones de campo) nunca es accesible sin autenticación de rol interno, y nunca es indexable por buscadores o crawlers de IA.

4. **Simplicidad mantenible:** monolito modular hexagonal. Funciones ≤ 30 líneas. Archivos de lógica de negocio ≤ 200 líneas. Routers/schemas ≤ 150 líneas si la cohesión lo justifica.

5. **Transparencia y auditabilidad:** toda acción crítica (pagos, facturación, cambios de contrato, excepciones de semáforo, aceptaciones/rechazos tácitos, ajustes de continuidad de contador) queda en bitácora inmutable. Las soluciones de campo pasan por validación de Supervisor antes de integrarse al catálogo interno de experiencia técnica.

6. **Experiencia centrada en el cliente:** el flujo de reporte de fallas permite autoservicio vía biblioteca antes de crear una orden de servicio. Confirmaciones de lectura, confirmaciones de recibido de cotizaciones y avisos se comunican por el canal unificado de mensajería de la app. El sistema distingue entre cliente de renta, comprador y escuela prepago; cada uno tiene una experiencia post-contratación distinta (la escuela no tiene semáforo ni facturación recurrente; el comprador tiene garantía con bolsa mensual; el rentista tiene semáforo y facturación PPD mensual).

7. **El PDF es fuente, no repositorio.** Los manuales técnicos no se exponen al usuario como documentos para hojear. Se procesan una sola vez (extracción, chunking, embeddings en pgvector) y de ahí se derivan registros estructurados (`FallaCatalogo`) que sí se muestran en pantalla. El PDF original queda como referencia trazable (S3 key + página), no como la interfaz de consulta.

8. **La experiencia de campo es un activo protegido, no un artículo de biblioteca pública.** La solución que el fabricante documenta en el manual (laboratorio controlado) y la solución que un técnico aplica realmente en campo son dos fuentes distintas, mostradas por separado en la pantalla del técnico. Solo la primera (`FallaCatalogo`, de origen fabricante) es candidata a biblioteca pública o de cliente. La segunda (`SolucionCampo`) es material de capacitación interna exclusivo de servicio técnico — nunca se publica, es la ventaja competitiva de la operación.

---

## 5. Bibliotecas del Sistema

El sistema opera **tres bibliotecas con audiencias y reglas de visibilidad distintas.** No son vistas de un mismo contenido con permisos — son colecciones de contenido diferentes por diseño.

| Biblioteca | Audiencia | Contenido | Indexable por buscadores/LLMs |
|---|---|---|---|
| **Pública** | Público general, prospectos | Guías de compra y renta, funcionamiento general de MFP, fallas comunes y su solución **de fabricante** (`FallaCatalogo`), recomendaciones, comparativas renta vs compra, calculadora de costo por impresión | **Sí, deliberadamente** — es la estrategia de descubrimiento por buscadores y LLMs |
| **Cliente** | Usuario principal, contabilidad, representante legal de cada RFC | Manuales de operación **filtrados por los equipos realmente instalados** en su `grupo_cliente_id` (no el catálogo completo), tips y recomendaciones específicas de sus equipos | No — requiere autenticación |
| **Interna** | Técnico, Coordinadora, Supervisor | Catálogos de partes, costos, tablas de mantenimiento del fabricante completas, **`SolucionCampo` (caja negra / expertise de campo)**, información comercial no compartible | No — nunca, bajo ninguna circunstancia. `robots: noindex` explícito y sin excepción. |

**Regla explícita sobre `SolucionCampo`:** es material de capacitación interna del área técnica. No se promueve a biblioteca de cliente ni pública de forma automática bajo ninguna condición. Si en el futuro se decide publicar algo derivado de la experiencia de campo, requiere curación manual explícita por Ventas/Contenido y debe reformularse — nunca se expone el registro original.

**Regla explícita sobre `FallaCatalogo`:** al ser de origen fabricante (información de dominio público en esencia, disponible en manuales de servicio), sí es candidata natural a biblioteca pública, curada por Ventas.

**Regla explícita sobre costos y cotizaciones — tres niveles de acceso, no dos.**
El costo real por impresión (CPI) es información distinta de la metodología para calcularlo. La biblioteca pública puede enseñar el *cómo* sin regalar el *cuánto* de la operación de la compañía. Esto se modela como tres niveles de profundidad, cada uno requiriendo más del prospecto a cambio de más precisión:

| Nivel | Qué recibe el prospecto | Qué entrega el prospecto | Dónde vive |
|---|---|---|---|
| **Público (metodología)** | Fórmula del CPI explicada, regla 1X/2X, diferencia brochure vs. campo, comparativas ilustrativas con cifras genéricas o de rango — **nunca el desglose real de costos de refacciones por pieza** | Nada — libre acceso | `ArticuloBiblioteca`, nivel_acceso = publico |
| **Estimado rápido (embudo ligero)** | Un rango aproximado de renta/CPI mensual, calculado con una fórmula simplificada de referencia, explícitamente etiquetado como estimado no vinculante | Volumen mensual global aproximado + datos de contacto básicos | `SolicitudCotizacion`, tipo = estimado_rapido |
| **Cotización real (embudo calificado)** | El CPI real, calculado con costos reales de refacciones, tóner y servicio para su volumen específico | Volumen global, volumen por área, volumen por MFP, o solicitud explícita de ser atendido por un asesor técnico | `SolicitudCotizacion`, tipo = cotizacion_detallada o asesor_tecnico |

**Ningún artículo de biblioteca pública ni el cotizador de autoservicio del sitio expone montos reales de refacciones, tóner o servicio.** Esos valores solo existen en `SolucionCampo`/catálogo interno y en el resultado calculado de una `SolicitudCotizacion` calificada — nunca en contenido estático indexable.

---

## 6. Modelo de Dominio

### 6.1 Entidades centrales de clientes y contratos

**GrupoCliente**, **RFC** — sin cambios respecto a lo ya definido: agregado raíz para agrupar RFC de una misma entidad corporativa; cada RFC es unidad legal, fiscal y operativa independiente (ver 3.4).

**PersonaAutorizada** (reemplaza y amplía a `ContactoRFC`)
Registro de personas facultadas para autorizar acceso del técnico, confirmar contadores, recibir materiales, firmar órdenes de servicio y cerrar visitas — exigido por los tres contratos maestros.

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| rfc_id | FK → RFC | |
| nombre | string | |
| puesto | string | |
| celular | string | Obligatorio |
| email | string (nullable) | |
| alcance | JSONB / flags | autoriza_acceso, confirma_contador, recibe_materiales, firma_orden, cierra_visita |
| activo | boolean | |

**RegistroAccesoVisita** (nueva)
Captura obligatoria, previa al traslado del técnico al equipo, de quién autorizará el acceso en cada visita.

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| orden_servicio_id | FK → OrdenServicio | |
| persona_autorizada_id | FK → PersonaAutorizada (nullable) | Nulo si no hay persona disponible |
| coincide_con_registro_previo | boolean | |
| fecha_registro | timestamp | Debe preceder al traslado físico |
| tecnico_id | FK | |
| visita_realizada | boolean | False si no hubo persona autorizada disponible |
| motivo_no_realizada | enum (nullable) | sin_persona_autorizada, otro |

**Contrato**
Documento que vincula un RFC con un servicio de renta, venta o escuela prepago. Eje del sistema.

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| rfc_id | FK → RFC | |
| tipo_negocio | enum: RENTA, VENTA, ESCUELA_PREPAGO | |
| metodo_pago_sat | enum: PUE, PPD | PUE para VENTA y ESCUELA_PREPAGO; PPD para RENTA |
| minimo_impresiones | integer (nullable) | Solo RENTA |
| tipo_excedente | enum: FIJO, ESCALONADO (nullable) | Solo RENTA |
| costo_excedente_fijo | decimal (nullable) | |
| escalon_excedente | JSONB (nullable) | |
| tiene_garantia_incluida | boolean | Solo VENTA |
| fecha_vencimiento_garantia | date (nullable) | Solo VENTA |
| volumen_anual_pactado | integer (nullable) | Solo ESCUELA_PREPAGO |
| precio_anual_total_pue | decimal (nullable) | Solo ESCUELA_PREPAGO |
| tiene_seguro | boolean | Aplica principalmente a RENTA |
| deducible_seguro | decimal (nullable) | |
| estado_servicio | enum | activo, suspendido_por_morosidad, suspendido_por_consumo, suspendido_por_agotamiento_anual |
| modalidad_pf | boolean | True si es Venta post-garantía operando como renta (personal preferente) |
| fecha_inicio | date | |
| fecha_fin | date (nullable) | |
| estado | enum | borrador, activo, suspendido, cancelado, rescindido |

**BolsaGarantiaMFP** (solo VENTA)
Control de impresiones cubiertas por garantía, límite mensual no acumulativo.

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| contrato_id | FK → Contrato | |
| limite_impresiones_total | integer | Límite global de garantía (ej. hasta 12 meses o volumen total, lo que ocurra primero) |
| limite_mensual_no_acumulativo | integer | |
| impresiones_consumidas_total | integer | |
| mes_actual_consumo | integer | Se reinicia al cambiar el mes calendario |
| fecha_inicio | date | |
| fecha_fin_estimada | date (nullable) | min(fecha_vencimiento, proyección de consumo) |
| porcentaje_80_notificado | boolean | |
| porcentaje_90_notificado | boolean | |
| porcentaje_100_notificado | boolean | |
| fecha_suspension_consumo | timestamp (nullable) | |

**SeguimientoConsumoAnual** (nueva — solo ESCUELA_PREPAGO)
Equivalente conceptual a `BolsaGarantiaMFP` pero **acumulativo anual**, sin reinicio mensual, y sin efecto de facturación (solo control de vigencia y disparo de renovación).

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| contrato_id | FK → Contrato | |
| volumen_anual_pactado | integer | |
| volumen_consumido_acumulado | integer | Se actualiza con cada Meterclick, nunca se reinicia dentro del periodo de 12 meses |
| porcentaje_80_notificado | boolean | |
| porcentaje_90_notificado | boolean | |
| porcentaje_100_notificado | boolean | |
| fecha_vencimiento | date | |
| fecha_disparo_seguimiento_renovacion | date | Calculada: fecha_vencimiento − 60 días naturales |
| estado | enum | activo, en_seguimiento_renovacion, renovado, suspendido_por_agotamiento |

**SeguimientoRenovacion** (nueva — aplica a ESCUELA_PREPAGO y a VENTA post-garantía/PF)
Modela el flujo exacto de renovación con confirmación de recibido como condición para el rechazo tácito.

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| contrato_id | FK → Contrato | |
| tipo_origen | enum | escuela_prepago, venta_post_garantia |
| cotizacion_generada_en | timestamp | |
| dias_anticipacion_notificacion | integer | 60 (escuela) o 30 (venta PF) |
| notificacion_enviada_en | timestamp | Vía canal unificado de mensajería |
| confirmacion_recibido_en | timestamp (nullable) | Evento independiente de aceptar/rechazar contenido |
| confirmado_por | FK → PersonaAutorizada (nullable) | |
| dias_ventana_aceptacion | integer | 30 (escuela) o 15 (venta PF) — **cuenta a partir de `confirmacion_recibido_en`, no de `notificacion_enviada_en`** |
| ventana_aceptacion_vence_en | date (nullable) | Calculada solo una vez hay confirmación de recibido |
| decision | enum | pendiente, esperando_confirmacion_recibido, aceptada, rechazo_tacito, rechazo_expreso |
| consumo_real_periodo_anterior | integer (nullable) | Base del cálculo de renovación |
| saldo_bonificable_aplicado | decimal (nullable) | |

**SuspensionServicio**
Bitácora de suspensiones, con control de facturación detenida.

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| contrato_id | FK → Contrato | |
| tipo | enum | morosidad, consumo_agotado (venta garantía mensual), agotamiento_anual (escuela) |
| fecha_inicio | timestamp | |
| fecha_fin | timestamp (nullable) | |
| facturacion_detenida_desde | date | Última factura antes de la suspensión (aplica a Renta) |

**AutorizacionExcepcionSemaforo** (nueva)
Formaliza la autorización conjunta exigida por el contrato de Renta para operar en ROJO.

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| contrato_id | FK → Contrato | |
| coordinadora_id | FK → usuarios_roles | Obligatorio — no puede autorizarse sin ambos roles |
| supervisor_id | FK → usuarios_roles | Obligatorio |
| fecha | timestamp | |
| criterios_considerados | JSONB | historial_pagos, cuidado_equipos, reincidencias, consumo_materiales, criticidad_operativa |
| resultado | enum | autorizada, negada |
| resumen_visible_cliente | text | Para transparencia — versión resumida mostrada al cliente |

### 6.2 Catálogo de equipos y sugerencia de modelo

**ModeloMFP** (formalizada — antes solo referenciada, no detallada)
Además de los datos de ficha técnica, sostiene la regla de sugerencia automática de modelo para el estimado del Paso 1 del embudo, basada en rendimiento real de campo, no en el dato de brochure del fabricante.

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| marca | string | |
| modelo | string | |
| tipo | enum | negro, color |
| rendimiento_toner_fabricante | integer | Dato de brochure — se conserva solo como referencia comparativa educativa |
| rendimiento_toner_campo | integer | Dato real medido en campo — base de todos los cálculos operativos |
| volumen_optimo_1x | integer | = `rendimiento_toner_campo`. Volumen mensual de diseño |
| volumen_maximo_2x | integer | = `rendimiento_toner_campo * 2`. Límite seguro; por encima, desgaste acelerado |
| cpi_referencia_rango | JSONB | `{min, max}` — usado únicamente para el estimado rápido público, nunca desglosado por refacción |

**Regla de sugerencia de modelo (Paso 1 del embudo):**
Dado el `volumen_global_mensual` declarado por el prospecto, el sistema busca el `ModeloMFP` cuyo `volumen_optimo_1x` cubra ese volumen. Si el volumen cae entre `volumen_optimo_1x` y `volumen_maximo_2x` de un modelo, se sugiere ese modelo con nota de "límite seguro, evaluar equipo superior". Si excede `volumen_maximo_2x` de todos los modelos del catálogo, no se sugiere modelo automáticamente — se marca `requiere_evaluacion_personalizada = true` y se enruta directo a `asesor_tecnico`. Esta lógica vive en el catálogo, nunca como tabla fija en código o en el contenido del correo automático — así escala sin tocar la plantilla cada vez que se agrega un modelo nuevo.

### 6.3 Equipos, lecturas y evidencia

**Equipo (Asset)** — sin cambios respecto a lo ya definido: `asset_id` interno, `serial_number` único visible, historial completo aunque cambie de cliente, `propietario_legal` (EMPRESA en Renta y Escuela prepago, CLIENTE en Venta).

**Lectura**

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| equipo_id | FK → Equipo | |
| contrato_id | FK → Contrato | |
| tecnico_id | FK | |
| contador_anterior | integer | |
| contador_actual | integer | |
| diferencia | integer (calculado) | |
| excedente | integer (calculado) | Solo RENTA |
| excedente_garantia | integer (calculado) | Solo VENTA con garantía; no se factura |
| aplica_solo_control_vigencia | boolean | True en ESCUELA_PREPAGO — la lectura no genera facturación, solo actualiza `SeguimientoConsumoAnual` |
| fecha_captura | timestamp | |
| estado_confirmacion | enum | pendiente, confirmado_cliente, confirmado_sistema, anulada |
| metodo_confirmacion | enum | manual, aceptacion_tacita |
| confirmado_por | FK → PersonaAutorizada (nullable) | |
| fecha_confirmacion | timestamp (nullable) | |
| bloqueada_por_inconsistencia | boolean | True mientras espera recaptura/verificación |
| lectura_corregida_id | FK → Lectura (nullable) | Referencia a la lectura que esta corrige; la original nunca se sobrescribe |

**EvidenciaLectura**
Ampliada respecto a la versión anterior con el detalle exacto exigido por los tres contratos.

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| lectura_id | FK → Lectura | |
| registro_acceso_visita_id | FK → RegistroAccesoVisita | |
| tipo | enum | normal, contingencia |
| leyenda_manuscrita | enum | toma_lectura, recibo_toner_reserva, ambas |
| codigo_contingencia | enum (nullable) | |
| foto_documento_firmado_s3_key | string (nullable) | Procedimiento normal |
| foto_contador_display_s3_key | string (nullable) | Procedimiento de contingencia |
| firma_tecnico | string | Hash o referencia |
| firma_persona_autorizada | string | Digital o referencia a formato físico de contingencia |
| papel_original_en_poder_cliente | boolean | Default true |
| notificacion_email_enviada | boolean | |
| tecnico_id | FK | |

**AjusteContinuidadContador** (nueva)
Registra el evento que justifica un contador menor al anterior.

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| equipo_id | FK → Equipo | |
| orden_servicio_id | FK → OrdenServicio | |
| tipo_evento | enum | cambio_numero_serie, reemplazo_tarjeta_logica, reemplazo_memoria |
| contador_final_anterior | integer | |
| contador_inicial_posterior | integer | |
| numero_serie_saliente | string (nullable) | |
| numero_serie_entrante | string (nullable) | |
| parte_reemplazada | string | |
| evidencia_s3_key | string | |
| fecha | timestamp | |
| tecnico_id | FK | |
| fecha_fin_leyenda_obligatoria | date | Calculada: fecha + 3 meses. Toda factura/reporte emitido dentro de esta ventana debe incluir leyenda de justificación referenciando este registro |

### 6.4 Servicio técnico, fallas y biblioteca

**OrdenServicio** (equivalente operativo de "Ticket" cara a contrato — mismo agregado, nombre alineado al lenguaje contractual)

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| equipo_id | FK → Equipo | |
| contrato_id | FK → Contrato | |
| falla_reportada_id | FK → FallaCatalogo (nullable) | Nulo si es una falla no catalogada todavía |
| descripcion_falla_reportada | text | |
| tecnico_id | FK | |
| estado | enum | abierta, en_proceso, pendiente_solucion_campo, cerrada |
| requiere_solucion_campo_para_cierre | boolean | **Siempre true** — regla de negocio, no configurable |
| fecha_apertura | timestamp | |
| fecha_cierre | timestamp (nullable) | |

**FallaCatalogo** — fuente: manual del fabricante ("laboratorio controlado"). Candidata a biblioteca pública/cliente.

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| modelo_id | FK → ModeloMFP | |
| codigo_falla | string | |
| descripcion | text | |
| sintesis_solucion_fabricante | text | Generada por extracción del manual — no el PDF completo |
| manual_pdf_s3_key | string | Referencia trazable, no se presenta como documento a hojear |
| pagina_referencia | integer (nullable) | |
| publicable | boolean | Curado por Ventas/Contenido antes de aparecer en biblioteca pública |

**SolucionCampo** — caja negra interna. **Nunca pública, nunca visible a cliente.** Material de capacitación exclusivo de servicio técnico.

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| orden_servicio_id | FK → OrdenServicio | Obligatorio al cerrar la orden |
| falla_catalogo_id | FK (nullable) | Nulo si la falla es nueva, no catalogada aún |
| tecnico_id | FK | Responsable de la exactitud del dato |
| descripcion_solucion | text | |
| evidencia_s3_key | string (nullable) | |
| estado | enum | pendiente_validacion, validada, rechazada |
| validado_por | FK → supervisor (nullable) | |
| veces_confirmada_por_otros_tecnicos | integer | Señal de confiabilidad de campo |
| visibilidad | constante = interna | No configurable; no existe ruta de código que la exponga fuera de biblioteca interna |

**Regla de pantalla del técnico:** ante una falla dada, se muestran dos bloques separados y explícitamente etiquetados — nunca mezclados como si fueran la misma fuente:
- **"Según fabricante"** → `sintesis_solucion_fabricante` de `FallaCatalogo`.
- **"Resuelto en campo"** → lista de `SolucionCampo` validadas, ordenadas por `veces_confirmada_por_otros_tecnicos`.

**ArticuloBiblioteca** (nueva — contenido editorial, distinto de `FallaCatalogo`)

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| nivel_acceso | enum | publico, cliente, interno |
| tipo | enum | guia_compra, guia_renta, funcionamiento, manual_operacion, falla_solucion_fabricante, tip_recomendacion |
| modelo_id | FK → ModeloMFP (nullable) | Para manuales filtrados por equipo instalado (biblioteca cliente) |
| titulo | string | |
| contenido | text | |
| fuente | enum | editorial, fabricante (derivado de FallaCatalogo.publicable) |
| fecha_actualizacion | date | Visible en el artículo — señal de vigencia para SEO/LLM |
| datos_estructurados_json_ld | JSONB (nullable) | FAQPage / HowTo, según tipo |

### 6.5 Prospección y comunicación

**Prospecto** (nueva)

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| nombre_contacto | string | |
| escuela_o_empresa | string | |
| email | string | |
| telefono | string | |
| tipo_interes | enum | renta, venta, escuela_prepago |
| origen | enum | funnel_web, referido, biblioteca | 
| estado | enum | nuevo, contactado, cotizado, convertido, descartado |
| responsable_id | FK → usuarios_roles (Ventas) | Vendedor dueño del prospecto — todo prospecto debe tener responsable asignado |
| fecha_alta | timestamp | |
| convertido_a_rfc_id | FK → RFC (nullable) | |

**SeguimientoComercial** (nueva — "equivalente a Salesforce" acotado al problema real: que nada se pierda por omisión)

No es un CRM genérico; es un semáforo de gestión igual en principio al semáforo financiero, pero aplicado a que todo prospecto y toda renovación tengan dueño, próxima acción programada y una alerta automática si esa acción no ocurre. Se instancia sobre `Prospecto` o sobre `SeguimientoRenovacion` indistintamente — ambos son, en el fondo, el mismo problema: "una relación comercial que puede morir en silencio si nadie le da seguimiento".

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| entidad_tipo | enum | prospecto, renovacion |
| entidad_id | UUID | FK polimórfica a `Prospecto.id` o `SeguimientoRenovacion.id` |
| responsable_id | FK → usuarios_roles | Vendedor o Coordinadora dueño del seguimiento |
| etapa | enum | Depende de `entidad_tipo` (ver tabla de etapas abajo) |
| fecha_ultima_actividad | timestamp (nullable) | |
| fecha_proxima_accion | date | Obligatoria — no puede quedar un seguimiento sin próxima acción programada |
| tipo_proxima_accion | enum | llamada, mensaje, visita, envio_cotizacion, escalar_coordinacion |
| color_semaforo | enum: VERDE, AMARILLO, ROJO | Calculado por job diario, ver 7.2.1 |
| dias_umbral_amarillo | integer | Configurable por etapa (`EtapaComercialConfig`) |
| dias_umbral_rojo | integer | Configurable por etapa |

**Etapas por tipo de entidad:**

| entidad_tipo | Etapas |
|---|---|
| prospecto | nuevo → contactado → cotizado → en_negociacion → convertido / descartado |
| renovacion | cotizacion_generada → notificado → esperando_confirmacion_recibido → en_decision → cerrado |

**ActividadComercial** (nueva)
Bitácora de cada interacción — es lo que sostiene o reinicia el semáforo comercial.

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| seguimiento_id | FK → SeguimientoComercial | |
| tipo | enum | llamada, mensaje, visita, cotizacion_enviada, email, nota_interna |
| canal | enum | mensajeria_app, telefono, email, presencial |
| descripcion | text | |
| fecha | timestamp | |
| usuario_id | FK → usuarios_roles | |

**EtapaComercialConfig** (nueva — parametrizable, no hardcodeado)

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| entidad_tipo | enum | prospecto, renovacion |
| etapa | string | |
| dias_habiles_amarillo | integer | Días sin actividad/acción vencida para pasar a amarillo |
| dias_habiles_rojo | integer | Días sin actividad/acción vencida para pasar a rojo |

**SolicitudCotizacion** (nueva)
Formaliza el embudo escalonado: separa el "estimado rápido" del "cálculo real", y define qué datos habilitan cada uno. Los tres niveles corresponden a los tres pasos del embudo: estimado web (30 segundos) → llamada de asesor → propuesta formal post-visita.

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| prospecto_id | FK → Prospecto | |
| tipo_solicitud | enum | estimado_rapido, cotizacion_detallada, asesor_tecnico |
| volumen_global_mensual | integer (nullable) | Único dato de volumen requerido para `estimado_rapido` |
| ciudad_estado_detectada | string (nullable) | Capturada pasivamente por geolocalización de navegador/IP — nunca preguntada en el formulario |
| modelo_sugerido_id | FK → ModeloMFP (nullable) | Calculado, no hardcodeado — ver regla de sugerencia en `ModeloMFP` |
| cpi_estimado_rango | JSONB (nullable) | `{min, max}` — nunca un valor puntual, para no prometer una precisión que el dato de entrada no sostiene |
| costo_mensual_estimado_rango | JSONB (nullable) | `{min, max}` |
| volumen_por_area | JSONB (nullable) | `[{area, volumen_mensual}]` — recolectado por el asesor durante la llamada (paso 2), no en formulario público |
| equipos | JSONB (nullable) | `[{modelo_id, volumen_mensual}]` — máxima precisión, habilita el CPI real completo |
| confiabilidad_volumen_declarado | enum (nullable) | registrado_en_contador, estimado_aproximado, desconocido, calculo_proveedor_anterior — bandera para el asesor si no es `registrado_en_contador` |
| prefiere_asesor | boolean | Si es true, se omite el cálculo automático y se enruta directo a Ventas vía `SeguimientoComercial` |
| estado | enum | nueva, en_calculo, entregada |
| cpi_calculado | decimal (nullable) | Valor puntual real — solo se puebla para `cotizacion_detallada` o `asesor_tecnico`, nunca para `estimado_rapido` |
| fecha_solicitud | timestamp | |
| fecha_entrega | timestamp (nullable) | |

Toda `SolicitudCotizacion` con `tipo_solicitud != estimado_rapido` crea automáticamente un `SeguimientoComercial` (ver 6.5) con responsable asignado — así ninguna solicitud calificada queda solo como un registro pasivo en la base de datos.

**PropuestaFormal** (nueva — Paso 3, post-visita)

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| solicitud_cotizacion_id | FK → SolicitudCotizacion | Debe ser `tipo_solicitud = asesor_tecnico` con visita ya realizada |
| cpi_real | decimal | Calculado con volumen validado en campo — valor puntual, ya no rango |
| modelo_recomendado_id | FK → ModeloMFP | |
| opcion_contrato | enum | renta, venta, escuela_prepago |
| incluye | JSONB | Lista: equipo, tóner, refacciones, servicio, mantenimiento |
| terminos_condiciones_s3_key | string | |
| firma_digital | string (nullable) | |
| estado | enum | borrador, enviada, firmada, rechazada |
| fecha_visita | timestamp | |
| fecha_entrega | timestamp | Debe caer dentro de 24–48 horas posteriores a `fecha_visita` |

**Canal de mensajería unificado (Firestore):** un mismo mecanismo de conversación transporta, según contexto: chat de órdenes de servicio, confirmaciones de lectura, confirmaciones de recibido de `SeguimientoRenovacion`, y comunicación con `Prospecto`. Se modela por `entity_type` + `entity_id` (ticket, renovacion, prospecto) en vez de canales separados por función, para no duplicar infraestructura de mensajería.

**Actividad** (nueva)
Registro de interacción comercial, equivalente funcional a una "actividad" o "tarea" de un CRM. Es el insumo del semáforo comercial.

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| entidad_tipo | enum | prospecto, renovacion |
| entidad_id | UUID | `Prospecto.id` o `SeguimientoRenovacion.id`, según `entidad_tipo` |
| tipo | enum | llamada, email, visita, mensaje_app, nota |
| responsable_id | FK → usuarios_roles | Ventas o Coordinadora, según el caso |
| fecha | timestamp | |
| resultado | text | |
| fecha_proxima_accion | date (nullable) | Si queda nula, el semáforo lo trata como riesgo de omisión |

**SemaforoComercial** (nueva)
Estado calculado de seguimiento — mismo principio que `SemaforoGrupo`, pero mide "tiempo sin seguimiento" en vez de "tiempo sin pago". Polimórfico sobre `Prospecto` o `SeguimientoRenovacion`.

| Atributo | Tipo | Descripción |
|---|---|---|
| id | UUID PK | |
| entidad_tipo | enum | prospecto, renovacion |
| entidad_id | UUID | |
| color | enum | VERDE, AMARILLO, ROJO |
| dias_sin_actividad | integer | |
| fecha_proxima_accion_vencida | boolean | |
| responsable_actual_id | FK → usuarios_roles | |
| motivo | text | |
| fecha_actualizacion | timestamp | |

### 6.6 Facturación, pagos y semáforo

Sin cambios de fondo respecto a lo ya definido — **Factura**, **ReferenciaBancaria**, **Pago**, **SemaforoGrupo**, **HistorialSemaforo** — salvo la corrección ya señalada: cualquier `trigger_evento = OVERRIDE_SUPERVISOR` en `HistorialSemaforo` debe reflejar que la autorización es conjunta (Coordinadora + Supervisor), registrada formalmente en `AutorizacionExcepcionSemaforo`, no una decisión unilateral de Supervisor.

**Nota explícita:** `SemaforoGrupo` **no se instancia ni se evalúa** para contratos `tipo_negocio = ESCUELA_PREPAGO`. El control de ese modelo es exclusivamente `SeguimientoConsumoAnual` + `SeguimientoRenovacion`.

### 6.7 Inventario

**Insumo, Almacén, Stock, MovimientoInventario** — sin cambios respecto a lo ya definido.

---

## 7. Motor de Semáforos y Eventos Críticos

### 7.1 Principio de operación

El semáforo es una **Máquina de Estados Finitos (FSM)** reactiva, procesando eventos del negocio de forma asíncrona vía Event Bus + patrón Outbox. **Aplica solo a Renta y a Venta post-garantía/PF — nunca a Escuela prepago.**

### 7.2 Estados del Semáforo

**Motor RENTA (PPD):**

| Estado | Condición | Efecto |
|---|---|---|
| 🟢 VERDE | Sin facturas vencidas, o vencidas ≤ 5 días | Operación 100% liberada |
| 🟡 AMARILLO | Al menos un RFC con factura vencida entre 6 y 15 días | Aviso preventivo; sin restricción operativa |
| 🔴 ROJO | Al menos un RFC con factura vencida > 15 días naturales | Bloqueo total de nuevos tickets y suministros para ese RFC. Excepción posible solo con `AutorizacionExcepcionSemaforo` conjunta |

**Motor VENTA (PUE, previo a garantía):** binario. Rojo por defecto al crear el contrato; verde solo al confirmar pago y timbrar factura PUE — bloquea salida del equipo de almacén.

**Motor VENTA_GARANTIA:** mientras la garantía está activa, el semáforo es verde incondicional para suministros y soporte. Al finalizar, pasa a `POST_GARANTIA_PENDIENTE` y dispara `SeguimientoRenovacion`. Si se acepta modalidad PF, a partir de ahí sí aplica el motor RENTA por RFC individual.

**Modelo ESCUELA_PREPAGO:** no tiene motor de semáforo *financiero*. El control operativo es `SeguimientoConsumoAnual.estado` y `SeguimientoRenovacion.decision`.

**Motor COMERCIAL (nuevo — seguimiento de prospectos y renovaciones):** independiente del semáforo financiero, aplica a `SeguimientoComercial` sin importar el `tipo_negocio` del contrato. Su función no es cobranza, es evitar omisión.

| Estado | Condición | Efecto |
|---|---|---|
| 🟢 VERDE | `fecha_proxima_accion` está dentro de la ventana esperada para la etapa actual y no ha vencido | Sin alerta; aparece en el tablero normal del vendedor/Coordinadora |
| 🟡 AMARILLO | `fecha_proxima_accion` vencida entre `dias_habiles_amarillo` y `dias_habiles_rojo` sin nueva `ActividadComercial` registrada | Aviso al responsable asignado |
| 🔴 ROJO | Vencida más allá de `dias_habiles_rojo`, **o** una `SeguimientoRenovacion` permanece en `esperando_confirmacion_recibido` más allá del umbral configurado | Escalamiento automático a Coordinadora, visible en tablero de riesgo de omisión |

Esto resuelve directamente la pregunta P1 que había quedado abierta: una renovación que nunca recibe confirmación de recibido no vence automáticamente su decisión (eso seguiría siendo incorrecto — el contrato no lo autoriza), pero sí **se vuelve visible y escalable** antes de perderse, en vez de quedar en silencio indefinido.

### 7.3 Eventos críticos del sistema

- `PAGO.RECIBIDO`, `PAGO.CONCILIADO`
- `FACTURA.VENCIDA`, `FACTURA.TIMBRADA`
- `LECTURA.CAPTURADA`, `LECTURA.CONFIRMADA`, `LECTURA.BLOQUEADA_POR_INCONSISTENCIA`, `LECTURA.EVIDENCIA_CARGADA`
- `CONTRATO.ACTIVADO`
- `SEMAFORO.ACTUALIZADO`
- `SEMAFORO.EXCEPCION_AUTORIZADA` (requiere `coordinadora_id` y `supervisor_id` en el payload)
- `GARANTIA.PORCENTAJE_80`, `GARANTIA.PORCENTAJE_90`, `GARANTIA.CONSUMO_AGOTADO`
- `VOLUMEN_ANUAL.PORCENTAJE_80`, `VOLUMEN_ANUAL.PORCENTAJE_90`, `VOLUMEN_ANUAL.AGOTADO` (Escuela prepago)
- `SERVICIO.SUSPENDIDO`, `SERVICIO.REANUDADO`
- `RENOVACION.COTIZACION_GENERADA`, `RENOVACION.CONFIRMACION_RECIBIDO`, `RENOVACION.ACEPTADA`, `RENOVACION.RECHAZO_TACITO`, `RENOVACION.RECHAZO_EXPRESO`
- `RENOVACION.SALDO_BONIFICABLE`
- `CONTINUIDAD_CONTADOR.AJUSTE_REGISTRADO`
- `ORDEN_SERVICIO.CREADA`, `ORDEN_SERVICIO.CERRADA`
- `SOLUCION_CAMPO.REGISTRADA`, `SOLUCION_CAMPO.VALIDADA`
- `SEGUIMIENTO_COMERCIAL.CREADO`, `SEGUIMIENTO_COMERCIAL.ACTIVIDAD_REGISTRADA`
- `SEGUIMIENTO_COMERCIAL.AMARILLO`, `SEGUIMIENTO_COMERCIAL.ROJO`, `SEGUIMIENTO_COMERCIAL.ESCALADO`
- `SOLICITUD_COTIZACION.CREADA`, `SOLICITUD_COTIZACION.CALCULADA`, `SOLICITUD_COTIZACION.ENTREGADA`
- `MODELO.SUGERIDO_AUTOMATICAMENTE`, `PROPUESTA_FORMAL.ENVIADA`, `PROPUESTA_FORMAL.FIRMADA`

### 7.4 Job de monitoreo de consumo (generalizado a Venta y Escuela prepago)

```python
def evaluar_consumo_garantia():
    for bolsa in BolsaGarantiaMFP.activas():
        porcentaje = (bolsa.impresiones_consumidas_total / bolsa.limite_impresiones_total) * 100
        _notificar_umbral(bolsa, porcentaje, prefijo_evento='GARANTIA')
        if porcentaje >= 100 and not bolsa.porcentaje_100_notificado:
            bolsa.contrato.estado_servicio = 'suspendido_por_consumo'
            registrar_suspension(bolsa.contrato, tipo='consumo_agotado')

def evaluar_volumen_anual_escuela():
    for seguimiento in SeguimientoConsumoAnual.activos():
        porcentaje = (seguimiento.volumen_consumido_acumulado / seguimiento.volumen_anual_pactado) * 100
        _notificar_umbral(seguimiento, porcentaje, prefijo_evento='VOLUMEN_ANUAL')
        if porcentaje >= 100 and not seguimiento.porcentaje_100_notificado:
            seguimiento.contrato.estado_servicio = 'suspendido_por_agotamiento_anual'
            registrar_suspension(seguimiento.contrato, tipo='agotamiento_anual')

def evaluar_disparo_renovaciones():
    # Escuela: dispara a fecha_vencimiento - 60 días naturales
    # Venta PF: dispara a fecha_vencimiento_garantia - 30 días naturales
    for seguimiento in SeguimientoConsumoAnual.proximas_a_disparo():
        crear_seguimiento_renovacion(seguimiento.contrato, tipo_origen='escuela_prepago')
    for contrato in Contrato.venta_proxima_a_vencer_garantia():
        crear_seguimiento_renovacion(contrato, tipo_origen='venta_post_garantia')

def evaluar_ventanas_aceptacion_renovacion():
    # Solo evalúa vencimiento de ventana si YA hubo confirmación de recibido
    for seg in SeguimientoRenovacion.con_confirmacion_recibido_pendientes():
        if hoy() > seg.ventana_aceptacion_vence_en:
            seg.decision = 'rechazo_tacito'
            emitir_evento('RENOVACION.RECHAZO_TACITO', seg)
    # Los que NUNCA confirmaron recibido no se tocan aquí — quedan en
    # 'esperando_confirmacion_recibido' para seguimiento manual de Coordinadora.

def evaluar_semaforo_comercial():
    for seguimiento in SeguimientoComercial.abiertos():
        config = EtapaComercialConfig.para(seguimiento.entidad_tipo, seguimiento.etapa)
        dias_vencido = dias_habiles_desde(seguimiento.fecha_proxima_accion)

        # Caso especial: renovación esperando confirmación de recibido,
        # sin fecha de vencimiento propia — se evalúa por antigüedad del envío.
        if seguimiento.entidad_tipo == 'renovacion' and seguimiento.etapa == 'esperando_confirmacion_recibido':
            dias_vencido = dias_naturales_desde(seguimiento.fecha_ultima_actividad)

        nuevo_color = 'VERDE'
        if dias_vencido > config.dias_habiles_rojo:
            nuevo_color = 'ROJO'
        elif dias_vencido > config.dias_habiles_amarillo:
            nuevo_color = 'AMARILLO'

        if nuevo_color != seguimiento.color_semaforo:
            seguimiento.color_semaforo = nuevo_color
            emitir_evento(f'SEGUIMIENTO_COMERCIAL.{nuevo_color}', seguimiento)
            if nuevo_color == 'ROJO':
                emitir_evento('SEGUIMIENTO_COMERCIAL.ESCALADO', seguimiento)
```

---

## 8. Vistas de Arquitectura (C4)

### 8.1 Nivel 1: Contexto del Sistema

```
┌──────────────────────────────────────────────────────────┐
│         Escuela / Cliente Renta / Cliente Venta            │
│   (Usuario principal, Contabilidad, Representante Legal,   │
│                  Persona Autorizada)                        │
└─────────────────────┬────────────────────────────────────┘
                      │ HTTPS + canal de mensajería (Firestore)
                      ▼
┌──────────────────────────────────────────────────────────┐
│                       Prospecto                            │
│        (mismo canal de mensajería, antes de convertirse)   │
└─────────────────────┬────────────────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────────────┐
│                    Colaborador Interno                    │
│  (Técnico, Coordinadora, Supervisor, Almacén, Admin,      │
│                    Gerente, Ventas)                        │
└─────────────────────┬────────────────────────────────────┘
                      │ HTTPS
                      ▼
┌─────────────────────────────┐
│      Sistema MFP Rental     │
│      (Plataforma Web)       │
└──┬──────┬──────┬──────┬─────┘
   ▼      ▼      ▼      ▼
┌──────┐┌──────┐┌──────┐┌──────┐
│Factu-││Conekta││Fire- ││ AWS │
│rama  ││Pagos ││base ││ SES │
│PAC   ││SPEI  ││Auth/ ││Email│
│      ││      ││FCM/  ││     │
│      ││      ││Firest││     │
└──────┘└──────┘└──────┘└──────┘
```

### 8.2 Nivel 2: Contenedores

Sin cambios estructurales respecto a lo ya definido (API Backend FastAPI en Elastic Beanstalk, PostgreSQL + pgvector, Firebase Auth/FCM/Firestore, S3 + CloudFront, SES). El único ajuste conceptual: Firestore deja de describirse como "chat de tickets" y pasa a describirse como **canal de mensajería unificado**, con las cuatro funciones ya listadas en 6.4.

---

## 9. Estructura del Repositorio

```
mfp-rental/
├── .github/workflows/ci.yml
├── app/
│   ├── main.py
│   ├── core/
│   │   ├── config.py
│   │   ├── errors.py
│   │   ├── logging.py
│   │   ├── module_registry.py
│   │   ├── event_bus.py
│   │   └── outbox.py
│   ├── modules/
│   │   ├── auth_identidad/
│   │   ├── usuarios_roles/
│   │   ├── catalogos/
│   │   ├── grupos_clientes/
│   │   ├── contratos/
│   │   ├── prospectos/
│   │   ├── comercial_seguimiento/
│   │   ├── equipos/
│   │   ├── lecturas/
│   │   ├── continuidad_contador/
│   │   ├── ordenes_servicio/
│   │   ├── fallas_catalogo/
│   │   ├── soluciones_campo/
│   │   ├── biblioteca/
│   │   ├── facturacion/
│   │   ├── pagos/
│   │   ├── semaforo/
│   │   ├── renovaciones/
│   │   ├── inventario/
│   │   ├── mensajeria/
│   │   ├── notificaciones/
│   │   ├── reportes/
│   │   ├── archivos/
│   │   ├── auditoria/
│   │   └── jobs/
│   ├── db/
│   │   ├── engine.py
│   │   └── session.py
│   └── workers/
│       ├── outbox_worker.py
│       └── scheduled_jobs.py
├── tests/
├── docs/arquitectura/documento_maestro.md
├── docker/Dockerfile
├── requirements.txt
├── .gitignore
└── README.md
```

**Reglas de carpeta por módulo:** `router.py`, `service.py`, `repository.py`, `schemas.py`, `events.py`, `handlers.py`, `ports.py`, `adapters/` — sin cambios respecto a lo ya definido.

**Reglas de dependencia:** `core/` no depende de ningún módulo. Ningún módulo importa modelos o repositorios de otro módulo. Comunicación exclusivamente por eventos o puertos definidos.

---

## 10. Convenciones de Desarrollo

Sin cambios respecto a lo ya definido: límites de tamaño (funciones ≤ 30 líneas, servicios ≤ 200 líneas, routers/schemas ≤ 150 líneas), nombrado (snake_case, PascalCase, endpoints en plural), idempotencia obligatoria (`Idempotency-Key` en mutaciones), manejo de errores centralizado con excepciones de dominio.

---

## 11. Integraciones Externas

| Servicio | Propósito | Módulo adaptador |
|---|---|---|
| Facturama API | Timbrado de CFDI y complementos de pago | `facturacion/adapters/facturama_client.py` |
| Conekta API | Recepción de webhooks de pago SPEI | `pagos/adapters/conekta_client.py` |
| Firebase Auth | Validación de JWT y emisión de tokens | `auth_identidad/adapters/firebase_auth.py` |
| Firebase FCM | Notificaciones push | `notificaciones/adapters/fcm_client.py` |
| Firestore | Canal de mensajería unificado (órdenes de servicio, lecturas, renovaciones, prospectos) | `mensajeria/adapters/firestore_token_provider.py` |
| AWS S3 | Almacenamiento de archivos con URL firmada | `archivos/adapters/s3_client.py` |
| AWS SES | Envío de correos transaccionales | `notificaciones/adapters/ses_client.py` |

---

## 12. Glosario

| Término | Definición |
|---|---|
| **MFP** | Multifunctional Product. Equipo multifuncional de impresión, copia y escaneo. |
| **Meterclick** | Diferencia entre el contador actual y el anterior de un MFP; base para excedentes (Renta), consumo de garantía (Venta) o control de vigencia (Escuela prepago). |
| **PPD** | Pago en Parcialidades o Diferido. Método fiscal para Renta. |
| **PUE** | Pago en Una sola Exhibición. Método fiscal para Venta y Escuela prepago. |
| **CFDI** | Comprobante Fiscal Digital por Internet. |
| **PAC** | Proveedor Autorizado de Certificación. |
| **SPEI** | Sistema de Pagos Electrónicos Interbancarios. |
| **Outbox** | Patrón que garantiza publicación confiable de eventos en la misma transacción que los datos de negocio. |
| **FSM** | Máquina de Estados Finitos. |
| **Circuit Breaker** | Previene llamadas en cascada a un servicio externo fallando. |
| **Idempotency Key** | Identificador único que garantiza que una operación se ejecute una sola vez aunque se reintente. |
| **Aceptación tácita** | Confirmación automática de una lectura si el cliente no responde en 5 días hábiles (Renta y Venta). |
| **Persona autorizada** | Individuo registrado por nombre, puesto y celular, facultado para autorizar acceso, confirmar contadores, recibir materiales y firmar órdenes — exigido antes de cada visita. |
| **Bolsa no acumulativa** | Límite mensual de impresiones en garantía cuyo saldo no utilizado no se transfiere al mes siguiente (Venta). |
| **Volumen anual pactado** | Techo de impresiones cubierto por el pago único anual en Escuela prepago; no se reinicia mensualmente. |
| **Saldo bonificable** | Impresiones contratadas y no utilizadas que se convierten en crédito para la renovación. |
| **Confirmación de recibido** | Evento independiente de aceptar o rechazar una cotización de renovación; condición necesaria para que después pueda operar el rechazo tácito. |
| **Rechazo tácito** | Falta de decisión expresa dentro de la ventana de aceptación — solo aplica si hubo confirmación de recibido previa. |
| **Continuidad de contador** | Regla que exige justificar y documentar cualquier contador menor al anterior, válido solo por cambio de equipo o de componente lógico. |
| **FallaCatalogo** | Registro estructurado de una falla y su solución según el manual del fabricante ("laboratorio controlado"); candidato a biblioteca pública. |
| **SolucionCampo** | Solución real aplicada por un técnico en campo, distinta de la del fabricante; material de capacitación interno, nunca público — la "caja negra" de expertise de la compañía. |
| **Semáforo financiero** | FSM que evalúa el estado de pago por RFC individual en Renta y Venta post-garantía. No aplica a Escuela prepago. |
| **Excepción operativa en ROJO** | Autorización conjunta de Coordinadora y Supervisor para operar pese a bloqueo por morosidad; el técnico nunca puede autorizarla. |
| **Semáforo comercial** | FSM independiente del financiero, aplicado a `SeguimientoComercial`; mide riesgo de omisión en el seguimiento de prospectos y renovaciones, no capacidad de pago. |
| **SeguimientoComercial** | Registro de dueño, etapa y próxima acción programada para un prospecto o una renovación; su ausencia de actividad dispara alertas y escalamiento automático. |
| **CPI (Costo por Impresión)** | Costo real que incluye amortización del equipo, tóner, refacciones y servicio técnico — distinto del rendimiento de brochure del fabricante. Su desglose real nunca es público; solo el resultado final, y solo tras calificar en el embudo. |
| **Embudo escalonado de cotización** | Los tres niveles de `SolicitudCotizacion` (estimado rápido, cotización detallada, asesor técnico) que intercambian precisión del cálculo por profundidad de datos entregados por el prospecto. |

---

## 13. Preguntas abiertas registradas

| # | Pregunta | Estado |
|---|---|---|
| P1 | ¿Debe existir una alerta de antigüedad para `SeguimientoRenovacion` en estado `esperando_confirmacion_recibido` que nunca recibe confirmación, para que Coordinadora no lo pierda de vista? | **Resuelto** — cubierto por el Motor COMERCIAL (sección 7.2): escala a rojo por antigüedad sin tocar la decisión de renovación en sí, que sigue sin poder vencer automáticamente sin confirmación previa. |
| P2 | Hallazgos H1–H14 de la auditoría del Plan de Implementación (trigger de inmutabilidad, `RN-FISC-001` no definida, sincronización de SLA con clasificación de cliente, etc.) | No resueltos todavía en este documento — pertenecen a una fase de implementación posterior y quedan fuera de este consolidado hasta que se decidan explícitamente. |
| P3 | Umbrales exactos de `EtapaComercialConfig` (cuántos días hábiles sin actividad son amarillo/rojo por cada etapa de prospecto y de renovación) | Pendiente de definición comercial — el modelo de datos ya lo soporta como configuración, no como valor fijo en código. |

---

**Fin del Documento Maestro de Arquitectura.**
