# 🛠️ Taller 4: Mapa de Infraestructura y Diagnóstico Técnico

## 🎯 Objetivo

Construir el mapa lógico y/o físico de la infraestructura tecnológica del sistema base (RedExpress) y realizar un diagnóstico de debilidades, cuellos de botella y oportunidades de mejora.

---

## 🚚 Caso base de referencia: RedExpress (Plataforma de Logística)

RedExpress opera una infraestructura híbrida que combina servidores regionales, servicios en la nube, centros de distribución físicos y dispositivos móviles usados por sus mensajeros. La plataforma digital debe garantizar **alta disponibilidad y rendimiento**, especialmente durante campañas promocionales o temporadas de alto volumen como Navidad.

**Contexto del sistema:**
- RedExpress gestiona paquetes y rastreo de envíos a través de una app móvil y una plataforma web.
- La infraestructura incluye servicios desplegados en la nube, servidores regionales para procesamiento de rutas y una base de datos centralizada.

**Componentes de infraestructura a modelar:**

| Componente | Descripción |
|---|---|
| API Gateway | Punto de entrada unificado para todos los microservicios |
| Balanceadores de carga | Distribución de tráfico entre instancias de servicios |
| Base de datos distribuida | Almacenamiento transaccional con réplicas de lectura por región |
| Cola de mensajes (Kafka) | Desacoplamiento del servicio de rastreo en tiempo real |
| Caché distribuida (Redis) | Reducción de latencia en consultas frecuentes de estado |
| Servicios de monitoreo y alertas | Prometheus, Grafana, ELK Stack y alerting |
| Módulos de procesamiento de rutas | Routing Engine por zona geográfica con auto-scaling |
| Módulo de estados de paquetes | Tracking Service con persistencia en MongoDB |

**Áreas críticas a diagnosticar:**

| Área | Riesgo |
|---|---|
| Latencia en rastreo en tiempo real | Consultas directas a DB sin caché pueden superar los 200 ms |
| Punto único de falla (SPOF) | API Gateway y base de datos sin redundancia activa |
| Escalabilidad por zonas geográficas | Routing Engine centralizado satura en picos de demanda |

---

## 🧪 Trabajo en Clase

Durante la sesión se espera que el equipo:

1. **Modele el mapa de infraestructura** de RedExpress, diferenciando claramente las capas: acceso, borde, microservicios, datos y operaciones.
2. **Identifique los nodos críticos** del sistema: puntos de alta carga, posibles SPOFs y servicios sin redundancia.
3. **Diagnostique cuellos de botella** en base a el contexto del caso (temporadas de alto volumen, rastreo en tiempo real, escalabilidad regional).
4. **Proponga mejoras arquitectónicas** para cada debilidad identificada (ej. réplicas de lectura, caché, desacoplamiento con Kafka).
5. **Documente el trabajo** en `clase/notas.md` con las decisiones tomadas, el boceto del diagrama y la distribución de tareas.
6. **Reciba retroalimentación** en vivo del docente sobre el modelo propuesto.

**Herramientas recomendadas:** draw.io, Excalidraw, papel/pizarra, Astah.

---

## 📁 Estructura del repositorio

```
Taller4-arqEmp/
├── README.md                    ← Este archivo
├── clase/
│   ├── mapa-borrador.drawio     ← Diagrama elaborado en clase
│   └── notas.md                 ← Registro de actividades y decisiones de la sesión
└── entrega/
    ├── mapa-final.drawio        ← Diagrama final del cliente real
    ├── informe_taller.md        ← Informe técnico de diagnóstico
    └── referencias.md           ← Fuentes y bibliografía
```

---

## 📊 Rúbrica de Evaluación — Trabajo en Clase

| Criterio | Excelente (5) | Aceptable (3) | Insuficiente (1–2) |
|---|---|---|---|
| **Mapa de infraestructura (RedExpress)** | Representa claramente todas las capas: acceso, borde, microservicios, datos y operaciones, con nodos bien identificados | Incluye los componentes principales pero omite capas o no diferencia niveles | Diagrama incompleto, sin estructura de capas o con errores conceptuales |
| **Identificación de nodos críticos** | Se identifican SPOFs, zonas de alta carga y servicios sin redundancia con justificación técnica | Se mencionan algunos nodos críticos sin argumentación suficiente | No se identifican riesgos o la selección no tiene fundamento |
| **Diagnóstico de cuellos de botella** | Se detectan y explican los cuellos de botella del sistema (latencia, escalabilidad, disponibilidad) con evidencia del caso | Se mencionan problemas pero sin análisis técnico detallado | Diagnóstico ausente o superficial |
| **Propuestas de mejora** | Se proponen soluciones concretas y viables para cada debilidad identificada (caché, réplicas, Kafka, auto-scaling, etc.) | Se sugieren mejoras generales sin profundidad técnica | No hay propuestas o son inviables en el contexto |
| **Documentación en clase** | `notas.md` refleja fielmente las decisiones tomadas, el boceto, el análisis y la distribución de tareas del equipo | Notas parciales, con algunos elementos faltantes | Notas ausentes o que no corresponden al trabajo realizado |

---

## ✅ Licencia

Este taller hace parte del curso de Arquitectura Empresarial — Universidad de La Sabana. Uso académico bajo licencia MIT.
