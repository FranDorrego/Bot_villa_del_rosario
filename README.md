
# Bot Municipalidad – Villa del Rosario

## 🤖 Requisitos Funcionales

Este documento resume las funcionalidades requeridas para el desarrollo de un bot de WhatsApp destinado a ciudadanos de la municipalidad. Se agrupan los requerimientos por temática para facilitar su implementación por etapas.

## 📟 1. Información sobre Servicios Municipales

### ♻️ Recolección de residuos

- ¿Qué días y horarios pasa el recolector?
- ¿Qué tipos de residuos se pueden sacar a la calle y cuándo? (escombros, ramas, domiciliarios)

### 💧 Servicios básicos

- ¿En el barrio [X] hay agua potable, cloacas, gas?
- ¿Qué trámites se deben realizar para un desarrollo inmobiliario y venta de lotes?
- ¿Cuándo se va a terminar la obra de la calle [X]?

## 📍 2. Trámites y gestiones

### 🏢 Habilitaciones comerciales

- ¿Cómo habilitar un comercio en la ciudad?
- ¿Qué se necesita para habilitar un comercio de alimentos? (requisitos de bromatología)

### 🚗 Carnet de conducir

- ¿Requisitos para obtener el carnet?
  - Para: moto, auto, camión, maquinaria agrícola
- ¿Horarios de atención?

### 🏡 Impuestos

- ¿Cuánto debo del impuesto inmobiliario?
- ¿Cuánto debo de patentes?

## 📞 3. Reclamos y emergencias

### 🔧 Infraestructura

- Reportar pérdida de agua
- Reportar pozo negro tapado
- Reportar pérdida u olor a gas

## 🏫 4. Salud municipal

- Horarios de atención para:
  - Odontólogo
  - Psicólogo
  - Pediatra
  - Psiquiatra
- ¿Se puede realizar una tomografía computada en la municipalidad?

## 📋 5. Educación y formación

### Escuelas y formación superior

- Escuelas primarias y secundarias
- Instituto terciario
- Universidad o carreras disponibles

### Capacitación laboral

- Cursos brindados por la municipalidad
- Cursos del SEGEM (Centro de Gestión Municipal)

## 🎨 6. Cultura y turismo

### Eventos y fiestas

- Eventos culturales este o próximo fin de semana
- Fiesta del Folklore en el Agua
  - Qué es
  - Cuántos años hace que se realiza
  - Qué artistas han participado (ej. enero 25)

### Lugares y actividades

- Lugares para visitar en la ciudad
- Horarios del museo
- Muestras actuales en el museo

### Historia y personajes

- Fundador de la ciudad (tropas del general)
- Músicos, artistas o deportistas reconocidos del pueblo

---

> ✅ Este listado sirve como base para el desarrollo conversacional del bot, permitiendo responder a los ciudadanos con información ágil y precisa.

---

# 🤖 ¿Como se puede llevar a cabo?


## (Opcion 1) Flujo Completo – Bot de WhatsApp Municipal en Python

Este documento describe paso a paso la arquitectura y lógica de un bot municipal por WhatsApp, desarrollado 100% en Python, con FastAPI y Twilio.



## 🔄 Flujo General del Bot

### 1. 💌 Usuario envía mensaje por WhatsApp

Twilio recibe el mensaje y lo reenvía a tu servidor Python via webhook (ej: `POST /webhook`).

### 2. 📊 Backend (FastAPI) recibe el mensaje

- Identifica el número del usuario
- Consulta si ya tiene un **estado de conversación** activo

```python
usuarios_activos = {
    "+549351123456": {
        "intencion": "reclamo_gas",
        "estado": "esperando_direccion",
        "datos": {}
    }
}
```

### 3. 🧬 Si hay estado → continuar el flujo

Ejemplo:

- Usuario dijo "hay gas"
- Bot respondió: "📍 indicame la dirección"
- Usuario responde: "Belgrano 345"
- Bot guarda el dato y sigue: "🗰¿Querés agregar algún detalle extra?"

Cada paso se guarda con el estado actualizado.


### 4. 🧐 Si NO hay estado → detectar intención

- Se pasa el mensaje a un clasificador:
  - A: con palabras clave (rápido)
  - B: con GPT (para mayor flexibilidad)

Ejemplo:

```python
if "carnet" in mensaje and "auto" in mensaje:
    intencion = "tramite_carnet"
```


### 5. 💡 Se llama al handler correspondiente

Cada intención tiene su propio archivo o función:

```python
def handle_info_basura():
    return "🚿 La recolección de residuos se realiza los martes y viernes de 6 a 10hs."
```

Si el handler necesita recolectar más datos, cambia el estado del usuario para continuar.


### 6. 📃 Si la intención es de consulta informativa

