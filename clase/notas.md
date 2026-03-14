# 🗒️ Registro de Trabajo en Clase — Taller 4: Mapa de Infraestructura y Diagnóstico Técnico

## 📆 Fecha de la sesión
14 de marzo de 2026

## 👥 Integrantes presentes
- Miguel Ángel Ardila
- Daniel Moreno
- Sebastián Torres

---

## 🧠 Actividades realizadas en clase

Durante la sesión se trabajó sobre el **caso base de RedExpress**, una plataforma de logística con infraestructura híbrida. Las actividades desarrolladas fueron:

- **Lectura y análisis del caso:** se revisaron los componentes descritos en el README para entender el contexto de RedExpress: app móvil, plataforma web, servidores regionales, servicios en la nube y dispositivos móviles de mensajeros.
- **Discusión del equipo:** se identificaron los nodos más críticos del sistema (API Gateway, base de datos y servicios de rastreo) y se debatió cuáles representaban puntos únicos de falla.
- **Modelado inicial:** usando draw.io, se construyó el mapa de infraestructura preliminar con capas diferenciadas (usuario, borde, servicios, datos y operaciones).
- **Decisiones de modelado tomadas:**
  - Separar la capa de acceso (CDN + Load Balancer) de la capa de microservicios.
  - Modelar la base de datos como clúster con réplica de lectura para reducir la carga.
  - Incluir una cola de mensajes (Kafka) para desacoplar el servicio de rastreo en tiempo real del resto del sistema.
  - Incluir un nodo de caché (Redis) para disminuir latencia en consultas frecuentes de estado de paquetes.

---

## 🗺️ Descripción del diagrama de infraestructura (RedExpress)

El mapa de infraestructura modela el sistema de RedExpress dividido en **cinco capas lógicas**:

### Capa 1 — Acceso del Usuario
| Componente | Descripción |
|---|---|
| App móvil (iOS / Android) | Usada por clientes y mensajeros para consultar y actualizar estados de paquetes |
| Plataforma Web | Portal de clientes y operadores para gestión de envíos |
| CDN (CloudFront / Fastly) | Distribuye contenido estático con baja latencia desde nodos geográficamente distribuidos |

### Capa 2 — Borde y Enrutamiento
| Componente | Descripción |
|---|---|
| WAF (Web Application Firewall) | Filtra tráfico malicioso antes de ingresar al sistema |
| Load Balancer (NGINX / HAProxy) | Distribuye tráfico entre instancias de servicios; configurado con health checks |
| API Gateway (Kong / AWS API GW) | Punto de entrada único para todos los microservicios; gestiona autenticación, rate limiting y ruteo |

### Capa 3 — Microservicios
| Servicio | Función |
|---|---|
| Auth Service | Autenticación y gestión de tokens JWT |
| Tracking Service | Consulta y actualización del estado de paquetes en tiempo real |
| Routing Engine | Cálculo y optimización de rutas para mensajeros por zona geográfica |
| Notification Service | Envío de alertas por push, SMS y correo electrónico |
| Customer Service | CRUD de clientes, historial de envíos y perfil |
| Reporting Service | Generación de reportes operativos y de rendimiento |

> **Nota:** Cada microservicio se despliega en contenedores Docker sobre Kubernetes, lo que permite escalar independientemente por zona geográfica.

### Capa 4 — Datos y Mensajería
| Componente | Descripción |
|---|---|
| Kafka (Message Broker) | Cola de eventos para desacoplar actualizaciones de rastreo; garantiza entrega exactly-once |
| Redis (Cache) | Almacena el estado actual de paquetes en tránsito para respuesta en < 50 ms |
| PostgreSQL (DB primaria) | Almacena datos transaccionales (pedidos, clientes, rutas); con réplica de lectura |
| MongoDB (DB de rastreo) | Almacena el historial de eventos de cada paquete en estructura de documento |
| Backup & DR | Snapshots diarios en S3 / Cloud Storage; replicación entre zonas de disponibilidad |

