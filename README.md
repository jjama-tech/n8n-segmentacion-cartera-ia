# Automatización de Segmentación de Cartera con IA

Automatización desarrollada con **n8n** para segmentar cuentas vencidas según su nivel de riesgo utilizando inteligencia artificial.

## Descripción

El proyecto automatiza el proceso de análisis y segmentación de una cartera de clientes con cuentas vencidas.

El flujo obtiene diariamente las cuentas que se encuentran en estado `vencido`, analiza sus características mediante un modelo de lenguaje y asigna uno de tres niveles de riesgo:

- **Recuperable**
- **Dudosa**
- **Crítica**

Además de la clasificación, la inteligencia artificial genera una breve justificación de la decisión.

Una vez clasificada cada cuenta, el resultado se almacena automáticamente y el flujo utiliza un nodo `Switch` para dirigir cada caso hacia una gestión diferenciada.

---

## Estructura del repositorio

```text
n8n-segmentacion-cartera-ia/
│
├── README.md
│
├── workflow/
│   ├── README.md
│   └── workflow-demo.json
│
├── data/
│   ├── README.md
│   └── cartera_demo.xlsx
│
├── screenshots/
│   ├── README.md
│   └── workflow.png
│
└── docs/
    └── README.md
```

## Archivos principales
* `workflow/workflow-demo.json` — Workflow de n8n preparado para demostración.
* `data/cartera_demo.xlsx` — Dataset ficticio utilizado para las pruebas.
* `screenshots/workflow.png` — Captura visual de la automatización.
* `docs/` — Documentación complementaria del proyecto.

---

## Cómo probar el proyecto

### Requisitos

- Una instancia de n8n.
- Acceso a un modelo de lenguaje compatible.
- Una Data Table de n8n.
- Credenciales propias para el proveedor de IA y Email.

### Pasos

1. Descargar `workflow/workflow-demo.json`.
2. Importar el workflow en n8n.
3. Crear una Data Table utilizando `data/cartera_demo.xlsx` como fuente de datos.
4. Configurar las credenciales del modelo de IA.
5. Configurar las credenciales de Email.
6. Verificar las conexiones de los nodos.
7. Ejecutar el workflow manualmente.
8. Comprobar la clasificación de las cuentas en la Data Table.
9. Verificar el enrutamiento realizado por el nodo `Switch`.

> Las credenciales utilizadas en el entorno original no forman parte de este repositorio. Cada usuario debe configurar sus propias credenciales.

---

## Problema

No todas las cuentas vencidas requieren el mismo nivel de gestión.

Una cartera puede contener cuentas con pocos días de atraso y bajo número de intentos de cobro, mientras que otras pueden presentar períodos prolongados de vencimiento, múltiples intentos de gestión o montos elevados.

La revisión manual de cada cuenta dificulta la priorización y consume tiempo operativo.

---

## Solución

Se desarrolló un workflow automatizado que:

1. Se ejecuta automáticamente mediante un `Schedule Trigger`.
2. Obtiene las cuentas que se encuentran en estado vencido.
3. Procesa cada cuenta individualmente.
4. Envía los datos relevantes a un modelo de inteligencia artificial.
5. Clasifica la cuenta según su nivel de riesgo.
6. Genera una justificación de la clasificación.
7. Actualiza la información en la tabla de origen.
8. Utiliza un `Switch` para enrutar cada cuenta.
9. Ejecuta una notificación diferente según el nivel de riesgo.

---

## Arquitectura del workflow

```text
Schedule Trigger
       │
       ▼
Data Table - Get Rows
       │
       ▼
Loop Over Items
       │
       ▼
Wait
       │
       ▼
AI Agent
       │
       ├── OpenAI Chat Model
       │
       └── Structured Output Parser
       │
       ▼
Data Table - Update Rows
       │
       ▼
Switch
   ┌───┼───────────┐
   │   │           │
   ▼   ▼           ▼
Recuperable     Dudosa       Crítica
   │   │           │
   ▼   ▼           ▼
 Email           Email       Email
   │   │           │
   └───┴───────────┘
           │
           ▼
     Loop siguiente
```

