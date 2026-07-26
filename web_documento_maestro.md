# DOCUMENTO MAESTRO DE IMPLEMENTACIÓN

## Proyecto: Sistemas Documentales MX

### Web pública, estimadores, embudos comerciales y plataforma operativa por roles

**Dominio base:** sistemasdocumentales.mx
**Entorno de implementación:** WordPress / HostGator
**Directorio raíz de archivos web:** `public_html`

---

# 1. PROPÓSITO DE ESTE DOCUMENTO

Este documento es la fuente de continuidad del proyecto.

Su objetivo es permitir que cada fase de construcción pueda desarrollarse en un hilo separado sin perder:

* el objetivo general del proyecto;
* las decisiones arquitectónicas;
* los archivos ya construidos;
* las mitigaciones pendientes;
* la separación entre vistas públicas e internas;
* la relación entre los archivos HTML, JavaScript y la futura plataforma;
* las decisiones que ya fueron tomadas;
* las decisiones que todavía no deben suponerse.

Cada hilo nuevo de implementación debe comenzar leyendo este documento y siguiendo la fase correspondiente.

**Regla principal:**

> No se debe volver a diseñar el proyecto desde cero en cada hilo.
> Cada fase debe continuar el estado real de la fase anterior.

---

# 2. OBJETIVO INMEDIATO DEL PROYECTO

El objetivo inmediato es construir la web funcional del negocio:

**Sistemas Documentales MX**

La web debe comenzar con una experiencia pública orientada a:

1. Presentar el negocio.
2. Explicar sus servicios.
3. Ayudar al visitante a identificar su necesidad.
4. Utilizar estimadores como primer mecanismo de calificación.
5. Generar oportunidades comerciales.
6. Permitir posteriormente el seguimiento interno por parte de ventas y asesoría técnica.

Los embudos comerciales son solamente el inicio de la implementación.

El proyecto completo debe evolucionar hacia una plataforma con distintas vistas y niveles de acceso.

---

# 3. VISTAS Y ROLES DEL SISTEMA

El sistema debe contemplar las siguientes vistas:

## 3.1 Vista pública general

Para visitantes sin autenticación.

Debe permitir:

* conocer la empresa;
* consultar servicios;
* explorar soluciones;
* utilizar estimadores públicos;
* iniciar un proceso de contacto o calificación;
* solicitar atención.

No debe mostrar información interna ni costos sensibles.

---

## 3.2 Vista de clientes

Para clientes autenticados.

Debe permitir evolucionar hacia funciones como:

* consulta de información propia;
* solicitudes;
* seguimiento de servicios;
* información relacionada con equipos;
* documentos y procesos correspondientes al cliente.

El alcance exacto de esta vista se definirá en su fase.

---

## 3.3 Vista de colaboradores

Para personal operativo.

Debe permitir funciones internas de acuerdo con permisos.

---

## 3.4 Vista de coordinadora

Vista de coordinación operativa y/o comercial.

---

## 3.5 Vista de supervisor

Vista de supervisión, seguimiento y control.

---

## 3.6 Vista de administradores

Vista con permisos administrativos amplios.

---

## 3.7 Vista de administrador único

Nivel máximo de administración del sistema.

Debe conservarse como una capa separada de los administradores normales.

---

# 4. ARQUITECTURA DE IMPLEMENTACIÓN

La implementación debe realizarse por capas.

## CAPA 1 — Web pública

Archivos HTML, CSS y JavaScript públicos.

Ubicación inicial:

```text
public_html/
```

---

## CAPA 2 — Estimadores públicos

Herramientas que permiten al visitante proporcionar información inicial y recibir un estimado controlado.

Los estimadores no deben revelar información que el modelo de negocio haya decidido mantener interna.

---

## CAPA 3 — Catálogo único

Debe existir una única fuente de verdad para los modelos y sus parámetros.

No deben existir dos catálogos hardcodeados diferentes para el mismo modelo.

La información del catálogo debe ser reutilizable por:

* estimador standalone;
* sitio completo;
* futuros módulos;
* área interna;
* futuras APIs o backend.

---

## CAPA 4 — Flujo comercial

Incluye:

* captura de prospectos;
* calificación;
* temperatura;
* seguimiento;
* próxima acción;
* asesoría;
* visita;
* diagnóstico;
* propuesta formal.

---

## CAPA 5 — Área interna de ventas / asesor

No es una vista pública.

Debe estar destinada a:

