# CreacionYVisualizaciondeKPIs
Tarea
# INFORME DE PROYECTO: MIS PARA GESTIÓN DE CITAS MÉDICAS
## Diseño, Implementación e Identificación de KPIs
**Integrantes del Grupo:** Aguilera Alan, Flores Juan

---

## 2. Mapeo de Atributos y Boceto de Interfaz
### 2.1 Mapeo en el Esquema de Base de Datos
Se documentan las entidades y atributos extraídos de las tablas de texto plano (.txt delimitado por punto y coma) del TPS (Sistema de Gestión de Citas Médicas).

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
* **Gráfico de Pastel (Pie Chart) — KPI 1 (Tasa de Ausentismo y Estado):** Seleccionado específicamente porque la variable categórica de estado tiene únicamente tres clases mutuamente excluyentes (Atendida, Pendiente, Cancelada / Ausente). Esto permite a la gerencia percibir de inmediato la relación de las partes respecto al flujo total operativo (100% de las citas), apoyado por la renderización de porcentajes explícitos (`autopct`).
* **Gráficos de Barras Horizontales (Horizontal Barplots) — KPIs 2, 4, 6 y 8:** Se optó por la orientación horizontal de Seaborn para manejar adecuadamente variables categóricas con etiquetas de texto largas (nombres de especialidades, nombres de médicos y descripciones de diagnósticos). Esta disposición evita el solapamiento en el eje, mejora drásticamente la legibilidad en pantallas o proyecciones y permite jerarquizar los datos de mayor a menor (Top N) para identificar patrones de volumen al instante.
* **Histograma con Estimación de Densidad Kernel (KDE) — KPI 3 (Perfil Demográfico):** Representa la mejor herramienta estadística para analizar el comportamiento de una variable numérica continua como la edad. La subdivisión paramétrica en 15 contenedores (*bins*) agrupa a los pacientes por rangos etarios, mientras que la superposición de la curva KDE suaviza el ruido transaccional para revelar la verdadera función de densidad de probabilidad y la asimetría de nuestra población de usuarios.
* **Gráfico de Líneas con Marcadores (Lineplot) — KPI 5 (Demanda por Franja Horaria):** Implementado por ser el estándar analítico para mostrar la evolución de variables a través de un eje temporal secuencial. La adición de marcadores circulares explícitos (`marker='o'`) ancla la atención en los puntos de inflexión exactos, facilitando la detección táctica de picos de congestión (cuellos de botella) u horas de capacidad ociosa.
* **Gráfico de Barras Verticales (Vertical Barplot) — KPI 7 (Volumen Mensual):** Escogido para mapear una serie temporal discreta que posee etiquetas alfanuméricas cortas (Ene, Feb, Mar...). Al posicionar la cronología en el eje X, la lectura natural de izquierda a derecha ayuda a los *stakeholders* a evaluar la magnitud estacional del negocio mes a mes sin distracciones visuales.

---

