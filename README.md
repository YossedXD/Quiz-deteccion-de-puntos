# Quiz-practico-corte3
Detección de posición de una persona con mediapipe, y streamlit 
# deteccion-postura-hilos

Este proyecto demuestra cómo aplicar la programación concurrente (hilos, mutex y semáforos) en una aplicación de visión por computadora.  
El sistema detecta si una persona está **de pie o sentada** en tiempo real, utilizando la cámara del dispositivo y el modelo de **MediaPipe Pose**.  
El procesamiento de imágenes y la interfaz se ejecutan de forma paralela, logrando una respuesta fluida y eficiente gracias a la sincronización entre hilos.

## Variables Principales

El sistema mantiene varias variables globales que controlan la lógica del procesamiento de video y la comunicación entre hilos.

<img width="567" height="212" alt="image" src="https://github.com/user-attachments/assets/1a2c3c11-f7c4-4b39-830e-f3d4b3b91c93" />

*Fragmento de las variables globales*

Entre las más importantes están:

- `buffer_frames`: actúa como una cola donde se almacenan temporalmente los frames capturados.
- `mutex_postura`: evita que varios hilos accedan al buffer o a las variables visuales al mismo tiempo.
- `semaforo_espacios`: indica cuántos espacios disponibles quedan en el buffer.
- `semaforo_items`: indica cuántos frames están listos para ser procesados.
- `ultimo_frame` y `etiqueta_postura`: mantienen la última imagen y la postura detectada que se muestra en pantalla.

Estas variables son compartidas entre los hilos productor y consumidor, por lo que se requiere control de concurrencia para evitar conflictos de acceso o pérdida de datos.

---

## Hilos, Mutex y Semáforos

La aplicación utiliza **dos hilos principales** para dividir el trabajo:

- **Hilo Productor:** se encarga de capturar los frames desde la cámara.
- **Hilo Consumidor:** procesa los frames usando MediaPipe y determina la postura.

Ambos hilos se ejecutan en paralelo, comunicándose a través del buffer de frames.  
El **mutex** protege las secciones críticas, mientras que los **semáforos** controlan la cantidad de frames en espera o disponibles.

<img width="663" height="70" alt="image" src="https://github.com/user-attachments/assets/62df98b5-202f-4aa1-bcf3-347f63c8e96d" />

*Fragmento de inicialización de los hilos y mecanismos de concurrencia*

El **mutex (bloqueo mutuo)** garantiza que solo un hilo acceda a recursos compartidos como el buffer o la imagen mostrada en pantalla.  
El **semaforo**, por su parte, limita la cantidad de frames almacenados al mismo tiempo, evitando desbordamientos o bloqueos del sistema.

---

## Hilo Productor (Captura de Cámara)

El hilo productor se encarga de obtener las imágenes de la cámara usando OpenCV.  
Cada frame capturado se coloca dentro del buffer, siempre y cuando haya espacio disponible (controlado por el semáforo de espacios).

<img width="908" height="263" alt="image" src="https://github.com/user-attachments/assets/5f39c1a1-1c9f-47cf-9f89-9b998d4eb48d" />

*Fragmento del hilo productor*

Durante la inserción, se utiliza el `mutex_postura` para proteger la cola `buffer_frames` y evitar que el hilo consumidor lea al mismo tiempo.  
Si el buffer está lleno, el productor se bloquea hasta que el consumidor libere espacio, garantizando un flujo constante y sincronizado.

---

## Hilo Consumidor (Procesamiento y Detección)

El hilo consumidor toma los frames almacenados en el buffer y los procesa con **MediaPipe Pose**, que detecta los puntos del cuerpo humano.  
A partir de estos puntos, se calcula la distancia vertical entre los hombros y las caderas para determinar si la persona está de pie o sentada.

<img width="1041" height="778" alt="image" src="https://github.com/user-attachments/assets/456f3b48-5b7b-4e63-9c5a-70a4fdb31858" />

*Fragmento del hilo consumidor*

Si el torso tiene una altura considerable (diferencia mayor a 0.30), se clasifica como **persona de pie**; de lo contrario, como **persona sentada**.  
El resultado y el frame procesado se almacenan en variables globales protegidas por el `mutex`, que son actualizadas en la interfaz de Streamlit.

---

## Sección Crítica

En programación concurrente, una **sección crítica** es un bloque de código donde varios hilos podrían acceder al mismo recurso compartido simultáneamente.  
En esta aplicación, las secciones críticas aparecen cuando se modifican o leen variables globales como `buffer_frames`, `ultimo_frame` o `etiqueta_postura`.

<img width="931" height="172" alt="image" src="https://github.com/user-attachments/assets/94ad90a8-9f40-4b0b-9642-b77db42a3ac2" />

*Ejemplo de sección crítica en el código*

El uso del `mutex_postura` garantiza exclusión mutua, impidiendo que los hilos interfieran entre sí.  
Esto asegura que las imágenes mostradas y los resultados de detección siempre sean coherentes y estén sincronizados.

---

## Interfaz con Streamlit

La interfaz se construye con **Streamlit**, permitiendo visualizar en tiempo real la cámara y la postura detectada.  
Incluye dos botones principales:  
- **Iniciar detección:** activa ambos hilos y comienza el procesamiento.  
- **Detener:** detiene la ejecución de los hilos y libera la cámara.

<img width="890" height="312" alt="image" src="https://github.com/user-attachments/assets/f1d93b08-2131-409f-a5c2-3b20533a35b7" />

*Fragmento del control de interfaz con Streamlit*

Streamlit actualiza continuamente el área de video con el último frame procesado, mostrando sobre él los puntos detectados por MediaPipe y la etiqueta con el estado actual.

---

## Lógica de Detección con MediaPipe

El modelo de **MediaPipe Pose** identifica los puntos clave del cuerpo humano, como hombros y caderas.  
La postura se determina midiendo la distancia vertical promedio entre estos puntos.

## Criterio de detección:

- Si el **torso es largo** (`altura_torso > 0.30`) → la persona está **de pie**.  
- Si el **torso es corto** (`altura_torso ≤ 0.30`) → la persona está **sentada**.

Esta comparación se realiza **en cada frame**, lo que permite que el sistema actualice la postura en tiempo real al detectar movimientos de la persona frente a la cámara.