* asesores;
* ventas;
* usuarios autorizados.

Debe implementar las reglas de negocio del documento maestro.

---

## CAPA 6 — Área de clientes

Debe mantenerse separada del área interna de ventas.

---

## CAPA 7 — Plataforma de roles operativos

Posteriormente se implementarán:

* colaboradores;
* coordinadora;
* supervisor;
* administradores;
* administrador único.

---

# 5. ESTRUCTURA BASE DEL PROYECTO

La estructura inicial recomendada es:

```text
public_html/
│
├── index.html
│
├── nosotros.html
├── servicios.html
├── soluciones.html
├── contacto.html
│
├── estimador.html
│
├── css/
│   ├── styles.css
│   ├── public.css
│   └── responsive.css
│
├── js/
│   ├── catalogo.js
│   ├── estimador.js
│   ├── validaciones.js
│   ├── ubicacion.js
│   └── ui.js
│
├── assets/
│   ├── img/
│   ├── icons/
│   └── logos/
│
├── app/
│   ├── ventas/
│   ├── clientes/
│   ├── colaboradores/
│   ├── coordinacion/
│   ├── supervision/
│   ├── administracion/
│   └── superadmin/
│
└── docs/
    ├── README.md
    ├── estado-proyecto.md
    └── cambios.md
```

Esta estructura puede evolucionar.

No debe modificarse arbitrariamente durante una fase sin documentar el cambio.

---

# 6. ARCHIVOS YA EXISTENTES

Al inicio del proyecto existen tres archivos HTML desarrollados.

Los documentos de referencia de la auditoría fueron identificados como:

* Documento 7: calculador/estimador standalone.
* Documento 8: sitio completo.
* Documento 9: dashboard o vista interna del asesor.

Estos archivos deben considerarse como prototipos existentes.

No deben descartarse sin revisar.

La implementación debe:

1. conservar lo que ya funciona;
2. corregir inconsistencias;
3. unificar reglas;
4. evitar duplicación;
5. evolucionar gradualmente hacia una arquitectura mantenible.

---

# 7. DOCUMENTO MAESTRO COMO FUENTE DE REGLAS

El documento maestro del negocio es la fuente principal para:

* reglas comerciales;
* reglas de cálculo;
* niveles de acceso a costos;
* definición de campos;
* contratos;
* obligaciones;
* calificación;
* temperatura;
* visitas;
* diagnósticos;
* propuesta formal;
* datos requeridos.

Cuando exista conflicto entre:

* un archivo HTML;
* una decisión improvisada;
* una implementación anterior;

y el documento maestro, debe prevalecer el documento maestro, salvo que el usuario modifique explícitamente la decisión.

---

# 8. REGLAS COMERCIALES CONFIRMADAS

## 8.1 Paso 1

Los seis campos del Paso 1 coinciden en los tres documentos revisados.

La estructura existente debe conservarse salvo que el documento maestro indique lo contrario.

---

## 8.2 Rango en Paso 1

El CPI se muestra como rango en el Paso 1.

No debe mostrarse un desglose de costos de refacciones.

---

## 8.3 CPI puntual

El CPI puntual solamente debe aparecer después de que el proceso haya avanzado suficientemente.

La propuesta formal no debe generarse automáticamente únicamente con los datos iniciales de una llamada.

---

## 8.4 Visita de campo

La vista del asesor debe permitir avanzar hacia la visita de campo y el diagnóstico.

El modal de cierre actualmente implementa correctamente la idea de diferir la propuesta formal hasta después de la visita de campo.

Esta lógica debe conservarse.

---

# 9. MITIGACIONES CRÍTICAS OBLIGATORIAS

Estas mitigaciones deben implementarse antes de considerar estable la primera versión del flujo comercial.

---

## MITIGACIÓN 1 — Catálogo único

### Problema

Existen dos catálogos de modelos distintos con los mismos nombres.

Ejemplo identificado:

`IM 430`

Un catálogo usa:

```text
CPI: $0.25–$0.35
```

Otro usa:

```text
CPI: $0.27–$0.33
```

Esto puede producir resultados diferentes para el mismo modelo.

### Decisión

Debe existir una única fuente de verdad.

Archivo inicial:

```text
js/catalogo.js
```

El estimador standalone y el sitio completo deben consultar el mismo catálogo.

### Regla

No duplicar el catálogo en:

* `index.html`;
* `estimador.html`;
* scripts separados;
* dashboard.

