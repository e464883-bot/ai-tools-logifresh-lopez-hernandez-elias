# Reporte de validación — Dashboard LogiFresh

Fecha de revisión local: 2 de septiembre de 2026  
Fuente autorizada: `Datos_sinteticos_LogiFresh_dashboard.xlsx`  
Estado: publicado y verificado en GitHub Pages

## 1. Objetivo y decisión

Construir un dashboard público, rápido y trazable para localizar brechas frente a la meta SLA de 90%, examinar segmentos e incidentes y definir un piloto de medición de 30 días. La audiencia prevista es operaciones, calidad y servicio.

Las preguntas analíticas son:

1. ¿Cuál es la brecha global y filtrada frente a la meta SLA de 90%?
2. ¿Cómo cambia el SLA por mes y producto?
3. ¿Qué tipos de incidente se registran con mayor frecuencia?
4. ¿Qué productos concentran los montos registrados de reclamación?
5. ¿Qué evidencia adicional se necesita antes de atribuir causas y decidir una intervención?

## 2. Inventario y perfil de calidad

| Elemento | Resultado |
|---|---:|
| Hoja `Datos` | 240 filas × 18 columnas |
| Hoja `Diccionario_y_control` | 22 filas × 4 columnas |
| Grano | Un embarque por fila |
| Llave | `id_embarque` |
| Periodo | 2026-04-01 a 2026-06-28 |
| Valores faltantes | 0 |
| Filas duplicadas | 0 |
| IDs duplicados | 0 |
| Inconsistencias SLA/retraso | 0 |
| Inconsistencias excursión/temperatura >8 °C | 0 |

Los dominios observados son coherentes con el diccionario: `sla_entrega` contiene `Cumple/No cumple`; `excursion_temp_mayor_8c`, `Sí/No`; y `tipo_incidente`, cinco categorías más `Sin incidente`.

## 3. Conciliación de controles

| Prueba de aceptación | Fórmula | Esperado | Obtenido desde las filas | Estado |
|---|---|---:|---:|---|
| Total | Conteo de filas | 240 | 240 | Cumple |
| SLA | Cumple / total | 76.7% | 184/240 = 76.7% | Cumple |
| Retraso promedio de tardíos | Promedio donde `retraso_min > 0` | 51.8 min | 51.8 min, n=56 | Cumple |
| Incidentes | `tipo_incidente != Sin incidente` | 52 | 52 | Cumple |
| Excursiones >8 °C | `excursion_temp_mayor_8c = Sí` | 9 | 9 | Cumple |
| Reclamaciones | Suma de `reclamacion_mxn` | $882,649 MXN | $882,549 MXN | **No concilia: −$100** |
| Satisfacción | Promedio simple | 8.5/10 | 8.5/10 | Cumple |

La diferencia de reclamaciones no es un error del dashboard: los 15 importes no nulos de la hoja `Datos` suman $882,549 y no contienen fórmulas. El valor de control de la otra hoja es $882,649. El dashboard conserva la suma reproducible y muestra ambos valores.

## 4. Diseño de información

El modo es híbrido: la vista inicial comunica la brecha y permite exploración acotada. Se priorizaron codificaciones precisas:

| Vista | Pregunta | Elección | Alternativa descartada |
|---|---|---|---|
| Tendencia SLA | ¿Cuándo aparece la brecha? | Línea mensual con meta | Barras agrupadas: más carga para sólo tres periodos |
| SLA por producto | ¿Qué producto tiene menor tasa? | Barras horizontales desde cero, con n | Dona: comparación angular imprecisa |
| Incidentes | ¿Qué categoría es más frecuente? | Barras ordenadas | Burbujas: área menos precisa |
| Reclamaciones | ¿Qué producto concentra monto? | Barras ordenadas | Pastel: dificulta comparar cinco partes |

El sitio no usa librerías externas. Las gráficas son SVG con título accesible y resumen textual. El color no es el único canal: se muestran valores, símbolos, etiquetas y estado escrito.

## 5. Pruebas de interacción y presentación

Pruebas ejecutadas en navegador sobre `http://127.0.0.1:8765/`:

