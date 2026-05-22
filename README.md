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
# import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Configuración básica de estilo para los reportes
sns.set_theme(style="whitegrid")
path = "/content/"


# 1. CARGA DE DATOS (Archivos del TPS en C)
# Cargar citas - Forzamos strings en los IDs para evitar que borre ceros a la izquierda
cols_citas = ['id_cita', 'cedula_paciente', 'id_medico', 'fecha', 'hora', 'estado_cita', 'diagnostico']
df_citas = pd.read_csv(f"{path}citas.txt", sep=';', names=cols_citas, header=None, dtype=str)

# Traducimos los números del TPS a texto claro para el reporte gerencial
diccionario_estados = {'0': 'Cancelada', '1': 'Activa', '2': 'Atendida'}
df_citas['estado_desc'] = df_citas['estado_cita'].map(diccionario_estados)

# Cargar médicos
cols_medicos = ['id_medico', 'nombre_medico', 'especialidad', 'horario']
df_medicos = pd.read_csv(f"{path}medicos.txt", sep=';', names=cols_medicos, header=None, dtype=str)

# Cargar pacientes para graficar el histograma
cols_pacientes = ['nombre_paciente', 'cedula_paciente', 'edad', 'telefono', 'correo']
df_pacientes = pd.read_csv(f"{path}pacientes.txt", sep=';', names=cols_pacientes, header=None, dtype=str)
df_pacientes['edad'] = pd.to_numeric(df_pacientes['edad'], errors='coerce')



# 2. GENERACIÓN DEL DASHBOARD (Capa Visual MIS)
# Creamos el lienzo con 3 subplots 
fig, axes = plt.subplots(1, 3, figsize=(19, 6))
fig.suptitle('Dashboard de Gestión Hospitalaria - Indicadores Clave (MIS)', fontsize=16, fontweight='bold', y=1.02)

# KPI 1: Distribución y Estado de las Citas 
citas_por_estado = df_citas['estado_desc'].value_counts()
colores_pie = ['#2ca02c', '#d62728', '#1f77b4'] # Verde (Atendida), Rojo (Cancelada), Azul (Activa)

axes[0].pie(citas_por_estado, labels=citas_por_estado.index, autopct='%1.1f%%', startangle=140, colors=colores_pie)
axes[0].set_title('KPI 1: Estado Actual de Citas', fontsize=12, fontweight='bold')

# KPI 2: Demanda de Servicios por Especialidad
# Cruzamos las citas con la tabla de médicos para obtener el campo especialidad
df_citas_medicos = pd.merge(df_citas, df_medicos, on='id_medico', how='inner')
demanda_esp = df_citas_medicos['especialidad'].value_counts().reset_index()
demanda_esp.columns = ['Especialidad', 'Total Citas']

# Graficamos usando un color fijo para evitar advertencias de la librería
sns.barplot(data=demanda_esp, x='Total Citas', y='Especialidad', ax=axes[1], color='#4682B4')
axes[1].set_title('KPI 2: Demanda por Especialidad Médica', fontsize=12, fontweight='bold')
axes[1].set_xlabel('Número de Citas Registradas')
axes[1].set_ylabel('') 

# KPI 3: Perfil de Edad de Pacientes Atendidos
# Combinamos citas con pacientes y filtramos únicamente los casos con éxito ('Atendida')
df_citas_pacientes = pd.merge(df_citas, df_pacientes, on='cedula_paciente', how='inner')
pacientes_atendidos = df_citas_pacientes[df_citas_pacientes['estado_cita'] == '2']

sns.histplot(data=pacientes_atendidos, x='edad', bins=12, kde=True, ax=axes[2], color='#8a2be2')
axes[2].set_title('KPI 3: Rango de Edad (Pacientes Atendidos)', fontsize=12, fontweight='bold')
axes[2].set_xlabel('Edad')
axes[2].set_ylabel('Cantidad de Pacientes')

# Ajustes finales de empaquetado para que no se encima el texto
plt.tight_layout()
plt.show()
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
