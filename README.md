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

---

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

_02/03/2026_

En el tiempo dedicado hoy tuve que dar un pequeño paso atrás para poder avanzar mejor. Después de tantas pruebas y modificaciones, el código se había vuelto demasiado largo y difícil de seguir. Había añadido cambios sobre cambios, y eso estaba empezando a generar más confusión que mejoras reales. Por eso decidí simplificarlo y limpiarlo, dejando únicamente lo necesario para que el comportamiento fuera claro y controlable.

También añadí un sistema de recover para cuando el robot se sale de la línea. En lugar de quedarse bloqueado, ahora detecta la pérdida de la máscara y realiza un giro en la última dirección conocida hasta volver a encontrar la trayectoria. Esto hace que el sistema sea mucho más robusto y no dependa de que todo salga perfecto en cada curva.

He realizado muchas pruebas variando los parámetros. El objetivo era encontrar un equilibrio entre estabilidad y rapidez sin introducir más oscilaciones, y aunque he reducido el tiempo de recorrido a aproximadamente 1 minuto y medio, aún hace muchas oscilaciones bruscas al entrar y salir de las curvas ya que le cuesta volver a coger el camino al salirse. Además, he visto que pequeños cambios en los valores modifican bastante el comportamiento del robot.

Un inconveniente bastante grande ha sido que, cometando el tiempo de recorrido con un compañero, nos dimos cuenta de que el tiempo que marca el simulador está ligado al navegador, pero el simulador no funciona a velocidad real, sino que va más lento. Esto significa que el tiempo que se muestra no corresponde exactamente al tiempo real del circuito, por lo que no es una medida completamente fiable para evaluar el rendimiento, lo cual ha sido un poco frustrante después de todas las pruebas que he hecho sin éxito de bajar de 2.5 minutos el tiempo de recorrido.

Hoy principalmente me ha servido para ordenar el trabajo que tenía hasta ahora y entender mejor cómo influyen los parámetros en el robot a base de prueba y error.

_05/03/2026_

He realizado más pruebas con los parámetros del controlador. Después de las pruebas del día anterior, me centré especialmente en ajustar el valor del término derivativo para ver si conseguía reducir las oscilaciones que todavía aparecían en las curvas. Probé diferentes configuraciones y finalmente opté por aumentar bastante el valor de Kd. Este cambio dio un resultado bastante bueno, ya que el robot empezó a corregir de forma más suave cuando se desviaba de la línea. Con esto coseguí reducir bastante las oscilaciones al entrar en las curvas, aunque todavía aparece algo de movimiento lateral antes de estabilizarse.

Ya sabiendo que hay que fijarse en el tiempo de simulación para saber cuánto es el tiempo real del recorrido he visto que, con la configuración actual el tiempo de la vuelta se ha reducido bastante y ahora suele estar entre 50 y 60 segundos, dependiendo de la ejecución.

Además, modifiqué el cálculo del ratio que ajusta la velocidad en función del giro. Antes era lineal, pero ahora lo hice cuadrático, de forma que el robot reduce la velocidad mucho más cuando entra en curvas muy cerradas. Con el cálculo anterior, al entrar rápido en esas curvas el robot tendía a salirse bastante y luego oscilaba demasiado intentando volver a la línea. Con el ratio cuadrático el frenado es más fuerte en esas situaciones, lo que ayuda a recuperar la trayectoria de forma más estable, perdiendo también menos tiempo en la recuperación.

Aunque el comportamiento mejoró con respecto a las pruebas anteriores, todavía se aprecia un poco de oscilación en algunas curvas, por lo que probablemente aún sea necesario seguir afinando los parámetros en futuras pruebas.

---

### Solución final del sistema de seguimiento de línea
#### Seguimiento de línea mediante visión y control PD
En esta práctica se ha desarrollado un sistema de control para que un robot móvil sea capaz de seguir una línea en el suelo utilizando visión artificial. El objetivo principal ha sido detectar la trayectoria mediante una cámara frontal y ajustar el movimiento del robot en tiempo real para mantenerse sobre la línea. Para ello se ha implementado un sistema compuesto por tres partes principales:
1.	Procesamiento de la imagen para detectar la línea.
2.	Un controlador PD para calcular la velocidad y el giro del robot.
3.	Un sistema de recuperación cuando la línea mo se encuentra en el campo de visión.

