# Evaluación de Sistema de Recomendaciones — Test A/B Tienda Online Internacional
El equipo de producto quiere saber si su nuevo sistema de recomendaciones 
mejora la conversión. Pero antes de responder esa pregunta, hay otra más 
urgente: ¿la prueba fue ejecutada correctamente? Este análisis revela que 
la respuesta es no — y explica por qué los resultados no son concluyentes.

### Herramientas y tipo de proyecto
![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
![Pandas](https://img.shields.io/badge/pandas-%23150458.svg?style=for-the-badge&logo=pandas&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-013243.svg?style=for-the-badge&logo=numpy&logoColor=white)
![SciPy](https://img.shields.io/badge/SciPy-%230C55A5.svg?style=for-the-badge&logo=scipy&logoColor=%white)
![Matplotlib](https://img.shields.io/badge/MATPLOTLIB-blue?style=for-the-badge)
![Seaborn](https://img.shields.io/badge/SEABORN-blue?style=for-the-badge)
![Limpieza de Datos](https://img.shields.io/badge/LIMPIEZA_DE_DATOS-blue?style=for-the-badge)
![Análisis de Datos](https://img.shields.io/badge/AN%C3%81LISIS_DE_DATOS-blue?style=for-the-badge)
![Visualización de Datos](https://img.shields.io/badge/VISUALIZACI%C3%93N_DE_DATOS-blue?style=for-the-badge)
![Pruebas de Hipótesis](https://img.shields.io/badge/PRUEBAS_DE_HIP%C3%93TESIS-blue?style=for-the-badge)
![A/B Testing](https://img.shields.io/badge/A%2FB_TESTING-blue?style=for-the-badge)

## Preguntas clave:
1. ¿La prueba fue ejecutada bajo condiciones válidas para sacar conclusiones?
2. ¿El sistema de recomendaciones mejoró la conversión en al menos 10% 
   en cada etapa del embudo?
3. ¿Las diferencias observadas entre grupos son estadísticamente significativas?
4. ¿Qué factores externos pudieron contaminar los resultados?

## Metodología
Se auditó la calidad de la prueba antes de analizar sus resultados, 
verificando balance entre grupos, contaminación por eventos de marketing 
y solapamiento con otras pruebas activas. Se construyó el embudo de 
conversión por grupo (login → product_page → product_cart → purchase) 
y se calcularon diferencias relativas entre grupos. Para la validación 
estadística se aplicaron pruebas Z de proporciones con corrección de 
Bonferroni (α = 0.017) para controlar el error de tipo I en las tres 
comparaciones simultáneas.

## Problemas detectados en la ejecución de la prueba

Antes de los resultados, el análisis identificó cuatro problemas que 
comprometen la validez de la prueba:

1. **Desbalance severo entre grupos.** Tras filtrar por región EU y período 
de registro, el grupo A quedó con 2,604 usuarios y el grupo B con solo 877 
— una proporción de casi 3 a 1 sin explicación clara en los datos.

2. **Solapamiento con campaña de marketing activa.** La campaña 
*Christmas & New Year Promo* estuvo activa desde el 25 de diciembre, 
afectando el 10.6% de los eventos registrados. El comportamiento de los 
usuarios durante ese período no puede atribuirse exclusivamente al sistema 
de recomendaciones.

3. **Coexistencia de dos pruebas en el mismo dataset.** Se identificaron 
441 participantes registrados en más de una prueba A/B simultáneamente, 
lo que introduce contaminación cruzada.

4. **Anomalía en el embudo.** En ambos grupos, el número de usuarios en 
`purchase` supera al de `product_cart`, lo que indica compras directas 
desde `product_page` no documentadas o errores de registro.

## Insights clave:

1. **El grupo B no alcanzó el umbral de mejora del 10% en ninguna etapa.**
   
   | Etapa | Grupo A | Grupo B | Diferencia | Significativo |
   |---|---|---|---|---|
   | login → product_page | 64.7% | 56.3% | -13% (B menor) | ✅ p = 6.94e-06 |
   | product_page → product_cart | 46.4% | 49.5% | +6.6% (B mayor) | ❌ p = 0.215 |
   | product_cart → purchase | — | — | -4.2% (B menor) | ❌ p = 0.047 |

2. **La única diferencia significativa va en dirección contraria al objetivo.**
El grupo B convirtió un 13% menos que el grupo A en `product_page` — 
exactamente la etapa donde el sistema de recomendaciones debería brillar.

3. **La actividad por usuario es similar entre grupos.** El grupo B promedia 
5.71 eventos por usuario vs 6.79 del grupo A, con distribuciones similares 
y sin dependencia de usuarios atípicos. El desbalance en participantes no 
se explica por comportamiento diferencial.

4. **El solapamiento navideño afecta el 10.6% de los datos.** Filtrar ese 
período comprometería aún más la comparabilidad entre grupos dado el 
desbalance existente, por lo que se conservan los datos completos con 
la advertencia correspondiente.

## Conclusión
**El test A/B no cumplió su objetivo.** El sistema de recomendaciones no 
demostró mejorar la conversión en ninguna etapa del embudo. Sin embargo, 
los problemas de ejecución de la prueba impiden atribuir este resultado 
únicamente al sistema — la prueba no fue ejecutada bajo condiciones válidas.

Se recomienda **no implementar el sistema de recomendaciones** basándose 
en esta prueba, y repetir el experimento bajo condiciones controladas: 
grupos balanceados, un único test activo por período, y fechas alejadas 
de campañas de marketing activas.

## Diccionario de datos

**`ab_project_marketing_events_us.csv` — calendario de eventos de marketing 2020:**
- `name` — nombre del evento de marketing
- `regions` — regiones donde se llevó a cabo la campaña
- `start_dt` — fecha de inicio de la campaña
- `finish_dt` — fecha de finalización de la campaña

**`final_ab_new_users_upd_us.csv` — usuarios registrados del 7 al 21 de diciembre de 2020:**
- `user_id` — identificador único del usuario
- `first_date` — fecha de inscripción
- `region` — región del usuario
- `device` — dispositivo utilizado para la inscripción

**`final_ab_events_upd_us.csv` — eventos registrados del 7 de diciembre de 2020 al 1 de enero de 2021:**
- `user_id` — identificador único del usuario
- `event_dt` — fecha y hora del evento
- `event_name` — tipo de evento (product_page, product_cart, purchase)
- `details` — datos adicionales del evento (ej. total del pedido en USD para eventos `purchase`)

**`final_ab_participants_upd_us.csv` — participantes de la prueba:**
- `user_id` — identificador único del usuario
- `ab_test` — nombre de la prueba A/B
- `group` — grupo asignado al usuario (A o B)

## Cómo reproducir el análisis

```bash
git clone https://github.com/sgcuervo/ab-test-recommender-system

cd ab-test-recommender-system

pip install -r requirements.txt

jupyter notebook notebooks/analysis.ipynb
```
