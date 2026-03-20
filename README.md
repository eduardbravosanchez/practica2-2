
# ⏱️ Práctica 2B ESP32-S3: Interrupciones por Timer

## 📌 Introducción
Este proyecto demuestra el uso de **interrupciones por timer** en el ESP32-S3. El código configura un temporizador hardware que genera una interrupción cada segundo. En cada interrupción, se incrementa un contador y se muestra el número total de interrupciones por el monitor serie.

---

## 📋 Código Principal (main.cpp)

```cpp
#include <Arduino.h>

volatile int interruptCounter;
int totalInterruptCounter;
hw_timer_t * timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;

void IRAM_ATTR onTimer() {
    portENTER_CRITICAL_ISR(&timerMux);
    interruptCounter++;
    portEXIT_CRITICAL_ISR(&timerMux);
}

void setup() {
    Serial.begin(115200);
    
    timer = timerBegin(0, 80, true);
    timerAttachInterrupt(timer, &onTimer, true);
    timerAlarmWrite(timer, 1000000, true);
    timerAlarmEnable(timer);
}

void loop() {
    if (interruptCounter > 0) {
        portENTER_CRITICAL(&timerMux);
        interruptCounter--;
        portEXIT_CRITICAL(&timerMux);
        
        totalInterruptCounter++;
        Serial.print("An interrupt as occurred. Total number: ");
        Serial.println(totalInterruptCounter);
    }
}
```

## Diagrama de Flujo

```mermaid
graph TD
    A[Inicio] --> B[Configuración setup]
    B --> C[Serial.begin 115200]
    C --> D[timerBegin]
    D --> E[timerAttachInterrupt]
    E --> F[timerAlarmWrite 1s]
    F --> G[timerAlarmEnable]
    G --> H[Bucle loop]
    
    H --> I{interruptCounter > 0?}
    I -->|Sí| J[Entrar sección crítica]
    J --> K[interruptCounter--]
    K --> L[Salir sección crítica]
    L --> M[totalInterruptCounter++]
    M --> N[Serial.println]
    N --> H
    I -->|No| H
    
    subgraph Timer [Timer Hardware]
        O[Cada 1 segundo] --> P[Dispara interrupción]
        P --> Q[Ejecuta ISR onTimer]
    end
    
    Q --> R[Entrar sección crítica ISR]
    R --> S[interruptCounter++]
    S --> T[Salir sección crítica ISR]
    T --> U[Retornar]
```


## Diagrama de Tiempos

```mermaid
sequenceDiagram
    participant T as Timer Hardware
    participant C as CPU
    participant S as Serial
    
    Note over C: Setup: Configura timer cada 1s
    
    loop Cada 1 segundo
        T->>C: ⏰ Interrupción de timer
        activate C
        C->>C: ISR: interruptCounter++
        deactivate C
    end
    
    loop loop()
        C->>C: Comprueba interruptCounter
        C->>C: interruptCounter > 0?
        C->>C: interruptCounter--
        C->>C: totalInterruptCounter++
        C->>S: "An interrupt as occurred..."
    end
    
    Note over C: t=1s: Primera interrupción
    T->>C: Interrupción #1
    C->>S: "Total number: 1"
    
    Note over C: t=2s: Segunda interrupción
    T->>C: Interrupción #2
    C->>S: "Total number: 2"
    
    Note over C: t=3s: Tercera interrupción
    T->>C: Interrupción #3
    C->>S: "Total number: 3"
```


## Salida Monitor Serie

An interrupt as occurred. Total number: 1  
An interrupt as occurred. Total number: 2  
An interrupt as occurred. Total number: 3  
...  
