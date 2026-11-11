# CTRL-LEDs

**Controlador PWM para iluminación LED de banco de trabajo**

CTRL-LEDs es un proyecto de controlador de brillo para tiras LED de **12 V**, basado en el microcontrolador **PIC12F1822**. Utiliza una rueda de encoder incremental con pulsador, una etapa de potencia con MOSFET y memoria EEPROM para conservar la configuración y recuperar el comportamiento deseado después de un corte de alimentación.

> **Estado:** diseño en desarrollo. El hardware, la asignación definitiva de pines y el firmware están pendientes de implementación y validación.

## Objetivos

- Regular el brillo mediante PWM por hardware del PIC12F1822.
- Utilizar un encoder rotativo (canales A/B) para aumentar o disminuir el brillo.
- Encender y apagar la iluminación mediante el pulsador de la rueda.
- Recuperar el último brillo configurado.
- Conservar el estado lógico de encendido/apagado en EEPROM.
- Recuperarse automáticamente de cortes de alimentación **solo si la iluminación estaba encendida** antes del corte.

## Comportamiento esperado

| Acción o situación | Resultado |
| --- | --- |
| Pulsación con iluminación apagada | Enciende con el último brillo guardado. |
| Pulsación con iluminación encendida | Apaga la tira y guarda el estado `OFF`. |
| Giro del encoder con iluminación encendida | Aumenta o disminuye el brillo. |
| Se corta la alimentación mientras estaba encendida | Al volver la alimentación, recupera el estado `ON` y el último brillo persistido. |
| Se corta la alimentación después de apagar mediante pulsador | Al volver la alimentación, permanece apagada hasta una nueva pulsación. |
| Primer arranque o datos EEPROM inválidos | Permanece apagada por seguridad. |

**Nota:** no es necesario detectar físicamente el corte de alimentación: el firmware persiste el *estado deseado* (`ON`/`OFF`). Un corte inesperado no cambia ese estado almacenado.

## Arquitectura propuesta

```text
                  +------------------+
Encoder A --------|                  |
Encoder B --------|    PIC12F1822    |---- PWM ---- MOSFET N ---- Tira LED 12 V
Pulsador ---------|                  |
                  |      EEPROM      |
                  +------------------+
                           |
                     Fuente regulada
                         de 5 V

Fuente de 12 V ------------------------------ Tira LED (+)
GND de 12 V -------- GND del PIC y MOSFET
```

### Componentes principales

- PIC12F1822, con oscilador interno, ADC, EEPROM y PWM por hardware.
- Encoder incremental de dos canales con pulsador (o mecanismo recuperado de un mouse).
- MOSFET canal N **logic-level**, dimensionado según la corriente de la tira.
- Regulador de 12 V a 5 V para el PIC.
- Resistencias de polarización y compuerta; filtrado opcional de las entradas A/B.

### Pines propuestos (por validar)

| Señal | GPIO | Pin físico (DIP-8) |
| --- | --- | --- |
| Encoder A | RA5 | 2 |
| Encoder B | RA4 | 3 |
| Pulsador | RA3 / MCLR | 4 |
| PWM hacia MOSFET | RA2 / CCP1 (según configuración) | 5 |

El pulsador en RA3 requiere configuración de MCLR como entrada digital y una resistencia pull-up externa. La asignación PWM debe comprobarse con la hoja de datos y los bits de configuración del PIC antes de diseñar la placa.

## EEPROM y persistencia

Se propone almacenar como mínimo:

- **Brillo:** valor de ajuste persistido.
- **Estado deseado:** `ON` u `OFF`.

Estrategia prevista:

1. Al encender mediante pulsador, persistir `ON`.
2. Al apagar mediante pulsador, persistir `OFF`.
3. Después de girar el encoder, esperar aproximadamente 2–3 segundos sin cambios antes de guardar el nuevo brillo, evitando escrituras excesivas.
4. En el arranque, validar el registro y restaurar el estado y brillo almacenados.

Para proteger la información frente a un corte durante la escritura, se evaluará un esquema de registros alternativos con verificación de integridad. **Un ajuste aún no guardado puede perderse si la alimentación se interrumpe durante la espera.**

## PWM y entradas

- Frecuencia PWM inicial prevista: **2–4 kHz**, ajustable tras las pruebas.
- Decodificación en cuadratura de A/B, con antirrebote por software.
- Pull-ups para A/B y posibilidad de montar capacitores de filtrado de aproximadamente **10 nF** (a validar con mediciones).
- Encendido progresivo opcional para evitar cambios bruscos de luminosidad.

## Pendientes

- [ ] Definir longitud, potencia y corriente de la tira LED.
- [ ] Identificar el encoder recuperado del mouse y verificar su pinout.
- [ ] Seleccionar MOSFET, regulador, protecciones y componentes de entrada.
- [ ] Validar pinout y configuración de PWM del PIC12F1822.
- [ ] Desarrollar firmware en XC8.
- [ ] Probar antirrebote, persistencia y recuperación tras cortes de alimentación.
- [ ] Diseñar esquemático y PCB en KiCad.

## Herramientas previstas

- **MPLAB X / XC8:** firmware del PIC.
- **KiCad:** esquemático y PCB.
