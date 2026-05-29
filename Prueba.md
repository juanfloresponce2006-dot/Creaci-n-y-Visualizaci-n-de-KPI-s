Integrantes: Alan Aguilera, Juan José Flores

# 1. Resumen Ejecutivo y Propuesta de Valor
Este proyecto surge gracias a que la clínica ha recolectado gran cantidad de datos crudos mediante su TPS de citas, pero la gerencia prácticamente está operando a ciegas en la toma de decisiones, basándose en meras intuiciones. Al no tener reportes gerenciales, no tienen certeza de cuáles son los médicos más productivos, qué especialidades tienen cuellos de botella para atender pacientes ni cuáles son los números reales del ausentismo de los pacientes. Para solucionar esta adolescencia de conocimiento, se diseñó un prototipo de Sistema de Información Gerencial (MIS) que funciona como un procesador de información para extraer el verdadero valor de estos datos transaccionales, transformando los registros operativos en información estructurada para que la dirección médica y administrativa pueda tomar decisiones tácticas y estratégicas respaldadas por datos confiables y reales.

El MIS se encarga de solventar las necesidades de información de los tres stakeholders principales, que son la Dirección Médica, la Gerencia de Operaciones y la Jefatura de Planificación y Control de Gestión. A través de un proceso automatizado de extracción, transformación y carga programado con Pandas en Jupyter Notebooks, procesamos la data transaccional para calcular una matriz limpia de 8 KPIs esenciales, alineados estrictamente con el negocio. Toda esta información procesada se visualiza de forma directa en un dashboard unificado mediante gráficos, lo que permite monitorear la productividad del personal y generar archivos estructurados para proyectar la demanda de consultas sin tener que perder tiempo en procesos manuales.

La propuesta de valor de este MIS es que transforma simples datos en una ventaja competitiva lo que optimiza los procesos de soporte y gestión de la clínica lo que mejora directamente la atención al usuario. Mediante la automatización de los reportes programados y de indicadores clave, el sistema reduce el tiempo que el equipo utiliza haciendo auditorías mensuales y controles de gestión manuales. Además, el tablero proporciona las herramientas necesarias para cumplir las metas de reducir el ausentismo (no-show) y el personal médico ineficiente, demostrando cómo un buen uso de los sistemas de información resuelve los problemas del negocio de forma eficiente.

---

# 2. Fase 1: Planificación y Análisis

## 2.1. Análisis del Negocio
* **Misión de la Empresa:** Optimizar el acceso y la gestión de los servicios de salud mediante un software eficiente, lo que facilita, simplifica, asegura y organiza la interacción entre pacientes y profesionales médicos para garantizar una atención de calidad.
* **Visión de la Empresa:** Situarse como el instituto líder en eficiencia y servicio al cliente, mediante la digitalización e interconectividad total de sus servicios, consiguiendo que la tecnología supere las barreras de acceso entre los profesionales de la salud y los usuarios.
* **Objetivos Estratégicos:**
    * Reducir el personal médico ineficiente y el ausentismo (no-show) de pacientes en un 15% durante los próximos 6 meses.
    * Disminuir en un 25% el tiempo dedicado a la auditoría mensual y control de gestión de la clínica en los próximos 3 meses.
    * Reducir en un 20% el tiempo de espera para la asignación de consultas en especialidades críticas durante los próximos 5 meses.

## 2.2. Identificación de Stakeholders
1. **Chief Medical Officer (Dirección Médica):** El encargado de evaluar los reportes de ocupación y productividad por médico, tasas de ausentismo por parte de los pacientes por cada especialidad y estadísticas de tiempos de gestión de citas.
2. **Gerencia de Operaciones y Administración:** Requiere acceso al MIS para realizar auditorías, reportes de cuellos de botella en la asignación de turnos y tasas de cancelación de citas.
3. **Jefatura de Planificación y Control de Gestión:** Precisa archivos planos bien estructurados con la data histórica de los pacientes, patologías generales y proyecciones móviles de demanda para los meses consecutivos.

