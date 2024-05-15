# P4

### Objetivo:
El objetivo de la practica es comprender el funcionamiento de un sistema operativo en tiempo Real realizando una practica donde generaremos varias tareas y veremos como se ejecutan dividiendo el tiempo de uso de la cpu.


### Materiales:
ESP32
2 LEDS

### Código:
```cpp
void setup()
{
Serial.begin(112500);
/* we create a new task here */
xTaskCreate(
anotherTask, /* Task function. */
"another Task", /* name of task. */
10000, /* Stack size of task */
NULL, /* parameter of the task */
1, /* priority of the task */
NULL); /* Task handle to keep track of created task */
}
 
/* the forever loop() function is invoked by Arduino ESP32 loopTask */
void loop()
{
Serial.println("this is ESP32 Task");
delay(1000);
}
 
/* this function will be invoked when additionalTask was created */
void anotherTask( void * parameter )
{
/* loop forever */
for(;;)
{
Serial.println("this is another Task");
delay(1000);
}
/* delete a task when finish,
this will never happen because this is infinity loop */
vTaskDelete( NULL );
}
```


### Salida:
```
    this is another Task
    this is ESP32 Task
    this is another Task
    this is ESP32 Task
```

### Descripción:

Este código muestra cómo crear una tarea adicional permitiendo ejecutar múltiples tareas concurrentemente. La tarea adicional imprime un mensaje en serie periódicamente mientras que el bucle principal (loop()) también realiza una tarea similar. Esta tarea adicional se crea a partir de la funcion xTaskCreate y después se la llama en el void anotherTask.


### Código:
```cpp
#include <Arduino.h>
#include <FreeRTOS.h>
#include <task.h>
#include <semphr.h>

// Definiciones de pines
const int ledPin = 23; 

// Declaración de semáforos
SemaphoreHandle_t semaforoEncender;
SemaphoreHandle_t semaforoApagar;

// Prototipos de funciones
void tareaEncender(void *parameter);
void tareaApagar(void *parameter);

void setup() {
  Serial.begin(115200);

  // Inicialización de semáforos
  semaforoEncender = xSemaphoreCreateBinary();
  semaforoApagar = xSemaphoreCreateBinary();

  // Creación de tareas
  xTaskCreate(tareaEncender, "Encender LED", 1000, NULL, 1, NULL);
  xTaskCreate(tareaApagar, "Apagar LED", 1000, NULL, 1, NULL);

  // Inicialización del pin del LED
  pinMode(ledPin, OUTPUT);
}

void loop() {
  // No se utiliza en este ejemplo
}

void tareaEncender(void *parameter) {
  for (;;) {
    // Esperar hasta que se libere el semáforo para encender el LED
    xSemaphoreTake(semaforoEncender, portMAX_DELAY);

    Serial.println("Encendiendo LED");
    digitalWrite(ledPin, HIGH);
    delay(1000); // Espera 1 segundo

    // Liberar el semáforo para permitir que la otra tarea pueda apagar el LED
    xSemaphoreGive(semaforoApagar);
  }
}

void tareaApagar(void *parameter) {
  for (;;) {
    // Esperar hasta que se libere el semáforo para apagar el LED
    xSemaphoreTake(semaforoApagar, portMAX_DELAY);

    Serial.println("Apagando LED");
    digitalWrite(ledPin, LOW);
    delay(1000); // Espera 1 segundo

    // Liberar el semáforo para permitir que la otra tarea pueda encender el LED
    xSemaphoreGive(semaforoEncender);
  }
}
```

### Descripción:
En este código, se crean dos tareas (tareaEncender y tareaApagar) que se ejecutan en bucles infinitos. Cada tarea espera la liberación de un semáforo específico antes de realizar su acción (encender o apagar el LED). Después de realizar la acción, la tarea libera el semáforo correspondiente para permitir que la otra tarea realice su acción. Esto garantiza que las tareas estén sincronizadas correctamente.