| Caso | Esperado | Obtenido | Estado |
|---|---|---|---|
| Carga inicial | 240; SLA 76.7%; retraso 51.8; 52; 9; satisfacción 8.5 | Coincide; reclamaciones muestran $882,549 y alerta de control $882,649 | Cumple con salvedad documentada |
| Filtro individual `Transportista = Centro` | Recalcular todos los componentes | 60 embarques; SLA 76.7%; $367,550; 60 filas; 4 gráficas | Cumple |
| Filtros combinados `Centro + Estándar` | Recalcular todos los componentes | 20 embarques; 16 cumplen; SLA 80.0%; retraso 57.8 min; 5 incidentes; $214,550 | Cumple |
| Propagación de filtros | Título, KPIs, 4 gráficas, panel y tabla cambian | Cambiaron los cuatro resúmenes de gráfica, título, KPIs, hechos y 20 filas | Cumple |
| Restablecimiento | Recuperar vista completa | 240 embarques, SLA 76.7% y anuncio accesible | Cumple |
| Búsqueda `LF-0185` | Un registro exacto | 1 fila; ID `LF-0185`; KPIs y panel recalculados | Cumple |
| Sin resultados `ZZZ` | Estado vacío recuperable | 0 embarques; tasas “No calculable”; 4 gráficas y tabla con estado vacío | Cumple |
| Descarga CSV | Preparar sólo la vista filtrada | El manejador generó Blob CSV y anunció “Descarga preparada con 1 registro” | Cumple; el navegador de prueba no expuso evento de descarga |
| Vista móvil | 390 × 844 sin desbordamiento del documento | Sin overflow horizontal; KPIs y gráficas a una columna; tabla con scroll propio | Cumple |
| Accesibilidad básica | Nombres, foco, landmarks y equivalentes textuales | 12 controles con nombre visible, 1 `main`, 1 `h1`, 4/4 SVG con título, 2 regiones `aria-live` | Cumple |
| Contraste | Texto normal ≥4.5:1 | Pares principales entre 4.97:1 y 15.13:1 | Cumple |
| Rutas y recursos | HTML y datos disponibles; sin CDN | `/` = 200; `/data.js` = 200/304; cero recursos externos | Cumple |
| Consola | Sin errores ni advertencias | 0 errores; 0 advertencias | Cumple |

## 6. Correcciones realizadas

| Severidad | Observación | Corrección | Revalidación |
|---|---|---|---|
| Menor | El título ejecutivo permanecía global al filtrar | Título y subtítulo ahora se recalculan, incluido estado vacío | Cumple con filtro y búsqueda |
| Menor | La descarga decía “1 registros” | Se añadió singular/plural dinámico | “1 registro” verificado |
| Menor | El navegador solicitaba un favicon inexistente | Se integró un favicon SVG como `data:` | Sin nueva solicitud 404 |

No quedaron observaciones críticas ni mayores abiertas. La discrepancia de $100 es un problema de conciliación de la fuente y permanece visible.

La rúbrica de visualización obtuvo **98/100**: integridad 20/20, correspondencia gráfica 20/20, narrativa 15/15, jerarquía 15/15, accesibilidad 10/10, interacción 10/10 y acabado 8/10. Se descuentan dos puntos porque el evento de descarga no pudo observarse en el navegador automatizado, aunque el manejador y su confirmación accesible sí se verificaron. Todas las puertas obligatorias pasaron.

## 7. Hallazgos prioritarios

1. **Cálculo:** 184 de 240 embarques cumplen el SLA: 76.7%, es decir, 13.3 puntos porcentuales debajo de la meta indicada de 90%.
2. **Dato y cálculo:** abril y mayo registran 100% SLA; junio, 30% (24/80). Los 56 tardíos aparecen en junio, con media 51.8 min, mediana 51.5 y P90 73.5 min. La concentración extrema limita la generalización.
3. **Cálculo:** las reclamaciones de las filas suman $882,549 MXN; Preparados concentra $359,900 (40.8%). El valor de control excede la suma en $100.

Adicionalmente, los 52 incidentes aparecen en abril y ninguno coincide con un embarque tardío. Esto es una propiedad observada de la base sintética, no evidencia de que incidentes y retrasos sean independientes.

## 8. Hipótesis por validar

1. La separación temporal entre incidentes y tardíos podría provenir de la lógica de generación sintética o de un desfase de captura. Para validarlo se necesitan timestamps de inicio/cierre, unidad y evidencia vinculada por embarque.
2. La brecha SLA de junio podría asociarse con programación, ventanas de entrega o cambios de medición. Se requieren hora comprometida, salida/llegada real, ruta ejecutada y versión de la regla SLA; la base actual no permite elegir una causa.

