# blog-vision-robotica
Blog de prácticas - Carla Barbero Martín

## Ejercicio Basic Computer Vision
21/02
Al comenzar la práctica me encontré con un problema inicial: la imagen de salida no se visualizaba correctamente. Aunque el nodo parecía suscribirse al topic sin errores, en la interfaz solo aparecía un icono de imagen sin contenido. Tras reiniciar tanto el entorno de trabajo como el navegador, el problema se resolvió y la imagen comenzó a mostrarse con normalidad, por lo que todo apunta a que se trataba de un fallo puntual del entorno.

Una vez superado este primer inconveniente, pude avanzar con varios de los ejercicios propuestos en el módulo:
- **Change to grayscale:** el resultado fue el esperado, mostrando el vídeo correctamente en escala de grises.
- **Morphological processing:** en este caso sí observé cambios visibles en la imagen, aplicando distintos efectos sobre una imagen con letras. Sin embargo, no llegué a implementar el modo de segmentación en el que los objetos se representan en blanco y negro.
- **Implement a color filter:** probé el filtro con objetos de diferentes colores, pero únicamente funcionó de forma fiable con el color rojo. Mi hipótesis es que el rango HSV no estaba perfectamente ajustado para el resto de colores, lo que hace que este tipo de filtro sea especialmente sensible a los umbrales definidos.
- **Edge filters:** el resultado fue muy satisfactorio, mostrando los bordes de los objetos de forma clara y bien definida.
- **Convolutions:** realicé varias pruebas para mejorar el resultado y, aparentemente, la aplicación de convoluciones sí produce una mejora en la imagen.
- **Hough transform:** probé la transformada de Hough para la detección de círculos. Aunque el círculo principal se detecta correctamente, también aparecen detecciones adicionales debidas al ruido presente en la imagen.
