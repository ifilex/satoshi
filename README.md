# Satoshi

### Señales de extremos recientes y grid multiactivo para instrumentos cripto

Satoshi es un Expert Advisor desarrollado en **MQL4 para MetaTrader 4**.

Este repositorio comparte el código fuente de la **versión 1.00** para estudio, revisión y experimentación.

Combina señales basadas en la ubicación temporal de máximos y mínimos con entradas adicionales tipo grid y progresión configurable del lotaje.

## Contexto del proyecto

Como neuropsicólogo y desarrollador, me interesa explorar cómo se transforman observaciones en reglas explícitas y cómo esas reglas modifican la exposición ante movimientos adversos.

Satoshi permite examinar esas decisiones directamente en el código: cuándo entrar, cuándo agregar posiciones y cómo gestionar sus objetivos de salida.

Es un sistema basado en reglas. No incorpora inteligencia artificial ni aprendizaje automático.

## Señal de entrada

El programa identifica las posiciones de los máximos y mínimos dentro de dos ventanas consecutivas de barras cerradas.

Transforma esas posiciones en medidas normalizadas, calcula su diferencia y busca cruces de umbral:

- Cruce por encima del umbral positivo: señal compradora.
- Cruce por debajo del umbral negativo: señal vendedora.

La ventana predeterminada es de **10 barras** y el umbral es **50**.

La señal utiliza la ubicación temporal de los extremos. No mide directamente la distancia del precio actual a un soporte o resistencia.

## Entradas adicionales

Cuando el precio se desplaza en contra de una posición, puede agregar operaciones del mismo lado a una distancia configurada.

El lote adicional se calcula como:

    lote anterior × MultiplicadorLote

El multiplicador predeterminado es **1.05**.

El redondeo y las restricciones del bróker pueden hacer que el lote permanezca igual durante sucesivas entradas.

Los añadidos heredan el stop registrado en la operación tomada como referencia y se envían inicialmente sin take profit.

## Gestión de salida

Incluye:

- Stop loss y take profit configurables para las entradas iniciales.
- Trailing condicionado a que exista una sola orden con el identificador correspondiente en ese símbolo.
- Una rutina que intenta asignar un take profit conjunto cuando la operación tomada como referencia no tiene TP.

El cálculo del objetivo conjunto requiere revisión antes de atribuirle un breakeven monetario correcto.

## Filtros

Las entradas iniciales comprueban:

- Spread máximo.
- Horario, si está activado.
- Exclusión de fines de semana, si está activada.
- Disponibilidad de margen.
- Límite previsto de instrumentos simultáneos.

Las entradas adicionales del grid no vuelven a exigir los filtros de spread, horario o fin de semana.

Desactivar los filtros temporales permite intentar operar durante esas franjas; no garantiza que el instrumento esté disponible fuera de las sesiones del bróker.

## Instrumentos

La lista inicial contempla:

BTCUSD, ETHUSD, SOLUSD, XRPUSD, ADAUSD, DOGEUSD, LTCUSD, BNBUSD, DOTUSD, LINKUSD, AVAXUSD, MATICUSD, BCHUSD, XLMUSD, TRXUSD, ATOMUSD, NEARUSD y ALGOUSD.

Permite configurar hasta 30 símbolos.

Al inicializarse, intenta encontrar cada nombre con el sufijo detectado y, como alternativa, sin sufijo.

La disponibilidad depende del bróker. No incluye conexión directa a exchanges ni gestión de billeteras.

## Parámetros principales

| Parámetro | Valor predeterminado | Función |
|---|---:|---|
| `LoteFijo` | `0.01` | Lote inicial |
| `MultiplicadorLote` | `1.05` | Factor del lote adicional |
| `DistanciaPips` | `500.0` | Separación configurada del grid |
| `TakeProfitPips` | `500.0` | Distancia del objetivo inicial |
| `StopLossPips` | `2000` | Distancia del stop inicial |
| `MaxOrdenesPorLado` | `20` | Máximo por dirección y símbolo |
| `NumeroMagico` | `9999` | Identificador base |
| `ActivarTrailing` | `true` | Activa el trailing |
| `TrailingStopPips` | `300.0` | Distancia configurada del trailing |
| `PasoTrailingPips` | `50` | Paso configurado del trailing |
| `BarrasSenal` | `10` | Ventana de extremos |
| `UmbralSenal` | `50` | Umbral de señal |
| `MaxParesSimultaneos` | `2` | Límite previsto de instrumentos |
| `FiltrarHorario` | `false` | Activa la restricción horaria |
| `FiltrarFinSemana` | `false` | Excluye sábados y domingos |
| `SpreadMaximoPips` | `1500.0` | Umbral de spread para entradas iniciales |

