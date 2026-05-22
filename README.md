# CreacionYVisualizaciondeKPIs
Tarea
# INFORME DE PROYECTO: MIS PARA GESTIÓN DE CITAS MÉDICAS
## Diseño, Implementación e Identificación de KPIs
**Integrantes del Grupo:** Aguilera Alan, Flores Juan

---

## 2. Mapeo de Atributos y Boceto de Interfaz
### 2.1 Mapeo en el Esquema de Base de Datos
A continuación, se documentan las entidades y atributos extraídos de las tablas de texto plano (TXT delimitado por punto y coma) del TPS (Sistema de Gestión de Citas Médicas) necesarios para la construcción del cuadro de mando gerencial.

| Archivo TXT | Atributos / Columnas | Propósito en el MIS |
| :--- | :--- | :--- |
| `citas.txt` | id_cita, cedula_paciente, id_medico, fecha, hora, estado de la cita (0:cancelada, 1:Activa, 2:Atendida), diagnostico | Visualización de tasa de cancelaciones y volumen temporal. |
| `medicos.txt` | id_medico, nombre_medico, especialidad, horario | Visualización y agrupación de la demanda por especialidad. |
| `pacientes.txt` | nombre_paciente, cedula_paciente, edad, telefono, correo | Análisis demográfico de pacientes atendidos. |
| `usuarios.txt` | usuario, contraseña, rol, (cedula del paciente o ID del médico) | Contabilización de usuarios. |

**Relaciones entre entidades:**
* La tabla `citas.txt` actúa como el núcleo transaccional conectándose con `medicos.txt` mediante el atributo `id_medico`.
* La tabla `citas.txt` se conecta con `pacientes.txt` mediante la `cedula_paciente`.

### 2.2 Bocetado Visual (Mockup)

<img width="2560" height="4585" alt="Medical Center Appointment Operations Dashboard" src="https://github.com/user-attachments/assets/230257d9-4dcc-4d25-9731-be4b3b4ed316" />


**Justificación Técnica de las Visualizaciones Seleccionadas:**
* **KPI 1: Tasa de Citas Canceladas y Pendientes:** Se justifica el uso de un gráfico de dona para observar rápidamente la proporción de citas que están en estado 'Cancelada' frente a las 'Pendientes' o completadas.
* **KPI 2: Volumen de Citas por Especialidad Médica:** Se justifica el uso de un gráfico de barras verticales cruzando para facilitar la comparación de la demanda asistencial entre diferentes categorías.
* **KPI 3: Distribución Demográfica de Pacientes (Edad):** Se justifica un histograma utilizando los datos de edad extraídos de `pacientes.txt` para comprender qué grupos generan la mayor demanda en el sistema clínico.

---

## 3. Construcción e Implementación de los KPIs
**Código Fuente de la Implementación (Python / Pandas):**
```python
# Espacio reservado para el código fuente según el documento original
```

---

## 4. Validación de Atributos de Información e Interpretación Gerencial
### 4.1 Matriz de Atributos de Valor
Evaluación analítica crítica del comportamiento de las consultas y gráficos generados para asegurar su aplicabilidad ejecutiva.

| Característica | Pregunta de Validación | Análisis y Cumplimiento en el Sistema Clínico |
| :--- | :--- | :--- |
| **Simplicidad** | ¿El gráfico evita la sobrecarga de información y destaca lo realmente importante? | [Explicar cómo el gráfico procesa los registros brutos de los TXT mostrando únicamente tendencias claras, como la especialidad más solicitada] |
| **Oportunidad (Timeliness)** | ¿La consulta procesa los datos con la velocidad requerida para que el administrador actúe a tiempo? | [Detallar cómo la lectura rápida de los archivos delimitados por ';' en Pandas permite generar el reporte sin retrasos] |
| **Accesibilidad** | ¿El formato del reporte permite que un usuario no técnico interprete el resultado de un solo vistazo? | [Justificar el uso de etiquetas legibles en español y la omisión de IDs técnicos (como M-001) en las gráficas a favor de nombres legibles como 'Cardiología'] |

### 4.2 Documentación de Interpretación Gerencial
Definición analítica de las directrices estratégicas de respuesta ante los umbrales de cada indicador clave basado en los datos del sistema:

* **KPI 1: Estado Operativo de Citas (Canceladas vs. Pendientes)**
  * "Si este indicador clave de rendimiento muestra un comportamiento negativo (por ejemplo, el volumen de citas 'Canceladas' supera el 20% del total agendado), la acción correctiva o estratégica que la gerencia debe ejecutar de inmediato es implementar penalizaciones por inasistencia sin aviso previo o ajustar la sobreoferta de turnos en los horarios de mayor deserción."
* **KPI 2: Volumen de Citas por Especialidad (Pediatría, Cardiología, etc.)**
  * "Si este indicador clave de rendimiento muestra un comportamiento fuera de umbral (por ejemplo, una concentración atípica en Psiquiatría o Cardiología), la acción correctiva o estratégica que la gerencia debe ejecutar de inmediato es la apertura de nuevos cupos en los horarios de los médicos especialistas afectados (ej. extendiendo el horario de 14:00-20:00 a turnos matutinos)."
* **KPI 3: Perfil Demográfico de Pacientes (Edad)**
  * "Si este indicador clave de rendimiento muestra una predominancia significativa de pacientes jóvenes (por ejemplo, menores de 25 años), la acción correctiva o estratégica que la gerencia debe ejecutar es alinear los servicios de marketing y médicos (como Pediatría o Medicina General) para enfocar campañas preventivas o digitalizar canales de atención más acordes a este segmento poblacional."