---

## MITIGACIÓN 2 — Volumen fuera de catálogo

### Problema

El estimador standalone continúa mostrando un número cuando el volumen excede el límite de catálogo.

Esto contradice la regla de negocio.

### Regla correcta

Si el volumen excede el 2X de todos los modelos disponibles:

```text
requiere_evaluacion_personalizada = true
```

Y:

```text
modelo = null
```

No debe sugerirse automáticamente el modelo más grande.

No debe calcularse automáticamente un CPI para ese caso.

Debe mostrarse un flujo de evaluación personalizada y canalización a asesor.

---

## MITIGACIÓN 3 — Próxima acción según temperatura

### Problema

El dashboard utiliza:

```text
dias = 3
```

para todos los leads.

### Regla correcta

Debe utilizarse la configuración definida por temperatura:

```text
Caliente:
menos de 48 horas

Tibio:
5 días

Frío:
15 días hábiles
```

La lógica debe centralizarse.

No debe repetirse manualmente en varios archivos.

---

## MITIGACIÓN 4 — Persona autorizada

### Problema

La implementación actual utiliza únicamente un checkbox:

```text
Persona autorizada registrada
```

Esto es insuficiente.

### Deben capturarse:

```text
nombre
puesto
celular
si coincide con registro previo
```

La captura debe corresponder a:

```text
RegistroAccesoVisita
```

Esta información es contractual y no debe reducirse a un checkbox.

---

## MITIGACIÓN 5 — Temperatura del lead

### Problema

Actualmente el asesor puede seleccionar directamente:

```text
🔥
🌤️
❄️
```

sin que el sistema calcule una recomendación previa.

### Regla correcta

La temperatura debe:

1. calcularse mediante reglas;
2. mostrar una sugerencia;
3. permitir sobreescritura manual;
4. exigir motivo si se modifica.

El campo:

```text
motivo_ajuste_manual
```

es obligatorio cuando el asesor modifica la temperatura sugerida.

---

# 10. MITIGACIONES SECUNDARIAS

No bloquean la construcción inicial, pero deben mantenerse registradas.

## 10.1 Error de validación B/N + Color

Existe una validación que falla sin mostrar correctamente el mensaje de advertencia.

Debe implementarse un mensaje visible y comprensible.

---

## 10.2 Detección de ubicación

El sitio completo ya tiene detección de ubicación.

El calculador standalone no.

Debe unificarse el comportamiento.

---

## 10.3 PII en console.log

Antes de producción debe eliminarse información personal de:

```text
console.log
```

---

## 10.4 Aviso de privacidad

El enlace actual apunta a una página que todavía no existe.

Debe crearse antes del lanzamiento real a producción.

---

## 10.5 Equipos diagnosticados

Actualmente:

```text
vEquiposDetalle
```

es texto libre.

La versión futura debe utilizar entradas estructuradas repetibles para mapear correctamente a:

```text
equipos_diagnosticados
```

---

# 11. ORDEN OFICIAL DE IMPLEMENTACIÓN

## FASE 0 — Control del proyecto

Crear y mantener:

```text
docs/README.md
docs/estado-proyecto.md
docs/cambios.md
```

Objetivo:

* registrar decisiones;
* registrar archivos;
* registrar mitigaciones;
* evitar perder continuidad.

---

# FASE 1 — Web pública básica

Construir:

```text
index.html
```

Y los archivos necesarios para la primera versión pública.

Objetivo:

* tener una página web real;
* presentar la empresa;
* presentar soluciones;
* conducir al usuario hacia el estimador;
* establecer la identidad visual inicial.

No debe incluir todavía todo el sistema de roles.

---

# FASE 2 — Estimador público

Construir o actualizar:

```text
estimador.html
js/estimador.js
js/validaciones.js
js/ubicacion.js
```

Objetivo:

* mantener los seis campos del Paso 1;
* mostrar CPI como rango;
* no mostrar desglose de costos;
* respetar el límite de catálogo;
* enrutar volumen fuera de catálogo a evaluación personalizada.

---

# FASE 3 — Catálogo único

Crear:

```text
js/catalogo.js
```

Debe convertirse en la fuente única de modelos.

El estimador standalone y el sitio completo deben consumirlo.

Esta fase debe resolver definitivamente la inconsistencia del `IM 430` y cualquier otro modelo duplicado.

---

# FASE 4 — Mitigaciones del flujo comercial