A lo largo de la práctica se han realizado diferentes pruebas y ajustes para mejorar la estabilidad del seguimiento, especialmente en curvas pronunciadas y en situaciones en las que el robot pierde la línea.

#### Procesamiento de la imagen
El robot obtiene continuamente imágenes de la cámara y procesa únicamente una región inferior de la imagen. Esto se hace porque la parte cercana al robot contiene la información más relevante para el control inmediato y permite reducir el ruido y el coste computacional. 

La línea se detecta mediante segmentación por color en el espacio HSV. Se utilizan dos rangos de rojo para capturar correctamente el color, ya que en este espacio de color el rojo aparece dividido en dos zonas del espectro. Una vez obtenida la máscara binaria de la línea, se aplican operaciones morfológicas para eliminar ruido y mejorar la continuidad de la detección. Este paso resulta importante para evitar detecciones erróneas que puedan provocar cambios bruscos en el control del robot. 

<p align="center">
  <img width="450" height="300" alt="Mascara linea" src="https://github.com/user-attachments/assets/05900a70-4530-45b4-9250-1a3d7872e900">

<p align="center">
  <em>Máscara de detección de la línea roja</em>
</p>

Finalmente se calcula el centroide de la región detectada utilizando los momentos de la máscara. Este punto representa la posición aproximada de la línea dentro de la imagen y se utiliza posteriormente para calcular el error de seguimiento.

<p align="center">
    <img width="500" height="500" alt="Punto centroide a las 21 44 45" src="https://github.com/user-attachments/assets/8c7d9a3f-5ea9-457c-9d3d-ba7a50186e28" />
</p>

<p align="center">
  <em>Centroide detectado en la máscara de la línea</em>
</p>

#### Cálculo del error
El error se calcula como la diferencia entre el centro horizontal de la imagen y la posición del centroide de la línea detectada. Si la línea aparece desplazada hacia la izquierda o hacia la derecha, el error es negativo o positivo respectivamente lo que indica cuánto debe girar el robot para volver a centrarla. En el caso idela, cuando la línea está perfectamente centrada, el error es cero. Este error es la variable principal que se utiliza como entrada del controlador.

#### Controlador PD
Para controlar el movimiento del robot se ha utilizado un controlador proporcional-derivativo (PD).

El término proporcional corrige el error actual, generando un giro proporcional a la distancia de la línea respecto al centro de la imagen. Esto permite que el robot se dirija rápidamente hacia la trayectoria. Por otra parte, el término derivativo tiene en cuenta la velocidad de cambio del error. Su función es amortiguar el movimiento y evitar oscilaciones excesivas cuando el robot corrige su trayectoria. Durante las pruebas se observó que aumentar el término derivativo ayudaba a estabilizar el robot, especialmente en curvas, reduciendo el movimiento oscilatorio que aparecía al intentar volver al centro de la línea. Además, se ha limitado el valor máximo del giro para evitar movimientos demasiado bruscos que pudieran hacer que el robot se saliera de la trayectoria.

#### Ajuste dinámico de la velocidad
Otro aspecto importante del comportamiento del robot es la velocidad lineal. En lugar de utilizar una velocidad constante, se implementó una estrategia donde la velocidad depende del giro que está realizando el robot. Cuando el giro es pequeño, el robot puede avanzar más rápido. Sin embargo, cuando el giro aumenta, la velocidad se reduce. Esto permite que el robot tome curvas de forma más estable y evita que se salga de la trayectoria al entrar demasiado rápido en giros cerrados. Durante las pruebas se comprobó que un ajuste cuadrático de esta relación producía mejores resultados que una relación lineal. De esta forma el robot reduce más la velocidad cuando el giro es grande, mejorando el control en curvas muy cerradas.

#### Sistema de recuperación de línea
Uno de los problemas más importantes durante el seguimiento es la pérdida de la línea, algo que puede ocurrir especialmente en curvas cerradas o cuando el robot se desvía demasiado. Para resolver este problema se implementó un modo de recuperación que permite al robot intentar reencontrar la trayectoria de forma autónoma. 

En una primera versión, el recover consistía en avanzar muy despacio mientras el robot giraba hacia el último lado en el que se había detectado la línea. Sin embargo, este enfoque no resolvía el caso en el que el robot quedaba pegado a una pared o mal posicionado respecto a la trayectoria, dificultando la recuperación. Por ello se modificó la estrategia de recover para que, cuando la línea desaparece, el robot retroceda mientras gira. Esta solución resulta más efectiva, ya que permite deshacer parte del error cometido antes de perder la línea y generar más espacio para volver a orientarse correctamente.

