# 🤖 DeliveryBot — Sistema de Pedidos de Cafetería con IA

> Automatización de pedidos institucionales mediante Telegram + n8n + Google Sheets + Groq AI

---

## 📋 Descripción del Proyecto

**DeliveryBot** es una solución de automatización desarrollada con **n8n** que convierte a Telegram en una terminal de pedidos inteligente para entornos institucionales como oficinas, universidades o grandes centros de trabajo.

El sistema permite a los empleados y estudiantes consultar el menú, armar su carrito de compras y recibir notificaciones en tiempo real sobre el estado de su pedido (Recibido → Preparando → En camino → Entregado), mientras genera automáticamente reportes de ventas para la administración.

---

## 🎯 Objetivos

- Implementar un sistema de pedidos digital mediante una interfaz conversacional en Telegram
- Automatizar el cálculo de totales y la generación de números de orden únicos
- Gestionar el ciclo de vida del pedido a través de estados dinámicos
- Centralizar el inventario y menú en Google Sheets para actualización fácil
- Generar reportes diarios de ventas mediante procesos automáticos
- Optimizar la comunicación entre cocina y cliente mediante notificaciones push automáticas

---

## 🛠️ Stack Tecnológico

| Tecnología | Uso |
|---|---|
| **n8n Cloud** | Motor de automatización y orquestación de flujos |
| **Telegram Bot API** | Interfaz conversacional con el usuario |
| **Google Sheets** | Base de datos del menú, pedidos y sesiones |
| **Groq AI (Llama 3.3-70B)** | Procesamiento de lenguaje natural para el agente |
| **JavaScript (Code Nodes)** | Lógica de negocio y transformación de datos |

---

## 🏗️ Arquitectura del Sistema

El sistema está compuesto por 3 flujos modulares dentro de un único workflow de n8n:

### Flujo 1 — Pedidos de Usuarios (Principal)
```
Telegram Trigger → Normalizar Input → Obtener Sesión → Preparar Contexto
→ [Leer Menú + Leer Pedidos] → Combinar Datos → AI Agent DeliveryBot
→ Procesar Respuesta → ¿Es Pedido? → [Guardar Pedido / Actualizar Sesión]
→ Notificar Cocina → Responder Usuario
```

### Flujo 2 — Reporte Diario (Schedule 8PM)
```
Schedule Trigger → Leer Todos los Pedidos → Calcular Reporte → Enviar Reporte Admin
```

### Flujo 3 — Gestión de Estados (Admin)
```
Trigger Admin → Parsear Comando → Validar → Actualizar Estado en Sheets
→ Leer Pedido → Preparar Notificación → Notificar Usuario
```

---

## 🗄️ Modelo de Datos (Google Sheets)

La base de datos **DeliveryBot_DB** contiene 4 hojas:

### MENU
| id_producto | nombre | descripcion | precio | categoria | stock |
|---|---|---|---|---|---|
| P001 | Café negro | Café recién colado | 2500 | Bebidas | 50 |

### PEDIDOS
| id_pedido | id_usuario | detalles_pedido | total_pago | estado | fecha | hora |
|---|---|---|---|---|---|---|

### SESSIONS
| telegram_id | pantalla_actual | carrito_temporal | ultimo_cambio |
|---|---|---|---|

### USUARIOS
| telegram_id | nombre_completo | departamento | puntos_lealtad |
|---|---|---|---|

---

## ⚙️ Instalación y Configuración

