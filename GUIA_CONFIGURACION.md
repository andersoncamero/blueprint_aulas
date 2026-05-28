# Guía de Configuración - Blueprint Aula Inteligente

Esta guía te llevará paso a paso para configurar el sistema de automatización de aula inteligente en Home Assistant.

---

## 📋 Índice

1. [Requisitos Previos](#requisitos-previos)
2. [Paso 1: Crear Helpers](#paso-1-crear-los-helpers)
3. [Paso 2: Configurar Sensores](#paso-2-configurar-sensores)
4. [Paso 3: Configurar Dispositivos](#paso-3-configurar-dispositivos-de-salida)
5. [Paso 4: Instalar Blueprint](#paso-4-instalar-el-blueprint)
6. [Paso 5: Crear Automatización](#paso-5-crear-la-automatización)
7. [Paso 6: Pruebas](#paso-6-pruebas-y-verificación)
8. [Solución de Problemas](#solución-de-problemas)

---

## Requisitos Previos

Antes de comenzar, asegúrate de tener:

- ✅ Home Assistant instalado y funcionando
- ✅ Acceso de administrador
- ✅ Modo "Advanced Mode" activado (Perfil de Usuario)
- ✅ Dispositivos IoT configurados (sensores, luces, termostatos)

---

## Paso 1: Crear los Helpers

Los helpers son entidades virtuales que almacenan estados. Solo necesitas crear **2 helpers obligatorios**. Los demás son opcionales.

### 1.1 Modo Automático (Obligatorio)

1. Ve a **Configuración** → **Dispositivos y Servicios** → **Helpers**
2. Click en **+ Crear Helper**
3. Selecciona **Conmutador (Toggle)**
4. Configura:
   - **Nombre**: `Modo Auto Aula` (o el que prefieras)
   - **Icono**: `mdi:auto-fix` (opcional)
5. Click en **Crear**

> **Nota**: Este helper controla si la automatización funciona automáticamente (ON) o manual (OFF).

---

### 1.2 Estado del Aula (Obligatorio)

1. Ve a **Configuración** → **Dispositivos y Servicios** → **Helpers**
2. Click en **+ Crear Helper**
3. Selecciona **Menú desplegable (Dropdown)**
4. Configura:
   - **Nombre**: `Estado Aula`
   - **Opciones** (en este orden exacto):
     1. `LIBRE`
     2. `OCUPADO`
     3. `PUERTA_ABIERTA`
     4. `SCAN`
   - **Icono**: `mdi:door-sliding` (opcional)
5. Click en **Crear**

> ⚠️ **IMPORTANTE**: Las 4 opciones DEBEN escribirse exactamente como se muestra, en mayúsculas.

---

### 1.3 Indicador de Cuenta Regresiva (Opcional pero recomendado)

1. Ve a **Configuración** → **Dispositivos y Servicios** → **Helpers**
2. Click en **+ Crear Helper**
3. Selecciona **Número (Number)**
4. Configura:
   - **Nombre**: `Cuenta Regresiva Aula`
   - **Valor mínimo**: `0`
   - **Valor máximo**: `1800`
   - **Paso**: `1`
   - **Unidad de medida**: `segundos`
   - **Modo de visualización**: `Cuadro deslizante`
5. Click en **Crear**

---

### 1.4 Estado Anterior (Opcional - para debug)

1. Ve a **Configuración** → **Dispositivos y Servicios** → **Helpers**
2. Click en **+ Crear Helper**
3. Selecciona **Texto (Text)**
4. Configura:
   - **Nombre**: `Estado Anterior Aula`
   - **Longitud mínima**: `0`
   - **Longitud máxima**: `100`
   - **Patrón**: (dejar vacío)
5. Click en **Crear**

---

### Resumen de Helpers

| Helper | Tipo | Entity ID típico | Obligatorio | Descripción |
|--------|------|------------------|-------------|-------------|
| Modo Auto Aula | Toggle | `input_boolean.modo_auto_aula` | ✅ Sí | Control ON/OFF de la automatización |
| Estado Aula | Dropdown | `input_select.estado_aula` | ✅ Sí | Estados: LIBRE, OCUPADO, PUERTA_ABIERTA, SCAN |
| Cuenta Regresiva Aula | Number | `input_number.cuenta_regresiva_aula` | ❌ No | Visualiza tiempo restante en dashboard |
| Estado Anterior Aula | Text | `input_text.estado_anterior_aula` | ❌ No | Para debug y monitoreo |

> **Nota**: El sensor de puerta también es **opcional**. Si no lo tienes, la automatización funcionará solo con los sensores de movimiento y el control manual.

---

## Paso 2: Configurar Sensores

Necesitas tener sensores configurados en Home Assistant. Dependiendo de tu hardware, la configuración varía.

### 2.1 Sensores de Movimiento

#### Opción A: Sensores Zigbee (vía Zigbee2MQTT o ZHA)

Si usas Zigbee2MQTT, los sensores aparecerán automáticamente como:
- `binary_sensor.sensor_movimiento_aula_occupancy`

#### Opción B: Sensores WiFi (ESP8266/ESP32 con ESPHome)

Crea un archivo en `/config/esphome/sensor-movimiento.yaml`:

```yaml
esphome:
  name: sensor-movimiento-aula
  platform: ESP8266
  board: d1_mini

wifi:
  ssid: "TU_WIFI"
  password: "TU_PASSWORD"

  # Opcional: IP estática
  manual_ip:
    static_ip: 192.168.1.100
    gateway: 192.168.1.1
    subnet: 255.255.255.0

api:
  password: "API_PASSWORD"

ota:
  password: "OTA_PASSWORD"

binary_sensor:
  - platform: gpio
    pin: D5
    name: "Sensor Movimiento Aula"
    device_class: motion
    filters:
      - delayed_off: 30s
```

#### Opción C: Sensores MQTT

Si tienes sensores que publican por MQTT, añade a `configuration.yaml`:

```yaml
binary_sensor:
  - platform: mqtt
    name: "Sensor Movimiento Aula"
    state_topic: "aula/sensor/movimiento"
    payload_on: "ON"
    payload_off: "OFF"
    device_class: motion
```

---

### 2.2 Sensor de Puerta (Opcional)

> **Nota**: Este sensor es opcional. Si no lo tienes, la automatización funcionará igualmente con los sensores de movimiento.

#### Opción A: Sensor Zigbee de puerta

Aparecerá automáticamente como:
- `binary_sensor.puerta_aula_contact` o similar

#### Opción B: ESPHome

```yaml
# Añade a tu configuración ESPHome
binary_sensor:
  - platform: gpio
    pin: 
      number: D6
      mode: INPUT_PULLUP
      inverted: true
    name: "Sensor Puerta Aula"
    device_class: door
```

#### Opción C: MQTT

```yaml
binary_sensor:
  - platform: mqtt
    name: "Sensor Puerta Aula"
    state_topic: "aula/puerta"
    payload_on: "OPEN"
    payload_off: "CLOSED"
    device_class: door
```

---

## Paso 3: Configurar Dispositivos de Salida

### 3.1 Luces

Asegúrate de tener tus luces configuradas como entidades `light.*` o `switch.*`.

Si usas switches para luces, puedes convertirlos a lights en `configuration.yaml`:

```yaml
light:
  - platform: switch
    name: "Luz Principal Aula"
    entity_id: switch.luz_principal
```

### 3.2 Termostatos

Los termostatos deben ser entidades `climate.*`. 

#### Ejemplo con sensores y switches (climate genérico):

```yaml
climate:
  - platform: generic_thermostat
    name: "Aire Acondicionado Aula"
    heater: switch.aire_acondicionado
    target_sensor: sensor.temperatura_aula
    min_temp: 20
    max_temp: 26
    ac_mode: true
    target_temp: 22
    cold_tolerance: 0.5
    hot_tolerance: 0.5
    min_cycle_duration:
      seconds: 120
    keep_alive:
      minutes: 3
```

#### Para termostatos inteligentes (Tuya, Midea, etc.):

Estos suelen integrarse automáticamente mediante sus respectivas integraciones.

---

## Paso 4: Instalar el Blueprint

### Método 1: Mediante la UI (Recomendado)

1. Ve a **Configuración** → **Automatizaciones y Escenas** → **Blueprints**
2. Click en el botón azul **Importar Blueprint** (esquina inferior derecha)
3. Pega esta URL:
   ```
   https://github.com/TU_USUARIO/TU_REPO/blob/main/edu_control.yaml
   ```
   O si lo tienes local, copia el archivo a:
   ```
   /config/blueprints/automation/edu_control.yaml
   ```
4. Click en **Importar**
5. Verás el blueprint "Aula inteligente - Multi dispositivos" en la lista

### Método 2: Manual (Archivo Local)

1. Crea la carpeta si no existe:
   ```bash
   mkdir -p /config/blueprints/automation
   ```

2. Copia el archivo `edu_control.yaml` a:
   ```
   /config/blueprints/automation/edu_control.yaml
   ```

3. Reinicia Home Assistant o recarga los blueprints:
   - **Herramientas de Desarrollador** → **YAML** → **Recargar Blueprints**

---

## Paso 5: Crear la Automatización

1. Ve a **Configuración** → **Automatizaciones y Escenas** → **Automatizaciones**
2. Click en **+ Crear Automatización** (esquina inferior derecha)
3. Click en **Usar Blueprint**
4. Busca y selecciona **"Aula inteligente - Multi dispositivos"**

### Configuración de Parámetros:

#### Sección 1: Sensores

| Parámetro | Selección | Descripción |
|-----------|-----------|-------------|
| **Sensores de Movimiento** | Tus sensores de movimiento | Puedes seleccionar múltiples |
| **Sensor de puerta** | Tu sensor de puerta | Opcional pero recomendado |

#### Sección 2: Control

| Parámetro | Selección | Descripción |
|-----------|-----------|-------------|
| **Modo Automático** | `input_boolean.modo_auto_aula` | El helper toggle que creaste |
| **Estado del Aula** | `input_select.estado_aula` | El helper dropdown |

#### Sección 3: Dispositivos

| Parámetro | Selección | Descripción |
|-----------|-----------|-------------|
| **Iluminación del Aula** | Tus luces | Puedes seleccionar múltiples luces/grupos |
| **Termostatos** | Tus climas | Puedes seleccionar múltiples termostatos |

#### Sección 4: Opciones Avanzadas (Opcionales)

| Parámetro | Valor Recomendado | Descripción |
|-----------|-------------------|-------------|
| **Temperatura Objetivo** | `22` | Temperatura cuando hay presencia |
| **Tiempo de Scan** | `300` | Segundos antes de considerar libre (5 min) |
| **Tiempo apagado puerta** | `300` | Segundos tras abrir puerta (5 min) |
| **Indicador Cuenta Regresiva** | `input_number.cuenta_regresiva_aula` | Helper number opcional |
| **Visualizador Estado Anterior** | `input_text.estado_anterior_aula` | Helper text opcional |

5. Click en **Guardar**
6. Asigna un nombre descriptivo: `"Automatización Aula Inteligente"`

---

## Paso 6: Pruebas y Verificación

### 6.1 Verificar Helpers

1. Ve a **Herramientas de Desarrollador** → **Estados**
2. Busca tus helpers y verifica que existan:
   - `input_boolean.modo_auto_aula`
   - `input_select.estado_aula`
   - Deben mostrar valores válidos

### 6.2 Verificar Sensores

1. En **Herramientas de Desarrollador** → **Estados**
2. Busca tus sensores de movimiento
3. Verifica que cambien de estado al detectar movimiento
4. Verifica el sensor de puerta abriendo y cerrando

### 6.3 Prueba Paso a Paso

#### Prueba 1: Modo Automático Básico

1. Asegúrate de que `input_boolean.modo_auto_aula` esté en **ON**
2. El estado del aula debe estar en **LIBRE**
3. Abre la puerta → Debería:
   - Encender las luces
   - Encender el aire acondicionado
   - Cambiar estado a **PUERTA_ABIERTA** tras el delay

#### Prueba 2: Detección de Presencia

1. Cierra la puerta
2. El estado cambia a **SCAN**
3. Muévete frente a los sensores
4. El estado debería cambiar a **OCUPADO**

#### Prueba 3: Modo Manual

1. Cambia `input_boolean.modo_auto_aula` a **OFF**
2. Abre la puerta → No debería encender nada automáticamente
3. Selecciona manualmente el estado **LIBRE** en el dropdown
4. Verifica que se apague el clima

#### Prueba 4: Timeout

1. Deja el aula vacía sin movimiento
2. Después del tiempo configurado (default 5 min)
3. El estado debería cambiar a **LIBRE**
4. Luces y clima deberían apagarse

---

## Solución de Problemas

### Problema: Los dispositivos no responden

**Causa probable**: Las entidades no existen o están mal escritas.

**Solución**:
```yaml
# Verifica en Herramientas de Desarrollador → Estados que existan:
- binary_sensor.tu_sensor_movimiento
- binary_sensor.tu_sensor_puerta
- light.tu_luz
- climate.tu_termostato
```

---

### Problema: El estado no cambia correctamente

**Causa probable**: El `input_select` no tiene las opciones exactas requeridas.

**Solución**:
Verifica que las opciones sean EXACTAMENTE:
```
LIBRE
OCUPADO
PUERTA_ABIERTA
SCAN
```

---

### Problema: La automatización no se activa

**Causa probable**: El modo automático está en OFF.

**Solución**:
- Ve a **Configuración** → **Dispositivos y Servicios** → **Helpers**
- Busca tu toggle "Modo Auto Aula"
- Actívalo (debe estar azul/encendido)

---

### Problema: El clima no enciende

**Causa probable**: El termostato no acepta el modo "cool".

**Solución**:
Verifica en **Herramientas de Desarrollador** → **Servicios** que el termostato acepte:
```yaml
service: climate.set_hvac_mode
target:
  entity_id: climate.tu_termostato
data:
  hvac_mode: "cool"
```

Algunos termostatos usan: `heat`, `cool`, `auto`, `off`

---

### Problema: Cuenta regresiva no funciona

**Causa probable**: El helper number no está configurado correctamente.

**Solución**:
Asegúrate de que:
- Valor mínimo: 0
- Valor máximo: 1800 (o mayor)
- No esté vacío en la configuración del blueprint

---

## Ejemplo Completo de Configuración

### configuration.yaml

```yaml
# Helpers (también puedes crearlos por UI)
input_boolean:
  modo_auto_aula:
    name: Modo Auto Aula
    icon: mdi:auto-fix
    initial: true

input_select:
  estado_aula:
    name: Estado Aula
    options:
      - LIBRE
      - OCUPADO
      - PUERTA_ABIERTA
      - SCAN
    initial: LIBRE
    icon: mdi:door-sliding

# Helpers opcionales (descomenta si los quieres usar):
# input_number:
#   cuenta_regresiva_aula:
#     name: Cuenta Regresiva Aula
#     min: 0
#     max: 1800
#     step: 1
#     unit_of_measurement: segundos
#     mode: slider
#     icon: mdi:timer-outline

# input_text:
#   estado_anterior_aula:
#     name: Estado Anterior Aula
#     max: 100
#     icon: mdi:history

# Sensores MQTT (ejemplo)
binary_sensor:
  - platform: mqtt
    name: "Sensor Movimiento Aula"
    state_topic: "aula/sensores/movimiento"
    payload_on: "ON"
    payload_off: "OFF"
    device_class: motion
    
  # Sensor de puerta opcional:
  # - platform: mqtt
  #   name: "Sensor Puerta Aula"
  #   state_topic: "aula/sensores/puerta"
  #   payload_on: "OPEN"
  #   payload_off: "CLOSED"
  #   device_class: door

# Conversión de switches a lights
light:
  - platform: switch
    name: "Luz Principal Aula"
    entity_id: switch.rele_luz_principal
    
  - platform: switch
    name: "Luz Trasera Aula"
    entity_id: switch.rele_luz_trasera

# Termostato genérico
climate:
  - platform: generic_thermostat
    name: "Clima Aula"
    heater: switch.aire_acondicionado
    target_sensor: sensor.temperatura_aula
    min_temp: 20
    max_temp: 26
    ac_mode: true
    target_temp: 22
```

---

## Dashboard/Panel de Control (Lovelace)

Crea una tarjeta para monitorear y controlar el aula:

```yaml
type: entities
title: Control Aula Inteligente
entities:
  - entity: input_boolean.modo_auto_aula
    name: Modo Automático
  - entity: input_select.estado_aula
    name: Estado Actual
  - entity: input_number.cuenta_regresiva_aula
    name: Tiempo Restante
  - entity: input_text.estado_anterior_aula
    name: Estado Anterior
  - type: divider
  - entity: binary_sensor.sensor_movimiento_aula
    name: Movimiento Detectado
  - entity: binary_sensor.sensor_puerta_aula
    name: Estado Puerta
  - type: divider
  - entity: light.luz_principal_aula
    name: Luz Principal
  - entity: light.luz_trasera_aula
    name: Luz Trasera
  - entity: climate.clima_aula
    name: Aire Acondicionado
```

---

## Consejos Finales

1. **Siempre prueba primero en modo manual**: Desactiva el modo automático y prueba cada dispositivo individualmente.

2. **Ajusta los tiempos según tu necesidad**: Los 5 minutos por defecto pueden ser mucho o poco según tu uso.

3. **Monitorea el estado anterior**: Es útil para entender qué pasó antes de un cambio de estado.

4. **Usa grupos para múltiples luces**: Si tienes muchas luces, créalas como un `light.group` para simplificar.

5. **Respaldos**: Guarda una copia de tu configuración antes de hacer cambios importantes.

---

## Soporte

Si encuentras problemas:

1. Revisa los **Registros** en **Configuración** → **Sistema** → **Registros**
2. Busca errores relacionados con tu automatización
3. Verifica que todas las entidades existan en **Herramientas de Desarrollador** → **Estados**

---

**¡Listo! Tu aula inteligente debería estar funcionando.** 🎉
