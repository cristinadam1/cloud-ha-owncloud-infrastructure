# Práctica 1: Despliegue de servicio ownCloud

Este repositorio contiene el diseño y despliegue de una infraestructura de almacenamiento en la nube basada en **ownCloud**, utilizando tecnologías de contenerización y orquestación para garantizar escalabilidad, persistencia y alta disponibilidad.

---

## 📂 Estructura 

El repositorio se organiza en tres escenarios evolutivos que representan diferentes necesidades empresariales:

### 1. [Escenario 1](./escenario1) - Pequeña Empresa
Despliegue básico de microservicios interconectados para un grupo reducido de usuarios.
* **Servicios:** ownCloud Web, MariaDB, Redis y LDAP.
* **Autenticación:** Gestión de identidades centralizada con OpenLDAP.
* **Persistencia:** Configuración de volúmenes locales para asegurar los datos de la BD y el directorio.

### 2. [Escenario 2](./escenario2) - Alta Disponibilidad (HA)
Arquitectura robusta para empresas medianas con redundancia y tolerancia a fallos.
* **Balanceador de Carga:** Uso de **HAProxy** como proxy inverso para distribuir el tráfico HTTP.
* **Escalabilidad:** Implementación de réplicas de servicios críticos para evitar puntos únicos de fallo.
* **Monitoreo:** Panel de estadísticas de HAProxy para control del rendimiento en tiempo real.

### 3. [owncloud-k8s](./owncloud-k8s) - Orquestación con Kubernetes
Archivos YAML para el despliegue automatizado del stack completo en un clúster de Kubernetes.
* **Objetos K8s:** Deployments, Services, PersistentVolumes (PV) e Ingress.
* **Automatización:** Gestión del ciclo de vida de los contenedores y escalado dinámico.

---

## 🛠️ Tecnologías Utilizadas
* **Motores de contenedores:** Docker / Podman.
* **Composición:** Docker-compose / Podman-compose.
* **Orquestación:** Kubernetes.
* **Servicios:** OpenLDAP, MariaDB, Redis, HAProxy y ownCloud.

---

## 🚀 Cómo Desplegar

Cada carpeta contiene su propio archivo `README.md` detallado con las instrucciones específicas.