### Requisitos Previos
- Cuenta en [n8n.cloud](https://n8n.cloud)
- Cuenta de Google con Google Sheets
- Bot de Telegram creado con @BotFather
- API Key de [Groq](https://console.groq.com) (gratuita)

### Paso 1 — Crear el Bot de Telegram

1. Abre Telegram y busca **@BotFather**
2. Envía `/newbot` y sigue las instrucciones
3. Guarda el token con formato `123456789:AAFxxx...`
4. Crea dos grupos en Telegram: **"Cocina DeliveryBot"** y **"Admin DeliveryBot"**
5. Agrega el bot a ambos grupos
6. Obtén los Chat IDs visitando:
   ```
   https://api.telegram.org/bot<TOKEN>/getUpdates
   ```

### Paso 2 — Configurar Google Sheets

1. Crea una hoja de cálculo llamada `DeliveryBot_DB`
2. Crea las 4 pestañas: `MENU`, `PEDIDOS`, `SESSIONS`, `USUARIOS`
3. Agrega los encabezados exactos según el modelo de datos
4. Puebla la hoja `MENU` con los productos de la cafetería

### Paso 3 — Obtener API Key de Groq

1. Ve a [console.groq.com/keys](https://console.groq.com/keys)
2. Crea una API Key gratuita
3. Guarda la key con formato `gsk_...`

### Paso 4 — Configurar Credenciales en n8n

En **Settings → Credentials** crea 3 credenciales:

| Nombre | Tipo | Valor |
|---|---|---|
| `Telegram DeliveryBot` | Telegram API | Token del bot |
| `Google Sheets DeliveryBot` | Google Sheets OAuth2 | Sign in with Google |
| `Groq DeliveryBot` | Groq API | API Key `gsk_...` |

### Paso 5 — Importar el Workflow

1. En n8n ve a **Workflows → New Workflow**
2. Menú `⋯` → **Import from File**
3. Sube el archivo `DeliveryBot_workflow.json`
4. Asigna las credenciales en cada nodo
5. Activa con **Publish**

---

## 🧪 Pruebas

### Flujo de pedido completo:
1. Abrir Telegram y buscar `@Cafeteria_Delivery_bot`
2. Enviar `/start`
3. El bot responde con el menú completo por categorías
4. Pedir un producto: `"Quiero un café negro"`
5. Confirmar: `"Sí, agrégalo y confirma el pedido"`
6. Verificar que llegue notificación al grupo Cocina
7. Verificar que se guardó el pedido en Google Sheets → hoja PEDIDOS

### Cambio de estado (Admin):
En el grupo Admin enviar:
```
/estado ORD-XXXX-XXXXXX Preparando
```
El cliente recibe automáticamente una notificación con el nuevo estado.

---

## 📊 Reporte Diario

Todos los días a las **8:00 PM** el sistema envía automáticamente al grupo Admin:

```
📊 REPORTE DIARIO DELIVERYBOT
📅 08/06/2026

🛒 Pedidos: 15
💰 Total: $37,500
⭐ Estrella: Café negro (8 uds)
🕐 Hora pico: 12:00 (6 pedidos)

📦 Detalle:
  • Café negro: 8
  • Almuerzo del día: 4
  • Empanada de carne: 3
```

---

## 🔄 Estados del Pedido

| Estado | Emoji | Descripción |
|---|---|---|
| Recibido | 📥 | Pedido registrado automáticamente |
| Preparando | 👨‍🍳 | Cocina comenzó la preparación |
| En camino | 🛵 | Pedido en camino al cliente |
| Entregado | ✅ | Pedido entregado exitosamente |
| Cancelado | ❌ | Pedido cancelado |

---

## 🚧 Dificultades Encontradas Durante el Desarrollo

Durante el desarrollo del proyecto se presentaron varios obstáculos técnicos que requirieron investigación y soluciones creativas:

### 1. Limitación de Telegram: Un solo Trigger por Bot
**Problema:** n8n lanza un error `"The service is receiving too many requests"` cuando se intenta activar dos nodos `Telegram Trigger` con el mismo bot simultáneamente. El flujo principal de pedidos y el flujo de gestión de estados del admin no podían coexistir activos al mismo tiempo.

**Solución:** Se desactivó el `Trigger Admin Estados` y se dejó el flujo principal activo. La solución definitiva requiere un segundo bot de Telegram exclusivo para el administrador.

### 2. Compatibilidad de API Key de Google AI Studio con n8n
**Problema:** Las API Keys generadas en Google AI Studio tienen un formato `AQ.xxxxx` que no es compatible con el nodo `Google Gemini(PaLM) API` de n8n, el cual espera keys con formato `AIza...`. Esto generó errores de `"No matching data"` al intentar cargar los modelos disponibles.

**Solución:** Se migró a **Groq** como proveedor de IA, que ofrece acceso gratuito al modelo `llama-3.3-70b-versatile` con una API Key estándar y total compatibilidad con n8n.

### 3. El Workflow se Detenía en "Obtener Sesión"
**Problema:** Para usuarios nuevos, el nodo `Obtener Sesión` no encontraba registros en Google Sheets y devolvía `"No output data"`, lo que detenía toda la ejecución del workflow sin llegar al agente de IA.

**Solución:** Se activó la opción `"Always Output Data"` en el nodo, permitiendo que continúe con datos vacíos cuando el usuario es nuevo y no tiene sesión registrada.

### 4. Error de Parsing de Markdown en Telegram
**Problema:** El agente de IA generaba respuestas con caracteres especiales (`*`, `_`, `$`) que el modo `Markdown (Legacy)` de Telegram no podía procesar, resultando en el error `"Can't parse entities: Can't find end of the entity"` y mensajes que nunca llegaban al usuario.

**Solución:** Se cambió el `Parse Mode` del nodo `Responder Usuario` de `Markdown (Legacy)` a `HTML`, resolviendo completamente el problema de parsing.

### 5. Operación "upsert" Deprecada en Google Sheets
**Problema:** Los nodos de Google Sheets `Actualizar Sesion` y `Limpiar Sesion Post-Pedido` utilizaban la operación `upsert` que fue deprecada en versiones recientes de n8n, generando el error `"Could not get parameter: columns.schema"`.

**Solución:** Se migró a la operación `Append or Update Row` con `Map Automatically`, especificando `telegram_id` como columna de coincidencia.

### 6. El Agente de IA no Generaba los Bloques ACTION
**Problema:** El modelo Llama 3 de Groq no seguía consistentemente las instrucciones del System Prompt para generar los bloques `|||ACTION:{}|||` necesarios para que el workflow ejecutara acciones como agregar al carrito o confirmar pedidos. El bot respondía conversacionalmente pero sin disparar las acciones.

**Solución:** Se reescribió el System Prompt de manera más imperativa y directa, con instrucciones explícitas de cuándo y cómo generar cada tipo de acción, reduciendo la ambigüedad y mejorando el seguimiento de instrucciones del modelo.

### 7. Datos del Carrito no Llegaban al Nodo de Guardado
**Problema:** El nodo `Generar ID Pedido` buscaba el carrito en `data.carrito_nuevo` pero el agente enviaba los datos en `data.accion.carrito`, resultando en pedidos guardados con detalles vacíos y total `$0.00`.

**Solución:** Se actualizó el código del nodo para verificar ambas rutas de datos con fallback:
```javascript
const carrito = (data.accion?.carrito && data.accion.carrito.length > 0) 
  ? data.accion.carrito 
  : data.carrito_nuevo;
```

---

## 📁 Estructura del Repositorio

```
Proyecto_DeliveryBot/
├── README.md
├── DeliveryBot_workflow.json    # Workflow completo de n8n
└── docs/
    └── arquitectura.png         # Diagrama de flujo del sistema
```

---

## 🔮 Mejoras Futuras

- Activar el sistema de cambio de estados del admin con un segundo bot dedicado
- Implementar descuento de stock automático en la hoja MENU al confirmar pedidos
- Agregar sistema de puntos de lealtad para usuarios frecuentes
- Integrar pasarela de pagos (Stripe, MercadoPago) para cobro digital
- Implementar sistema de reservas con horario de entrega
- Panel web administrativo para gestión del menú sin tocar Google Sheets
- Soporte multi-idioma para entornos internacionales

---

## 👨‍💻 Autor

Desarrollado como proyecto de automatización institucional con n8n.

**Tecnologías:** n8n · Telegram Bot API · Google Sheets · Groq AI · JavaScript

---

## 📄 Licencia

Este proyecto está bajo la licencia MIT. Ver el archivo `LICENSE` para más detalles.
