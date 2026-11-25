# Cuaderno de Síntesis — Sistemas Digitales

| Datos | Registro |
| --- | --- |
| Estudiante | Richard Ariel Chamba Santin |
| Unidad | Unidad 2 · Semana 1 de la unidad |
| Semana del curso | Semana 5 |

| Fecha de clase | Tema |
| --- | --- |
| 24/11/2026 | Introducción al procesamiento digital de señales |
| 25/11/2026 | Manejo de pantallas LCD 16×2 |

## 24/11/2026 — Introducción al procesamiento digital de señales

### Síntesis de clase

**Temperatura → LM35 → voltaje analógico → ADC del ESP32 → procesamiento → salida**

El LM35 convierte la temperatura en voltaje. El ADC convierte ese voltaje en una secuencia de números que el ESP32 puede escalar a grados Celsius, filtrar y usar para activar una alarma o mostrar una lectura.

| Etapa | Qué ocurre |
| --- | --- |
| Muestreo | Señal continua → lecturas a intervalos fijos. Cada lectura se representa como `x[n]`. |
| Cuantización | Valor medido → uno de los niveles disponibles. Entre dos niveles hay un escalón de tamaño aproximado `LSB`. |
| Codificación | Nivel seleccionado → código binario. Con 12 bits se obtienen 4096 códigos, de 0 a 4095. |
| Procesamiento | Código o voltaje medido → valor en °C → filtrado → decisión o visualización. |

**Muestreo y Nyquist.** El período `Ts` es el intervalo entre muestras y la frecuencia de muestreo es `fs = 1 / Ts`. Se elige `fs > 2 × fmax`, donde `fmax` es la mayor frecuencia de interés. Un filtro antialiasing antes del ADC atenúa las componentes altas que podrían aparecer como frecuencias falsas.

**ADC del ESP32.** En el ejemplo de clase se usa `GPIO34`, que es una entrada del `ADC1`. En el ESP32 estudiado, `ADC1` se puede usar mientras está activo el Wi‑Fi, que comparte recursos con `ADC2`. La atenuación ajusta el rango de entrada y para el LM35 en el rango de temperatura del aula se analizó el ajuste de 0 dB. `analogRead()` devuelve un valor crudo y `analogReadMilliVolts()` devuelve una lectura calibrada en milivoltios.

### Actividades y ejercicios

| Actividad | Desarrollo |
| --- | --- |
| Calcular la frecuencia de muestreo | `Ts = 100 ms → 0,1 s → fs = 1 / 0,1 = 10 Hz`. Se toman 10 muestras por segundo. |
| Identificar una frecuencia falsa | Señal de `3 Hz` y muestreo a `4 Hz` → se incumple `fs > 6 Hz` → aparece una frecuencia de `abs(3 − 4) = 1 Hz`. |
| Calcular el tamaño de un escalón | `LSB = fondo de escala / 2^N`. Con el valor nominal del ejemplo, `1100 mV / 4096 ≈ 0,27 mV` por escalón. La exactitud también depende del ruido y la calibración. |
| Convertir la lectura del LM35 | `10 mV/°C → 213 mV / 10 = 21,3 °C`. En código se divide por `10.0` para conservar el decimal. |

**Entrada ruidosa → media móvil → salida suavizada.** Se promedian las últimas 10 muestras. Cuando llega una lectura, el búfer circular reemplaza la más antigua y calcula **suma anterior − muestra más antigua + muestra nueva**. Con muestras cada 0,1 s, un cambio tarda alrededor de 1 s en reflejarse por completo en la ventana. Aumentar el número de muestras suaviza más la salida y retrasa su respuesta.

**Temperatura filtrada → decisión con histéresis → LED.** En el ejemplo, el LED se enciende al llegar a 28 °C y se apaga al bajar a 26 °C. Entre ambos umbrales conserva su estado. Así se evitan cambios repetidos por pequeñas variaciones alrededor del umbral.

### Lectura y palabras clave

La lectura relaciona el muestreo y la resolución con el tratamiento de la temperatura antes de tomar una decisión.

- **ADC** — conversor analógico-digital.
- **Aliasing** — aparición de una frecuencia aparente por un muestreo insuficiente.
- **LSB** — bit menos significativo. El tamaño de un LSB equivale a un escalón de cuantización.
- **Media móvil** — promedio de las últimas muestras de una señal.
- **Histéresis** — uso de dos umbrales para cambiar de estado.
- **FFT** — cálculo rápido del espectro de frecuencias, útil para identificar una frecuencia dominante o una interferencia. Se registró como concepto complementario en los apuntes.

## 25/11/2026 — Manejo de pantallas LCD 16×2

### Síntesis de clase

**ESP32 → código del carácter → HD44780 → puntos encendidos en la LCD**

