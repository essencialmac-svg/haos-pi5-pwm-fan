# Conexión / Wiring

Vista del conector GPIO de 40 pines de la Raspberry Pi 5 (pin 1 junto al logo/esquina).
GPIO header seen from above (pin 1 is the square pad).

```
              3V3  (1) (2)  5V   ◄── Amarillo / Yellow  (+5V)
            GPIO2  (3) (4)  5V
            GPIO3  (5) (6)  GND  ◄── Negro / Black      (GND)
            GPIO4  (7) (8)  GPIO14
              GND  (9) (10) GPIO15
           GPIO17 (11) (12) GPIO18 ◄── Azul / Blue      (PWM)
           GPIO27 (13) (14) GND
           GPIO22 (15) (16) GPIO23 ◄── Verde / Green    (Tach, opcional / optional)
```

| Cable Noctua (4 pines) | Función | Pin físico | GPIO |
|---|---|---|---|
| Amarillo / Yellow | +5 V | 2 | — |
| Negro / Black | GND | 6 | — |
| Azul / Blue | PWM | 12 | GPIO18 |
| Verde / Green | Tacómetro / Tach | 16 (opcional) | GPIO23 |

> ⚠️ Solo ventiladores **PWM de 5 V** (p. ej. Noctua NF-A6x15 5V PWM, NF-A4x10 5V PWM).
> Un ventilador de 12 V no girará bien y uno sin cable PWM (3 pines) no se puede regular.
> Only **5 V PWM** fans. 12 V fans won't spin properly; 3-pin fans can't be speed-controlled.

> ℹ️ Los colores son los del adaptador Noctua. Otros fabricantes usan otros colores: guíate por la función.
> Colors are Noctua's; other brands differ — go by function.
