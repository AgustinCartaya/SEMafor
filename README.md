# 🚦 SEMàfor

**Sistema de apoyo a la decisión para el Sistema d'Emergències Mèdiques (SEM) de Catalunya que elige el mejor hospital de destino para cada paciente, en tiempo real.**

🔗 **Demo interactiva (mockup):** https://agustincartaya.github.io/SEMafor/

---

## 📌 El problema

Cuando una ambulancia del SEM atiende una emergencia (un ictus, un infarto, un politraumatismo…), hay que decidir **a qué hospital llevar al paciente**. Hoy esa decisión depende en gran parte de **llamadas telefónicas** desde la central del SEM a los hospitales para preguntar:

- ¿Hay TAC / sala de hemodinámica / quirófano disponible?
- ¿Está de guardia el especialista necesario (neurología, cardiología, cirugía…)?
- ¿Hay camas de UCI libres?
- ¿Cómo está de saturado el servicio de urgencias?

Estas llamadas **consumen minutos críticos**, ocupan al personal de la central y de los hospitales, y la información puede quedar desactualizada enseguida. Además, el hospital más cercano no siempre es el mejor: si no puede tratar la patología, el paciente necesitará un **traslado secundario**, y se pierde todavía más tiempo.

## 💡 La solución

SEMàfor **reduce (o elimina) esas llamadas** obteniendo automáticamente el estado de los recursos de cada hospital y usa **Inteligencia Artificial** para recomendar el destino que minimiza el **tiempo hasta el tratamiento definitivo** (*"Fins al tractament"*), no solo el tiempo de llegada.

> ⏱️ **Fins al tractament** = trayecto con tráfico real + tiempo habitual del centro desde la puerta hasta el tratamiento (+ posible traslado secundario).

La recomendación se muestra al operador del SEM en una interfaz gráfica clara: el operador **siempre tiene la última palabra**; SEMàfor le da la información y una propuesta razonada.

## 🏗️ Arquitectura

SEMàfor se basa en **dos bloques** y se **acopla al sistema actual del SEM** (gestión de incidentes, ambulancias, localización de recursos…) sin reinventar la rueda.

```
 ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
 │  Hospital A  │   │  Hospital B  │   │  Hospital N  │   Sistemas propios de cada
 │ (HIS, camas, │   │ (HIS, camas, │   │ (HIS, camas, │   hospital, cada uno con su
 │  quirófanos) │   │  quirófanos) │   │  quirófanos) │   formato
 └──────┬───────┘   └──────┬───────┘   └──────┬───────┘
        │ ETL              │ ETL              │ ETL        ① Extracción y normalización
        ▼                  ▼                  ▼
 ┌─────────────────────────────────────────────────────┐
 │        Datos unificados de recursos hospitalarios    │
 └──────────────────────────┬──────────────────────────┘
                            │
   Sistema actual del SEM ──┤  (incidente, ubicación, ambulancias,
   Tráfico en tiempo real ──┤   patología, gravedad…)
                            ▼
 ┌─────────────────────────────────────────────────────┐
 │   Motor de decisión con IA (Graph Neural Networks)   │   ② Decisión inteligente
 └──────────────────────────┬──────────────────────────┘
                            ▼
 ┌─────────────────────────────────────────────────────┐
 │      Interfaz gráfica para el operador del SEM       │   Recomendación + explicación
 └─────────────────────────────────────────────────────┘
```

### ① Extracción de información (ETL)

Cada hospital tiene sus propios sistemas y formatos. SEMàfor coloca **una ETL por hospital** que actúa como "traductor":

- **Extract**: lee el estado de los recursos del hospital (camas, UCI, TAC, RM, hemodinámica, quirófanos, especialistas de guardia, saturación de urgencias…).
- **Transform**: convierte esos datos a un **formato común** y validado.
- **Load**: los publica de forma continua en un repositorio unificado accesible para el SEM.

Así el SEM dispone de una **visión actualizada de toda la red hospitalaria** sin tener que llamar.

### ② Decisión con IA (Graph Neural Networks)

La red sanitaria se modela de forma natural como un **grafo**: hospitales, ubicación del incidente y ambulancias son nodos; las carreteras y las rutas de traslado entre centros son aristas. Una **GNN** aprovecha esta estructura para puntuar cada hospital candidato teniendo en cuenta:

| Parámetro | Ejemplo |
|---|---|
| 🏥 Disponibilidad de recursos necesarios | TAC + neurología de guardia para un código ictus |
| 📍 Distancia | Km reales por carretera |
| 🚗 Tráfico | Tiempo de trayecto estimado en tiempo real |
| 🔁 Traslados secundarios | Probabilidad / coste de tener que derivar después |
| ⏳ Tiempo puerta-tratamiento | Rendimiento habitual de cada centro |
| 📈 Saturación | Carga actual de urgencias y UCI |
| 🩺 Patología y gravedad | Códigos de activación (ictus, IAM, politrauma, sepsis…) |

El resultado es un **ranking de hospitales** con la opción recomendada y el motivo, que el operador puede aceptar o modificar.

### 🔌 Integración con el SEM

SEMàfor **no sustituye** las herramientas actuales del SEM: se conecta a ellas para reutilizar lo que ya existe (gestión de incidentes, flota de ambulancias, geolocalización) y añade una capa de información hospitalaria y de recomendación inteligente.

## 🖥️ Mockup

Este repositorio contiene un **mockup interactivo** ([`SEMafor.html`](SEMafor.html)) que simula el flujo completo: una emergencia, la extracción de datos vía ETL de los hospitales y la recomendación de destino de SEMàfor según el tiempo hasta el tratamiento.

- **Online:** https://agustincartaya.github.io/SEMafor/
- **Local:** descarga el repositorio y abre `SEMafor.html` en cualquier navegador (no requiere instalación ni servidor).

> ⚠️ Es una demo conceptual con datos simulados; no está conectada a sistemas reales del SEM ni de los hospitales.

## 📂 Estructura del repositorio

```
SEMafor/
├── SEMafor.html   # Mockup interactivo (autocontenido)
├── index.html     # Redirige a SEMafor.html (página de inicio de GitHub Pages)
├── .nojekyll      # Sirve los archivos tal cual en GitHub Pages
└── README.md
```

## 🗺️ Próximos pasos

- [ ] Definir el esquema común de datos hospitalarios.
- [ ] Prototipo de ETL con datos de un hospital piloto.
- [ ] Entrenar y validar el modelo GNN con datos históricos de traslados.
- [ ] Integración con la API del sistema de gestión del SEM.
- [ ] Validación con operadores del SEM.

## 👥 Equipo

Proyecto desarrollado en el marco de **HackIDIBGI** (VICOROB – Universitat de Girona).