## 3. Construcción e Implementación de los KPIs
**Código Fuente de la Implementación (Python / Pandas):**
```python
import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from streamlit_autorefresh import st_autorefresh

# --- CONFIGURACIÓN DEL DASHBOARD INTERACTIVO ---
# Configuramos la página para que sea ancha y quepan bien tus gráficas
st.set_page_config(page_title="MIS Hospitalario", layout="wide")

# Actualización automática cada 30 segundos (30000 milisegundos)
st_autorefresh(interval=30000, key="datarefresh")

st.title(" Dashboard Analítico de Gestión Hospitalaria (MIS)")
st.markdown("Este panel se actualiza en tiempo semi-real (Falso real) al registrar datos en el sistema TPS (ZinjaI).")

# Configuración estética de Seaborn 
sns.set_theme(style="whitegrid", context="paper")
plt.rcParams['font.sans-serif'] = 'DejaVu Sans'

# Ruta local (vacía para que busque en la misma carpeta del script)
path = ""

try:
    # --- 1. CARGA Y TRANSFORMACIÓN DE DATOS (ETL) ---
    cols_citas = ['id_cita', 'cedula_paciente', 'id_medico', 'fecha', 'hora', 'estado_cita', 'diagnostico']
    df_citas = pd.read_csv(f"{path}citas.txt", sep=';', names=cols_citas, header=None, dtype=str)
    diccionario_estados = {'0': 'Cancelada / Ausente', '1': 'Pendiente', '2': 'Atendida'}
    df_citas['estado_desc'] = df_citas['estado_cita'].map(diccionario_estados)

    cols_medicos = ['id_medico', 'nombre_medico', 'especialidad', 'horario']
    df_medicos = pd.read_csv(f"{path}medicos.txt", sep=';', names=cols_medicos, header=None, dtype=str)

    cols_pacientes = ['nombre_paciente', 'cedula_paciente', 'edad', 'telefono', 'correo']
    df_pacientes = pd.read_csv(f"{path}pacientes.txt", sep=';', names=cols_pacientes, header=None, dtype=str)
    df_pacientes['edad'] = pd.to_numeric(df_pacientes['edad'], errors='coerce')

    # Cruces relacionales
    df_citas_medicos = pd.merge(df_citas, df_medicos, on='id_medico', how='inner')

    # Procesamiento de fechas para los nuevos KPIs
    df_citas['fecha_dt'] = pd.to_datetime(df_citas['fecha'], format='%d/%m/%Y', errors='coerce')
    df_citas['mes'] = df_citas['fecha_dt'].dt.month

    # --- 2. GENERACIÓN DEL DASHBOARD (8 KPIs) ---
    fig, axes = plt.subplots(4, 2, figsize=(18, 24))
    fig.suptitle('Análisis Operativo y Estratégico', fontsize=22, fontweight='bold', color='#1A365D', y=0.99)

    # [ KPI 1: Tasa de Ausentismo y Estado ]
    citas_estado = df_citas['estado_desc'].value_counts()
    color_map = {'Atendida': '#2E8B57', 'Pendiente': '#4682B4', 'Cancelada / Ausente': '#E67E22'}
    axes[0, 0].pie(citas_estado, labels=citas_estado.index, autopct='%1.1f%%', startangle=140, 
                   colors=[color_map.get(x, '#333') for x in citas_estado.index], wedgeprops={'linewidth': 1.5, 'edgecolor': 'white'})
    axes[0, 0].set_title('KPI 1: Tasa de Ausentismo y Estado General', fontsize=14, fontweight='bold')

    # [ KPI 2: Volumen de Citas por Especialidad ]
    demanda_esp = df_citas_medicos['especialidad'].value_counts().reset_index()
    demanda_esp.columns = ['Especialidad', 'Total']
    sns.barplot(data=demanda_esp, x='Total', y='Especialidad', ax=axes[0, 1], color='#3182CE')
    axes[0, 1].set_title('KPI 2: Demanda por Especialidad Médica', fontsize=14, fontweight='bold')
    axes[0, 1].set_xlabel(''); axes[0, 1].set_ylabel('')

    # [ KPI 3: Distribución Demográfica ]
    sns.histplot(data=df_pacientes, x='edad', bins=15, kde=True, ax=axes[1, 0], color='#2B6CB0')
    axes[1, 0].set_title('KPI 3: Perfil Demográfico (Edad)', fontsize=14, fontweight='bold')
    axes[1, 0].set_xlabel('Edad (Años)'); axes[1, 0].set_ylabel('Pacientes Registrados')

    # [ KPI 4: Productividad Médica ]
    df_atendidas = df_citas_medicos[df_citas_medicos['estado_cita'] == '2']
    if not df_atendidas.empty:
        prod_med = df_atendidas['nombre_medico'].value_counts().head(8).reset_index()
        prod_med.columns = ['Médico', 'Total']
        sns.barplot(data=prod_med, x='Total', y='Médico', ax=axes[1, 1], color='#4299E1')
    axes[1, 1].set_title('KPI 4: Top 8 - Productividad por Médico (Atendidas)', fontsize=14, fontweight='bold')
    axes[1, 1].set_xlabel(''); axes[1, 1].set_ylabel('')

    # [ KPI 5: Demanda por Franja Horaria ]
    demanda_hora = df_citas['hora'].value_counts().sort_index().reset_index()
    demanda_hora.columns = ['Hora', 'Total']
    sns.lineplot(data=demanda_hora, x='Hora', y='Total', ax=axes[2, 0], marker='o', markersize=8, color='#E53E3E', linewidth=2.5)
    axes[2, 0].set_title('KPI 5: Demanda Operativa por Franja Horaria', fontsize=14, fontweight='bold')
    axes[2, 0].tick_params(axis='x', rotation=45)
    axes[2, 0].set_xlabel('Hora del Día'); axes[2, 0].set_ylabel('Citas Programadas')

    # [ KPI 6: Top Diagnósticos Frecuentes ]
    df_diag = df_citas[df_citas['diagnostico'] != 'Pendiente']['diagnostico'].value_counts().head(6).reset_index()
    df_diag.columns = ['Diagnóstico', 'Total']
    sns.barplot(data=df_diag, x='Total', y='Diagnóstico', ax=axes[2, 1], color='#805AD5')
    axes[2, 1].set_title('KPI 6: Top 6 - Diagnósticos Clínicos Frecuentes', fontsize=14, fontweight='bold')
    axes[2, 1].set_xlabel(''); axes[2, 1].set_ylabel('')

    # [ KPI 7: Tendencia Mensual (2026) ]
    meses_nombres = {1:'Ene', 2:'Feb', 3:'Mar', 4:'Abr', 5:'May', 6:'Jun', 7:'Jul', 8:'Ago', 9:'Sep', 10:'Oct', 11:'Nov', 12:'Dic'}
    tendencia_mes = df_citas['mes'].value_counts().sort_index().reset_index()
    tendencia_mes.columns = ['MesNum', 'Total']
    tendencia_mes['Mes'] = tendencia_mes['MesNum'].map(meses_nombres)
    sns.barplot(data=tendencia_mes, x='Mes', y='Total', ax=axes[3, 0], color='#38B2AC')
    axes[3, 0].set_title('KPI 7: Volumen Mensual de Citas', fontsize=14, fontweight='bold')
    axes[3, 0].set_xlabel(''); axes[3, 0].set_ylabel('Cantidad de Citas')

    # [ KPI 8: Tasa de Cancelación por Especialidad ]
    df_canceladas = df_citas_medicos[df_citas_medicos['estado_cita'] == '0']['especialidad'].value_counts()
    df_total_esp = df_citas_medicos['especialidad'].value_counts()
    tasa_cancel = (df_canceladas / df_total_esp * 100).fillna(0).sort_values(ascending=False).reset_index()
    tasa_cancel.columns = ['Especialidad', 'Tasa']
    sns.barplot(data=tasa_cancel, x='Tasa', y='Especialidad', ax=axes[3, 1], color='#DD6B20')
    axes[3, 1].set_title('KPI 8: Tasa de Cancelación por Área Médica (%)', fontsize=14, fontweight='bold')
    axes[3, 1].set_xlabel('Ausentismo / Cancelaciones (%)'); axes[3, 1].set_ylabel('')

    # Ajuste de márgenes
    plt.tight_layout(rect=[0, 0, 1, 0.98])
    
    # Renderizamos todo el lienzo de Matplotlib/Seaborn en Streamlit
    st.pyplot(fig)

except FileNotFoundError as e:
    st.warning(" Esperando a que el sistema en C genere los archivos .txt (citas, medicos, pacientes)...")
except Exception as e:
    st.error(f"Ocurrió un error al procesar los datos: {e}")
```

