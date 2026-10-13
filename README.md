# plexo

Multiplexor de 6 potenciómetros para Raspberry Pi Pico.

Usa un 74HC4051 (multiplexor analógico de 8 canales) para leer 6 potenciómetros con una sola entrada analógica (ADC) de la Pico y 3 pines digitales de selección.

## Componentes

| Referencia | Valor | Encapsulado | Notas |
| --- | --- | --- | --- |
| U1 | 74HC4051 | SOIC-16 | LCSC C9386 |
| RV1–RV6 | 10k lineal (B10K) | 9 mm vertical, tipo Alpha RV09 | Se sueldan a mano |
| C1–C6 | 100 nF X7R | 0805 | Filtro en cada potenciómetro, LCSC C49678 |
| C7 | 100 nF X7R | 0805 | Desacople de U1, LCSC C49678 |
| J3 | Conector hembra 1×6 | 2,54 mm | Ver conexiones |

## Conexiones

J3 es un conector hembra de 6 pines, paso 2,54 mm.

| Pin | Señal | Pico (sugerido) |
| --- | --- | --- |
| 1 | 3V3 | 3V3(OUT), pin 36 |
| 2 | salida (COM del multiplexor) | GP26 / ADC0, pin 31 |
| 3 | GND | AGND, pin 33 |
| 4 | S0 | GP2, pin 4 |
| 5 | S1 | GP3, pin 5 |
| 6 | S2 | GP4, pin 6 |

Conviene conectar la tierra de plexo al pin AGND de la Pico (pin 33) en vez de una GND digital: las lecturas salen con menos ruido.

No alimentes los potenciómetros desde ADC_VREF. En la Pico ese pin tiene una resistencia de 200 Ω en serie, y los 6 potenciómetros consumen unos 2 mA en total, lo que hace caer el voltaje unos 0,4 V.

## Tabla de selección

| S2 | S1 | S0 | Canal | Lectura |
| --- | --- | --- | --- | --- |
| 0 | 0 | 0 | A0 | potenciómetro 1 |
| 0 | 0 | 1 | A1 | potenciómetro 2 |
| 0 | 1 | 0 | A2 | potenciómetro 3 |
| 0 | 1 | 1 | A3 | potenciómetro 4 |
| 1 | 0 | 0 | A4 | potenciómetro 5 |
| 1 | 0 | 1 | A5 | potenciómetro 6 |
| 1 | 1 | 0 | A6 | GND (lee 0) |
| 1 | 1 | 1 | A7 | GND (lee 0) |

Los canales A6 y A7 están conectados a tierra. Sirven para revisar cuánto mide el "cero" del ADC.

## Ejemplo en MicroPython

```python
from machine import ADC, Pin
import time

seleccion = [Pin(2, Pin.OUT), Pin(3, Pin.OUT), Pin(4, Pin.OUT)]  # S0, S1, S2
salida = ADC(26)

def leer_potenciometro(canal):
    for bit, pin in enumerate(seleccion):
        pin.value((canal >> bit) & 1)
    time.sleep_us(10)  # tiempo para que se estabilice el multiplexor
    salida.read_u16()  # se descarta la primera lectura
    return salida.read_u16()

while True:
    valores = [leer_potenciometro(canal) for canal in range(6)]
    print(valores)
    time.sleep_ms(50)
```

`read_u16()` entrega valores entre 0 y 65535.

## Notas

- Si el valor de un potenciómetro baja al girarlo en sentido horario, el potenciómetro tiene los pines al revés. Se puede invertir en el código con `65535 - valor`.
- Los potenciómetros genéricos de 9 mm (por ejemplo de AliExpress) varían un poco en la posición de las patas de montaje. Antes de mandar a fabricar, imprime el PCB a escala 1:1 y prueba que calce uno real.

## Archivos

- `plexo-v-0-rev-a/`: proyecto de KiCad (esquemático y PCB).
