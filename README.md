# Simulador de optimización de implementación WMS

Tablero interactivo (`wms_dashboard.html`) y libro de Excel (`WMS_Simulador_Optimizado.xlsx`) para planear, simular y optimizar la implementación de un sistema WMS en 15 sites de México y Colombia. Objetivo del programa: terminar en **máximo 8 meses**, al **menor costo posible**.

Abre `wms_dashboard.html` directo en cualquier navegador — es un solo archivo, no necesita servidor ni instalación.

## Qué contiene y de dónde salen los datos

Todos los cálculos parten de las 5 pestañas originales del archivo Excel fuente. El dashboard no inventa ningún dato: toma estas tablas y las convierte en un motor de cálculo en el navegador.

| Pestaña original | Qué contiene | Para qué se usa |
|---|---|---|
| **Sites Master** | Los 15 sites, su país y a qué **Cluster** (1-4) pertenece cada uno | Define qué tabla de duraciones y de recursos le aplica a cada site |
| **Implementation Phases** | Para cada Cluster y cada nivel de **Deployment Maturity** (A o B), la duración en semanas de las 8 fases (Site Readiness → Design Adaptation → Functional Review → UAT → Training → DIALT → Go Live → Hypercare) | Calcula cuánto dura cada fase de cada site, y por lo tanto la fecha de inicio/fin de cada uno |
| **Phase-Resource Allocation** | Para cada Cluster y cada fase, qué rol se necesita y cuánta capacidad consume (ej. 0.5 = medio tiempo) | Calcula cuántas personas de cada rol se necesitan cada semana mientras esa fase está activa en algún site |
| **Resource Master** | Cada rol, su costo mensual promedio (MXN) y cuántas personas hay disponibles (columnas "Actual" y "Escenario 1") | Da el costo por persona-mes y el punto de partida de disponibilidad; el dashboard permite editarlo o "ajustar automáticamente" |
| **Financials (USD)** | Por site: beneficio mensual, costos de implementación, y las opciones alternativas de etiquetado (impresoras vs. manual) y WiFi (Full / Optimizado / Priorizado) | Calcula el costo de implementación (USD) por site según la opción de etiquetado/WiFi elegida |

**Cómo se conectan:** `Site → Cluster → (Implementation Phases) → duración de sus 8 fases` y, en paralelo, `Site → Cluster → (Phase-Resource Allocation) → qué roles necesita cada fase → (Resource Master) → costo y disponibilidad de esos roles`. El costo de implementación (USD) sale directo de `Financials`, independiente del cronograma; el costo de recursos (MXN) sí depende del cronograma porque es `personas disponibles × costo mensual × duración total del programa en meses`.

**Nota sobre moneda:** los costos de implementación están en USD (fuente: Financials) y los de recursos en MXN (fuente: Resource Master). El archivo original no trae un tipo de cambio, así que el dashboard los muestra y suma por separado en vez de inventar una conversión.

## Cómo se calcula el cronograma (motor del simulador)

Con **N sites en paralelo** (parámetro editable), los primeros N sites (en orden de prioridad) arrancan el día 0. Cada site que sigue arranca apenas se libera un cupo: el día en que termina el site que ocupaba ese lugar N puestos antes. Dentro de cada site, sus 8 fases corren en secuencia estricta (no se puede alterar el orden ni acortar artificialmente una fase). La duración total del programa es la fecha en que termina el último site.

Por cada semana del programa, se suma la capacidad requerida de cada rol en todos los sites con una fase activa esa semana; el pico semanal de cada rol es lo que en teoría se necesita contratar para no superar el 100% de utilización.

## Resultados verificados (sin necesidad de ejecutar el dashboard)

Estos números salen de correr el motor de cálculo descrito arriba sobre los datos originales. Coinciden exactamente con lo calculado en `WMS_Simulador_Optimizado.xlsx`.

| Escenario | Deployment Maturity | Sites en paralelo | Duración | ¿Cumple 8 meses? | Costo implementación (USD) | Costo recursos (MXN) | Payback |
|---|---|---|---|---|---|---|---|
| **Base** (punto de partida, sin optimizar) | A | 3 | 86 semanas (19.8 meses) | **No** | $9,768,848 | $13,795,627 | 13.9 meses |
| **Óptimo** (menor costo que sí cumple 8 meses) | A | 15 (todos en paralelo) | 19 semanas (4.4 meses) | **Sí** | $4,991,383 | $24,831,185 | 7.1 meses |

El escenario Óptimo se encontró evaluando 54 combinaciones de Deployment Maturity, sites en paralelo, método de etiquetado y estrategia de WiFi, y ajustando en cada una los recursos al mínimo necesario para no tener ninguna sobrecarga (0 de 15 roles sobrecargados en el escenario Óptimo, contra 11 de 15 en el Base). El dashboard carga el escenario Óptimo por defecto.

## Funcionalidad del dashboard

- **Plan general del programa** — Gantt de los 15 sites de inicio a fin, con la línea de límite de 8 meses marcada.
- **Plan de trabajo por site** — selector de site con su cronograma de 8 fases, costo y demanda de recursos.
- **Uso de recursos, editable** — tabla por rol con el campo "Disponible" editable a mano; recalcula la utilización y el semáforo (verde ≤80%, ámbar ≤100%, rojo >100%) en vivo. Botón "Ajustar automáticamente" que fija cada rol al mínimo necesario para no tener sobrecarga.
- **Resumen de costos** — por site (USD), por tipo de recurso (MXN) y totales del escenario.
- **Guardar y comparar escenarios** — puedes nombrar y guardar tu configuración actual (Deployment Maturity, paralelismo, recursos, etiquetado, WiFi); queda almacenada en el navegador (`localStorage`) y aparece en la tabla comparativa junto al Base y al Óptimo, con botones para cargarla de nuevo o borrarla. Así se pueden comparar tantos escenarios como se quiera, no solo los dos de referencia.
- Todos los controles (Deployment Maturity, sites en paralelo, fuente de recursos, etiquetado, WiFi, y los inputs de recursos) recalculan el cronograma, la utilización y los costos en tiempo real — nada es un valor fijo.

## Archivos en este repositorio

- `wms_dashboard.html` — el tablero interactivo (autocontenido, sin dependencias externas).
- `WMS_Simulador_Optimizado.xlsx` — la versión en Excel del mismo simulador, con el motor construido en fórmulas nativas (sin macros).