- El bot busca respuestas en **documentos organizados** (ver más abajo)
- O usa templates predefinidos cargados desde base local
- Puede usar GPT para armar mejor la redacción (opcional)


### 7. 📝 Se responde al usuario por WhatsApp

- Se guarda el estado de conversación si el flujo no terminó
- Si se completó (ej: se hizo reclamo, se dio info), se borra el estado del usuario


## 📥 Organización de datos para respuestas informativas

Para mantener respuestas actualizadas sin tocar código, se recomienda usar un **Google Drive compartido** organizado así:

```
MunicipalidadBot/
├── salud/
│   └── horarios_medicos.pdf
├── residuos/
│   └── calendario_basura.docx
├── cultura/
│   └── eventos_actuales.pdf
├── cursos/
│   └── capacitaciones.xlsx
```

### ✅ Ventajas:

- Los empleados municipales pueden actualizar info directamente
- Desde Python podés usar `pydrive` o `google-api-python-client` para:
  - Leer archivos
  - Indexar su contenido
  - Buscar frases clave y responder

### Ejemplo:

> Usuario: "¿Cuándo es el próximo evento de cultura?"
> ✉️ El bot busca en `/cultura/eventos_actuales.pdf`, extrae el texto y responde con el próximo evento listado.



## 🤝 Tecnologías sugeridas

| Propósito                   | Herramienta                |
| --------------------------- | -------------------------- |
| Bot WhatsApp                | Twilio API                 |
| Backend                     | Python (FastAPI)           |
| Clasificación IA (opcional) | GPT (via API)              |
| Persistencia                | SQLite, Redis o JSON local |
| Archivos de información     | Google Drive compartido    |



Este flujo está pensado para mantener control total del bot, facilitar su mantenimiento, y permitir escalarlo con nuevos temas sin romper lo existente.

---

# 🤖 (Opcion 2) Implementación del Bot Municipal en n8n

Este documento explica cómo replicar el flujo del bot de WhatsApp para una municipalidad usando **n8n**, adaptado a sus capacidades.


## 🧠 Consideraciones Generales

n8n es ideal para flujos simples y tareas automatizadas, pero tiene limitaciones cuando se trata de:
- Mantener **estado conversacional**
- Procesar respuestas abiertas
- Lógica condicional compleja


## 🏗️ Estructura del Flujo en n8n

### 🔁 1. Webhook (entrada desde WhatsApp)
- Se usa un nodo `Webhook` que recibe los mensajes.
- Ejemplo de configuración:
  - Method: POST
  - Path: `/bot-municipal`

### 🧾 2. Extraer datos del mensaje
- Nodo `Set` para tomar:
  - `phone_number`
  - `message`

### 📊 3. Determinar si hay contexto previo
- Nodo `HTTP Request` a Google Sheets o Supabase para consultar si ese número ya está en proceso.
- Si hay contexto:
  - Redirigir al subflujo correspondiente (ej: esperando dirección de reclamo)
- Si no hay:
  - Seguir al paso de clasificación de intención



### 🧠 4. Clasificación de intención
- Nodo `OpenAI` (si tenés API Key) o `IF` con reglas tipo:
```text
IF message contains "carnet" AND "auto"
→ ruta: tramite_carnet
```
- Otra opción: usar `Function` node con JavaScript para categorizar


### ⚙️ 5. Ejecutar subflujo por intención
- Cada tema (basura, salud, cursos, reclamos) se maneja en un subworkflow
- Estos flujos pueden:
  - Leer archivos (Google Drive, Sheets, Docs)
  - Enviar mensajes con texto fijo
  - Solicitar más datos al usuario



## 📃 Manejo de consultas informativas (Google Drive)

### Opciones:
- Usar el nodo `Google Drive` para buscar archivos por carpeta/tema
- Usar el nodo `Google Docs` o `Google Sheets` para leer información

### Ejemplo:
1. Usuario: "¿Cuándo se recolecta basura?"
2. Nodo: Google Drive → buscar archivo en carpeta `residuos`
3. Nodo: `Google Docs` → extraer texto
4. Nodo: `Respond with WhatsApp message`


## 🗂️ Estado del usuario

### Opciones:
- Guardar en Google Sheets (una fila por usuario con: teléfono, estado, datos parciales)
- Usar Firebase, Supabase o Airtable como backend

Ejemplo de estructura:
```
| Telefono      | Intencion      | Estado               | Datos                |
|---------------|----------------|----------------------|----------------------|
| +549351...    | reclamo_gas    | esperando_direccion  | {"barrio": "Sur"}   |
```



## 📤 Enviar respuesta a WhatsApp
- Nodo `HTTP Request` a Twilio API
- O si usás WhatsApp Cloud API, también HTTP POST con plantilla o texto