Implementar:

* próxima acción por temperatura;
* registro completo de persona autorizada;
* cálculo sugerido de temperatura;
* motivo obligatorio de ajuste manual;
* validación visible B/N + Color;
* detección de ubicación consistente;
* eliminación progresiva de PII de logs.

---

# FASE 5 — Área interna de Ventas / Asesor

Construir la vista interna para:

* asesores;
* ventas.

Debe contemplar:

* leads;
* calificación;
* temperatura;
* seguimiento;
* próxima acción;
* visita;
* persona autorizada;
* diagnóstico;
* cierre de visita.

La propuesta formal debe continuar bloqueada hasta que se cumplan las condiciones correspondientes del proceso.

---

# FASE 6 — Área de clientes

Construir la experiencia autenticada para clientes.

Debe mantenerse separada de la vista de ventas.

El alcance se definirá a partir de los procesos reales del negocio y del documento maestro.

---

# FASE 7 — Colaboradores, coordinación y supervisión

Implementar:

```text
colaboradores
coordinadora
supervisor
```

Con permisos separados.

---

# FASE 8 — Administración

Implementar:

```text
administradores
administrador único
```

El administrador único debe mantener permisos superiores.

---

# 12. PROTOCOLO PARA CADA NUEVO HILO

Cada hilo de implementación debe seguir esta secuencia.

## Paso 1

Indicar:

```text
Este hilo corresponde a la FASE X.
```

## Paso 2

Confirmar los archivos que se modificarán.

## Paso 3

Confirmar las reglas del documento maestro aplicables.

## Paso 4

Revisar los archivos actuales antes de reemplazarlos.

## Paso 5

Entregar archivos completos cuando se solicite implementación.

No entregar únicamente fragmentos si el objetivo es reemplazar un archivo completo.

## Paso 6

Indicar exactamente:

```text
CREAR:
ruta/archivo.ext

REEMPLAZAR:
ruta/archivo.ext

CONSERVAR:
ruta/archivo.ext
```

## Paso 7

Indicar pruebas.

## Paso 8

Actualizar el estado del proyecto.

---

# 13. REGLAS DE CONTINUIDAD PARA LA IA

En cada nuevo hilo:

1. No asumir que un archivo existe si no fue confirmado.
2. No asumir que una mitigación ya fue implementada.
3. No crear una segunda versión de un catálogo sin revisar la primera.
4. No cambiar reglas del documento maestro silenciosamente.
5. No eliminar archivos existentes sin justificarlo.
6. No inventar campos contractuales.
7. No inventar datos de modelos.
8. No sustituir una decisión de negocio por una preferencia técnica.
9. No declarar una fase terminada si no se entregaron los archivos correspondientes.
10. Si falta información crítica, preguntar antes de construir.

---

# 14. FORMATO DE ESTADO DEL PROYECTO

Debe mantenerse una tabla como esta:

