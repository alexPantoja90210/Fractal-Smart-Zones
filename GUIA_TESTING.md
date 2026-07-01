# Guía de Testing Paso a Paso — Fractal Smart Zonez
*by Vic-Forex | Pine Script v5 | Arquitectura OOP*

---

## Filosofía de Testing por Versiones

Cada versión prueba **una sola capa** de la estrategia. No avances a la siguiente versión hasta que la actual funcione correctamente. Este enfoque permite aislar errores y entender qué parte del sistema falla.

---

## VERSIÓN 1: `v1_fractal_detector.pine`
**Objetivo:** Verificar que los fractales se detectan correctamente

### Cómo instalarlo
1. Abre TradingView → Pine Editor
2. Pega el contenido de `v1_fractal_detector.pine`
3. Haz clic en "Add to chart"

### Instrumento y temporalidad
- **Primario:** XAUUSD, 30 minutos
- **Secundario:** EURUSD, 1H

### Qué verificar
- [ ] Los triángulos rojos (↓) aparecen en máximos locales reales
- [ ] Los triángulos azules (↑) aparecen en mínimos locales reales
- [ ] Los cuadros rectangulares representan el rango de un impulso real
- [ ] La tabla muestra `Fractales detectados > 0`
- [ ] Los cuadros azules = estructura alcista (SL antes del SH)
- [ ] Los cuadros rojos = estructura bajista

### Parámetros a ajustar
| Parámetro | Valor inicial | Rango | Efecto |
|-----------|--------------|-------|--------|
| Lookback Fractal | 5 | 3-8 | Más alto = fractales más grandes y menos frecuentes |

### Criterio de aprobación
✅ Los fractales coinciden visualmente con los impulsos/retrocesos del PDF (página 1)

---

## VERSIÓN 2: `v2_fibo_zone.pine`
**Objetivo:** Validar el cálculo de la zona dorada 50%-61.8%

### Verificación cruzada (IMPORTANTE)
1. Aplica el indicador nativo de TradingView: `Fibonacci Retracement`
2. Dibuja manualmente el FIBO desde el swing low al swing high del último fractal detectado
3. Compara los niveles 50% y 61.8% del indicador nativo con los de la estrategia

### Qué verificar
- [ ] La zona amarilla aparece dentro del retroceso esperado
- [ ] Los niveles 50% y 61.8% coinciden con el FIBO manual (tolerancia: 0.0001)
- [ ] El fondo amarillo de la pantalla aparece cuando el precio toca la zona
- [ ] En el cuadro "Precio zona" aparece "DENTRO" cuando corresponde

### Casos de prueba
Identifica estos 3 escenarios en el gráfico y verifica:
1. **Fractal alcista reciente**: La zona dorada debe estar entre el SH y el SL
2. **Fractal bajista reciente**: La zona debe estar entre el SL y el SH (invertido)
3. **Precio atravesando la zona**: El fondo debe iluminarse al entrar

### Criterio de aprobación
✅ Niveles coinciden con FIBO nativo de TradingView en ±0.001

---

## VERSIÓN 3: `v3_confluences.pine`
**Objetivo:** Detectar confluencias dentro de la zona FIBO

### Proceso de validación manual
Para cada confluencia, verifica manualmente en el gráfico:

#### Order Block (morado)
1. Busca la última vela bajista (para alcista) antes del impulso
2. Debe estar dentro o cerca de la zona 50%-61.8%
3. ¿El cuadro morado cubre esa vela?

#### FVG / Imbalance (naranja)
1. Busca zonas donde hay 3 velas y la mecha de la vela 1 no se superpone con la mecha de la vela 3
2. ¿El cuadro naranja cubre ese espacio?

#### Pivot S/R (cian)
1. ¿Hay algún máximo o mínimo histórico relevante en la zona dorada?
2. ¿La línea cian puntea por ahí?

### Parámetros a calibrar
| Parámetro | XAUUSD 30m | EURUSD 1H | Efecto |
|-----------|-----------|-----------|--------|
| Lookback OB | 3-5 | 2-3 | Más alto = busca más atrás |
| FVG mínimo | 0.5-2.0 | 0.0001-0.0005 | Filtra FVGs pequeños |
| Confluencias mínimas | 2 | 2 | No cambiar |

### Criterio de aprobación
✅ En al menos 5 setups revisados manualmente, el score coincide con lo que ves visualmente
✅ El fondo verde aparece cuando hay ≥2 confluencias reales

---

## VERSIÓN 4: `v4_signal_entry.pine`
**Objetivo:** Verificar señales de entrada con CHoCH y vela de rechazo

### Flujo de verificación
Para cada señal de entrada (flecha), verifica:

```
¿Había fractal válido antes?     → SÍ
¿Precio entró a zona FIBO?       → SÍ
¿Había ≥2 confluencias?          → SÍ
¿Se detectó CHoCH (×)?           → SÍ (antes de la flecha)
¿La vela de entrada tiene mecha? → SÍ
¿Estructura general intacta?     → SÍ (verificar manualmente)
```