---
<img width="1189" height="366" alt="WhatsApp Image 2026-05-29 at 00 17 56" src="https://github.com/user-attachments/assets/dc6168ab-7fd7-4aba-b742-df51ac85c60e" />
<img width="1188" height="397" alt="WhatsApp Image 2026-05-29 at 00 17 25" src="https://github.com/user-attachments/assets/10f76ede-4825-40a1-a912-06b336d3f3ea" />
<img width="1194" height="407" alt="WhatsApp Image 2026-05-29 at 00 17 06" src="https://github.com/user-attachments/assets/f1e52b2d-9099-4193-b749-91025d9a3ed4" />
<img width="1190" height="428" alt="WhatsApp Image 2026-05-29 at 00 16 50" src="https://github.com/user-attachments/assets/b554b976-4cb8-4fe1-ad81-bd5645d6cc9b" />

Figura obtenida mediante el código implementado

## 4. Validación de Atributos de Información e Interpretación Gerencial
### 4.1 Matriz de Atributos de Valor
Evaluación analítica crítica del comportamiento de las consultas y gráficos generados para asegurar su aplicabilidad ejecutiva.

| Característica | Pregunta de Validación | Análisis y Cumplimiento en el Sistema Clínico |
| :--- | :--- | :--- |
| **Simplicidad** | ¿El gráfico evita la sobrecarga de información y destaca lo realmente importante? | Se generó los gráficos con las variables necesarias que se necesitaba monitorear. El gráfico tipo pastel es limpio y no se ve saturado debido a la simplicidad de los atributos. El gráfico de barras se explica a el mismo con las etiquetas adecuadas. Para el histogrma, agrupamos las edades en rangos lógicos para que la distrubución sea fácil de visualizar. |
| **Oportunidad (Timeliness)** | ¿La consulta procesa los datos con la velocidad requerida para que el administrador actúe a tiempo? | Al conectarse el MIS mediante la lectura de los TPS con archivos .txt y vinculandolas con las librerias pandas la velocidad de procesamiento es prácticamente inmediata. Al utilizar googlecollab y utilizar archivos .txt se observó un inconveniente al momento de actualizar los datos en tiempo real con el MIS. Por ello el gerente deberá actualizar la celda de datos en googlecollab para que al final se vea reflejado el estado real de la clínica |
| **Accesibilidad** | ¿El formato del reporte permite que un usuario no técnico interprete el resultado de un solo vistazo? | La capa gráfica fue pensada para un uso intuitivo y para aquellos que no poseen conocimientos técnicos de programación. El uso de colores y titulos facilita que cualquier persona del equipo de salud puede interpretar los gráficos. |


