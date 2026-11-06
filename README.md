[README S-25.md](https://github.com/user-attachments/files/34452823/README.S-25.md)
# Proyecto de analítica financiera | IX Concurso ITM 2026

**Curso:** Lenguajes de Programación · Ingeniería Financiera  
**Concurso:** 12 de noviembre de 2026  
**Simulacro:** 10 de noviembre de 2026  
**Tecnología:** Python + Streamlit

## 1. Identificación del proyecto

- **Nombre de la aplicación:** Stattum
- **Equipo e integrantes:** Isabella Hincapie - Manuela Bohorquez - Yuliana Pineda
- **Enlace a la aplicación:** [Pendiente: URL de Streamlit Community Cloud]
- **Repositorio del proyecto:** https://github.com/isahincapiecontacto-netizen/Stattum 
- **Última actualización:** 2026-11-05

## 2. ¿Qué queremos investigar? · S25

- **Situación o fenómeno de interés:** Los inversionistas en Colombia pueden elegir empresas de sectores muy distintos (comercio, energía, banca), y cada una implica un nivel de riesgo diferente. Queremos entender cómo se comportaron en la práctica en la Bolsa de Valores de Colombia (BVC).
- **Pregunta principal:** *¿Cómo se comportaron el rendimiento y el riesgo de Almacenes Éxito (EXITO.CL), Ecopetrol (ECOPETROL.CL) y Grupo Cibest (PFCIBEST.CL) frente al mercado colombiano (ICOLCAP.CL) durante los últimos cinco años y cuál ofreció la mejor relación rendimiento-riesgo?*
- **Objetivo del análisis:** Comparar retorno, volatilidad, caídas máximas y correlación entre los cuatro activos, usando datos históricos diarios.
- **Importancia financiera:** Medir el rendimiento sin medir el riesgo conduce a malas decisiones. Comparar ambos y la correlación entre activos permite evaluar su aporte a la diversificación de un portafolio.
- **Preguntas complementarias:**
  - ¿Qué activo sufrió la mayor caída desde un máximo (drawdown)?
  - ¿Qué par de activos ofrece mejor diversificación (menor correlación)?
  - ¿Qué tan sensible es cada activo al mercado (beta frente al mercado colombiano, ICOLCAP)?

**Qué puede y qué no puede responderse:** con datos históricos podemos describir rendimientos, volatilidad, correlaciones y caídas pasadas. **No** podemos predecir precios futuros, ni garantizar que el comportamiento pasado se repita, ni explicar las causas de cada movimiento.

## 3. Activos, datos y alcance · S25

| Activo | Ticker | Tipo | Razón para incluirlo |
|---|---|---|---|
| Almacenes Éxito | EXITO.CL | Acción (comercio minorista) | Principal cadena de tiendas que cotiza en la BVC; representa el consumo y el sector retail |
| Ecopetrol | ECOPETROL.CL | Acción (energía) | Mayor empresa del país; su precio depende del petróleo y de factores políticos |
| Grupo Cibest (Bancolombia) | PFCIBEST.CL | Acción preferencial (banca) | Mayor grupo bancario colombiano; representa el sector financiero |
| iShares COLCAP | ICOLCAP.CL | ETF | Replica el índice COLCAP; sirve de referencia del mercado (benchmark y base para la beta) |

- **Fuente de datos:** Yahoo Finance mediante la librería [yfinance](https://ranaroussi.github.io/yfinance/)
- **Período:** 2020-01-01 – 2026-01-01 (ajustable en la app)
- **Frecuencia:** Diaria
- **Variables disponibles:** Fecha, apertura, máximo, mínimo, cierre ajustado, volumen. Usamos el **cierre ajustado** (`auto_adjust=True`).
- **Limitaciones de los datos:**
  - Las acciones colombianas tienen menor liquidez: hay días sin negociación. Usamos únicamente **fechas comunes** a todos los activos y, si hace falta, rellenamos huecos de hasta 5 días.
  - Los precios están en **pesos colombianos (COP)**; no se considera el efecto de la tasa de cambio.
  - Cibest cambió de nombre y de ticker (antes Bancolombia, PFBCOLOM). **Verificar desde qué fecha tiene historial** `PFCIBEST.CL` en Yahoo Finance.
  - Éxito tuvo cambios de control accionario y tiene poca liquidez: su historial puede mostrar saltos.
  - La tasa libre de riesgo de la app (4 % por defecto) debe ajustarse a una referencia colombiana, por ejemplo TES, y citar la fuente.
  - Los símbolos `.CL` deben confirmarse en Yahoo Finance antes de usarse.
  - Yahoo Finance es una fuente gratuita, no oficial, y puede tener faltantes o correcciones.

**Primera prueba:** [Enlace al notebook o archivo; anota qué funcionó y qué no al descargar con `yf.download`]

## 4. Metodología de análisis · S25–S26

| Indicador / técnica | ¿Qué mide? | ¿Por qué lo necesitamos? | Implementación |
|---|---|---|---|
| Retorno diario | `r_t = P_t / P_{t-1} − 1` | Base de todos los cálculos de riesgo | `metricas.retornos` |
| Retorno acumulado | `P_final / P_inicial − 1` | Ganancia total en el período | `metricas.retorno_acumulado` |
| Retorno anualizado (CAGR) | `(P_final / P_inicial)^(1/años) − 1` | Comparar activos en una misma escala anual | `metricas.retorno_anualizado` |
| Volatilidad anual | `σ_diaria × √252` | Mide el riesgo (dispersión de retornos) | `metricas.volatilidad_anual` |
| Ratio de Sharpe | `(μ_anual − r_f) / σ_anual` | Retorno obtenido por unidad de riesgo; responde la pregunta principal | `metricas.sharpe` |
| Drawdown máximo | Mayor caída desde un máximo previo | Riesgo de pérdida real que sufriría un inversionista | `metricas.max_drawdown` |
| Beta | `Cov(activo, mercado) / Var(mercado)` | Sensibilidad frente al mercado colombiano (ICOLCAP) | `metricas.beta` |
| Correlación | Relación lineal entre retornos | Potencial de diversificación | `app.py` (pestaña Correlación) |
| Medias móviles | Promedio de precios de 20 y 100 días | Identificar tendencia | `app.py` (pestaña Precios) |

Seleccionamos estos indicadores porque cubren las dos caras de la pregunta (rendimiento y riesgo) y su combinación (Sharpe). Un Sharpe alto indica mejor retorno por unidad de riesgo, pero asume retornos aproximadamente normales, algo que no se cumple bien en cripto.

### Visualizaciones previstas

| Gráfico | Pregunta que ayuda a responder | Variables | Interpretación esperada |
|---|---|---|---|
| Base 100 (líneas) | ¿Cuánto habría crecido una inversión de 100? | Precio normalizado | Comparar trayectorias y momentos de caída |
| Mapa riesgo-retorno (dispersión) | ¿Qué activo da más retorno por riesgo? | Volatilidad vs. retorno anualizado | Mejor posición: arriba a la izquierda |
| Drawdown (líneas) | ¿Qué tan profundas fueron las caídas? | Caída desde el máximo | Duración y severidad de las pérdidas |
| Volatilidad móvil | ¿El riesgo cambió con el tiempo? | Desv. estándar móvil anualizada | Periodos de calma vs. estrés |
| Histograma de retornos | ¿Cómo se distribuyen los retornos diarios? | Retorno diario | Colas gruesas, dispersión por activo |
| Mapa de calor de correlación | ¿Qué activos diversifican? | Correlación de retornos | Correlaciones bajas = mejor diversificación |

**Primer resultado verificable:** 
| Ticker | Retorno acumulado | Retorno anualizado | Volatilidad anual | Sharpe | Máx. drawdown | Beta vs ICOLCAP.CL |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **ECOPETROL.CL** | 73.86% | 11.72% | 37.65% | 0.37 | -37.99% | 0.82 |
| **EXITO.CL** | 41.61% | 7.22% | 44.56% | 0.28 | -66.59% | 0.43 |
| **ICOLCAP.CL** | 118.76% | 16.98% | 18.02% | 0.72 | -30.22% | 1.00 |
| **PFCIBEST.CL** | 267.24% | 29.78% | 31.98% | 0.83 | -32.58% | 0.93 |


## 5. Diseño de la aplicación · S25

- **Usuarios previstos:** Estudiantes, inversionistas principiantes y docentes que quieran comparar riesgo y rendimiento entre activos.
- **Entradas y filtros:** Selección de activos (más un ticker libre), rango de fechas, tasa libre de riesgo, ventana de volatilidad móvil y referencia de mercado.
- **Secciones de la interfaz:** Resumen · Precios · Riesgo · Correlación · Conclusiones.
- **Resultados visibles:** Tabla de indicadores, mapa riesgo-retorno, gráficos de precio, drawdown, volatilidad, histograma, mapa de calor, lectura automática de resultados y descarga en CSV.
- **Interacción principal:** Al cambiar activos, fechas o parámetros, la app vuelve a descargar los datos (con caché de 1 hora), recalcula todos los indicadores y actualiza gráficos y conclusiones.

**Flujo previsto:**  
Selección de activos y parámetros → Descarga con yfinance → Alineación de fechas y cálculo de retornos → Indicadores y gráficos → Lectura automática e interpretación.

**Boceto o esquema:** 

**Decisión técnica:** Python por su ecosistema financiero (`pandas`, `numpy`) y acceso directo a datos con `yfinance`. Streamlit permite construir una interfaz interactiva sin programar frontend. Separamos los cálculos en `metricas.py` para poder probarlos de forma independiente.

## 6. Construcción y pruebas · S26 (5 de noviembre)

- **Componentes implementados:** Código escrito para todas las secciones descritas arriba. **[Actualiza este punto con lo que verifiques que funciona en el laboratorio.]**
- **Cómo ejecutar la aplicación:**
  ```bash
  pip install -r requirements.txt
  streamlit run app.py
  ```
- **Dependencias:** `requirements.txt` (streamlit, yfinance, pandas, numpy, plotly)
- **Pruebas realizadas:**

| Caso | Resultado esperado | Resultado obtenido |
|---|---|---|
| Beta del benchmark contra sí mismo (datos sintéticos) | 1.00 | 1.00 ✅ |
| Drawdown siempre ≤ 0 (datos sintéticos) | Verdadero | Verdadero ✅ |
| Base 100 inicia en 100 (datos sintéticos) | 100 en todos los activos | 100 ✅ |
| Descarga real con EXITO.CL, ECOPETROL.CL, PFCIBEST.CL, ICOLCAP.CL | Tabla con 4 filas | [Pendiente de probar] |
| Ticker inválido | Aviso y exclusión del activo | [Pendiente de probar] |
| Fecha inicial posterior a la final | Mensaje de error | [Pendiente de probar] |

- **Errores conocidos y cómo se resolvieron:** [Completa durante el laboratorio]
- **Pendientes antes del simulacro:** Desplegar en Streamlit Community Cloud, probar con datos reales, completar el boceto e integrantes.

## 7. Hallazgos e interpretación · S26–S28

*Completa con los resultados reales de la app. Plantilla para cada hallazgo:*

1. **Evidencia:** 

<img width="1197" height="295" alt="image" src="https://github.com/user-attachments/assets/2a6e9d22-27fa-4617-9488-a0d22ee32642" />
<img width="1185" height="311" alt="image" src="https://github.com/user-attachments/assets/974986b9-0dc9-4085-b11e-cec884ebd557" />
<img width="1192" height="310" alt="image" src="https://github.com/user-attachments/assets/1f1b62ff-e98d-413a-9cae-b255a1555479" />
<img width="1154" height="349" alt="image" src="https://github.com/user-attachments/assets/1d1b553b-5cdd-43f7-a4da-65772f9862c5" />


3. **Qué muestra:** ### recimiento de una inversión de 100, Tendencia con medias móviles, Distribución de retornos diarios, correlación

4. **Interpretación financiera:** ¿qué activos ayudan a diversificar? Correlación baja o negativa = mejor diversificación ¿cuánto habría crecido una inversión inicial y en qué momentos cayó?
   
6. **Limitaciones:** 

**Conclusión provisional o final:** 

> Los resultados históricos no garantizan rendimientos futuros.

## 8. Simulacro · S27 (10 de noviembre)

**Guion de sustentación (3–5 minutos):**
1. **Problema (30 s):** comparar riesgo y rendimiento de Éxito, Ecopetrol, Cibest y el mercado colombiano (COLCAP).
2. **Datos y metodología (45 s):** Yahoo Finance, 5 años diarios; retorno, volatilidad, Sharpe, drawdown, beta y correlación.
3. **Demostración (90 s):** cambiar activos y fechas, mostrar mapa riesgo-retorno y drawdown.
4. **Hallazgos (60 s):** activo con mejor Sharpe, más volátil y mejor diversificador.
5. **Conclusión y limitaciones (30 s):** análisis descriptivo, no predictivo.

**Retroalimentación recibida:** [Observaciones del docente o compañeros]

| Ajuste necesario | Responsable | Estado |
|---|---|---|
| [Ajuste] | [Integrante] | [Pendiente] |

## 9. Presentación final · S28 (12 de noviembre)

- **Enlace funcional de la aplicación:** [URL]
- **Versión final / commit:** [Enlace]
- **Evidencia de presentación:** [Enlace autorizado]
- **Conclusiones finales:** [Síntesis]
- **Limitaciones y posibles mejoras:** Solo se usan fechas comunes; no se modelan costos de transacción ni impuestos; mejoras posibles: optimización de portafolio (frontera eficiente), VaR, RSI y Bollinger Bands.

## 10. Bitácora de trabajo y aportes

| Fecha | Integrante(s) | Investigación o decisión | Avance técnico / evidencia | Próximo paso |
|---|---|---|---|---|
| 2026-11-03 | [Todos] | Nombre del equipo (FinPy Labs), pregunta y activos | Código base en `app.py` y `metricas.py` | Probar con datos reales |
| 2026-11-05 | [Nombre] | [Decisiones] | [Evidencia] | [Acción] |
| 2026-11-10 | [Nombre] | [Retroalimentación] | [Cambio] | [Acción] |

## Fundamento en los recursos de consulta

| Recurso | Qué tomamos de él | Dónde se usa |
|---|---|---|
| Libro del curso (programación y analítica financiera) | Conceptos de retorno, riesgo y visualización. **[Pendiente: citar capítulos y sesiones S25–S26 exactas]** | Secciones 2 y 4 |
| [yfinance](https://ranaroussi.github.io/yfinance/) | `yf.download` con varios tickers a la vez. La fecha `start` es inclusiva y `end` es exclusiva, por eso la app suma un día a la fecha final. Se fija `auto_adjust=True` para usar precios ajustados. Con varios tickers devuelve columnas multinivel; la app toma `["Close"]`. | `app.py` → `cargar_precios` |
| [pandas](https://pandas.pydata.org/docs/) | `pct_change` (retornos), `rolling` (medias y volatilidad móvil), `cummax` (drawdown), `corr` y `cov` | `metricas.py` y `app.py` |
| [Streamlit](https://docs.streamlit.io/) | `st.sidebar`, `st.multiselect`, `st.slider`, `st.tabs`, `st.cache_data` (caché de descargas) y `st.download_button` | `app.py` |
| [Shiny para R](https://shiny.posit.co/r/) | Descartado: el proyecto se implementa en Python + Streamlit. Se conserva como alternativa equivalente. | Decisión técnica (sección 5) |

**Extensión opcional con empresas colombianas:** la app permite agregar tickers propios (por ejemplo, acciones de la BVC). Se debe verificar cada símbolo en Yahoo Finance y declarar que sus precios están en pesos colombianos y que son menos líquidos, lo que genera días sin datos (hay una opción para rellenar huecos de hasta 5 días).

## Recursos de consulta

- [Libro de programación y analítica financiera del curso](https://github.com/vyolete/proyecto_analitica_financiera)
- [Documentación de yfinance](https://ranaroussi.github.io/yfinance/)
- [Documentación de pandas](https://pandas.pydata.org/docs/)
- [Documentación de Streamlit](https://docs.streamlit.io/)