La pantalla tiene **16 columnas × 2 filas = 32 casillas**. Cada carácter se forma en una matriz de **5 × 8 puntos**. El ESP32 envía el código y el controlador HD44780 determina qué puntos encender. Las posiciones se indican como `(columna, fila)` y se cuentan desde cero. La primera casilla es `(0, 0)` y la última es `(15, 1)`.

**Ejemplo de clase.** `A → ASCII 65 → 0100 0001 → carácter visible`. En modo paralelo de 4 bits, primero se envía `0100` y después `0001` por `D4–D7`, cada mitad con su pulso en `E`.

| Memoria | Función |
| --- | --- |
| CGROM | Guarda las formas de los caracteres incorporados de fábrica, como la letra `A`. |
| DDRAM | Guarda los códigos de los caracteres colocados en las posiciones de la pantalla. |
| CGRAM | Permite definir hasta ocho caracteres propios de 5 × 8, como el símbolo de grados. |

### Pines y formas de conexión

| Grupo | Pines del LCD | Función |
| --- | --- | --- |
| Alimentación | 1 `VSS`, 2 `VDD` | Tierra y alimentación. En el módulo de clase, `VDD` es de 5 V. |
| Contraste | 3 `V0` | Ajusta la visibilidad de los caracteres. |
| Control | 4 `RS`, 5 `R/W`, 6 `E` | `RS = 0` selecciona un comando y `RS = 1` un dato. `R/W` selecciona lectura o escritura y se conecta a GND si solo se escribe. El pulso en `E` indica cuándo tomar los bits. |
| Datos | 7–14 `D0–D7` | Transportan el código. En modo de 4 bits se emplean `D4–D7`. |
| Luz de fondo | 15 `A`, 16 `K` | Ánodo y cátodo del LED de iluminación. |

| Aspecto | Paralelo de 4 bits | Con adaptador I²C |
| --- | --- | --- |
| GPIO del ESP32 | Seis GPIO para `RS`, `E` y `D4–D7`. | Dos GPIO para `SDA` (datos) y `SCL` (reloj). En el ejemplo se usan `GPIO21` y `GPIO22`, respectivamente. |
| Cableado | Líneas de control, datos y alimentación. | Cuatro conexiones al módulo mediante `VCC`, `GND`, `SDA` y `SCL`. |
| Contraste | Potenciómetro externo. | Potenciómetro del adaptador. |
| Comunicación | El ESP32 acciona las líneas del HD44780. | El PCF8574 recibe por I²C y acciona las líneas del HD44780. |

**ESP32 → bus I²C → PCF8574 → HD44780.** El adaptador puede responder en direcciones como `0x27` o `0x3F`. Si se desconoce la dirección, se identifica con un escáner I²C.

### Actividad de aplicación — planificar el monitor de temperatura

El ejemplo de las diapositivas distribuye la temperatura y el estado de la alarma en dos filas. Los rótulos se escriben una sola vez y después se actualizan los valores en sus posiciones.

| Fila | Vista de ejemplo | Texto fijo | Dato que se actualiza |
| --- | --- | --- | --- |
| 0 | `Temp: 21.3  C` | `Temp:` y la unidad | Temperatura desde la columna 6. |
| 1 | `Alarma: NO` | `Alarma:` | Estado desde la columna 8. |

Con `LiquidCrystal_I2C` se prepara el módulo mediante `lcd.init()` y se enciende la luz con `lcd.backlight()`. Después, **`lcd.setCursor(col, fila)` → `lcd.print(valor)`** ubica y escribe el dato. Si el valor anterior era `25.10` y el nuevo es `9.5`, se escriben dos espacios después del nuevo valor para ocupar las mismas cinco casillas. Así se borran los dígitos sobrantes sin llamar a `lcd.clear()` en cada actualización.

### Lectura y palabras clave

La lectura amplía el papel del controlador, el cableado mediante I²C y la actualización de campos en la pantalla.

- **LCD 16×2** — pantalla alfanumérica de 16 columnas y dos filas.
- **HD44780** — controlador que interpreta comandos y representa los caracteres.
- **ASCII** — codificación que asigna números a caracteres como letras y dígitos.
- **I²C** — bus de comunicación de dos líneas, `SDA` (datos) y `SCL` (reloj).
- **PCF8574** — expansor de entradas y salidas que adapta I²C a las señales de la LCD.

## Declaración de uso de IA

Para elaborar este cuaderno utilicé ChatGPT como apoyo para transcribir y organizar mis apuntes manuscritos, resumir las dos presentaciones de la semana y ampliar la sección sobre pantallas LCD. Las explicaciones y los ejemplos se basan en los apuntes y el material de clase indicado en cada entrada.