Además, el sistema recuerda el último lado en el que se vio la línea. De esta forma el robot inicia la búsqueda girando en la dirección más probable. Si tras un cierto tiempo no la encuentra, alterna el sentido del giro de forma periódica, realizando un pequeño barrido que aumenta las probabilidades de recuperar la trayectoria. 

También se añadió un pequeño ajuste al volver a detectar la línea, si el robot venía del modo _recover_, la parte derivativa del controlador se reinicia momentáneamente. Esto evita que aparezca un pico brusco en el giro debido al cambio repentino del error y hace que la transición entre recuperación y seguimiento normal sea más suave. Esta estrategia de recuperación mejoró el resultado especialmente en situaciones en las que el robot quedaba mal colocado o atrapado cerca de los límites del circuito.

#### Desarrollo de la solución
El sistema final no se obtuvo directamente desde el principio, sino que fue evolucionando a lo largo de distintas pruebas y ajustes.

En una primera fase se implementó únicamente el seguimiento de línea mediante el controlador PD. Aunque el robot era capaz de seguir la trayectoria, aparecían oscilaciones al corregir el error y en algunas curvas el robot tendía a salirse debido a la velocidad. Posteriormente se introdujo la adaptación dinámica de la velocidad en función del giro. Esta modificación permitió que el robot redujera su velocidad al tomar curvas, mejorando notablemente la estabilidad del seguimiento.

Finalmente se trabajó en mejorar el sistema de recuperación cuando la línea desaparecía. Las primeras versiones del recover no siempre conseguían reencontrar la línea con rapidez, por lo que se modificó la estrategia para incluir el retroceso mientras se realiza el giro de búsqueda. Este proceso de prueba y ajuste fue clave para mejorar progresivamente el comportamiento del robot hasta alcanzar una solución más estable y robusta.

#### Validación y pruebas del sistema
Para evaluar el comportamiento del sistema se realizaron diferentes pruebas en varios circuitos del simulador. El objetivo no era únicamente comprobar que el robot podía seguir la línea en condiciones ideales, sino también analizar su comportamiento en trayectorias más complejas y comprobar la robustez del sistema.

En primer lugar se realizaron pruebas en el circuito _Simple Circuit_, que fue el utilizado principalmente durante el desarrollo del algoritmo. Con la configuración final del controlador y del sistema de velocidad, el robot es capaz de completar el recorrido de forma estable en aproximadamente 60 segundos, dependiendo de la ejecución. En este circuito el robot consigue mantenerse sobre la línea durante todo el recorrido y corregir su trayectoria de forma generalmente suave al entrar en las curvas.

<p align="center">
  <a href="https://youtu.be/DzrHEuiWoQ4">
    <img src="https://img.youtube.com/vi/DzrHEuiWoQ4/0.jpg" alt="Video Simple Circuit">
  </a>
</p>

<p align="center">
  <em>Demostración del seguimiento de línea en el circuito Simple Circuit (≈60s).</em>
</p>

Para comprobar la robustez del sistema también se realizaron pruebas en otros circuitos disponibles en el simulador, como _Montreal_ y _Montmeló_. Estos circuitos presentan curvas más pronunciadas y trayectorias más exigentes, lo que permite evaluar mejor el comportamiento del controlador. En ambos casos el robot es capaz de seguir la línea y completar el recorrido de forma razonablemente estable, lo que demuestra que el algoritmo no está ajustado únicamente para un único circuito concreto.

<p align="center">
  <a href="https://youtu.be/oOFyQ14TyKk">
    <img src="https://img.youtube.com/vi/oOFyQ14TyKk/0.jpg" alt="Video Montmeló Circuit">
  </a>
</p>

<p align="center">
  <em>Demostración del seguimiento de línea en el circuito Montmeló.</em>
</p>

Sin embargo, en el circuito _Nürburgring_ el sistema no consigue completar el recorrido. Desde el inicio, en la primera recta, el robot se desvía hacia la izquierda y pierde inmediatamente la línea, quedándose sin capacidad de continuar el seguimiento. 