## 2.3. Preguntas Clave
* ¿Qué información consolidada necesitas los tomadores de decisiones y con qué frecuencia?
* ¿Cómo se transforman los datos operativos obtenidos mediante el TPS en información gerencial?
* ¿Cuáles son las restricciones técnicas de almacenamiento y persistencia para el MIS?
* ¿Quiénes tendrán acceso a la información del MIS y cómo se garantizará la seguridad de los datos?
* ¿Cómo interactuará el/los usuario/s gerencial con el MIS?

---

# 3. Fase 2: Diseño de la Solución (Matriz de KPIs)

| Nombre del KPI | Tipo de Reporte (MIS) | Fórmula / Definición Lógica | Propósito Estratégico |
| :--- | :--- | :--- | :--- |
| **1. Tasa de ausentismo de pacientes** | Programado | Citas canceladas + Citas pendientes / Total de citas x 100 | Evaluar la demanda insatisfecha acumulada en el TPS para ajustar las políticas de sobreventa o liberación de turnos. |
| **2. Volumen de citas por especialidad médica** | Programado | (Citas completadas agrupadas por especialidad) | Identificar las áreas clínicas de mayor demanda operativa para planificar la infraestructura física y la asignación de presupuestos a corto y mediano plazo. |
| **3. Distribución demográfica de pacientes** | Programado | Conteo y porcentaje de pacientes por rangos de edad | Caracterizar epidemiológicamente a la población de usuarios para diseñare e implementar campañas preventivas. |
| **4. Productividad Médica (Top 8)** | Resumido | Conteo descendente de citas en estado 'Atendida' por nombre_medico | Reducir el personal médico ineficiente y optimizar el rendimiento global del staff de salud. |
| **5. Demanda Operativa por Franja Horaria** | Resumido | Frecuencia temporal indexada secuencialmente por la variable hora | Detectar picos de saturación diaria para optimizar la distribución del personal de recepción y abatir cuellos de botella. |
| **6. Productividad de consultas por médico** | Programado | Número de pacientes atendidos efectivo / Horas totales de consulta activa | Reducir el personal médico ineficiente y optimizar el rendimiento por hora de consulta. |
| **7. Volumen Mensual de Citas** | Resumido | Agrupación cronológica secuencial del volumen transaccional por mes calendario | Proveer data estructurada a control de gestión para proyectar la demanda móvil de consultas sin reprocesos manuales. |
| **8. Tasa de Cancelación por Área Médica (%)** | Resumido | Tasa = (Citas canceladas de la especialidad/Total de citas de la especialidad)*100 | Detectar fallas de retención y el ausentismo (no-show) por área específica para maximizar el uso del recurso humano. |

## 3.2. Distribución del Dashboard
El diseño e implementación del dashboard se desarrolla como estrategia para una institución de salud que, tras acumular una gran cantidad de datos crudos mediante su TPS, necesita optimizar la toma de decisiones. El mismo muestra un tablero estructurado en una matriz de 4 filas por 2 columnas (4x2), distribuyendo los gráficos de forma balanceada e integrada de la siguiente manera:

* **Fila 1:** Aloja a la izquierda el gráfico de pastel de la Tasa de Ausentismo y Estado General y a la derecha las barras horizontales de la Demanda por Especialidad Médica.
* **Fila 2:** Dispone a la izquierda el histograma continuo del Perfil Demográfico (Edad) y a la derecha el gráfico de barras horizontales de Productividad por Médico (Top 8).
* **Fila 3:** Presenta a la izquierda el gráfico de líneas de la Demanda Operativa por Franja Horaria y a la derecha el diagrama de barras del Top 6 de Diagnósticos Clínicos Frecuentes.
* **Fila 4:** Ubica a la izquierda las barras verticales de tendencia del Volumen Mensual de Citas y a la derecha el gráfico analítico de la Tasa de Cancelación por Área Médica (%).

---

# 4. Fase 3: Implementación Técnica

