# Sistema Price-Wise

Price-Wise es un sistema desacoplado basado en microservicios diseñado para el análisis de productos de comercio electrónico en tiempo real. Permite a los usuarios buscar productos en múltiples plataformas, extraer datos, analizarlos para detectar posibles estafas y evaluar la relación calidad-precio, recibiendo actualizaciones en tiempo real.

## 🚀 Descripción General de la Arquitectura

El sistema sigue un pipeline asíncrono impulsado por eventos:

1.  **Inicio del Trabajo:** Un cliente envía una solicitud de búsqueda al Broker.
2.  **Activación del Scraper:** El Broker activa el servicio Scraper.
3.  **Ingesta de Datos:** El Scraper encuentra productos y envía los datos crudos de vuelta al Broker.
4.  **Análisis de IA:** El Broker envía estos datos al servicio de IA en lotes o mediante streaming.
5.  **Resultados en Tiempo Real:** El Broker recibe los datos analizados y los envía al usuario en tiempo real a través de WebSockets.

### Componentes Principales

- **Broker (Go):** El orquestador central. Gestiona los estados de los trabajos, distribuye tareas y transmite resultados.
- **Scraper (Python/FastAPI):** Servicio de extracción que utiliza Playwright y Crawl4ai para recopilar datos crudos de productos.
- **Servicio de IA (Python/FastAPI):** Servicio de análisis que utiliza modelos LightGBM para evaluar productos y detectar anomalías o estafas.

## 🛠 Stack Tecnológico

- **Broker:** Go 1.24+, Gorilla WebSocket.
- **Scraper:** Python 3.11+, FastAPI, Playwright, Crawl4ai.
- **Servicio de IA:** Python 3.11+, FastAPI, LightGBM, Pandas, Scikit-learn.

## 🚦 Primeros Pasos

### Usando Docker Compose (Recomendado)

La forma más sencilla de ejecutar todo el sistema:

```bash
docker-compose up --build
```

Esto expone el **Broker** en el puerto `8000`.

### Ejecución Local

Para ejecutar el sistema localmente, inicie los servicios en este orden:

1.  **Servicio de IA:**
    ```bash
    cd AI-python
    pip install -r requirements.txt
    uvicorn app.main:app --port 5001
    ```
2.  **Servicio Scraper:**
    ```bash
    cd scraping-python
    pip install -r requirements.txt
    playwright install chromium
    uvicorn scraper_server.main:app --port 5000
    ```
3.  **Servicio Broker:**
    ```bash
    cd brocker-go
    go run cmd/broker/main.go
    ```

## 📁 Estructura del Proyecto

```text
Price-Wise-system/
├── AI-python/           # Servicio de Análisis ML (FastAPI)
├── brocker-go/          # Servicio de Orquestación y WebSocket (Go)
└── scraping-python/     # Servicio de Scraping Web (FastAPI)
```

## 📜 Licencia

[Agregar información de licencia aquí]
