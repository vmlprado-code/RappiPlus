[README (rappiplus).md](https://github.com/user-attachments/files/33087247/README.rappiplus.md)
# RappiPlus: de datos a decisiones de negocio

Proyecto de análisis de datos desarrollado como parte de la formación en **Data Analyst de TripleTen**. Integra limpieza de datos en Python, análisis financiero, consultas SQL, un embudo de conversión, cohortes y una prueba A/B para evaluar el desempeño del negocio.

**Autor:** Víctor Manuel López Prado  
**Periodo de pedidos:** enero–junio de 2025  
**Mercados:** México, Colombia y Argentina

[Ver dashboard en Tableau Public](https://public.tableau.com/app/profile/victor.manuel.lopez.prado/viz/sprit12rappiplus/RAPPIPLUS1SEMESTRE)

## Objetivo

Transformar datos crudos en información útil para responder preguntas sobre rentabilidad, comportamiento de compra y conversión, y proponer acciones que apoyen decisiones comerciales y financieras.

## Herramientas

| Herramienta | Aplicación |
| --- | --- |
| Python y Pandas | Exploración, limpieza, integración de tablas y cálculo de indicadores. |
| SQL / PostgreSQL | Consultas con CTEs, joins, agregaciones y funciones de fecha para embudo y cohortes. |
| SQLAlchemy | Conexión a la base de datos desde Python. |
| Matplotlib y Seaborn | Gráficos de productos, marketing, embudo y mapa de calor de cohortes. |
| Statsmodels | Prueba Z para comparar proporciones de conversión. |
| Tableau | Presentación interactiva del análisis comercial y financiero. |

## Fuentes de datos

| Fuente | Descripción |
| --- | --- |
| `rappiplus_orders_raw.csv` | 25,100 registros iniciales de pedidos: clientes, fechas, productos, cantidades, precios, descuentos y montos. |
| `rappiplus_catalog.csv` | Catálogo de 7 productos en 3 categorías, con costos unitarios y proveedores. |
| `rappiplus_marketing_spend.csv` | 1,620 registros de inversión por fecha, país y canal. |
| `events` | Eventos de navegación y compra para analizar el embudo. |
| `users` | Información de registro de 8,000 usuarios para definir cohortes. |
| `user_activity` | Actividad posterior al registro de los usuarios. |
| `experiment_checkout_ui.csv` | 10,000 observaciones de un experimento de interfaz del checkout. |

Los CSV se cargan desde enlaces incluidos en el notebook. Las tablas SQL requieren acceso a la base de datos del proyecto o a una base equivalente.

## Proceso de análisis

1. **Calidad de datos:** conversión de fechas, eliminación de 100 pedidos duplicados, estandarización de categorías y países, tratamiento de valores faltantes y revisión de valores numéricos.
2. **Rentabilidad:** integración de pedidos con el catálogo para calcular costos de productos; cálculo de ingresos, gasto de marketing, resultado y ticket promedio.
3. **Embudo de conversión:** conteo de usuarios únicos por etapa mediante SQL y cálculo de pérdidas entre etapas.
4. **Cohortes:** agrupación por mes de registro y exploración de actividad posterior mediante un mapa de calor.
5. **Prueba A/B:** comparación de la conversión entre las variantes control y tratamiento mediante una prueba Z bilateral, con significancia de 0.05.
6. **Comunicación:** exportación de datasets limpios y presentación de resultados en Tableau.

## Resultados principales

### Desempeño financiero

Resultados registrados en las salidas del notebook; importes expresados en USD según el reporte del proyecto.

| Indicador | Resultado |
| --- | ---: |
| Pedidos únicos después de eliminar duplicados | 25,000 |
| Ingresos totales, redondeados | USD 51,989,757 |
| Costo de productos, redondeado | USD 43,131,278 |
| Costo de productos / ingresos | 82.96% |
| Inversión en marketing | USD 2,871,843.53 |
| Marketing / ingresos | 5.52% |
| Resultado después de costo de productos y marketing | USD 5,986,635.47 |
| Margen del resultado calculado | 11.52% |
| Ticket promedio | USD 2,079.59 |

El resultado se calcula como **ingresos − costo de productos − marketing**. No representa utilidad neta ni contempla todos los posibles gastos operativos e impuestos. El cálculo utiliza ingresos y costo de productos previamente redondeados.

### Productos y marketing

- **Blender-XL-Red** registra el mayor número de pedidos: **4,179**. Este conteo corresponde a operaciones, no a unidades vendidas.
- **Social** concentra la mayor inversión de marketing identificada: **USD 918,043.21**. Este indicador mide gasto, no ingresos ni retorno de inversión.
- El costo de productos absorbe una proporción elevada de los ingresos, por lo que conviene evaluar márgenes junto con el volumen de ventas.

### Embudo de conversión

La consulta encadena usuarios que aparecen en las etapas anteriores, partiendo de `first_visit`.

| Etapa | Usuarios | Conversión desde la etapa anterior |
| --- | ---: | ---: |
| `first_visit` | 7,796 | — |
| `select_item` | 7,393 | 94.83% |
| `add_to_cart` | 7,052 | 95.39% |
| `begin_checkout` | 6,364 | 90.24% |
| `add_payment_info` | 4,967 | 78.05% |
| `purchase` | 3,857 | 77.65% |

La conversión entre la primera visita y la compra es de **49.47%**. La mayor pérdida absoluta se observa entre el inicio del checkout y la captura de información de pago: **1,397 usuarios**.

Este hallazgo sugiere investigar posibles fricciones en esa etapa. La consulta identifica presencia en los eventos, pero no verifica su orden cronológico ni que ocurran en una misma sesión.

### Prueba A/B del checkout

| Variante | Usuarios | Conversiones | Tasa de conversión |
| --- | ---: | ---: | ---: |
| Control | 4,965 | 779 | 15.69% |
| Tratamiento | 5,035 | 820 | 16.29% |

- **Estadístico Z:** −0.8133.
- **Valor p:** 0.4161.
- **Nivel de significancia:** 0.05.

No se rechaza la hipótesis nula: la muestra no aporta evidencia estadística suficiente para afirmar que las variantes tienen tasas de conversión diferentes. Esto no demuestra que sean equivalentes.

### Cohortes

El notebook incluye consultas de actividad por mes de registro y un mapa de calor. La implementación actual agrupa los días 15–28 en una sola ventana y define la siguiente como días posteriores al 28. Además, cuenta registros de actividad en lugar de usuarios únicos en las ventanas.

Por ello, los porcentajes deben revisarse antes de interpretarlos como retención semanal: el repunte de la tercera ventana y el cero de la cuarta pueden responder a la definición de las ventanas y no a cambios reales del comportamiento.

## Recomendaciones de negocio

- Investigar la experiencia de captura de información de pago, donde se registra la mayor pérdida absoluta de usuarios del embudo.
- Evaluar productos, países y canales por margen, además de ingresos y volumen de pedidos.
- Comparar el gasto de marketing con conversiones y margen atribuible antes de reasignar presupuesto.
- Continuar evaluando cambios en el checkout con experimentos que permitan detectar un efecto relevante para el negocio.
- Refinar el cálculo de retención para contar usuarios únicos en ventanas comparables.

## Cómo ejecutar el notebook

1. Descarga este repositorio y abre `S12_Estudiante_Proyecto_Final(1).ipynb` en Jupyter Notebook, JupyterLab o Google Colab.
2. Instala las dependencias:

   ```bash
   pip install pandas matplotlib seaborn sqlalchemy psycopg2-binary statsmodels jupyter
   ```

3. Comprueba que los enlaces de los CSV sean accesibles. Si trabajas con copias locales, cambia las rutas en las llamadas a `pd.read_csv()`.
4. Configura la conexión SQLAlchemy con una base PostgreSQL que contenga `events`, `users` y `user_activity`, usando credenciales propias y evitando publicarlas en el repositorio.
5. Ejecuta las celdas en orden. Las secciones de embudo y cohortes requieren la conexión SQL; las secciones basadas en CSV pueden explorarse por separado.
6. La etapa de limpieza exporta al directorio de trabajo:

   - `orders_clean.csv`
   - `catalog_clean.csv`
   - `marketing_clean.csv`

7. Consulta el dashboard de Tableau Public para explorar la presentación visual del proyecto.

## Consideraciones metodológicas

- Los resultados aquí descritos provienen de las salidas guardadas del notebook; no se ha reejecutado la conexión a la base de datos.
- La limpieza convierte valores negativos a positivos y utiliza imputaciones. Estas decisiones deben validarse según las reglas del negocio y el posible significado de devoluciones o ajustes.
- El costo de productos se calcula con un `inner join` al catálogo, que excluye los pedidos sin un producto identificado; los ingresos totales se calculan sobre todos los pedidos depurados.
- Algunas gráficas utilizan listas de valores fijos. Si se actualizan las fuentes, estos valores deben sincronizarse con los nuevos resultados.
- El porcentaje de abandono de la etapa final en una consulta utiliza las compras como denominador. Para medir abandono desde la etapa anterior, el denominador debe ser el número de usuarios en `add_payment_info`.

## Autor

**Víctor Manuel López Prado**  
Profesional de administración y finanzas con 12 años de experiencia en el sector hotelero, orientado al análisis de datos. Certificado en Análisis de Datos por TripleTen (2026).