## 4.1. Proceso ETL (Extracción, Transformación y Carga) en Jupyter Notebooks
El pipeline de datos desarrollado en Python se encarga de estructurar los datos del TPS mediante los siguientes pasos automatizados:
* **Extracción:** Lectura automatizada de archivos planos fuente en bruto (citas.txt, medicos.txt, pacientes.txt) delimitados por caracteres de punto y coma (;), asegurando la integridad de llaves y campos de identidad mediante tipado inicial de cadenas de texto (str).
* **Transformación:** Limpieza y tipado forzado de variables numéricas (edad) y temporales (fecha_dt), extracción indexada del mes calendario, mapeo descriptivo de códigos operacionales de la clínica (0, 1, 2) a estados legibles de negocio, y consolidación de la base de datos mediante cruces relacionales internos de tipo inner join.
* **Carga:** Estructuración estructurada y optimizada de DataFrames limpios listos para alimentar de forma directa el bloque de visualización analítica de la gerencia.

## 4.2. Implementación de Visualizaciones (Matplotlib y Seaborn)
Para la construcción estética del dashboard, se implementó la librería Seaborn bajo el tema visual de cuadrícula blanca (`style="whitegrid"`) y un escalado apto para reportes corporativos (`context="paper"`). Se forzó el uso del conjunto tipográfico DejaVu Sans de forma global para mitigar advertencias de renderizado en el sistema.

La paleta cromática se seleccionó con un enfoque puramente funcional y ejecutivo:
* **Métricas de Estatus Global y Fugas (KPI 1 y KPI 8):** El diagrama de pastel emplea un mapeo explícito (`#2E8B57` para Atendidas, `#4682B4` para Pendientes, y `#E67E22` para Canceladas/Ausentes) con bordes blancos de 1.5 de grosor. Para la tasa de cancelación por especialidad (KPI 8) se aplicó un tono naranja de advertencia (`#DD6B20`), aislando visualmente el impacto del no-show.
* **Volumen, Demanda y Eficiencia (KPI 2, KPI 4 y KPI 7):** Se emplearon variaciones armónicas de azul corporativo (`#3182CE`, `#4299E1`) y verde turquesa (`#38B2AC`) configurados en gráficos de barras, garantizando claridad en la lectura jerárquica de la productividad del personal y la afluencia para la junta directiva.
* **Estructuras de Población y Patologías (KPI 3 y KPI 6):** El perfil de edad se trazó mediante un histograma azul acero (`#2B6CB0`) subdividido en 15 contenedores con curva de estimación de densidad (KDE) superpuesta. Los diagnósticos frecuentes se diferenciaron con un tono púrpura profundo (`#805AD5`), separando analíticamente los hallazgos médicos de las métricas puramente logísticas.
* **Saturación Temporal (KPI 5):** La demanda horaria se plasmó en un gráfico de líneas continuas en rojo carmín de alta intensidad (`#E53E3E`), con un espesor de trazo de 2.5 y marcadores circulares de tamaño 8 para acentuar explícitamente los picos de congestión operativa.

El lienzo maestro unificado posee unas dimensiones físicas de 18x24 pulgadas, gobernado por el método `plt.tight_layout(rect=[0,0,1,0.98])` para evitar solapamientos y reservar la franja superior para el título principal, configurado en tipografía azul marino oscuro (`#1A365D`, `fontsize=22`).

---

# 5. Fase 4: Estrategia de Venta Comercial y Conclusiones
La defensa final del proyecto ante la Junta Directiva se fundamenta en un lenguaje puramente empresarial, omitiendo tecnicismos algorítmicos. La solución propuesta se posiciona no como un gasto tecnológico, sino como una inversión estratégica capaz de aumentar los márgenes operativos mediante la visibilidad absoluta de los datos. Al transformar la analítica en una herramienta interactiva, la consultora de SI dota a la empresa de la capacidad de anticiparse a los cambios de la demanda, mitigando riesgos financieros y consolidando una clara ventaja competitiva en el ecosistema empresarial de la Escuela Politécnica Nacional. 

La solución propuesta se posiciona no como un gasto tecnológico, sino como una inversión estratégica capaz de aumentar los márgenes operativos mediante la visibilidad absoluta de información. Al transformar la analítica en una herramienta interactiva, la consultora de SI dota a la empresa de la capacidad de anticiparse a los cambios de la demanda, mitigando riesgos financieros y consolidando una clara ventaja competitiva.