<p align="center">
  <a href="https://youtu.be/4LWieVPGa0k">
    <img src="https://img.youtube.com/vi/4LWieVPGa0k/0.jpg" alt="Video Nürburgring Circuit">
  </a>
</p>

<p align="center">
  <em>Demostración del seguimiento de línea en el circuito Nürburgring.</em>
</p>

Además, se ha incluido una demostración del funcionamiento del modo recover forzando al robot a girar completamente a la izquierda al empezar lo que provoca que se posiciones mirando a la pared. En este vídeo se puede observar cómo, cuando el robot pierde la línea, el sistema detecta la ausencia de la máscara y comienza a retroceder mientras gira en la última dirección conocida. Si la línea no aparece inmediatamente, el robot alterna el sentido del giro hasta reencontrarla, permitiendo continuar el recorrido de forma autónoma.

<p align="center">
  <a href="https://youtu.be/RWa0rHS_2go">
    <img src="https://img.youtube.com/vi/RWa0rHS_2go/0.jpg" alt="Video Recover">
  </a>
</p>

<p align="center">
  <em>Demostración del modo recover.</em>
</p>

#### Conclusiones
Esta práctica ha permitido comprender cómo combinar técnicas de visión artificial y control para resolver un problema de navegación autónoma.

Uno de los aspectos más interesantes ha sido observar cómo pequeños cambios en los parámetros del controlador pueden afectar significativamente al comportamiento del sistema. Ajustar correctamente estos parámetros requiere realizar múltiples pruebas y analizar el comportamiento del robot en diferentes situaciones. También se ha visto la importancia de adaptar la velocidad al contexto del movimiento. Un robot que siempre se mueve a máxima velocidad puede resultar inestable, mientras que ajustar la velocidad en función del giro mejora notablemente la precisión del seguimiento.

Por otro lado, el desarrollo del sistema ha remarcado la importancia de contar con mecanismos de recuperación cuando la detección falla. La mejora del modo recover permitió que el robot pudiera salir con mayor facilidad de situaciones comprometidas y volver a encontrar la trayectoria.

---

## Práctica 2 - 3D Reconstruction
_11/03/2026_

Como primer paso comencé detectando los bordes en la imagen izquierda utilizando el detector _Canny_. El objetivo de esta parte inicial era simplemente visualizar los bordes obtenidos y comprobar que el detector funcionaba correctamente sobre las imágenes del sistema estéreo.

Una vez verificado el resultado, utilicé estos bordes para seleccionar puntos de interés en la imagen izquierda, ya que las zonas de borde suelen contener más información visual y son más adecuadas para encontrar correspondencias entre ambas imágenes.

_15/03/2026_

Con los puntos de interés ya definidos, el siguiente paso fue buscar sus puntos homólogos en la imagen derecha.

En un primer intento, decidií hacer un caso trivial y asumí que las cámaras estaban rectificadas, por lo que el punto correspondiente debía encontrarse aproximadamente en la misma fila de la imagen derecha. Esto limitaba la búsqueda a una ventana horizontal alrededor del punto original, reduciendo bastante el espacio de búsqueda.

Sin embargo, para hacer el método más robusto y resistente a movimientos de cámara, utilicé geometría epipolar. En lugar de buscar únicamente sobre una línea horizontal, se calcula la recta epipolar asociada a cada punto de la imagen izquierda mediante retroproyección del píxel a un rayo 3D y su posterior proyección en la cámara derecha. Una vez definida la recta epipolar, la búsqueda del punto homólogo se realiza únicamente sobre una banda alrededor de dicha recta, en lugar de recorrer toda la imagen derecha. Esto permite mantener la restricción geométrica del sistema estéreo y al mismo tiempo tolerar pequeños errores numéricos o de discretización.

Para determinar cuál de los candidatos es el punto homólogo correcto, comparé parches de imagen alrededor de cada punto. Para cada punto de interés en la imagen izquierda se extrae un pequeño parche cuadrado, que se utiliza como plantilla. Este parche se compara con los parches candidatos en la imagen derecha utilizando correlación normalizada (`cv2.matchTemplate`). El candidato con mayor valor de correlación se considera el mejor match. No obstante, para evitar correspondencias incorrectas se introduce un umbral de similitud, descartando aquellos candidatos cuyo valor de correlación sea demasiado bajo.