### Capa 5 — Operaciones y Monitoreo
| Componente | Descripción |
|---|---|
| Prometheus + Grafana | Métricas de rendimiento de servicios, latencia, tasa de errores y uso de recursos |
| ELK Stack (Elasticsearch, Logstash, Kibana) | Centralización y análisis de logs distribuidos |
| PagerDuty / Alertmanager | Generación y escalamiento de alertas ante incidentes |
| CI/CD Pipeline (GitHub Actions / Jenkins) | Despliegue continuo y automatizado a producción y ambientes de prueba |

---

## 🔍 Diagnóstico de cuellos de botella y debilidades identificadas

### 1. Punto único de falla — API Gateway
El API Gateway actúa como único punto de entrada. Si no tiene redundancia activa-activa, un fallo en este nodo deja todo el sistema inaccesible.

**Propuesta:** Configurar al menos dos instancias del API Gateway detrás del Load Balancer con failover automático.

---

### 2. Cuello de botella — Base de datos centralizada
En temporadas de alto volumen (Navidad, promociones), la base de datos PostgreSQL puede saturarse con escrituras simultáneas de actualización de estados.

**Propuesta:**
- Agregar réplicas de lectura por región geográfica.
- Implementar CQRS (Command Query Responsibility Segregation) para separar las escrituras de las lecturas.
- Usar Redis como capa de caché para los estados más consultados.

---

### 3. Latencia en rastreo en tiempo real
El Tracking Service requiere baja latencia (< 100 ms) para mostrar el estado actualizado del paquete al usuario. Sin caché ni mensajería asíncrona, cada consulta impacta directamente en la base de datos.

**Propuesta:**
- Kafka para recibir eventos de actualización de forma asíncrona.
- Redis para servir el estado actual sin tocar la base de datos en cada request.

---

### 4. Escalabilidad por zona geográfica
Los servidores regionales procesan rutas de mensajeros de forma centralizada. En ciudades con alta densidad de pedidos, el procesamiento puede presentar cola de espera.

**Propuesta:**
- Desplegar instancias del Routing Engine por región usando Kubernetes con auto-scaling horizontal.
- Enrutar tráfico regional al nodo más cercano mediante DNS geolocalizado.

---

### 5. Falta de observabilidad
Sin un stack de monitoreo unificado, es difícil detectar degradaciones antes de que afecten al usuario final.

**Propuesta:**
- Implementar distributed tracing con Jaeger u OpenTelemetry para rastrear llamadas entre microservicios.
- Definir SLOs (Service Level Objectives) por servicio crítico.

---

## 🧩 Boceto inicial del modelo (descripción textual)

```
[App Móvil]  [Plataforma Web]
       \          /
        [CDN / WAF]
             |
       [Load Balancer]
             |
        [API Gateway]  ←→ [Auth Service]
             |
  ┌──────────┼──────────────┐
  │          │              │
[Tracking] [Routing]  [Notification]
  │          │              │
  └─────────→[Kafka]←───────┘
                  │
            [Redis Cache]
                  │
       ┌──────────┴─────────┐
  [PostgreSQL]          [MongoDB]
   (con réplica)       (eventos)
                  │
        [Prometheus / Grafana]
        [ELK Stack / Alerting]
```

> El diagrama completo en draw.io se encuentra en: `clase/mapa-borrador.drawio`

---

## 🔁 Tareas definidas para complementar el taller

| Tarea asignada | Responsable | Fecha estimada |
|----------------|-------------|----------------|
| Refinar diagrama final en draw.io con todos los nodos y capas | Miguel Ángel Ardila | 17/03 |
| Redacción del informe de diagnóstico técnico | Daniel Moreno | 18/03 |
| Investigación de buenas prácticas (cloud, HA, redundancia) | Sebastián Torres | 18/03 |
| Revisión conjunta y entrega final | Todo el equipo | 19/03 |

---

_Este documento resume el trabajo colaborativo realizado durante la sesión del Taller 4 en el curso de Arquitectura Empresarial (AREM) — Universidad de La Sabana._
