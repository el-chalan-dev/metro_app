# Calibración TypeSafe — Misión 3 (pasajeros en espera) — rubric-m3-v1 — 2026-10-26

- **Estado:** completada. **Veredicto: `not ready`**: el caso fuerte queda en 80 exacto en las tres corridas y no califica (hace falta más de 80).
- **Misión:** `mission-m3`, brief [`public-brief.md`](public-brief.md) de esta rama (texto exacto en cada fixture).
- **Rúbrica:** `rubric-m3-v1` + `m3-policy`, sin cambios.
- **Modelo:** `jev-latest`, resuelto a `jev-1.14.0`.
- **Persistencia:** desactivada (`RECORD_ATTEMPTS=0`, sin `--record`). No se escribió ningún intento en GitHub.
- **Corridas:** 3 por caso, 2026-10-26 entre 09:27 y 09:28 (America/Mexico_City, UTC-6), en un clon separado del VPS (`~/m3-calib/metro_app` en `d2433b1`), no en el checkout de Hermes.

## Casos

Los tres diffs son código real sobre `dev` `acd8258` y solo tocan `apps/game/`. Antes de la calibración se comprobó en el VPS:

| Caso | Fixture | Diff | Qué hace el código | lint / typecheck | Expectativa manual |
| --- | --- | --- | --- | --- | --- |
| Débil | [`passengers-weak.json`](../../../../../apps/community/fixtures/calibration-m3/passengers-weak.json) | 0.8 KB, 1 archivo | Un punto gris fijo junto a cada estación; sin generación, pausa, límite, destino ni orden | ok / ok | `core-gap`, 0 |
| Medio | [`passengers-mid.json`](../../../../../apps/community/fixtures/calibration-m3/passengers-mid.json) | 3.8 KB, 1 archivo | Pasajeros cada 2 s tras **Ejecutar** con `Math.random`; siguen durante **Parar**; sin límite de 12. El prompt pide pausa, límite y orden reproducible | ok / ok | `core-gap`, 0 |
| Fuerte | [`passengers-strong.json`](../../../../../apps/community/fixtures/calibration-m3/passengers-strong.json) | 6.3 KB, 2 archivos | Generación cada 1,5 s de simulación en el bucle existente, secuencia de semilla fija por número de pasajero, destino ≠ origen, máximo 12, pausa sin acumulación, Reset/recarga vacían; colas 2×6 bajo cada estación | ok / ok / `build:web` ok | > 80, califica |

La lógica del caso fuerte se verificó con un script aparte: los primeros diez pasajeros coinciden con dos ritmos de frame distintos (`E8 △, E1 □, E7 ○, E7 △, E6 ○, E2 ○, E5 △, E8 □, E2 □, E7 △`), ninguna estación pasa de 12 tras 5000 s simulados y ningún destino repite la forma de su estación. Las pruebas declaradas en el fixture usan esos valores.

## Resultados

Niveles: cobertura / fidelidad / alcance / evidencia / trazabilidad. "Rúbrica" es el total antes de la política; "Final" es el total que recibe el participante.

| Caso | Corrida | Rúbrica | Final | Niveles | Hard-fails | Califica |
| --- | --- | ---: | ---: | --- | --- | --- |
| Débil | 1 | 47.5 | 0 | 2/1/3/1/3 | core-gap | no |
| Débil | 2 | 47.5 | 0 | 2/1/3/1/3 | core-gap | no |
| Débil | 3 | 47.5 | 0 | 2/1/3/1/3 | core-gap | no |
| Medio | 1 | 51.3 | 0 | 2/1/3/2/3 | core-gap | no |
| Medio | 2 | 46.3 | 0 | 2/1/2/2/3 | core-gap | no |
| Medio | 3 | 46.3 | 0 | 2/1/2/2/3 | core-gap | no |
| Fuerte | 1 | 80.0 | 80.0 | 3/3/4/3/3 | — | **no** (80 no es > 80) |
| Fuerte | 2 | 80.0 | 80.0 | 3/3/4/3/3 | — | **no** |
| Fuerte | 3 | 80.0 | 80.0 | 3/3/4/3/3 | — | **no** |

## Lectura

- **Estabilidad:** ningún caso cambia de elegibilidad entre corridas y ninguna dimensión se mueve más de un nivel (solo alcance del caso medio, 3→2). Cumple el criterio del plan.
- **Separación:** débil y medio quedan en 0 por `core-gap`, como se esperaba. El fuerte queda 80 puntos arriba de ellos, pero en el borde exacto: 3/3/4/3/3 = 80.0, la combinación que la calibración del 22-oct ya marcó como "no califica".
- **Confianza:** Jev reporta confianza `low` en cobertura, fidelidad y trazabilidad del caso fuerte, y el enrutamiento queda `defer` en los nueve casos.
- **Feedback del fuerte:** genérico ("hay requisitos del prompt que el código no implementa", "el código hace algo distinto en algunos valores"), sin señalar un requisito concreto. En la revisión manual no se encontró un requisito del prompt sin implementar.
- Con `jev-1.14.0`, un PR pequeño (6 KB) que cumple todas las reglas no supera 80 en esta misión. El caso de ensayo `syn-small-strong` del 22-oct (reiniciar tablero, 87.5) sí lo hizo, pero su prompt tenía menos requisitos.

## Veredicto

`not ready`. Según el plan, no se ajustaron la rúbrica, el umbral ni el alcance de la misión. La decisión queda para Artur.

## Opciones (requieren decisión)

1. **Mantener misión y umbral, mejorar el caso fuerte y recalibrar.** Reescribir el fixture fuerte con requisitos numerados uno a uno y una prueba declarada por requisito, y correr otra vez. Si llega a > 80, la misión queda `ready` con la advertencia de que el margen es pequeño.
2. **Bajar el umbral de M3 a ≥ 80** (cambio de ADR-0011 y de `M3_QUALIFICATION_THRESHOLD`). El fuerte calificaría; débil y medio siguen en 0 por `core-gap`. Hay que comunicarlo antes de abrir.
3. **Ajustar la rúbrica** (`rubric-m3-v1` → `v2`), por ejemplo para que la confianza baja no impida el nivel 4 en cobertura o fidelidad. Requiere recalibrar todo el set de M3.
4. **Reducir la misión** (por ejemplo, sacar la reproducibilidad de los diez primeros a la Misión 4) para tener menos requisitos por prompt. Cambia el alcance aprobado.
5. **Abrir el viernes de todos modos** y registrar el riesgo: es probable que casi nadie supere 80 y que la cola técnica quede vacía.
