# Wireless Fencing

Sistema de esgrima inalambrica basado en ESP32 para detectar tocados en tiempo real entre dos tiradores.

El proyecto usa una arquitectura cliente-servidor sobre WiFi y UDP:

- Un ESP32 actua como servidor (Access Point + receptor UDP).
- Uno o dos ESP32 actuan como clientes (tirador izquierda/derecha).
- El servidor procesa paquetes, detecta tocados, analiza dobles y controla marcador/tiempo.

## Caracteristicas

- Comunicacion UDP de baja latencia (puertos 4210 y 4211).
- Deteccion de tocado valido/no valido.
- Ventana temporal para doble tocado (300 ms).
- Estadisticas de comunicacion: paquetes perdidos, PPS y porcentaje de perdida.
- Marcador y cronometro para combate.
- Soporte para matriz LED MAX72XX (funciones incluidas en el servidor).

## Estructura del proyecto

```text
Wireless_Fencing/
  Cliente/
    Cliente.ino
    config.h
  Server/
    Server.ino
    maquina.h
  libraries/
    IRremote-4.4.0/
    WiFi-1.2.7/
  .vscode/
    arduino.json
```

## Arquitectura

### Servidor (ESP32)

Archivo principal: `Server/Server.ino`

- Crea un Access Point:
  - SSID: `ESP32_Server`
  - Password: `12345678`
- Escucha paquetes UDP en:
  - Puerto `4210` (tirador izquierda)
  - Puerto `4211` (tirador derecha)
- Espera paquetes de este formato de texto:

```text
ID:<numero>
TICK:<millis>
V0:<valor>
V1:<valor>
```

Logica principal:

- `V0 == 0` indica tocado del florete del tirador emisor.
- `V1` en el rival se usa para validar tocado sobre blanco valido.
- Si hay eventos cercanos, evalua doble tocado dentro de 300 ms.

### Cliente (ESP32)

Archivo principal: `Cliente/Cliente.ino`

- Se conecta al AP `ESP32_Server`.
- Envia paquetes UDP periodicamente (~cada 20 ms).
- Puede configurarse como:
  - `IZQ` (puerto 4210)
  - `DER` (puerto 4211)

## Requisitos

- 2 o 3 placas ESP32 (1 servidor + 1/2 clientes).
- Arduino IDE o extension Arduino en VS Code.
- Core ESP32 instalado (`esp32 by Espressif`).
- Librerias:
  - `WiFi` (incluida en el core ESP32)
  - `MD_MAX72XX` (si se usa la matriz LED)
  - `SPI` (core)

## Configuracion rapida

### 1. Configurar clientes

En `Cliente/Cliente.ino`:

- Define rol del tirador:
  - `#define IZQ` o `#define DER`
- Mantener IP del servidor/AP en `192.168.4.1` (por defecto de SoftAP).

En `Cliente/config.h`:

- SSID y password del AP (por defecto ya apuntan a `ESP32_Server`).

### 2. Configurar servidor

En `Server/Server.ino`:

- Verifica SSID/password del AP.
- Si usas matriz LED MAX72XX, habilita la inicializacion en `setup()` (hay lineas comentadas listas para activar).

## Compilacion y carga

### Opcion A: Arduino IDE

1. Abre `Server/Server.ino` y subelo al ESP32 servidor.
2. Abre `Cliente/Cliente.ino` y subelo a cada ESP32 cliente.
3. Para el segundo tirador, compila cliente con `#define DER`.

### Opcion B: VS Code + extension Arduino

El proyecto ya incluye configuracion en `.vscode/arduino.json` con placa:

- `esp32:esp32:esp32doit-devkit-v1`

Ajusta el puerto serie segun tu equipo y sube cada sketch por separado.

## Flujo de uso

1. Encender el ESP32 servidor.
2. Encender cliente IZQ y cliente DER.
3. Cada cliente se conecta al AP y comienza a enviar paquetes UDP.
4. El servidor procesa tocados y muestra diagnostico por Serial.
5. (Opcional) Visualizar estado en matriz LED si esta habilitada.

## Pines y hardware (resumen)

Definidos en `Server/maquina.h`:

- `CLK_PIN`: 13
- `DATA_PIN`: 11
- `CS_PIN`: 10
- `IR_RECEIVE_PIN`: 12

Nota:

- En comentarios del proyecto aparece mapeo alternativo para ESP32 tipico (`MOSI 23`, `MISO 19`, `SCK 18`, `SS 5`). Ajusta segun tu cableado real.

## Debug y telemetria

Por puerto serie, el servidor imprime:

- Paquetes perdidos por conexion.
- Paquetes por segundo (PPS).
- Porcentaje de perdida.
- Deteccion de tocado valido/no valido.

## Estado actual del codigo

- El motor principal UDP y logica de deteccion esta implementado.
- La parte IR esta en preparacion y comentada en varias secciones.
- Hay comentarios de mejoras futuras (por ejemplo, ajustes de cronometro y visualizacion).