### Tipos de vela a reconocer
1. **pin_bar**: Mecha inferior larga (≥60% del rango total), cuerpo pequeño (≤35%)
2. **engulfing**: Vela alcista que envuelve completamente la vela anterior
3. **wick_rejection**: Mecha inferior > 2× el cuerpo

### Verificar niveles SL/TP
- SL debe estar BAJO el Order Block (o bajo el swing low si no hay OB)
- TP1 = 2R desde el entry
- TP2 = 4R desde el entry
- ¿Estos niveles tienen sentido estructuralmente?

### Red flags (señales inválidas)
- [ ] La flecha aparece cuando la estructura ya fue rota
- [ ] El SL está demasiado ajustado (< 3 pips en Forex)
- [ ] No hay CHoCH visible antes de la flecha
- [ ] La zona FIBO está muy cerca del SL del fractal anterior

### Criterio de aprobación
✅ 8 de 10 señales revisadas manualmente cumplen todas las condiciones
✅ No hay señales "fantasmas" (entradas sin confluencias reales)

---

## VERSIÓN 5: `v5_strategy_backtest.pine`
**Objetivo:** Backtest estadístico completo

### Configuración del backtester de TradingView
```
Propiedades del backtester:
- Capital inicial: $10,000
- Comisión: 0.07% por lado (spreads Forex)
- Slippage: 2 ticks
- Order size: 2% del capital
```

### Periodos de prueba por instrumento
| Instrumento | TF | Periodo mínimo | Trades esperados |
|-------------|-----|---------------|-----------------|
| XAUUSD | 30m | 6 meses | 30-80 |
| EURUSD | 1H | 12 meses | 40-100 |
| GBPUSD | 4H | 24 meses | 25-60 |

### Métricas objetivo para versión estable

| Métrica | Mínimo aceptable | Bueno | Excelente |
|---------|-----------------|-------|-----------|
| Win rate | 40% | 50% | 60%+ |
| Expectancy | 0.20 R | 0.35 R | 0.50 R+ |
| Net RR | > 0 | > 10R | > 20R |
| Max consec. SL | ≤ 6 | ≤ 4 | ≤ 3 |
| Profit Factor | > 1.3 | > 1.6 | > 2.0 |

### Proceso de optimización de parámetros
**IMPORTANTE**: Optimizar en este orden exacto para evitar overfitting:

```
PASO 1: Fija todo, optimiza solo i_lookback (3, 4, 5, 6, 7)
PASO 2: Fija lookback, optimiza i_min_conf (1, 2, 3)
PASO 3: Fija los anteriores, optimiza i_wick_ratio (0.5, 0.6, 0.7)
PASO 4: Optimiza i_rr_tp1 (1.5, 2.0, 2.5, 3.0)
PASO 5: Optimiza i_rr_tp2 (3.0, 4.0, 5.0)
```

⚠️ **Anti-overfitting**: Optimiza en los primeros 2/3 del periodo de datos.
Valida los parámetros óptimos en el último 1/3 sin tocarlos.

### Interpretación de resultados del Strategy Tester
```
✅ Señales positivas:
- Net Profit > 15% del capital inicial (en 12 meses)
- Profit Factor > 1.4
- Sharpe Ratio > 1.0
- Max Drawdown < 20%

⚠️ Señales de ajuste necesario:
- Profit Factor 1.1-1.4 → ajustar i_min_conf o i_wick_ratio
- Win rate < 35% → aumentar selectividad (subir i_min_conf)
- Max Drawdown > 25% → reducir risk% o ajustar SL buffer

❌ Señales de problema estructural:
- Net RR negativo en +6 meses → revisar lógica de CHoCH
- Max consec. SL > 8 → revisar filtro de estructura general
```

---

## Proceso para llegar a versión estable

```
v1 (fractales)
  ↓ ¿fractales correctos?
v2 (fibo)
  ↓ ¿zona dorada exacta?
v3 (confluencias)
  ↓ ¿OB/FVG se detectan bien?
v4 (señal)
  ↓ ¿CHoCH + rechazo válidos?
v5 (backtest)
  ↓ ¿métricas en rango aceptable?
Optimización de parámetros
  ↓ ¿hold-out period positivo?
✅ VERSIÓN ESTABLE
```

---

## Notas importantes de Pine Script v5

### Limitaciones conocidas del backtester
1. **Repainting**: Los fractales se calculan con `lookback` barras de lag. Esto es real (no repainting), pero significa que la entrada siempre llega `i_lookback` barras tarde.
2. **Datos históricos de M5**: Si usas CHoCH de M5 en un script de 30m, Pine no tiene acceso a datos interbarra. La lógica de CHoCH es una aproximación en la misma temporalidad.
3. **Max bars back**: Con arrays y UDTs, el limit de 500 barras de historial puede afectar setups muy antiguos.

### Mejoras posibles para v6
- [ ] Multi-timeframe real: confirmar CHoCH en timeframe inferior con `request.security()`
- [ ] Filtro de volatilidad: ATR para ajustar mínimos de zona
- [ ] Filtro de sesión: solo operar en horario de mayor liquidez
- [ ] Dashboard de liquidez: proyectar niveles de liquidez objetivo (equal highs/lows)
- [ ] Modo de alerta: notificaciones en tiempo real cuando la zona se activa