Los valores describen el archivo publicado. No son recomendaciones.

Las distancias son unidades configuradas que el código convierte a precios. No representan porcentajes constantes del valor del activo.

## Visualización

Incluye:

- Panel con balance, equity y margen.
- Información de compras y ventas del símbolo procesado.
- Niveles de máximos y mínimos.
- Canal de rango.
- Marcas de señales.
- Referencia visual de una EMA.

Los niveles, el canal y la media móvil no participan en las condiciones de entrada.

## Instalación para pruebas

1. Abrir MetaTrader 4 en una cuenta demo.
2. Seleccionar **Archivo → Abrir carpeta de datos**.
3. Copiar `SATOSHI.mq4` dentro de `MQL4/Experts`.
4. Abrir el archivo en MetaEditor y compilar.
5. Revisar errores y advertencias.
6. Actualizar la lista de Expert Advisors.
7. Cargar el EA y comprobar símbolos, históricos, parámetros y permisos.

El procesamiento multiactivo depende de los ticks del gráfico donde se instala.

Utiliza `NumeroMagico` para compras y `NumeroMagico + 1` para ventas. Cambiar estos valores si coinciden con los de otros sistemas o instancias, incluido Fionn.

## Estado y limitaciones conocidas

Puntos identificados mediante lectura del código:

- **Conversión de unidades:** utiliza `MODE_POINT` para la separación del grid y `MODE_TICKSIZE` en otros cálculos. La equivalencia debe validarse por instrumento.
- **Factor de pip:** depende de los decimales y no garantiza las equivalencias monetarias indicadas en algunos comentarios.
- **Objetivo conjunto:** convierte beneficio monetario a precio sin incorporar de forma completa el valor del tick.
- **Límite de símbolos:** el conteo se realiza antes del recorrido y no se actualiza tras cada apertura.
- **Última operación:** depende del orden de recorrido, sin selección explícita por fecha de apertura.
- **Lotaje:** el redondeo por decimales no garantiza compatibilidad con todos los pasos de lote.
- **Stops:** si no se supera la comprobación de distancia mínima, la entrada inicial puede enviarse con SL o TP en cero.
- **Panel:** se actualiza para cada símbolo y termina mostrando el último procesado; su P/L total no representa toda la cartera.
- **Media móvil visual:** se establece al inicializar los elementos gráficos, sin actualización continua de esa referencia.
- **Balance mínimo:** el aviso de 500 incluido en el código es informativo; no bloquea aperturas ni acredita capital suficiente.
- **Ejecución y datos:** faltan comprobaciones integrales de parámetros, históricos y resultados de varias solicitudes.

Esta versión no incorpora un circuit breaker por drawdown ni un límite monetario agregado de riesgo.

## Riesgo

El grid agrega exposición cuando el precio se mueve en contra. Un multiplicador pequeño no elimina la acumulación de pérdidas ni el consumo de margen.

Operar varios instrumentos cripto puede concentrar riesgo por movimientos correlacionados.

## Resultados y validación

La documentación se basa en la lectura del código fuente. No acredita compilación, ejecución ni resultados de backtesting.

El repositorio no incluye un historial auditado ni una validación independiente de rentabilidad.

## Autor

Desarrollado por **Emanuel**, neuropsicólogo y desarrollador del proyecto **WineBOX**.

- https://winebox.cloud
- https://wineboxtool.cloud

## Contribuciones

Se reciben reportes de errores y propuestas mediante Issues y Pull Requests.

Incluir versión de MetaTrader 4, instrumento, período del gráfico, parámetros y registros relevantes, sin credenciales ni datos personales.

## Licencia

Este proyecto se distribuye bajo la licencia **Apache License 2.0**.

Consultar el archivo [LICENSE](LICENSE) para conocer los términos completos.

## Aviso

Material técnico y educativo. No constituye asesoramiento financiero ni una promesa de resultados.
