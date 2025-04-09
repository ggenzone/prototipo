---
layout: post
title:  "Transmitiendo UART sin Serial"
date:   2025-04-05 18:30:00 +0000
categories: [electronica, experimentos]
tags: [UART, bitbanging, serial]
---

> ⚡️ Entender UART desde adentro: bit a bit, sin magia.

Este experimento busca **transmitir datos vía UART sin usar la librería Serial de Arduino**. En lugar de delegar la comunicación a una abstracción, vamos a **generar la señal UART manualmente** con código, usando `digitalWrite()` y `delayMicroseconds()`.

---

## 🎯 Objetivo

Transmitir un byte vía UART desde un pin digital, con formato `9600 8N1`, y **leerlo desde el Monitor Serial del IDE** mediante un adaptador USB-TTL o el propio puerto USB del Arduino.

---

## ⚙️ Materiales

- Arduino UNO o similar
- Cable USB
- (Opcional) Adaptador USB–Serial (FTDI, CH340, etc)
- Cable jumper para conectar TX manual

---

## 🔌 Diagrama de conexión

Conectamos el **pin digital 8 (TX manual)** al pin **RX del adaptador USB–Serial** (o al RX del Arduino si tiene otro puerto serie disponible).

```
Arduino        ↘
Pin 8 (TX)  --->  RX del adaptador USB–Serial
GND          --->  GND
```

> 💡 No conectes este pin al RX de la misma placa si estás usando el puerto USB, puede haber conflicto con el Serial hardware.

---

## 💻 Código: bit banging UART

```cpp
const int txPin = 8; // Pin de transmisión UART "manual"

void setup() {
  pinMode(txPin, OUTPUT);
  digitalWrite(txPin, HIGH); // Línea en reposo
}

void loop() {
  sendByte('H');
  sendByte('o');
  sendByte('l');
  sendByte('a');
  sendByte('\n');
  delay(1000);
}

// Enviar un byte en formato UART 8N1 a 9600 baudios
void sendByte(byte b) {
  const int bitDuration = 104; // microsegundos para 9600 baudios

  digitalWrite(txPin, LOW); // Start bit
  delayMicroseconds(bitDuration);

  // Enviar bits de datos (LSB primero)
  for (int i = 0; i < 8; i++) {
    bool bit = (b >> i) & 0x01;
    digitalWrite(txPin, bit);
    delayMicroseconds(bitDuration);
  }

  digitalWrite(txPin, HIGH); // Stop bit
  delayMicroseconds(bitDuration);
}
```

---

## 🧪 Lectura desde el monitor serial

1. Conectá el pin **TX manual** al **RX del adaptador USB–Serial**
2. Abrí el Monitor Serial del Arduino IDE
3. Configurá la velocidad a **9600 baudios**
4. Deberías ver la palabra **“Hola”** cada segundo

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

## 📦 Próximos pasos

- Recepción UART por software (más complejo, pero posible)
- Ver la señal en un osciloscopio o analizador lógico
- Implementar un protocolo propio sobre UART

---

> 🧠 **Aprender a enviar un byte manualmente es entender realmente qué pasa cuando usás `Serial.print()`.** Todo está hecho de ceros, unos… y paciencia.
```

---
