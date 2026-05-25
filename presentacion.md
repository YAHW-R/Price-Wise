# Presentación del Proyecto: Price-Wise System

## Introducción
Price-Wise es un ecosistema de microservicios diseñado para el análisis de productos de e-commerce en tiempo real. El sistema permite buscar productos en múltiples plataformas, extraer datos detallados, analizarlos mediante Inteligencia Artificial para detectar estafas o evaluar la relación calidad-precio, y entregar los resultados al usuario final de forma instantánea.

---

## 📦 Módulo 1: Message Broker (Orquestador Central)

**Tecnología:** Go 1.24+

### 🎯 Finalidad
Es el cerebro del sistema. Su función principal es desacoplar y coordinar la comunicación entre la aplicación móvil, el servicio de scraping y el servicio de IA. Actúa como un gestor de estado que garantiza que cada solicitud de búsqueda sea procesada y entregada correctamente.

### ⚙️ Funcionamiento
1. **Recepción de Solicitudes:** Recibe órdenes de búsqueda desde la App, genera un identificador único (`JobID`) y lo devuelve inmediatamente al cliente para seguimiento.
2. **Activación del Pipeline:** Dispara de forma proactiva el proceso de scraping.
3. **Ingesta Asíncrona:** Recibe datos crudos del Scraper y los pone en cola para su análisis.
4. **Análisis en Paralelo:** Distribuye la carga de trabajo a un grupo de "workers" que consultan al servicio de IA.
5. **Streaming en Tiempo Real:** Envía los resultados analizados al usuario a través de **WebSockets**, permitiendo que la interfaz se actualice conforme se encuentran productos.

### 🧩 Partes y Componentes Clave
- **Job Manager:** Gestiona el ciclo de vida de cada búsqueda (pendiente, en proceso, completado).
- **Engine (Pub/Sub):** Lógica central que utiliza canales de Go para el paso de mensajes entre servicios.
- **Dispatcher & Worker Pool:** Un conjunto de hilos (goroutines) dedicados a procesar datos concurrentemente, optimizando el uso de recursos y evitando cuellos de botella.
- **WebSocket Server:** Mantiene conexiones persistentes con los clientes para la entrega de datos tipo "Push".

---

## 🕷️ Módulo 2: Scraper Service (Extractor de Datos)

**Tecnología:** Python 3.11+ / FastAPI

### 🎯 Finalidad
Actúa como un servicio "esclavo" bajo demanda. Su único objetivo es navegar por sitios de e-commerce (como Amazon o Mercado Libre), extraer la información técnica de los productos y enviarla de vuelta al Broker lo más rápido posible.

### ⚙️ Funcionamiento (Streaming Inverso)
A diferencia de los scrapers tradicionales que terminan todo el trabajo antes de devolver datos, este módulo implementa **Streaming Inverso**:
1. Recibe una orden del Broker con el producto a buscar.
2. Inicia la navegación en múltiples tiendas simultáneamente.
3. **Inmediatez:** Cada vez que encuentra un producto individual, lo envía instantáneamente al Broker sin esperar a que termine la búsqueda completa.
4. Una vez agotadas las fuentes, notifica la finalización del trabajo.

### 🧩 Partes y Componentes Clave
- **API (FastAPI):** Expone los endpoints para activar búsquedas y verificar el estado del servicio.
- **Motor de Scraping (Playwright/Crawl4AI):** Utiliza tecnologías modernas de automatización de navegadores para superar protecciones y extraer datos precisos (precios, imágenes, valoraciones).
- **Parser de Tiendas:** Lógica especializada para cada plataforma:
    - **Amazon:** Navegación profunda en páginas individuales para obtener precios reales.
    - **Mercado Libre:** Extracción optimizada mediante el análisis de estados JSON embebidos en el HTML.
- **Cliente de Ingestión:** Módulo encargado de la comunicación asíncrona de alta velocidad con el Broker.

---

## 🔄 Flujo de Integración
La sinergia entre estos dos módulos permite un flujo de datos continuo y eficiente:
1. El **Broker** manda y el **Scraper** obedece.
2. El **Scraper** encuentra y el **Broker** distribuye.
3. El **Usuario** recibe información valiosa en segundos, no minutos.

Este diseño modular permite que el sistema sea altamente escalable, pudiendo añadir más scrapers o servicios de análisis sin afectar la estabilidad de la plataforma.
