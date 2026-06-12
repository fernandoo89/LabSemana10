# Solución del Laboratorio - Observabilidad (Grafana, Prometheus y Loki)


A continuación se presentan las respuestas detalladas y técnicas a las preguntas de la sección 13 de la guía de laboratorio:

### Pregunta 1: ¿Por qué necesitamos Loki además de Prometheus si ya tenemos `/metrics`?
* **Respuesta:** 
  Prometheus y Loki manejan dos señales de observabilidad distintas y complementarias:
  * **Prometheus (Métricas):** Está optimizado para almacenar **números a lo largo del tiempo** (ej. porcentaje de uso de CPU, cantidad total de peticiones, memoria libre). Es ideal para saber *cuándo* algo anda mal (ej. "el uso de memoria superó el 90%") y para cálculos agregados rápidos, pero no puede decirte el detalle o la causa raíz del evento porque no almacena texto ni datos de alta cardinalidad.
  * **Loki (Logs):** Almacena y procesa **texto estructurado o no estructurado** (los logs/salidas estándar de las aplicaciones). Es ideal para saber el *porqué* ocurrió un error (ej. ver el stack trace exacto de una excepción `NullPointerException`, o saber qué ID de usuario falló en una transacción).
  * **Conclusión:** Con `/metrics` (Prometheus) vemos la salud general del sistema y detectamos anomalías, pero con Loki (Logs) realizamos la investigación profunda del error específico que causó esa anomalía.

---

### Pregunta 2: ¿Qué ventaja aporta que las fuentes de datos (Data Sources) de Grafana estén aprovisionadas como código y no creadas a mano?
* **Respuesta:** 
  El aprovisionamiento de fuentes de datos como código (declaradas en archivos YAML dentro de la carpeta `provisioning/datasources/` de Grafana) ofrece las siguientes ventajas fundamentales:
  * **Reproducibilidad y Consistencia:** Garantiza que cualquier miembro del equipo (o entorno de integración/producción) levante exactamente la misma configuración sin necesidad de pasos manuales propensos a errores humanos.
  * **Control de Versiones:** Los cambios en las URLs de conexión, métodos de autenticación o nombres de las fuentes quedan registrados en el historial de Git, permitiendo auditorías y retornos a versiones estables (rollbacks).
  * **Despliegues Automatizados (CI/CD):** Permite levantar entornos efímeros de prueba desde cero (por ejemplo, con `docker compose up -d`) y que todo funcione inmediatamente, lo cual es clave para el enfoque de **Infraestructura como Código (IaC)**.
  * **Facilidad de Recuperación ante Desastres:** Si el contenedor de Grafana se daña o se elimina, no se pierde la configuración de conexión porque reside en el código y no en la base de datos interna de Grafana.

---

### Pregunta 3: El panel "CPU contenedor" y el panel "CPU host" pueden mostrar valores muy distintos. ¿Por qué? ¿Cuál usarías para alertar sobre una aplicación concreta?
* **Respuesta:** 
  * **¿Por qué difieren?:**
    * La métrica de **CPU del Host** (obtenida mediante `node-exporter`) mide el uso total de procesamiento en toda la máquina real o virtual, el cual incluye al sistema operativo, al propio Docker daemon, procesos de fondo y la suma de **todos** los contenedores ejecutándose en ella.
    * La métrica de **CPU de Contenedor** (obtenida mediante `cAdvisor` para `lab-backend`) mide únicamente el porcentaje de tiempo de CPU consumido por los hilos de ejecución de ese contenedor específico. Por ejemplo, en una máquina de 4 núcleos (400% de capacidad total), un contenedor puede estar saturado al 100% de un hilo, mientras que la CPU global del host solo reporta un 25% de uso.
  * **¿Cuál usar para alertar sobre una aplicación concreta?:**
    * Se debe utilizar el panel de **CPU del contenedor** (ej. filtrado por `name="lab-backend"`). Alertar usando la CPU del host causaría falsos positivos (la alarma se dispara porque otra aplicación ajena saturó el servidor) o falsos negativos (nuestra aplicación específica se colgó por falta de recursos, pero la alarma nunca sonó porque el host general tenía mucha CPU libre).

---

### Pregunta 4: ¿Qué diferencia hay entre el *evaluation interval* y el *pending period* de una alarma?
* **Respuesta:** 
  * **Evaluation Interval (Intervalo de Evaluación):** Es la frecuencia con la que el motor de alertas de Grafana ejecuta la consulta (PromQL/LogQL) para comprobar si se cumple la condición de alerta (ej. si está configurado en `10s`, Grafana consultará a Prometheus cada 10 segundos para ver si el uso de CPU superó el 50%).
  * **Pending Period (Período de Espera / Pendiente):** Es el tiempo de gracia que debe mantenerse la condición en estado de incumplimiento antes de que la alerta pase formalmente al estado de **Firing** (Disparada) y envíe notificaciones.
    * *Ejemplo:* Si el límite es 50% de CPU, el intervalo de evaluación es `10s` y el pending period es `30s`: cuando la CPU sube a 55%, la alerta entra en estado `Pending`. Si en las siguientes evaluaciones de los 30 segundos la CPU se mantiene arriba del 50%, la alerta se dispara (`Firing`). Si baja a 40% antes de que pasen los 30 segundos, la alerta vuelve a `Normal` sin haber molestado al equipo, evitando alertas por picos breves o transitorios de carga.
