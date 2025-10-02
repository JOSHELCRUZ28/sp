*NOMBRE ALUMNO: JORGE JOSHEL LEON CRUZ*

*NOMBRE DEL MAESTRO: RENE SOLIS REYES*

*MATERIA: SISTEMAS PROGRAMALES*

*HORARIO: 3:00PM A 4:00PM*

---

# Impacto de 5G en la transmisión de datos IoT usando MQTT: oportunidades y desafíos

---

## Resumen ejecutivo

La llegada de *5G* transforma la conectividad IoT ofreciendo mayor capacidad de dispositivos, baja latencia y nuevas capacidades de red (p. ej. network slicing y MEC). Esto abre oportunidades para usar *MQTT* en escenarios de mayor exigencia (control industrial en tiempo real, V2X, telemedicina), pero también introduce desafíos técnicos y de seguridad: orquestación entre dominios, garantías de aislamiento en slices, handovers y adaptación de QoS, y cambios en arquitectura (más borde/MEC). Las recomendaciones clave incluyen adoptar MQTT 5.0 y sus propiedades, despliegues con brokers en el borde, estrategias de QoS+idempotencia, y mecanismos de seguridad alineados con 5G (mTLS, JWT, claves rotativas).

---

## 1. Contexto: ¿por qué 5G importa para IoT?

5G no es solo “más velocidad”: define tres categorías de servicio (eMBB, URLLC, mMTC) pensadas para distintos tipos de IoT — desde banda ancha mejorada hasta comunicaciones ultra-fiables y conectividad masiva. Estas capacidades permiten casos de uso donde MQTT, por su modelo pub/sub ligero, puede encajar muy bien si se alinean QoS y arquitecturas de red.

---

## 2. Breve repaso técnico — 5G y MQTT

* *5G*: tecnologías clave (sub-6 GHz y mmWave, Massive MIMO, beamforming), capacidades de servicio (eMBB, URLLC, mMTC), y funciones de red (network slicing, MEC/edge).
* *MQTT*: protocolo pub/sub ligero popular en IoT. MQTT 5.0 aporta propiedades útiles (Message Expiry, user properties, reason codes) que facilitan interoperabilidad con capacidades de la red 5G.

---

## 3. Oportunidades (cómo 5G potencia a MQTT en IoT)

### 3.1 Baja latencia y fiabilidad (URLLC)

* Permite aplicaciones MQTT con requisitos de latencia estricta (control industrial, teleoperación, V2X) si se combinan QoS adecuados y colocación del broker cerca del borde (MEC). Esto reduce el RTT y mejora la capacidad de respuesta de flujos MQTT críticos.

### 3.2 Escalabilidad masiva (mMTC)

* 5G está diseñada para millones de dispositivos por km², facilitando despliegues IoT masivos (sensores ciudad, telemetría). MQTT es eficiente en ancho de banda y puede aprovechar esa densidad cuando el diseño de tópicos, QoS y control de sesión están bien pensados.

### 3.3 Network slicing — tráfico con SLAs diferenciados

* Slicing permite garantizar recursos y SLAs por tipo de servicio (p. ej. slice URLLC para control, slice mMTC para telemetría). MQTT puede mapear tópicos o clases de servicio a slices dedicados, obteniendo mejores garantías para mensajes críticos.

### 3.4 Edge computing (MEC) y brokers en el borde

* Llevar brokers MQTT y lógica de ingestión al MEC reduce latencia y tráfico troncal, permite pre-procesado y políticas locales (filtrado, deduplicación, HMAC checks) y mejora la resiliencia ante desconexiones móviles.

---

## 4. Desafíos (lo que hay que resolver)

### 4.1 Seguridad y superficie de ataque ampliada

* 5G introduce nuevas APIs (operador/IoT platforms) y slicing que, si no están bien implementadas, pueden exponer vectores de ataque (aplicaciones inseguras, aislamiento imperfecto entre slices). Estudios han mostrado APIs vulnerables en plataformas de operadoras. Es crítico aplicar controles (mTLS, autenticación fuerte, RBAC por tópico y validación de payload).

### 4.2 Orquestación y correspondencia QoS ↔ SLA

* Garantizar que la semántica de QoS de MQTT (0/1/2) se traduzca en SLAs de la red (slice con latencia o pérdida limitada) no es trivial. Requiere coordinación entre la capa de aplicación (broker/configuración) y el proveedor 5G para mapear prioridades y recursos.

### 4.3 Movilidad, handovers y consistencia de sesión

* Dispositivos móviles (vehículos, drones) cambian celdas y slices; mantener sesiones MQTT estables y evitar publicaciones duplicadas o pérdida temporal exige estrategias: brokers en borde con re-anchoring, reconexiones rápidas, LWT bien diseñadas y uso de clean session según caso.

### 4.4 Gestión de energía en dispositivos masivos

* Aunque 5G soporta mMTC, modos de eficiencia energética (e.g., NB-IoT/RedCap) y la gestión de radio pueden afectar latencia y ventana de transmisión; el diseño MQTT debe considerar ventanas de awake, batching y tamaño de payload para ahorrar energía.

### 4.5 Interoperabilidad y heterogeneidad

