---
date: 2025-02-18
categories:
  - Experimentos
tags:
  - Experimentos
  - Spanish
---

# Bot De Apuestas Del Clima - Openclaw (Simmer Market & Polymarket)

### Índice

1. [Requisitos](#requisitos)
2. [Instalar OpenClaw](#instalar-openclaw)
3. [Integrar tu Agente en Simmer Market](#integrar-con-simmer-market)
4. [Estrategia de Apuestas](#estrategia-de-apuestas)
5. [Algo Interesante](#algo-interesante)

<br>


## Requisitos

Antes de comenzar, asegúrate de tener:

- **Telegram** instalado
- **WSL/2 o WSL/1** (solo si eres usuario de Windows)
- **Cuenta en Copilot** (pago) o **ChatGPT** (gratuito)


## Instalar OpenClaw

### Paso 1: Instalar la CLI de OpenClaw
 
Según su [documentación oficial](https://docs.openclaw.ai/install), la instalación es simple y funciona en WSL. Ejecuta el siguiente comando: 

    curl -fsSL https://openclaw.ai/install.sh | bash

<br>

### Paso 2: Configuración Inicial

Una vez instalado, aparecerá el asistente de configuración (onboarding) en la terminal. Deberás seleccionar las siguientes opciones:

```
Security warning      → Sí
Onboarding            → Quick Start
Model provider        → OpenAI (Codex OAuth, inicia sesión con el enlace proporcionado)
Model                 → openai-codex/gpt-5.2
Auth method           → ChatGPT OAuth
Skills                → Sí, selecciona Clawdhub (importante)
Communication channel → Telegram
```

  <br>

  <br>

  ![Paso 0](./assets/images/bets-weather/steps-00.png)

- En la barra de búsqueda de Telegram, escribe "BotFather" y haz clic en él.

  ![Paso 01](./assets/images/bets-weather/steps-01.png)
    
- Ejecuta el comando `/newbot`. El bot te pedirá que nombres tu bot y definas un nombre de usuario (ej: `any_bot`).
  
- Copia el token que te proporciona BotFather. Lo necesitarás cuando el asistente de OpenClaw te solicite el canal de comunicación (Telegram).

  ![Paso 02](./assets/images/bets-weather/steps-02.png)
    
- Finalmente, ejecuta `/start` en el chat con tu bot. Recibirás un código de emparejamiento. Cópialo y ejecuta este comando en tu terminal:

```bash
openclaw pairing approve telegram TU_CÓDIGO
```

  <br>

### Paso 3: Finalizar la Configuración

Continúa con las siguientes opciones:

```
Google Suite → No
Gateway      → Localhost
Interfaz     → TUI
```

¡Listo! Si seguiste todos los pasos correctamente, ya deberías poder chatear con OpenClaw desde Telegram. 


## Integrar con Simmer Market

### ¿Qué es Simmer?

Simmer es un mercado de predicciones diseñado específicamente para agentes de IA. Su característica más destacada es que permite a los agentes **auto-custodiar billeteras digitales** (una nueva tecnología en Web3) y operar de forma autónoma en mercados de apuestas reales. Esto abre posibilidades fascinantes para la experimentación con agentes autónomos que manejan fondos reales.


### Paso 1: Agregar Fondos

1. Conecta tu wallet EVM a la plataforma.
2. Verás dos carteras en tu perfil: una personal y otra para el bot.
3. Deposita **USDC.e** y **POL** **exclusivamente en la red Polygon**.

<br>

!!! note

    Tú decides cuánto depositar. En nuestro caso, depositamos 20 USDC.e y 10 POL para hacer pruebas.

!!! warning

    ⚠️ **Crítico**: Usa únicamente la red Polygon. Si depositas en otra red, **perderás tus criptomonedas**.

![Wallet EVM](./assets/images/bets-weather/steps-03.png)

<br>

### Paso 2: Integrar el Bot

En la interfaz de Simmer, dirígete a: **Página de Inicio** → **Overview Tab** → **Manual Installation**

![Wallet EVM](./assets/images/bets-weather/steps-04.png)

- Haz clic en el botón de copiar del recuadro.
- Ve a tu terminal y pega el comando en OpenClaw, luego presiona Enter.
- Tu bot responderá con un enlace. Haz clic en él para completar la integración.

![Bot integrated](./assets/images/bets-weather/steps-05.png)

✅ **¡Excelente!** Tu bot ya tiene acceso a fondos reales y puede comenzar a operar en Simmer Market.


### Paso 3: Instalar Habilidades (Skills)

En Simmer, ve a la pestaña **Skills** → **Weather Trader** y copia el comando de instalación.

Envía el comando a tu bot de OpenClaw a través de Telegram y espera su respuesta.

## Estrategia de Apuestas

!!! note

    Antes de continuar debo aclarar que en estos momentos mi bot lleva 3 dias funcionando con dinero paper (dinero falso), yo recomiendo decirle al bot que ejecute con "paper" y luego si lo deseas usa dinero real.

### Configuración Actual

A continuación se presenta la configuración que utilizamos actualmente en OpenClaw para operar en Weather Trader, puedes copiar y pegarle esto a tu openclaw si lo deseas:

#### Señales de Entrada y Salida

- **Entry threshold**: 34% (`SIMMER_WEATHER_ENTRY=0.34`)
- **Exit threshold**: 45% (`SIMMER_WEATHER_EXIT=0.45`)

#### Control de Posición

- **Max position size**: $2.00 (`SIMMER_WEATHER_MAX_POSITION=2.00`)
- **Máximo de trades por ciclo**: 3 (`SIMMER_WEATHER_MAX_TRADES=3`)
- **Slippage máximo permitido**: 12% (`SIMMER_WEATHER_SLIPPAGE_MAX_PCT=0.12`)

#### Ubicaciones de Operación (Plan A)

- NYC, CHICAGO, SEATTLE, ATLANTA, DALLAS

#### Seguridad y Activación

- **Safeguards**: Activado ✅
- **Trend detection**: Activado ✅
- **Frecuencia de escaneo**: Cada 30 minutos (optimizado para reducir rate limits)
- **Venue**: Simmer (paper/virtual)

#### Fuentes de Datos Climáticos

- **USA**: Pronóstico principal con NOAA
- **Internacional**: Pronóstico principal con Meteostat
- **Validación cruzada** (solo USA): Máxima diferencia de 2.5°F entre NOAA y Meteostat (`SIMMER_FORECAST_MAX_DIFF`)
- **Límite de consultas Meteostat**: 3 por ciclo (`METEOSTAT_MAX_QUERIES_PER_RUN`)
- **Smart sizing**: Ajusta automáticamente el tamaño para cumplir con el mínimo operativo de 5 shares

### Flujo de Ejecución

¿Cómo funciona el bot en cada ciclo? Así es como el script de OpenClaw procesa y ejecuta las operaciones:

1. **Descubre** mercados de clima activos en Simmer
2. **Agrupa** los mercados por evento (fecha y ciudad)
3. **Obtiene** pronóstico NOAA para cada ciudad/fecha
4. **Mapea** el pronóstico al bucket de temperatura correspondiente
5. **Evalúa salvaguardas** de seguridad:
   - Detección de flip-flop warnings
   - Validación de slippage
   - Verificación: ¿está el mercado cerca de resolución?
6. **Evalúa señal de entrada** (nuevas posiciones):
   - Si precio < $0.15 → **candidato a compra**
7. **Evalúa señal de salida** (posiciones abiertas):
   - Si precio > $0.45 → **candidato a venta**
8. **Ejecuta operaciones** hasta el máximo de `SIMMER_WEATHER_MAX_TRADES` por ciclo
9. **Registra** la actividad en logs

### Gestión de Riesgo

La configuración está diseñada específicamente para **limitar y controlar el riesgo**. En lugar de exponer el 50% del capital en un mal trade, controlamos la exposición para que las pérdidas no superen el ~5% del capital total.

**Límites de riesgo implementados:**

- 💰 **Riesgo máximo por trade**: $2.00
- 🔢 **Tope de operaciones por ciclo**: 3 trades
- 🛡️ **Filtros de seguridad**: Activados por defecto
- ⚠️ **Recomendación**: No incrementes el tamaño de posición sin un historial comprobado de edge (ventaja) estable en backtest

!!! tip

    Comienza con estos parámetros conservadores. Una vez que verifiques que el bot es rentable en papel (paper trading), puedes ajustar gradualmente los límites de riesgo.