| Fase   | Estado                | Archivos principales | Observaciones          |
| ------ | --------------------- | -------------------- | ---------------------- |
| Fase 0 | En progreso           | docs/*               | Control de continuidad |
| Fase 1 | Pendiente/En progreso | index.html           | Web pública            |
| Fase 2 | Pendiente             | estimador.html       | Estimador              |
| Fase 3 | Pendiente             | js/catalogo.js       | Catálogo único         |
| Fase 4 | Pendiente             | varios               | Mitigaciones           |
| Fase 5 | Pendiente             | app/ventas/*         | Área interna           |
| Fase 6 | Pendiente             | app/clientes/*       | Clientes               |
| Fase 7 | Pendiente             | app/*                | Roles operativos       |
| Fase 8 | Pendiente             | app/*                | Administración         |

El estado debe actualizarse solamente con base en archivos realmente entregados e implementados.

---

# 15. PLANTILLA MAESTRA PARA ABRIR CADA HILO

El siguiente texto debe copiarse al iniciar un hilo nuevo:

---

## CONTEXTO DE IMPLEMENTACIÓN — SISTEMAS DOCUMENTALES MX

Estoy continuando el proyecto de construcción de la web de:

**Sistemas Documentales MX**

Dominio:

```text
https://sistemasdocumentales.mx/
```

La implementación se realiza inicialmente en:

```text
HostGator
WordPress
public_html
```

Este hilo corresponde a:

```text
FASE: [INDICAR FASE]
```

El objetivo general es construir toda la web del negocio.

Los embudos y estimadores son solamente el inicio.

El sistema tendrá estas vistas:

1. Público general.
2. Clientes.
3. Colaboradores.
4. Coordinadora.
5. Supervisor.
6. Administradores.
7. Administrador único.

El documento maestro del negocio es la fuente de verdad para las reglas comerciales, contractuales y de cálculo.

No debes suponer decisiones que no estén confirmadas.

Antes de reemplazar archivos, debes considerar los archivos ya existentes.

Cuando la tarea sea implementar código, debes entregar:

1. El contenido completo de cada archivo nuevo.
2. El contenido completo de cada archivo reemplazado.
3. La ruta exacta de cada archivo.
4. Qué archivo crear.
5. Qué archivo reemplazar.
6. Qué archivo conservar.
7. Las pruebas que debo realizar en HostGator.
8. Las dependencias entre archivos.
9. Los cambios realizados respecto a la versión anterior.

No debes limitarte a decir que vas a entregar un archivo posteriormente.

Si afirmas que una fase está implementada, debes haber entregado los archivos correspondientes.

---

## REGLAS YA CONFIRMADAS

* Los seis campos del Paso 1 están alineados en los documentos existentes.
* El CPI en Paso 1 debe mostrarse como rango.
* No debe mostrarse desglose de costos de refacciones.
* El CPI puntual debe aparecer solamente después de la etapa correspondiente del proceso.
* La propuesta formal no debe generarse solamente con la llamada inicial.
* La visita de campo y el diagnóstico son etapas relevantes del proceso.

---

## MITIGACIONES OBLIGATORIAS

1. Un catálogo único para todos los estimadores.
2. Volumen fuera de catálogo:

   * `modelo = null`
   * `requiere_evaluacion_personalizada = true`
   * no mostrar CPI automático.
3. Próxima acción según temperatura:

   * caliente: menos de 48 horas;
   * tibio: 5 días;
   * frío: 15 días hábiles.
4. Persona autorizada:

   * nombre;
   * puesto;
   * celular;
   * coincidencia con registro previo.
5. Temperatura:

   * cálculo sugerido;
   * posibilidad de modificación manual;
   * motivo obligatorio para modificarla.

---

## MITIGACIONES SECUNDARIAS PENDIENTES

* mensaje visible para validación B/N + Color;
* detección de ubicación consistente;
* eliminar PII de `console.log`;
* crear aviso de privacidad;
* sustituir texto libre de equipos diagnosticados por estructura repetible.

---

## ESTADO DE LA FASE ANTERIOR

Antes de iniciar, revisar y actualizar:

```text
docs/estado-proyecto.md
```

Si el archivo no existe, crear el registro correspondiente.

---

## REGLA DE ENTREGA

La respuesta debe centrarse en la implementación de esta fase.

No anunciar una entrega futura sin incluir en la respuesta los archivos solicitados.

---

# 16. PRÓXIMO ORDEN DE TRABAJO

La continuidad recomendada es:

## HILO 1

Construcción de:

```text
index.html
```

Primera versión funcional de la web pública.

---

## HILO 2

Construcción de:

```text
js/catalogo.js
```

Con base en los datos reales del documento maestro y los catálogos existentes.

---

## HILO 3

Actualización completa del estimador:

```text
estimador.html
js/estimador.js
```

Usando el catálogo único.

---

## HILO 4

Implementación de las cinco mitigaciones críticas.

---

## HILO 5

Área interna de Ventas / Asesor.

---

## HILO 6

Área de clientes.

---

## HILO 7

Colaboradores, coordinadora y supervisor.

---

## HILO 8

Administradores y administrador único.

---

# 17. PRINCIPIO FINAL DEL PROYECTO

El objetivo no es construir una colección de páginas HTML aisladas.

El objetivo es construir progresivamente:

```text
Web pública
        ↓
Estimadores
        ↓
Calificación
        ↓
Prospectos
        ↓
Ventas / Asesor
        ↓
Visita
        ↓
Diagnóstico
        ↓
Propuesta
        ↓
Cliente
        ↓
Operación
        ↓
Supervisión
        ↓
Administración
```

Cada fase debe conectar con la siguiente.

La prioridad es mantener:

* coherencia;
* trazabilidad;
* una sola fuente de verdad;
* reglas comerciales consistentes;
* separación de permisos;
* continuidad técnica.

La implementación debe avanzar de forma incremental, pero sin perder la arquitectura completa del proyecto.