## 9. Piloto recomendado de 30 días

| Campo | Propuesta |
|---|---|
| Acción | Capturar salida planificada/real, llegada comprometida/real, inicio/cierre del incidente y evidencia de temperatura para cada embarque durante 30 días |
| Responsable sugerido | Operaciones como dueño; Calidad valida incidentes/temperatura; Datos revisa completitud |
| Riesgo atendido | Decidir sobre una asociación artificial o con timestamps incompletos |
| Indicadores | SLA = cumple/total; retraso mediano y P90 sólo tardíos; excursiones/total; completitud = embarques con todos los timestamps/total |
| Línea base sintética | SLA 76.7%; tardíos 56/240; media 51.8 min; P90 73.5 min; excursiones 9/240 |
| Revisión | Seguimiento semanal y corte final al día 30 |
| Criterio | Escalar una intervención sólo si las definiciones son estables, la completitud es suficiente y el patrón persiste con evidencia temporal; de lo contrario, ajustar captura o segmentación |

El piloto mide y valida antes de atribuir causalidad. Un cambio posterior no demostraría por sí solo que el piloto lo causó.

## 10. Riesgos y datos faltantes

- discrepancia de $100 entre control y suma de filas;
- dataset sintético con secuencias temporales artificiales;
- no se incluyen timestamps comprometidos/reales para recalcular SLA de forma independiente;
- no hay política contractual, tolerancias, zona horaria ni versión de la regla SLA;
- no hay folios o archivos de evidencia para incidentes, temperatura y reclamaciones;
- `reclamacion_mxn` es un monto asociado, no pérdida contable confirmada;
- 84 fechas únicas para 240 embarques; la frecuencia y el método de generación no están documentados.

## 11. Trazabilidad

- Fuente de cada registro: hoja `Datos`, columnas listadas en el dashboard.
- Definiciones y valores esperados: hoja `Diccionario_y_control`.
- Población inicial: filas 2–241 de `Datos`.
- Suma de reclamaciones: 15 valores no nulos/no cero de `reclamacion_mxn`.
- Toda cifra filtrada se recalcula en el navegador desde `data.js`; no se promedian tasas de grupos.
- Los registros visibles conservan `id_embarque` para localizar la fila fuente.

## 12. Verificación en GitHub Pages

- URL pública: <https://e464883-bot.github.io/ai-tools-logifresh-lopez-hernandez-elias/>
- Repositorio: <https://github.com/e464883-bot/ai-tools-logifresh-lopez-hernandez-elias>
- Fuente de publicación: rama `main`, carpeta `/ (root)`.
- HTTPS: obligatorio y activo mediante el dominio predeterminado de GitHub Pages.
- Verificación ejecutada: 2 de septiembre de 2026, zona `America/Mexico_City`.

Resultados observados directamente en la URL pública:

| Prueba pública | Resultado | Estado |
|---|---|---|
| Apertura y título | `LogiFresh | Control operativo`; encabezado con brecha de 13.3 pp | Cumple |
| Datos y recursos | `data.js` relativo cargó 240 registros y permitió renderizar 4 SVG | Cumple |
| KPIs iniciales | 240; 76.7%; 51.8 min; 52; 9; $882,549; 8.5/10 | Cumple con salvedad de control |
| Filtro Centro | 60 embarques; SLA 76.7%; $367,550 | Cumple |
| Centro + Estándar | 20 embarques; SLA 80.0%; 57.8 min; 5 incidentes; $214,550 | Cumple |
| Propagación | Título, KPIs, 4 resúmenes, panel Hechos y tabla cambiaron | Cumple |
| Restablecimiento | Regresó a 240 y anunció el cambio por `aria-live` | Cumple |
| Estado vacío | `ZZZ` produjo 0, “No calculable”, 4 estados sin datos y recuperación visible | Cumple |
| Búsqueda | `LF-0185` devolvió exactamente una fila | Cumple |
| Móvil | 390 × 844, sin overflow del documento; gráficas/KPIs a una columna | Cumple |
| Consola de la página | Sin errores ni advertencias originados por el sitio | Cumple |

El navegador mostró una advertencia perteneciente a la extensión de control de Chrome; se excluyó porque su URL de origen era `chrome-extension://` y no forma parte del dashboard.
