# Práctica 2: Reconocimiento Facial como Servicio (FaaS)

En este archivo se explica la práctca 2 para la implementación de un sistema de identificación biométrica basado en el paradigma **Functions-as-a-Service (FaaS)**. [cite_start]La infraestructura permite la detección automática de rostros en imágenes mediante el despliegue de funciones escalables sobre un clúster de **Kubernetes**[cite: 1].

---

## 📂 Estructura del Proyecto

El proyecto documenta el flujo completo desde la provisión de la plataforma hasta el desarrollo de la lógica de visión artificial:

### 1. [Infraestructura OpenFaaS](./platform-deployment)
Configuración de la capa de computación sin servidor sobre el orquestador de contenedores.
* **Orquestación**: Despliegue de OpenFaaS utilizando **Kubernetes (Minikube)** como base para el catálogo de funciones.
* **Gestión de Funciones**: Instalación y configuración de **faas-cli** para administrar el ciclo de vida (build, push, deploy) de los servicios.
* **Gateway**: Exposición del API Gateway para la invocación de funciones mediante peticiones HTTP.

### 2. [Detección Facial con Python](./facesdetection-python)
Desarrollo de una función personalizada optimizada para el procesamiento de imágenes en tiempo real.
* **Lógica**: Implementación en **Python 3** utilizando la librería **OpenCV** para el análisis computacional.
* **Algoritmo**: Uso de clasificadores pre-entrenados (*Haar Cascades*) para la detección precisa de coordenadas faciales.
* **Funcionalidad**: La función recibe una URL de imagen, identifica los rostros presentes y devuelve el archivo con marcos delimitadores dibujados.

### 3. [Evaluación de Catálogo](./function-store)
Análisis de rendimiento y precisión de herramientas de biometría disponibles en la comunidad.
* **Modelos Evaluados**: Pruebas comparativas con las funciones `face-detect-pigo` y `face-detect-opencv` del OpenFaaS Store.
* **Pipeline de CI/CD**: Automatización del flujo de actualización de funciones hacia Docker Hub y su despliegue inmediato en el clúster.

---

## 🛠️ Tecnologías Utilizadas
* **Plataforma FaaS**: OpenFaaS sobre Kubernetes.
* **Lenguajes**: Python (Handler de la función).
* **Librerías de Visión**: OpenCV para detección de objetos y procesamiento de imagen.
* **Herramientas**: faas-cli para despliegue, Docker Hub para registro de imágenes y cURL para testeo de endpoints.
