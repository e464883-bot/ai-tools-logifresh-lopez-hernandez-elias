# Dashboard operativo LogiFresh

Dashboard HTML interactivo para revisar desempeño de entrega, retrasos, incidentes, excursiones de temperatura, reclamaciones y satisfacción en una base sintética de 240 embarques.

## Abrir el dashboard

- Dashboard: <https://e464883-bot.github.io/ai-tools-logifresh-lopez-hernandez-elias/>
- Repositorio: <https://github.com/e464883-bot/ai-tools-logifresh-lopez-hernandez-elias>

Para revisarlo localmente, basta servir esta carpeta con cualquier servidor estático y abrir `index.html`.

## Qué incluye

- siete KPIs que se recalculan con los filtros;
- meta SLA de 90% claramente identificada como meta indicada;
- filtros de fecha, origen, destino, producto, transportista, tipo de ruta, estado SLA e ID;
- tendencia mensual del SLA, ranking por producto, incidentes por tipo y reclamaciones por producto;
- panel dinámico de Hechos, Hipótesis y Próximo paso;
- tabla trazable y descarga CSV de la vista filtrada;
- estados sin datos, navegación por teclado, foco visible y diseño responsive.

## Fuente y alcance

- Archivo analizado: `Datos_sinteticos_LogiFresh_dashboard.xlsx`.
- Hojas: `Datos` y `Diccionario_y_control`.
- Grano: un embarque por fila.
- Llave: `id_embarque`.
- Periodo: 1 de abril a 28 de junio de 2026.
- Fuente: sintética; no contiene datos personales ni secretos según el diccionario.

El libro original no se publica como archivo separado. El dashboard contiene únicamente los 18 campos sintéticos necesarios en `data.js` para que los filtros funcionen en un sitio estático.

## Métricas

| Indicador | Fórmula |
|---|---|
| Embarques | Conteo de filas filtradas |
| SLA | `sla_entrega = Cumple` / embarques comparables |
| Retraso de tardíos | Promedio de `retraso_min` donde `retraso_min > 0` |
| Incidentes | Conteo donde `tipo_incidente != Sin incidente` |
| Excursiones | Conteo donde `excursion_temp_mayor_8c = Sí` |
| Reclamaciones | Suma de `reclamacion_mxn` |
| Satisfacción | Promedio simple de `satisfaccion_1_10` |

La meta SLA de 90% procede de la instrucción del proyecto, no de una política contractual incluida en el Excel.

## Arquitectura

Sitio estático sin framework, rastreadores ni dependencias externas:

- `index.html`: estructura, estilos, cálculos, gráficas SVG e interacciones;
- `data.js`: 240 registros sintéticos serializados;
- `.nojekyll`: evita transformaciones innecesarias en GitHub Pages;
- `REPORTE_VALIDACION.md`: perfil de datos, trazabilidad, pruebas y limitaciones.

Esta arquitectura reduce solicitudes de red, evita fallas por CDN y permite publicar directamente desde la rama principal.

## Advertencia de conciliación

La suma reproducible de `reclamacion_mxn` en las 240 filas es **$882,549 MXN**. La hoja `Diccionario_y_control` y la prueba de aceptación indican **$882,649 MXN**. La diferencia de **$100 MXN** se muestra en el dashboard y no se corrige de manera silenciosa.

## Validación

Consulta [REPORTE_VALIDACION.md](./REPORTE_VALIDACION.md) para ver resultados esperados/obtenidos, pruebas de filtros, vista móvil, accesibilidad, carga de recursos, correcciones y riesgos.
