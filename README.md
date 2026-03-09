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

<img width="450" height="300" alt="Mascara linea" src="https://github.com/user-attachments/assets/05900a70-4530-45b4-9250-1a3d7872e900" />  <img width="500" height="500" alt="Punto centroide a las 21 44 45" src="https://github.com/user-attachments/assets/8c7d9a3f-5ea9-457c-9d3d-ba7a50186e28" />

Finalmente se calcula el centroide de la región detectada utilizando los momentos de la máscara. Este punto representa la posición aproximada de la línea dentro de la imagen y se utiliza posteriormente para calcular el error de seguimiento.

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

[VIDEO SIMPLE CIRCUIT](https://youtu.be/DzrHEuiWoQ4)

Para comprobar la robustez del sistema también se realizaron pruebas en otros circuitos disponibles en el simulador, como _Montreal_ y _Montmeló_. Estos circuitos presentan curvas más pronunciadas y trayectorias más exigentes, lo que permite evaluar mejor el comportamiento del controlador. En ambos casos el robot es capaz de seguir la línea y completar el recorrido de forma razonablemente estable, lo que demuestra que el algoritmo no está ajustado únicamente para un único circuito concreto.

[VIDEO MONTMELÓ](https://youtu.be/oOFyQ14TyKk)

Sin embargo, en el circuito _Nürburgring_ el sistema no consigue completar el recorrido. Desde el inicio, en la primera recta, el robot se desvía hacia la izquierda y pierde inmediatamente la línea, quedándose sin capacidad de continuar el seguimiento. 

[VIDEO NURBURGUIN](https://youtu.be/4LWieVPGa0k)

Además, se ha incluido una demostración del funcionamiento del modo recover forzando al robot a girar completamente a la izquierda al empezar lo que provoca que se posiciones mirando a la pared. En este vídeo se puede observar cómo, cuando el robot pierde la línea, el sistema detecta la ausencia de la máscara y comienza a retroceder mientras gira en la última dirección conocida. Si la línea no aparece inmediatamente, el robot alterna el sentido del giro hasta reencontrarla, permitiendo continuar el recorrido de forma autónoma.

[VIDEO RECOVER](https://youtu.be/RWa0rHS_2go)

#### Conclusiones
Esta práctica ha permitido comprender cómo combinar técnicas de visión artificial y control para resolver un problema de navegación autónoma.

Uno de los aspectos más interesantes ha sido observar cómo pequeños cambios en los parámetros del controlador pueden afectar significativamente al comportamiento del sistema. Ajustar correctamente estos parámetros requiere realizar múltiples pruebas y analizar el comportamiento del robot en diferentes situaciones. También se ha visto la importancia de adaptar la velocidad al contexto del movimiento. Un robot que siempre se mueve a máxima velocidad puede resultar inestable, mientras que ajustar la velocidad en función del giro mejora notablemente la precisión del seguimiento.

Por otro lado, el desarrollo del sistema ha remarcado la importancia de contar con mecanismos de recuperación cuando la detección falla. La mejora del modo recover permitió que el robot pudiera salir con mayor facilidad de situaciones comprometidas y volver a encontrar la trayectoria.
