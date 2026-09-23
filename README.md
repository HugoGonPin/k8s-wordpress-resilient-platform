# Plataforma Web Resiliente y Autogestionada en Kubernetes (K3s)

[![Kubernetes](https://img.shields.io/badge/Orchestrator-K3s-326CE5?logo=kubernetes&logoColor=white)](https://k3s.io/)
[![Docker](https://img.shields.io/badge/Containers-Docker-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![Cloudflare](https://img.shields.io/badge/Security-Cloudflare_Zero_Trust-F38020?logo=cloudflare&logoColor=white)](https://www.cloudflare.com/)
[![Prometheus](https://img.shields.io/badge/Monitoring-Prometheus-E6522C?logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Observability-Grafana-F46800?logo=grafana&logoColor=white)](https://grafana.com/)

Infraestructura contenerizada desacoplada de servidores tradicionales monolíticos, implementando alta disponibilidad, autorreparación, persistencia desacoplada y exposición perimetral segura basada en arquitectura **Zero Trust**.

---

## Resumen Ejecutivo del Proyecto

El objetivo de este proyecto fue diseñar e implementar una arquitectura de producción que sustituye los servidores tradicionales (monolitos frágiles) por un ecosistema de orquestación moderno. Mediante el uso de **Kubernetes (K3s)**, se eliminan los puntos únicos de fallo (SPOF) gracias a réplicas redundantes, autorreparación en caliente de pods y almacenamiento independiente del ciclo de vida de los contenedores.

La exposición pública a internet se realiza mediante un túnel de capa 7 sin abrir puertos en el cortafuegos, garantizando seguridad perimetral avanzada y cifrado TLS continuo.

---

## Arquitectura Técnica y Decisiones de Diseño

### 1. Patrón Sidecar (WordPress + Nginx)
* **Desacoplamiento optimizado:** En lugar de desplegar Nginx y WordPress en pods aislados a través de la red de Kubernetes, se agruparon en una misma unidad de despliegue (`Pod`).
* **Comunicación Localhost:** Nginx actúa como proxy inverso y procesador estático, redirigiendo el código dinámico a WordPress (PHP-FPM) a través de `localhost:9000`. Esto elimina la latencia de red interna y asegura tiempos de respuesta mínimos.

### 2. Motor de Base de Datos y Persistencia
* **MariaDB 10.11 Personalizado:** Despliegue independiente a partir de una imagen personalizada para forzar el juego de caracteres `utf8mb4`, optimizando el consumo de recursos frente a MySQL estándar.
* **Almacenamiento Desacoplado (PV / PVC):** Implementación de volúmenes persistentes de 5GB asociados al host. Si el pod de base de datos se destruye, el nuevo pod se reconecta automáticamente al reclamo de volumen sin pérdida de datos.

### 3. Seguridad de Credenciales
* **Kubernetes Secrets:** Las credenciales de la base de datos se desacoplaron del código fuente (`mysql-secret.yaml`), manteniéndolas codificadas e inyectadas dinámicamente como variables de entorno seguras.

### 4. Perímetro Zero Trust con Cloudflare
* **Cero Puertos Abiertos:** El cortafuegos de la máquina host mantiene todos los puertos entrantes bloqueados.
* **Túnel Seguro (`cloudflared`):** La comunicación hacia el exterior se establece mediante una conexión saliente gestionada por Cloudflare Tunnel, proporcionando mitigación de ataques DDoS y terminación SSL/TLS automática para el dominio corporativo.

### 5. Observabilidad y Métricas
* Monitorización en tiempo real del uso de CPU y memoria de los nodos y pods mediante la integración de **Prometheus** y tableros visuales en **Grafana**.
* Consola administrativa desacoplada mediante un pod auxiliar de **phpMyAdmin**.

---

## Hitos y Resultados Conseguidos

| Métrica / Prueba | Resultado Obtenido | Impacto Técnico |
| :--- | :--- | :--- |
| **Prueba del Caos (Destrucción de Pod)** | Autorecuperación en **< 6 segundos** | Recuperación ante desastres inmediata sin corte de servicio. |
| **Escalabilidad** | Escalado de 1 a 3 réplicas en caliente | Distribución eficiente de carga ante picos de tráfico. |
| **Superficie de Ataque** | **0 puertos abiertos** en el firewall perimetral | Mitigación completa de escaneos de puertos no autorizados. |
| **Integridad de Datos** | **0% pérdida de datos** tras reinicios y caídas de pod | Persistencia de datos desacoplada exitosa con PV/PVC. |

---

## Comandos Clave Utilizados

### Despliegue y Orquestación
```bash
# Comprobación de salud del clúster K3s
sudo k3s kubectl get nodes

# Despliegue de los manifiestos de infraestructura
sudo k3s kubectl apply -f mysql-secret.yaml
sudo k3s kubectl apply -f mysql-persistence.yaml
sudo k3s kubectl apply -f mysql-deployment.yaml
sudo k3s kubectl apply -f wordpress-deployment.yaml

# Comprobación de pods y servicios
sudo k3s kubectl get pods -o wide
sudo k3s kubectl get svc,pvc,pv