## ✅ Consejos para n8n

- Usá nodos `Switch` o `IF` para ramificar flujos
- Mantené los flujos simples: 1 intención por workflow
- Guardá estados externos, n8n no es bueno para flujos largos sin persistencia
- Nombrá los nodos y conectores para no perderte



## 🚫 Limites de n8n

| Limitación | Alternativa |
|------------|-------------|
| Manejo de contexto largo | Usar backend en Python y que n8n sólo dispare tareas |
| Procesamiento de lenguaje complejo | Usar GPT desde backend, no desde n8n directo |
| Múltiples flujos simultáneos | Implementar identificadores por usuario y estados |



Este enfoque te permite usar n8n para prototipos o casos simples, pero si querés robustez real, eventualmente migrá la lógica crítica a un backend en Python.


---

# 🤖 Comparación: Bot Municipal en n8n vs Python (FastAPI)

Comparamos las dos alternativas para implementar el bot de WhatsApp municipal: **n8n** (low-code) y **Python con FastAPI** (código completo).

---

## 🏋️ Comparativa General

| Característica                       | n8n                                | Python (FastAPI)                      |
|-------------------------------------|-------------------------------------|---------------------------------------|
| 💻 Complejidad inicial         | Baja                               | Media                                  |
| 🔋 Flexibilidad total          | Limitada                           | Total                                  |
| 📈 Escalabilidad               | Baja-media                         | Alta                                   |
| 📁 Manejo de contexto          | Muy limitado                       | Completo y persistente                 |
| 🚀 Velocidad de desarrollo    | Rápido para cosas simples          | Más lento pero profesional             |
| 🔧 Integración con APIs externas| Visual, pero engorrosa a gran escala| Limpia y ordenada con requests         |
| 🔄 Mantenimiento a largo plazo| Difícil si crece el flujo         | Escalable y modular                   |
| 🚫 Offline / Desconexión       | No funciona sin Internet           | Puede correr en servidores locales     |
| 🤝 Ideal para...               | Prototipos o flujos ultra simples  | Proyectos serios, con lógica compleja  |

---

## ✅ Ventajas de usar n8n

- Rápida puesta en marcha
- Ideal para no-programadores
- Buena integración visual con:
  - Google Sheets, Docs, Drive
  - Webhooks
  - APIs REST básicas
- Buena para flujos lineales tipo:
  - "Consulta de horarios"
  - "Respuestas estáticas desde archivo"

### ❌ Desventajas de n8n
- No puede manejar bien **flujos ramificados complejos**
- Pésimo manejo de **contexto entre mensajes**
- Complicado de mantener si el flujo crece
- Depende 100% de conexión y cuenta en n8n o Docker
- Depende de otros sistemas externos para guardar estados (ej: Google Sheets)

---

## ✅ Ventajas de usar Python (FastAPI)

- Control total sobre el flujo y estados
- Muy buen manejo de:
  - Múltiples intenciones
  - Usuarios paralelos
  - Persistencia en DB
- Escalable: podés agregar autenticación, reportes, analítica
- Fácil integración con GPT para mejorar respuestas

### ❌ Desventajas de Python
- Requiere experiencia técnica
- Lleva más tiempo levantar la estructura inicial
- Hay que desplegarlo y mantenerlo (hosting, logs, seguridad)

---

## 🚀 Recomendación

| Escenario | Recomendado |
|----------|--------------|
| Bot con 5-10 respuestas rápidas | n8n |
| Bot con manejo de flujos + contexto | Python |
| Escalabilidad a más servicios (impuestos, reclamos, cursos, salud) | Python |
| Necesitás que personal no técnico edite respuestas | n8n + Google Drive |
| Flujo dependiente de validaciones externas/API complejas | Python |

---

## 📄 Conclusión

> **n8n** es ideal para comenzar, probar y validar flujos simples. Pero **Python (FastAPI)** es la mejor opción si querés robustez, mantenimiento limpio, y control completo del bot.

Para este caso municipal, donde hay reclamos, trámites, estado y muchas intenciones diferentes, **la solución escalable, segura y confiable es Python**.

---

# 💼 Presupuesto y Plan de Trabajo – Consultoría Bot WhatsApp Municipal

Presento el plan de trabajo y estimación de tiempos y tareas para acompañar el desarrollo del bot de WhatsApp municipal, en tres escenarios:
- Acompañamiento técnico sobre infraestructura actual con n8n
- Desarrollo completo desde cero con backend en Python
- Trabajo colaborativo con backend API compartido y manejo de lógica desde IA


## 🧩 Parte 1 – Consultoría sobre plataforma n8n existente

**Duración estimada**: 2 a 3 semanas  
**Costo estimado**: EUR 500

