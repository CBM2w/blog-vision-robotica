# blog-vision-robotica
Blog de prácticas - Carla Barbero Martín

## Ejercicio Basic Computer Vision
_21/02/2026_

Al comenzar la práctica me encontré con un problema inicial: la imagen de salida no se visualizaba correctamente. Aunque el nodo parecía suscribirse al topic sin errores, en la interfaz solo aparecía un icono de imagen sin contenido. Tras reiniciar tanto el entorno de trabajo como el navegador, el problema se resolvió y la imagen comenzó a mostrarse con normalidad, por lo que todo apunta a que se trataba de un fallo puntual del entorno.

Una vez superado este primer inconveniente, pude avanzar con varios de los ejercicios propuestos en el módulo:
- **Change to grayscale:** el resultado fue el esperado, mostrando el vídeo correctamente en escala de grises.
- **Morphological processing:** en este caso sí observé cambios visibles en la imagen, aplicando distintos efectos sobre una imagen con letras. Sin embargo, no llegué a implementar el modo de segmentación en el que los objetos se representan en blanco y negro.
- **Implement a color filter:** probé el filtro con objetos de diferentes colores, pero únicamente funcionó de forma fiable con el color rojo. Mi hipótesis es que el rango HSV no estaba perfectamente ajustado para el resto de colores, lo que hace que este tipo de filtro sea especialmente sensible a los umbrales definidos.
- **Edge filters:** el resultado fue muy satisfactorio, mostrando los bordes de los objetos de forma clara y bien definida.
- **Convolutions:** realicé varias pruebas para mejorar el resultado y, aparentemente, la aplicación de convoluciones sí produce una mejora en la imagen.
- **Hough transform:** probé la transformada de Hough para la detección de círculos. Aunque el círculo principal se detecta correctamente, también aparecen detecciones adicionales debidas al ruido presente en la imagen.

## Práctica 1 - Visual Follow Line
_23/02/2026_

El primer día mi objetivo principal fue conseguir detectar correctamente la línea roja mediante una máscara de color. Implementé la segmentación en HSV, definiendo los rangos del rojo para generar una máscara binaria donde únicamente apareciera la línea. Tras varios ajustes en los valores, se consiguió que la detección funcionara satisfactoriamente y que la línea se viera diferenciada claramente en la imagen.

Una vez obtenida la máscara, utilicé la posición de la línea para que el robot comenzara a corregir su trayectoria con un controlador proporcional básico. El resultado fue positivo en rectas, el robot conseguía mantenerse sobre la línea y avanzar sin demasiados problemas. Sin embargo, en las curvas el comportamiento no era tan bueno, de hecho era más bien desastroso. Cuando llegaba a la primera, el robot se salía completamente del circuito y se quedaba estancado sin poder recuperar la trayectoria por sí solo.

El trabajo del primer día sirvió sobre todo para comprobar que la detección funcionaba y que el control básico respondía, pero también dejó claro que el sistema aún no era estable y que las curvas suponían un problema en el recorrido.

_25/02/2026_

El segundo día me centré en intentar solucionar el problema de las curvas. Revisando el comportamiento con más detalle, me di cuenta de que estaba calculando el error haciendo la resta al revés. En lugar de medir correctamente la desviación respecto al centro, estaba invirtiendo el signo. Esto provocaba que el robot corrigiera hacia el lado contrario, lo que explicaba la salida al llegar a la primera curva. Al detectar este fallo entendí que parte del problema no era solo de ajuste del parámetro proporcional, sino de cómo se estaba calculando la desviación.

Una vez corregido el problema de las curvas, ajusté también la forma en la que el robot analizaba la imagen. Empecé a trabajar con una región de interés en la parte inferior para centrarme en la zona más cercana al robot y mejorar la reacción ante los giros. Además, limpié mejor la máscara aplicando operaciones para reducir el ruido, lo que hizo que la detección fuera más estable. Con estos cambios el comportamiento mejoró notablemente, ya no se salía en la primera curva y era capaz de mantenerse dentro del circuito. Sin embargo, apareció un nuevo inconveniente. Aunque ahora completaba el recorrido, lo hacía muy lentamente. El robot tardaba casi 10 minutos en terminar el circuito, con un movimiento demasiado conservador y poco fluido. En ese momento la velocidad establecida era constante y lineal, simplemente para comprobar que el control funcionaba correctamente antes de hacer más ajustes. Se notaba que, aunque el sistema ya era más estable en las curvas, todavía faltaba optimizar la velocidad para que el recorrido fuera más natural y eficiente.

_27/02/2026_

El tercer día me centré en mejorar el rendimiento general del robot. Una vez que ya conseguía completar el circuito sin salirse, el objetivo pasó a ser hacerlo más rápido y con un movimiento más fluido.

Para ello añadí el controlador derivativo (D) además del proporcional. De esta forma el sistema no solo reaccionaba al error actual, sino también a cómo estaba cambiando ese error en el tiempo. Esto ayudó a suavizar las correcciones y reducir los movimientos bruscos que aparecían al entrar en las curvas, donde el coche oscilaba bastante antes de estabilizarse. También añadí un pequeño filtrado al error para evitar que el ruido de la imagen provocara cambios repentinos en el giro. Además, limité la velocidad angular máxima para impedir giros excesivos.

También hice varias pruebas variando los valores de las distintas variables, especialmente las ganancias del controlador y los límites de velocidad. Fue necesario ajustar poco a poco cada parámetro hasta encontrar un equilibrio entre rapidez y estabilidad, ya que pequeños cambios podían hacer que el robot volviera a oscilar o que se volviera demasiado lento.

Uno de los cambios más importantes fue introducir velocidad variable. En lugar de mantener una velocidad lineal constante, la velocidad pasó a depender de cuánto estuviera girando el robot. En rectas, donde el giro es pequeño o inexistente, el robot alcanza la velocidad máxima. En curvas cerradas, al aumentar el giro, la velocidad disminuye automáticamente. Este ajuste hizo que el movimiento fuera mucho más natural.

Otra mejora fue calcular el centroide utilizando solo la parte inferior de la máscara, centrándome en la zona más cercana al robot. Esto permitió una reacción más precisa en curvas y redujo aún más las oscilaciones.

Con todos estos cambios el tiempo de recorrido se redujo prácticamente a la mitad, pasando de casi 10 minutos a aproximadamente 5 minutos por vuelta. Además, el comportamiento general se volvió algo más estable y fluido, aunque aún oscila un poco en las curvas y el tiempo de recorrido es lento.