Además, se aplica una restricción geométrica adicional basada en la disparidad horizontal. En un sistema estéreo convencional, el punto correspondiente en la imagen derecha debe aparecer desplazado hacia la izquierda respecto al de la imagen izquierda. Por este motivo, solo se aceptan matches cuya disparidad horizontal sea positiva.

Por último, para mejorar el rendimiento del algoritmo se introducen varias optimizaciones prácticas. Por ejemplo, no se procesan puntos demasiado cercanos a los bordes de la imagen, ya que no permiten extraer parches completos, y se limita el rango de búsqueda horizontal mediante un radio máximo para evitar recorrer regiones innecesarias de la imagen.

_21/03/2026_

Una vez obtenidos los puntos homólogos entre ambas imágenes, el siguiente paso fue hacer la reconstrucción 3D mediante triangulación. A partir de cada par de puntos correspondientes, se calcularon sus respectivos rayos en el espacio y se estimó su punto de intersección para obtener la posición 3D.

En una primera versión, no se tuvo en cuenta el color de los píxeles, por lo que la nube de puntos resultante aparecía completamente en negro. Esto dificultaba bastante la interpretación visual de la reconstrucción, ya que no se apreciaban correctamente las formas ni la estructura de la escena. Posteriormente, se añadió el color a cada punto 3D tomando como referencia el valor del píxel correspondiente en la imagen original. Este cambio mejoró notablemente la visualización, permitiendo distinguir mejor las distintas partes de la escena.

Sin embargo, la reconstrucción seguía siendo bastante pobre. Esto se debía principalmente a la baja cantidad de puntos utilizados, ya que únicamente se estaban considerando un número reducido de correspondencias (alrededor de 4000 puntos). Para mejorar este aspecto, se decidió aumentar significativamente el número de puntos, tomando una muestra aleatoria de aproximadamente 20000 puntos de interés. Con este aumento, la densidad de la nube de puntos creció y la reconstrucción era más completa.

A pesar de esta mejora, al observar el resultado final se aprecia claramente una limitación importante, las zonas internas de los objetos no se reconstruyen bien. Esto pasa porque los puntos de interés utilizados son del detector de bordes, por lo que únicamente se seleccionan píxeles situados en los contornos de los objetos, nunca en el interior.

--- 

### Solución final del sistema de reconstrucción 3D
#### Obtención y preprocesado de imágenes
El primer paso consiste en capturar las imágenes de las cámaras izquierda y derecha mediante el uso de la interfaz proporcionada ```(HAL.getImage)```. Posteriormente, ambas imágenes se convierten a escala de grises, ya que el proceso de búsqueda de correspondencias se realiza sobre intensidad y no sobre color.

A continuación, se aplica el detector de bordes de Canny sobre la imagen izquierda. Esto permite identificar puntos de interés relevantes, ya que los bordes suelen corresponder a zonas con cambios bruscos de intensidad y, por tanto, son más fácilmente identificables en ambas imágenes.

<p align="center">
    <img width="500" height="500" alt="Punto centroide a las 21 44 45" src="https://github.com/user-attachments/assets/80878faa-d6c3-4a8e-9bc6-b57b18ccc1ac" />
</p>

<p align="center">
  <em>Detección de bordes</em>
</p>

#### Selección de puntos de interés
Una vez obtenidos los píxeles de borde, se extraen sus coordenadas utilizando ```np.where```. Inicialmente, se probó a trabajar con un subconjunto muy reducido de puntos (uno de cada 20), obteniendo alrededor de 900 puntos, con el objetivo de reducir el tiempo de ejecución. Sin embargo, la reconstrucción obtenida resultaba muy incompleta, ya que se perdía demasiada información de la escena.

<p align="center">
    <img width="500" height="500" alt="Punto centroide a las 21 44 45" src="https://github.com/user-attachments/assets/72b98133-eacb-4af5-96dc-f6b3ecab15bd" />
</p>

<p align="center">
  <em>Selección de puntos 1 de cada 20</em>
</p>

Posteriormente, se optó por seleccionar aproximadamente 20000 puntos de forma aleatoria. Este enfoque mejoró notablemente los resultados, ya que se obtenía una nube de puntos más densa y representativa de la escena.

<p align="center">
    <img width="500" height="500" alt="Punto centroide a las 21 44 45" src="https://github.com/user-attachments/assets/7ce32c05-bc56-4dad-a4f1-66465809a440" />
</p>

<p align="center">
  <em>Selección de 20000 puntos aleatorios</em>