### Objetivos
- Ayudar al equipo actual a:
  - Implementar manejo de contexto (por teléfono)
  - Guardar estados (en Google Sheets o Supabase)
  - Resolver errores actuales de flujos
  - Automatizar seguimiento de reclamos

### Tareas
| Tarea | Tiempo estimado | Descripción |
|-------|------------------|--------------|
| Diagnóstico del flujo actual | 1 día | Entender configuración actual y nudos de error |
| Asistencia en lógica de flujos | 2-3 días | Recomendar ajustes y simplificar estructura |
| Implementación de estados | 2-3 días | Diseño y prueba de lectura/escritura de estado por teléfono |
| Scripts adicionales (Python/JS en n8n) | 1-2 días | Automatizaciones y transformación de datos |

> 🧠 *Ideal para mejorar lo que ya tienen sin migrar de tecnología.*

---

## 🐍 Parte 2 – Desarrollo completo en Python (FastAPI)

**Duración estimada**: 4 a 5 semanas  
**Costo estimado**: EUR 1.500 

### Etapas y tiempos estimados

| Módulo | Tiempo | Descripción |
|--------|--------|--------------|
| Setup de servidor (VPS, entorno) | 1-2 días | Configuración de hosting, dominio, SSL, despliegue inicial |
| Recepción Webhook desde Twilio | 1-2 día | Endpoint funcional para recibir mensajes |
| Manejo de usuarios y sesiones | 3-4 días | Seguimiento de conversaciones y estados por usuario |
| Clasificación de mensajes (lógica/IA) | 3 días | Detectar intención y redirigir al handler adecuado |
| Implementación de flujos (basura, reclamos, salud, etc.) | 10-12 días | Crear flujos individuales y modularizados |
| Conexión a Google Drive (info actualizable) | 2-3 días | Lectura dinámica de archivos PDF, Sheets |
| Sistema de logging y backup | 1-2 días | Registro de interacciones para seguimiento |
| Pruebas + documentación | 3-4 días | Validación, test con usuarios, instructivo técnico |

> 🧱 *Permite una solución escalable, confiable y mantenible a largo plazo.*

---

## 🤝 Parte 3 – Trabajo en colaboración con desarrollador actual (API externa)

**Duración estimada**: 3 a 4 semanas  
**Costo estimado**: EUR 1000 

### Objetivos
- Dividir responsabilidades para acelerar el desarrollo:
  - Desarrollador A (otro programador): manejar integración con WhatsApp (recepción de textos y audios), preparar el entorno de produccion y gestion de APIs e informacion.
  - Desarrollador B (mi parte): manejar lógica, IA y generación de respuestas vía API

### Requerimientos para el otro programador
- Endpoint para recibir texto limpio del mensaje de usuario (`POST /mensaje`)
- Endpoint para recibir texto transcripto si es audio (`POST /audio-transcripto`)
- Lista centralizada con todos los endpoints necesarios
- Especificación de qué se espera de cada endpoint (campos, formato, respuestas)

### Responsabilidades propias
| Tarea | Tiempo estimado | Descripción |
|-------|------------------|-------------|
| Estructura del backend (FastAPI) | 3-4 días | Configurar API modular para recibir mensajes y responder |
| Motor de clasificación de intención | 2-3 días | Con o sin GPT, identifica qué desea el usuario |
| Flujos personalizados por intención | 6-7 días | Manejo de preguntas, respuestas, estado por usuario |
| API REST para respuestas | 2 días | Endpoint `GET /respuesta/{user_id}` para integración |
| Persistencia y contexto | 2 días | Base de datos para retener sesiones y flujos activos |
| Pruebas + documentación API | 2 días | Swagger + ejemplos de uso de cada endpoint |

> 🎯 *Permite combinar lo mejor de dos mundos: WhatsApp controlado por un programador, IA y flujos por otro.*

---

## 🧾 Observaciones
- Se puede comenzar por la consultoría n8n como etapa 1, y luego escalar a Python si se decide migrar
- También puede iniciarse por la **opción 3 colaborativa**, como paso intermedio eficiente
- Las tarifas pueden ajustarse si hay apoyo del equipo técnico local

---

## 📌 Recomendación
> Comenzar con consultoría n8n para resolver urgencias y validar estructura.
>
> Evaluar trabajo colaborativo (opción 3) como camino intermedio escalable.
>
> Si el flujo crece y requiere robustez, migrar a una solución integral en Python (opción 2).


## 📎 ¿Qué necesito para comenzar?
- Acceso al sistema n8n actual (si ya existe)
- API Keys de Twilio y OpenAI (si se usan)
- Usuario de prueba con WhatsApp
- Documentación actual o flujo en producción (si existe)