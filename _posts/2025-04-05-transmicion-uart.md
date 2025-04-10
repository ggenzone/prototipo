---
layout: post
title:  "UART por software en Arduino Mega"
date:   2025-04-05 18:30:00 +0000
categories: [electronica, experimentos]
tags: [UART, bitbanging, serial]
---


## UART por software (bit banging)

En este experimento, exploramos cómo **transmitir datos en serie utilizando un pin digital común (sin usar la librería `Serial`)** y cómo leer esa transmisión en otro UART disponible del **Arduino Mega**. La técnica utilizada se conoce como _bit banging_ y permite generar una señal UART "a mano", controlando el tiempo de cada bit directamente.

---

## 🧪 Objetivo

Transmitir un mensaje desde un pin digital (`pin 8`) generando una señal UART manualmente y recibirlo en el **pin 19 (RX1)** del **Arduino Mega**, utilizando `Serial1`.

---

## 🔌 Conexión

| Arduino Mega      | Conecta a              |
| ----------------- | ---------------------- |
| Pin 8 (TX manual) | Pin 19 (RX1 - Serial1) |
| GND               | GND                    |

> 🛠️ **Opcional:** Se puede colocar una **resistencia de 1kΩ en serie** entre pin 8 y pin 19 para limitar corriente en caso de conflicto de estados. No es estrictamente necesaria si ambos pines están bien configurados.

---

## 📦 Código

```cpp
// CONFIGURACIÓN DE VELOCIDAD Y TIEMPO DE BIT
const long BAUD_RATE = 4800;         // Opciones: 9600, 4800, 2400...
const int BIT_DURATION = 208;        // en microsegundos: 104 p/9600, 208 p/4800

const int TX_PIN = 8;

void setup() {
  pinMode(TX_PIN, OUTPUT);
  digitalWrite(TX_PIN, HIGH); // Línea en reposo

  Serial.begin(9600);          // Debug por USB
  Serial1.begin(BAUD_RATE);    // RX1 en Mega (pin 19)
}

void loop() {
  sendByte('H');
  sendByte('o');
  sendByte('l');
  sendByte('a');
  sendByte('\n');

  delay(1000);

  // Mostrar lo recibido por Serial1 (pin 19)
  while (Serial1.available()) {
    char c = Serial1.read();
    Serial.print("Recibido: ");
    Serial.println(c);
  }
}

/* Transmite un byte por UART usando bit banging
 * 
 * Estructura UART típica:
 * [START] [ D0 D1 D2 D3 D4 D5 D6 D7 ] [STOP]
 *    ↓            ↓ datos ↓             ↓
 *    0     b bits del dato (LSB primero) 1
 *    
 * Ejemplo si 'H' (0x48 = 01001000):
 * TX → __|‾|_|‾‾|_|_|_|‾|‾‾‾‾‾‾ (cada segmento dura BIT_DURATION µs)
 * No usamos bit de paridad
 * */
void sendByte(byte b) {
  digitalWrite(TX_PIN, LOW); // bit de start
  delayMicroseconds(BIT_DURATION);

  for (int i = 0; i < 8; i++) {
    digitalWrite(TX_PIN, (b >> i) & 0x01);
    delayMicroseconds(BIT_DURATION);
  }

  digitalWrite(TX_PIN, HIGH); // bit de stop
  delayMicroseconds(BIT_DURATION);
}
```

---

## 🧠 Problemas comunes y cómo resolverlos

### ❌ Carácter ilegible o ruido (`⸮`, `ÿ`, etc.)

Si ves símbolos extraños en el monitor serial, probablemente se deba a **errores de sincronización temporal**. El UART no tiene reloj, por lo tanto:

- Cada bit debe durar exactamente el mismo tiempo
- El receptor asume la posición de los bits basándose en ese tiempo

### 🔧 Sugerencias

- Probá distintos valores de `BIT_DURATION` (por ejemplo, 100, 104, 106).
- Bajá la velocidad a 4800 o incluso 2400 baudios si sigue fallando.
- Usá cables cortos y conexión directa para evitar ruido.
- Probá transmitir letras simples como `'A'`, `'H'`, o `'U'`, que tienen patrones de bits reconocibles para testear estabilidad.

---

## 📊 Tabla de referencia

| Velocidad (baudios) | Tiempo por bit (µs) |
| ------------------- | ------------------- |
| 9600                | 104                 |
| 4800                | 208                 |
| 2400                | 416                 |

---

## 🔍 ¿Qué aprendimos?

- Cómo funciona el protocolo UART bit a bit
- Qué significa un start bit, stop bit, y por qué el reposo es HIGH
- Cómo sincronizar los tiempos con precisión
- Cómo se transmite un byte manualmente en LSB-first

---

## 📝 Notas técnicas

- Este método se llama comúnmente **bit banging**
- No es fiable a velocidades altas (por errores de timing)
- Ideal para aprender o para situaciones donde no tenés UART hardware disponible

---

## 🎯 Conclusión

Este experimento muestra que es posible **transmitir datos en serie sin usar las funciones UART del Arduino**, lo cual puede ser útil en casos donde el hardware está ocupado o se desea entender el protocolo en profundidad.

El **Arduino Mega** facilita mucho las cosas al contar con múltiples puertos UART reales, evitando conflictos con el USB (como ocurre en el UNO con los pines 0 y 1).
