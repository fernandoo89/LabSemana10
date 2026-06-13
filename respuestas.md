# Respuestas Resumidas – Laboratorio de Observabilidad

### Pregunta 1: ¿Por qué necesitamos Loki además de Prometheus si ya tenemos `/metrics`?
* **Prometheus (Métricas):** Registra números a lo largo del tiempo (ej. % CPU, RAM). Sirve para saber **cuándo** hay un problema general de rendimiento, pero no los detalles.
* **Loki (Logs):** Almacena el texto detallado de las aplicaciones (ej. stack traces, errores específicos). Sirve para saber el **porqué** (causa raíz) del problema.
* **Conclusión:** Prometheus detecta la anomalía y Loki ayuda a investigar la causa exacta.

---

### Pregunta 2: ¿Qué ventaja aporta que las fuentes de datos (Data Sources) de Grafana estén aprovisionadas como código y no creadas a mano?
* **Reproducibilidad:** Asegura que cualquier desarrollador o entorno (testing/producción) levante exactamente la misma configuración.
* **Control de Versiones:** Permite registrar y revertir cambios en las conexiones a través del historial de Git.
* **Automatización y Recuperación:** Si el contenedor se borra o daña, las conexiones se auto-configuran al iniciar el stack sin necesidad de clics manuales.

---

### Pregunta 3: El panel "CPU contenedor" y el panel "CPU host" pueden mostrar valores muy distintos. ¿Por qué? ¿Cuál usarías para alertar sobre una aplicación concreta?
* **¿Por qué difieren?:** La CPU del Host mide el consumo total de toda la máquina (OS + Docker + todos los contenedores). La CPU del contenedor mide únicamente los recursos consumidos por ese proceso específico (ej. `lab-backend`).
* **¿Cuál usar?:** Se debe usar **CPU de contenedor**. Alertar con la del host generaría falsos positivos (alarmas por otras aplicaciones) o falsos negativos (si tu app falla pero la máquina tiene CPU de sobra).

---

### Pregunta 4: ¿Qué diferencia hay entre el *evaluation interval* y el *pending period* de una alarma?
* **Evaluation Interval:** Es la frecuencia con la que Grafana ejecuta la consulta (ej. cada `10s` consulta a Prometheus).
* **Pending Period:** Es el tiempo que debe mantenerse la condición en falla para que la alerta pase formalmente a disparada (ej. esperar `30s` para evitar falsas alarmas por picos transitorios).