---

## Criterios de clasificación

La IA utiliza como referencia las siguientes reglas:

| Nivel |	Criterio |
| :--- | :--- |
| Recuperable |	Pocos días de vencimiento (≤30), pocos intentos y monto bajo o medio. |
| Dudosa | Entre 30 y 90 días de vencimiento, 2–3 intentos y monto medio. |
| Crítica |	Más de 90 días, muchos intentos (≥4), o monto alto con mucho atraso. |

Los criterios sirven como guía para la clasificación realizada por el modelo.

---

## Datos utilizados

El proyecto utiliza datos ficticios de prueba.

Las variables principales utilizadas por el modelo son:

* id_factura
* monto
* fecha_emision
* fecha_vencimiento
* dias_vencido
* estado
* intentos
* ultima_gestion

El identificador **id_factura** se utiliza para localizar el registro correspondiente al momento de actualizar la información.

---

## Salida de la inteligencia artificial

La IA genera una salida estructurada que contiene:

```text
{
  "id_factura": "F-0009",
  "nivel_riesgo": "critica",
  "justificacion": "La cuenta presenta un nivel elevado de atraso y múltiples intentos de cobro."
}
```

La información generada se utiliza posteriormente dentro del workflow para actualizar la tabla y determinar la ruta correspondiente.

---

## Tecnologías utilizadas

* n8n
* Inteligencia Artificial / LLM
* AI Agent
* Structured Output Parser
* Data Table
* Switch
* Email
* JSON
* APIs
* Expresiones de n8n

---

## Retos técnicos

Durante el desarrollo se presentaron diferentes retos relacionados con la integración del modelo de inteligencia artificial y el procesamiento de múltiples registros.

---

## Control de solicitudes a la API

Para evitar problemas asociados a límites de solicitudes, se implementó Loop Over Items junto con un nodo Wait, procesando los registros individualmente.

---

## Salida estructurada

La respuesta del modelo debía ser utilizada posteriormente por otros nodos del workflow. Para ello se implementó un Structured Output Parser, permitiendo trabajar con campos estructurados como nivel_riesgo y justificacion.

---

## Actualización de registros

El campo id_factura se utiliza como identificador para localizar y actualizar el registro correspondiente en la tabla.

---

## Resultado

El proceso permite automatizar el ciclo completo de segmentación:

```text
Cuenta vencida
      ↓
Análisis mediante IA
      ↓
Clasificación de riesgo
      ↓
Justificación
      ↓
Actualización de datos
      ↓
Enrutamiento
      ↓
Gestión diferenciada
```
La automatización reduce la necesidad de revisar manualmente cada cuenta y permite priorizar la gestión según el nivel de riesgo identificado.

---

## Aprendizajes y retos

Durante el desarrollo se trabajó principalmente en:

- Integración de modelos de lenguaje dentro de workflows de n8n.
- Diseño de salidas estructuradas para que la IA pueda interactuar con otros nodos.
- Procesamiento individual de registros mediante `Loop Over Items`.
- Control del ritmo de solicitudes mediante `Wait`.
- Uso de identificadores para actualizar registros específicos.
- Enrutamiento de resultados mediante condiciones.
- Integración de IA con acciones posteriores dentro de un proceso automatizado.

Uno de los principales retos fue controlar el procesamiento de múltiples registros y los límites de solicitudes de la API. Esto llevó a implementar un procesamiento individual de las cuentas antes de continuar con el siguiente registro.

---

## Consideraciones de seguridad

Este repositorio utiliza únicamente datos ficticios y material preparado para demostración.

No se incluyen:

* Credenciales
* API Keys
* Contraseñas
* Datos reales de clientes
* Información confidencial
* Configuraciones internas de infraestructura

La versión pública del workflow será adaptada antes de ser publicada para evitar exponer información perteneciente a entornos corporativos.

---

## Autor

**Jimmy Jama**

Profesional de TI enfocado en soporte, operaciones, análisis de datos, automatización e inteligencia artificial aplicada a procesos.

GitHub: [jjama-tech](https://github.com/jjama-tech)