</p>

Finalmente, por recomendación de un compañero, se probó una estrategia intermedia consistente en seleccionar un punto sí y uno no ```(points[::2])```. Con esta técnica se consigue una distribución más uniforme de los puntos. 

<p align="center">
    <img width="500" height="500" alt="Punto centroide a las 21 44 45" src="https://github.com/user-attachments/assets/45216113-abf4-4d4c-8fd3-1b66343f172f" />
</p>

<p align="center">
  <em>Selección de puntos uno sí uno no</em>
</p>

#### Cálculo de la recta epipolar
Para cada punto seleccionado en la imagen izquierda, se calcula su recta epipolar correspondiente en la imagen derecha. Este proceso se realiza en varios pasos:
1. Se obtiene el rayo 3D correspondiente al píxel mediante retroproyección.
2. A partir de ese píxel, se generan dos puntos sobre ese rayo en el espacio (la posición de la cámara y la dirección del rayo).
3. Estos puntos se proyectan sobre la cámara derecha.
4. Los dos puntos resultantes definen la recta epipolar en la imagen derecha.


De este modo, la búsqueda del punto homólogo no se realiza en toda la imagen, sino únicamente sobre una región coherente con la geometría del sistema estéreo.

<p align="center">
    <img width="500" height="500" alt="Punto centroide a las 21 44 45" src="https://github.com/user-attachments/assets/14e10c69-c78a-4468-bd93-87a051e49336" />
</p>

<p align="center">
  <em>Submuestra de rectas epipolares</em>
</p>

#### Búsqueda de correspondencias
Una vez calculada la recta epipolar, se busca el punto homólogo en la imagen derecha. Para ello, alrededor del punto de la imagen izquierda se extrae un parche cuadrado de tamaño fijo 15x15. A continuación, se recorre la recta epipolar en la imagen derecha dentro de un rango horizontal limitado y, además, se considera una pequeña banda vertical alrededor de dicha recta para tolerar pequeños errores geométricos (franja epipolar).

En cada candidato se extrae un parche y se compara con el parche original de la imagen izquierda mediante ```cv2.matchTemplate``` con correlación normalizada. El punto con mayor puntuación se toma como la mejor correspondencia candidata, determinando así los puntos homólogos.

#### Validación de correspondencias
No todas las correspondencias obtenidas son válidas, por lo que se aplica una fase de validación antes de aceptar el punto. Se han empleado tres condiciones:

- La puntuación de correlación debe ser superior a un umbral ```(threshold = 0.7)```.
- La disparidad horizontal debe ser positiva y suficientemente grande ```(dx > 2)```.
- La diferencia vertical entre ambos puntos debe ser pequeña ```(abs(dy) < 4)```.

Estas restricciones permiten eliminar muchos falsos emparejamientos y conservar únicamente correspondencias geométricamente consistentes.

#### Triangulación
Una vez validada la correspondencia entre el punto izquierdo y el derecho, se reconstruye el punto 3D utilizando la función ```cv2.triangulatePoints```. Para ello, primero se definen las matrices de proyección de ambas cámaras a partir de sus parámetros intrínsecos y extrínsecos. En concreto, se parte de las matrices K y RT de cada cámara y se construye la matriz de proyección como:

$$P=K⋅RT$$

Posteriormente, los píxeles emparejados de las cámaras izquierda y derecha se convierten de coordenadas gráficas a coordenadas ópticas, y se pasan a la función ```cv2.triangulatePoints``` en formato homogéneo. El resultado es un punto 4D homogéneo, que después se normaliza para obtener sus coordenadas cartesianas 3D.

#### Corrección del sistema de referencia y visualización
Una vez obtenido el punto 3D, se transforma al sistema de referencia usado por el visor mediante ```HAL.project3DScene```. Durante las pruebas se observó que la nube de puntos aparecía invertida verticalmente, por lo que fue necesario corregir la orientación invirtiendo el eje Y. Además, se añadió un pequeño desplazamiento vertical para elevar la escena y facilitar su visualización. De esta forma, antes de almacenar el punto reconstruido se aplica una corrección sobre la coordenada vertical para que la nube final se muestre correctamente orientada y mejor centrada en el visor.

#### Asignación de color
A cada punto 3D se le asigna un color calculado como la media entre el color del píxel correspondiente en la imagen izquierda y el de la imagen derecha. Dado que OpenCV maneja el color en formato BGR, al final se invierte el orden de los canales para almacenarlo en RGB, que es el formato esperado en la visualización de la nube.