* Diferentes operadores, vendors y entornos (públicos/privados/industrial) significan diferentes prestaciones reales de 5G; la solución MQTT debe ser tolerante a variaciones (QoS adaptable, buffering, expiración de mensajes).

---

## 5. Buenas prácticas recomendadas (arquitectura y operaciones)

1. *MQTT 5.0 + Propiedades*: usar Message Expiry, User Properties y Reason Codes para controlar vigencia y meta-datos que ayuden a la orquestación con la red.
2. *Brokers en el borde (MEC)*: desplegar brokers regionales/edge para casos URLLC y usar replicación/colocación para coherencia global.
3. *Mapear clases de mensajes a slices*: acordar con el operador cómo mapear tópicos/clases de QoS a slices con SLAs correspondientes. Documentar prioridades y fallbacks.
4. *Seguridad multicapa*: mTLS entre clientes y brokers, autorización por tópico (ACL), validación de payload (HMAC o firmas), y rotación/gestión de credenciales integrada con el plano 5G. Monitorizar APIs expuestas por operadores.
5. *Idempotencia y deduplicación*: incluir message_id y seq en payload; consumidores deben mantener store de IDs con TTL para evitar efectos de duplicados durante handovers o re-publicaciones.
6. *Adaptación de QoS dinámica*: si la red indica degradación, degradar QoS o batch/pipeline los mensajes no críticos para mantener disponibilidad.
7. *Pruebas en condiciones reales*: validar en celdas 5G reales con handovers, slices, y tráfico mixto; medir latencia, Jitter, pérdida, y comportamiento en reconexiones.

---

## 6. Diseño de ejemplo (arquitectura propuesta)

* *Dispositivos* (sensors/actuators) → conexión 5G (SIM/Slice asignado) → *Edge Broker (MEC)* con TLS y validación HMAC → replicación segura (VPN / service mesh) → *Central Broker / Cloud* para almacenamiento y análisis.
* Mapping: tópicos críticos → slice URLLC + edge broker local; tópicos telemétricos masivos → slice mMTC (batching, QoS 0/1 según tolerancia).

---

## 7. Plan de pruebas y métricas a medir

* *Latencia (P95, P99)* de publicación → confirmación (end-to-end).
* *Disponibilidad / reconexión*: tiempo hasta re-estabelecer sesión tras handover.
* *Tasa de duplicados* bajo reconexiones y retrials.
* *Throughput* en escenarios masivos (nº de dispositivos por km² y mensajes/s).
* *Seguridad*: pruebas de fuzzing en APIs de operadora, tests de aislamiento de slices.

---

## 8. Checklist resumido para despliegue MQTT sobre 5G

* [ ] Evaluar SLA y tipos de slice requeridos (URLLC/mMTC/eMBB).
* [ ] Establecer arquitectura edge (MEC) con brokers replicados.
* [ ] Adoptar MQTT 5.0 y propiedades (Message Expiry, user properties).
* [ ] Definir mapeo tópico → slice → QoS.
* [ ] Implementar mTLS + ACL + HMAC/firmas para payloads críticos.
* [ ] Diseñar idempotencia con message_id`/seq` y store TTL.
* [ ] Probar handovers, degradación de red y reconexiones reales.
* [ ] Monitorizar métricas y auditar APIs 5G operadoras.

---

## 9. Conclusión

5G ofrece una plataforma poderosa para llevar MQTT a nuevos casos de uso IoT de alta exigencia (baja latencia, densidad masiva, slices con SLAs). No obstante, las ventajas son reales solo si se orquesta correctamente la interacción entre capa de aplicación (MQTT, brokers, seguridad) y capa de red (slicing, MEC, SLAs). Prestar atención a seguridad, gestión de sesiones y pruebas en condiciones reales es esencial para desplegar soluciones robustas.

---

## Bibliografía (APA — selección)

* 3GPP. (2022). 5G System Overview. 3GPP. [https://www.3gpp.org/technologies/5g-system-overview](https://www.3gpp.org/technologies/5g-system-overview).
* Reiher, L., et al. (2022). A Novel MQTT-based Interface Evaluated in a 5G Case Study (preprint). arXiv. [https://arxiv.org/pdf/2209.03630](https://arxiv.org/pdf/2209.03630).
* Dias, J., et al. (2025). 5G Network Slicing: Security Challenges, Attack Vectors and Defenses. Sensors. [https://www.mdpi.com/1424-8220/25/13/3940](https://www.mdpi.com/1424-8220/25/13/3940).
* Wired. (2022). One of 5G's Biggest Features Is a Security Minefield (report on carrier APIs vulnerabilities). Wired. [https://www.wired.com/story/5g-api-flaws](https://www.wired.com/story/5g-api-flaws).
* Cavli Wireless. (s. f.). Introduction of 5G Technology And Its Impact on IoT. [https://www.cavliwireless.com/blog/nerdiest-of-things/5g-technology-impact-on-iot](https://www.cavliwireless.com/blog/nerdiest-of-things/5g-technology-impact-on-iot).
* ETSI. (s. f.). 5G technologies. [https://www.etsi.org/technologies/5g](https://www.etsi.org/technologies/5g).
* Longo, E., et al. (2023). Design and implementation of an advanced MQTT broker for distributed systems. Journal / Elsevier. (artículo sobre arquitecturas de broker).