### 4.2 Documentación de Interpretación Gerencial
Definición analítica de las directrices estratégicas de respuesta ante los umbrales de cada indicador clave basado en los datos del sistema:

* **KPI 1: Tasa de Ausentismo y Estado General**
  * **Qué mide:** El porcentaje de citas que no se concretan (canceladas o ausentes) frente al total de turnos programados.
  * **Umbral de Alerta:** Cuando la inasistencia general supera el **15%** del volumen total operativo.

* **KPI 2: Demanda por Especialidad Médica**
  * **Qué mide:** Identifica cuáles son las áreas clínicas (ej. cardiología, pediatría) con mayor afluencia de pacientes.
  * **Umbral de Alerta:** Cuando una especialidad supera el **85%** de ocupación de su capacidad física instalada mensual.

* **KPI 3: Perfil Demográfico (Edad)**
  * **Qué mide:** Clasifica a los pacientes por rangos de edad para entender a qué público se está atendiendo mayoritariamente.
  * **Umbral de Alerta:** Cuando un grupo etario específico (ej. mayores de 65 años) representa más del **40%** de la población activa de la clínica.

* **KPI 4: Productividad Médica (Top 8)**
  * **Qué mide:** Evalúa el rendimiento real de los doctores, revisando cuántos pacientes atendieron efectivamente durante su turno.
  * **Umbral de Alerta:** Cuando el tiempo efectivo de consulta de un médico cae por debajo del **75%** respecto a las horas que tiene contratadas.

* **KPI 5: Demanda Operativa por Franja Horaria**
  * **Qué mide:** Muestra las horas del día en las que hay mayor congestión de pacientes en las instalaciones.
  * **Umbral de Alerta:** Cuando la acumulación de citas en una sola hora supera en un **90%** la capacidad de atención del personal de recepción.

* **KPI 6: Diagnósticos Clínicos Frecuentes (Top 6)**
  * **Qué mide:** Monitorea cuáles son las enfermedades, síntomas o motivos de consulta médica más recurrentes.
  * **Umbral de Alerta:** Cuando los casos de un diagnóstico específico crecen más de un **20%** de un mes a otro.

* **KPI 7: Volumen Mensual de Citas (Tendencia)**
  * **Qué mide:** Analiza si la cantidad global de pacientes atendidos en la clínica sube o baja conforme pasan los meses.
  * **Umbral de Alerta:** Cuando el volumen total de citas cae de forma sostenida durante **dos meses consecutivos**.

* **KPI 8: Tasa de Cancelación por Área Médica (%)**
  * **Qué mide:** Aísla y calcula el porcentaje exacto de turnos cancelados de forma específica para cada especialidad.
  * **Umbral de Alerta:** Cuando el ausentismo crónico (*no-show*) de una especialidad en particular sobrepasa el **20%** de su propia agenda.