De este modo, cada punto final queda definido por seis valores:

$$[x,y,z,r,g,b]$$

lo que permite reconstruir no solo la geometría de la escena, sino también su apariencia visual.

#### Resultados obtenidos
El procedimiento desarrollado permite obtener una nube de puntos 3D razonablemente representativa de la escena. El uso de la geometría epipolar reduce el espacio de búsqueda y hace más eficiente el proceso de correspondencia. Además, el empleo de la función de triangulación de OpenCV proporciona una triangulación robusta que la aproximación manual utilizada en versiones previas.

No obstante, la reconstrucción sigue dependiendo en gran medida de la calidad de los bordes detectados y de la capacidad del método de correlación para encontrar correspondencias correctas. Por ello, las zonas con poca textura o sin bordes marcados tienden a quedar menos representadas que las regiones con contornos bien definidos, como es la figura del pato.

A continuación se muestran los resultados obtenidos para las distintas estrategias de selección de puntos:
<p align="center">
    <img width="500" height="500" alt="Punto centroide a las 21 44 45" src="https://github.com/user-attachments/assets/00055a3e-ec0b-4405-95b1-7bf9e98f5335" />
</p>

<p align="center">
  <em>Reconstrucción con 900 puntos</em>
</p>

<p align="center">
    <img width="500" height="500" alt="Punto centroide a las 21 44 45" src="https://github.com/user-attachments/assets/4bec3170-a8f0-4880-8a3b-4da43d53a729" />
</p>

<p align="center">
  <em>Reconstrucción con 20000 puntos</em>
</p>

<p align="center">
    <img width="500" height="500" alt="Punto centroide a las 21 44 45" src="https://github.com/user-attachments/assets/b20b5959-9400-48f6-b8f7-457515451f59" />
</p>

<p align="center">
  <em>Reconstrucción con 8700 puntos</em>
</p>

Se observa que algunas figuras se completaban ligeramente mejor utilizando el segundo enfoque, aunque el cambio en la calidad global no era muy significativo. Por este motivo, se optó por mantener la estrategia de seleccionar puntos uno sí uno no, ya que ofrece un buen equilibrio entre calidad de reconstrucción y tiempo de ejecución. Esto se debe a que el número de puntos procesados se reduce notablemente, disminuyendo así el coste computacional del algoritmo sin comprometer en exceso la calidad del resultado.
  
#### Conclusiones
El sistema desarrollado permite realizar una reconstrucción 3D de la escena a partir de imágenes estéreo de forma efectiva, combinando técnicas de procesamiento de imagen, geometría epipolar y triangulación. A lo largo del desarrollo se ha comprobado la importancia de cada una de las etapas del pipeline, especialmente la selección de puntos de interés y la búsqueda de correspondencias.

Uno de los aspectos más relevantes ha sido el compromiso entre calidad de la reconstrucción y tiempo de ejecución. Se ha observado que utilizar un número elevado de puntos mejora la densidad de la nube, pero incrementa significativamente el coste computacional. Por el contrario, reducir excesivamente el número de puntos produce reconstrucciones incompletas. La estrategia intermedia de seleccionar un punto sí y uno no ha resultado ser una solución equilibrada, permitiendo mantener una representación razonable de la escena con un tiempo de ejecución mucho menor.

Asimismo, el uso de la geometría epipolar ha permitido restringir el espacio de búsqueda de correspondencias, mejorando la eficiencia del algoritmo y reduciendo la probabilidad de emparejamientos incorrectos. La validación de correspondencias mediante restricciones geométricas y de correlación ha sido clave para eliminar falsos matches y obtener resultados más consistentes.

En cuanto a la reconstrucción, se ha conseguido obtener una nube de puntos que representa de forma bastante fiel la estructura general de la escena, especialmente en aquellas zonas con mayor presencia de bordes. Sin embargo, también se ha observado que las regiones con menor textura o sin cambios de intensidad claros quedan menos definidas o incluso vacías.

En general, el resultado final es satisfactorio, ya que permite visualizar correctamente la forma de los objetos principales de la escena. Además, todo el proceso ha servido para comprender mejor cómo influyen cada una de las decisiones tomadas en la calidad de la reconstrucción y en el rendimiento del sistema.
