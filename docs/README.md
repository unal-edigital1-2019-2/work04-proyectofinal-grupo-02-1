# Electrónica Digital - Grupo 02

* David Miguel García Palacios - 1001315936
* Javier Andres Africano Pachón - 1193381320
* David Ricardo Lugo Venegas - 1075872618

----
# DOCUMENTACION PROYECTO FINAL

Se ejecuta una serie de actividades o “paquetes de trabajos” que tienen como  línea de enfoque  permitirnos desarrollar una cámara digital en su esencia más simple ayudados de herramientas tales como una tarjeta programable (Nexys 4 - FPGA),  Arduino y una cámara OV7670.

## Planteamiento del Problema
Realizar la síntesis de hardware (ayudados por la aplicación Xilinx) para recibir los datos de la cámara OV7670 con el protocolo I2C,  decodificarlos y proyectarlos en una pantalla con el estándar de video VGA. 

## Objetivos
* Lograr la visualización en una pantalla con entrada VGA de una foto tomada por la cámara OV7670, haciendo uso de una FPGA y su respectivo código en Verilog.
* Diseñar y crear un módulo de captura de la cámara que se encargue de recibir los datos de la imagen de la cámara, adecuarlos de formato RGB 565 a RGB 332, y guardarlos en la memoria RAM Dual Port
* Configurar los registros de la la cámara OV7670, haciendo uso de una tarjeta programable Arduino, ajustando correctamente sus parámetros de escalado (Formato CIF) y formato del pixel (RGB 565).

## Elementos de Trabajo
#### Hardware
* Tarjeta Nexys-4 (500) - FPGA: Artix 7
* Camara OV7670
* Tarjeta Programable Arduino UNO 
*  Pantalla VGA
#### Software
* Xilinx ISE
* Arduino Genuino IDE
#### Documentación
* Datasheet OV7670
* Datasheet Nexys-4

## Descripción de Modulos

Haciendo uso de una cámara y una tarjeta programable, la cual a partir de una señal de botón controlada por el usuario, realiza captura de una imagen de 320 x 240 pixeles de resolución, que se puede visualizar en una pantalla con conexión VGA, donde si dicho botón es oprimido por el usuario continuamente, la toma de imagen de igual manera se realiza constante dando opción de visualizar video en dicha pantalla.

![alt text](https://github.com/unal-edigital1-2019-2/work04-proyectofinal-grupo-02-1/blob/master/docs/figs/esquema%20global.png)

#### Camara OV7670

El módulo de cámara OV7670 está basado en el sensor OV7670 de omnivision. Es capaz de generar un máximo de 30 cuadros por segundo a una resolución de 640×480 pixeles (VGA). El OV7670 es un sistema en chip (SoC) por lo que además de ser un sensor de imagen, en su interior también encontramos circuitos destinados al procesamiento de la imagen capturada, por esta razón los procesos de control se realizan en el propio chip. Los parámetros de la cámara pueden configurarse a través del bus SCCB (Serial Camera Control Bus).En nuestro caso fue necesario modificar el parámetro com 7 en un valor de “0x24” para generar una imagen de tamaño 320x240  con formato RGB.

ENTRADAS: 
* SIO-C y SIO-D: El cual  series un puerto serial que en nuestro caso se define por  el protocolo de comunicación I2C proviene de la tarjeta Arduino y nos permite configurar la cámara.
* XCLK: Señal de reloj  24Mhz que necesita la cámara para su funcionamiento que proviene de módulo de clk_100MHZ_to_25M_24M.v
* Reset: señal que se debe dejar a cero voltios.
* PWDN: Señal que se debe dejar en 3.3 voltios.

SALIDAS: 
* VSYNC: Señal de sincronización vertical de la cámara.
* HREF: Señal de sincronización horizontal de la cámara.
* PCLK: Reloj para sincronización con la cámara.
* [7:0] D: Datos de salida de la cámara.

![alt text](https://github.com/unal-edigital1-2019-2/work04-proyectofinal-grupo-02-1/blob/master/docs/figs/camara.png)


#### Módulo de Captura

Se trata de un circuito secuencial, una máquina de estados diseñada por nosotros para la captura de los datos que provienen de la Cámara OV7670. Este módulo trabaja con una una cantidad finita de estados (5), y el sincronizado con el reloj de salida de la cámara. El módulo está compuesto por el módulo de definición de estados, un contador, y un conjunto de flip flops que se encargan de guardar temporalmente los datos de entrada de la cámara, para ser enviados a la memoria RAM.

ENTRADAS:
* VSYNC: Señal de sincronización vertical de la cámara.
* HREF: Señal de sincronización horizontal de la cámara.
* PCLK: Reloj para sincronización con la cámara.
* [7:0] D: Datos de salida de la cámara.
* CBtn: Señal botón de captura.

SALIDAS:
* [7:0] Data: Datos de salida de pixel.
* [16:0] Addr: Address de escritura para la memoria RAM.
* Regwrite: Señal de escritura para la memoria RAM.
![alt text](https://github.com/unal-edigital1-2019-2/work04-proyectofinal-grupo-02-1/blob/master/docs/figs/Captura%20de%20datos.png)

#### Generador de Reloj

El sistema requiere además de los 32, 50 o 100 Mhz de entrada, generar dos señales de reloj de 25Mhz y 24 Mhz para la pantalla VGA y la Cámara respectivamente, para ello es necesario generar un nuevo PLL con Clocking Wizard en el entorno de ISE, tomando la siguiente ruta “tools -> Core Generator ... y general el ip con Clocking Wizard”. Una vez, generado el nuevo bloque de Clk se copia el archivo en la carpeta hdl/scr/PLL, reemplazando en el proyecto test_cam.xise el archivo clk_32MHZ_to_25M_24M.v por el generado, clk_100MHZ_to_25M_24M.v.

ENTRADAS:
* Reloj 100Mhz: reloj de la tarjeta nexys 4.

SALIDAS:
* Reloj 24Mhz:Señal de reloj para sincronizar la escritura.
* Reloj 25Mhz:Señal de reloj para sincronizar la lectura.

![alt text](https://github.com/unal-edigital1-2019-2/work04-proyectofinal-grupo-02-1/blob/master/docs/figs/relog.png)

#### Dual Port RAM

Se crea un banco de memoria de lectura  y escritura simultánea con las siguientes dimensiones: Ancho de pixel: 8 bit. Cantidad de pixel: 2^17. Donde la velocidad de escritura es menor que la de lectura.

ENTRADAS:
* DP_RAM_regW: Señal que proviene del modula de captura de datos y da permiso de escritura en la memoria.
* DP_RAM_addr_in: Señal que proviene del modula de captura de datos e indica en qué posición de la memoria se va a guardar los datos.
* DP_RAM_data_in: Señal que proviene del modula de captura en forma de un byte y representa un pixel.
* DP_RAM_addr_out: Señal que proviene del conversor de dirección del módulo VGA 640x480 he indica que dirección o que pixel se va a leer. 
* Reloj 24Mhz:Señal de reloj para sincronizar la escritura.
* Reloj 25Mhz:Señal de reloj para sincronizar la lectura.

SALIDAS:
 *data_men: El valor de la posicion de lectura o  el valor del pixel solicitado por el módulo VGA 640x480 para visualizar en la pantalla.

![alt text](https://github.com/unal-edigital1-2019-2/work04-proyectofinal-grupo-02-1/blob/master/docs/figs/dp%20ram.png)

#### Módulo VGA

Serie de instrucciones que habilitar la  comunicación del sofware con el hardware. Los controladores VGA son usados para controlar información que es enviada a un monitor u otra forma de pantalla.
Nuestro módulo que es entregado por el profesor cuenta con un módulo central y dos módulos periféricos para codificar la dirección y convertir el formato RGB 322 al formato RGB444.

ENTRADAS:
* data_men: El valor binario de la posicion  o el valor del pixel solicitado por el módulo VGA 640x480 para visualizar en la pantalla.
* Reloj 25Mhz:Señal de reloj para sincronizar la lectura.

SALIDAS:
* VGA_pos_X y VGA_pos_X: valores codificados de la posicion del pixel que se quiere leer, esta señal llegara al periférico Convert_ addr el cual solicitará la posición decodificada al modula DP_RAM. 
* data_RGB322: Módulo que envía el valor del pixel en formato RGB322 al periférico de conversión de formato RGB322 to RGB444, donde este último envía el valor del pixel a la pantalla.
* VGA_hsync_n y VGA_Vsync_n: señal de sincronizacion entre el módulo VGA640x480 y la pantalla.

![alt text](https://github.com/unal-edigital1-2019-2/work04-proyectofinal-grupo-02-1/blob/master/docs/figs/driver_vga.png)

## Funcionamiento
El proyecto final lleva a cabo el proceso de captura, guardado en memoria y envío de los datos a la salida VGA, haciendo uso de diferentes procesos que se llevan a cabo en cada uno de los respectivos módulos. Su funcionamineto se describe a continuacion de manera secuencial.

#### Generador de Reloj
El módulo de generación de reloj, se encarga de recibir la señal de reloj nativa de la tarjeta de 100MHz para sincronizarse, luego esta genera dos señales de salida que serán usadas más adelante. La primera señal de reloj 24MHz es generada para la entrada de reloj para la cámara (Xclk), para que esta pueda funcionar correctamente. La siguiente señal de 25MHz es generada para el módulo VGA, para poder sincronizar correctamente con la pantalla.

#### Módulo de Captura
El funcionamiento de la cámara es independiente del proceso de captura, es precisamente para eso que fue creado este módulo. El módulo de captura se mantiene en estado inactivo mientras que la señal de Cbtn se mantiene en bajo. Al momento de recibir un alto, el módulo pasa a un estado de espera, a recibir un alto en VSync, en ese momento los registros para la señal de adress se resetean, y esperan a la bajada de VSync. Una vez pase esto, se esperará a una señal de Href, para capturar los bits correspondientes a Rojo y Verde, añadir 1 al conteo de Address, y mantener Regwrite en bajo, espera un ciclo de reloj para luego capturar los datos de los bits correspondientes a Azul, y enviar un alto a la señal de Regwrite. Este proceso se repite cíclicamente siempre y cuando Href se encuentre en alto. Finalmente, si en el estado de esperar a un alto de Href, recibe un alto de Vsync, este vuelve a estado inactivo y finaliza la captura de una imagen completa. Nótese que el método de captura no depende de las dimensiones de la imagen.

![alt text](https://github.com/unal-edigital1-2019-2/work04-proyectofinal-grupo-02-1/blob/master/docs/figs/Diagrama%20de%20estados.png)

#### Dual Port RAM
La memoria Dual port permite el almacenamiento de los datos provenientes del modulo de captura. Cada vez que un alto llega a la señal Regwrite, la RAM dual port se encarga de tomar los datos del bus data_in, y ubicarlos en el address correspondiente a adr_in. Cada uno de los datos tiene una longitud de 1 Byte, y el address correspondiente tiene una longitud de 17 bits. Los datos quedan almacenados dentro de la RAM. Simultaneamente se ofrece un puerto de lectura, en donde siempre que se le indique un address en el bus adr_out, este enviara los datos del adress indicado al bus data_out.

#### Driver VGA
El driver de la VGA recibe el reloj de 25Hz y va realizando un proceso de envio de datos por el puerto vga. Este driver identifica la cordenada en la que se encuentra en la pantalla, en base a esto busca el adress que corresponde al pixel que debe enviar, si no corresponde ninguno, pide el dato de la ultima posicion de memoria (Negro en nuestro caso). Una vez enviado el adress, recoje el dato del pixel desde la memoria RAM, para finalmente completarla para un formato RGB 444, y enviarla al puerto VGA, visualizando finalmnte la imagen.

![alt text](https://github.com/unal-edigital1-2019-2/work04-proyectofinal-grupo-02-1/blob/master/docs/figs/Proyecto-01.jpeg)

## Desarrollo
Durante todo el semestre se realizaron varias actividades que nos ayudaron a familiarizarnos con el entorno ISE xilinx herramienta con la cual aprendimos a sintetizamos hardware. Debido a que el profesor dividió el proyecto en paquetes de trabajo nos permitió  verla funcionalidad y por tanto el avance del proyecto en tiempo real, lo que significa que el paso a paso se convirtió en un mejor entendimiento del proyecto. Para realizar el módulo de captura de datos fue indispensable entender el diagrama de tiempo de las salidas de la cámara y contrastarlas con la funcionalidad de simulación en la aplicación de ISE. Para mas detalle en el desarrollo se sugiere consultar antiguos work pakages.

#### Definicion de la longitud de DW y AW
Se consulto en la datasheet de cuatro diferentes FPGA disponibles en las tarjetas Nexys, y se busco cuanto disponian en memoria de buffer, entre las cuales estaba la Nexys 2, Nexys 3, Nexys 4, y Nexys A7. Se decidio usar la Nexys 4 ya su FPGA (Artix-7) que era la que mayor cantidad de memoria disponia, con un total de 4860 Kb. A partir de esto se predijo que se podria tener un maximo de dimensiones de (320x240) con un formato 565 RGB.

Las predicciones fueron corroboradas realizando pruebas de sintesis dentro de ISE, verificando que no apareciera un errror que indicara que la memoria de la FPGA hubiese sido exedida. La primera preuba exitosa fue de la prediccion, y asi decidimos ajustar los valores de DW=16, y AW=17. Mas adelante se nos pidio que el el fromato de salida fuera necesariamente 322, asi que se decidio bajar el DW=8. A pesar de este cambio, las dimensiones de la pantalla no podian ser aumentadas.

Ajustando AW = 17 , DW = 16.

Los resulados de advertencias no incluyen problemas de memoria, y su sientesis fue exitosa:

`WARNING:LIT:701 - PAD symbol "clk" has an undefined IOSTANDARD.`

#### Prueba Test Bench RAM
Se realizo una pequeña prueba de simulacion de la RAM, en dodne se le pedia hcer un ciclo de lectura de un adrres, escritura del mismo, y luego nuevamente una lectura, para verificar que los datos fuesen escritos de manera exitosa. Una pequeña muestra exitosa de la simulacion se muestra a continuacion, verificando el funcionamiento de la memoria RAM.

![alt text](https://github.com/unal-edigital1-2019-2/work01-ramdp-grupo-02/blob/master/docs/figs/Testbench.png) 

#### Adaptacion de los requerimientos
A diferencia de otros grupos, nosotros estuvimos trabajando con dimensiones mayores, razon por la cual debimos hacer cambios a partir de el work package 2 para ajustar los valores de las dimensiones de salida (320x240).

* parameter CAM_SCREEN_X = 320;
* parameter CAM_SCREEN_Y = 240;
* DW = 8;
* AW = 17;

Adicionalmente nos dimos cuenta, que si la meta era lograr capturar una fotografia, se requeriria de una señal externa, para indicar que se tome la foto, razon por la cual creamos la señal CBtn.

#### Diseño de la capturadora
Para la realizacion del modulo de captura, se decidio pensar en este como una maquina de estados algoritmicos para comenzar a diseñarlos. La principal limitacion que notamos en el desarrolllo fue que el proceso debia sincornizarse corrrectamente con PCLK, ya que de este clok dependia la captura de los datos, y se debia sincronizar cualquier cambio utileizando el posedge.

![alt text](https://raw.githubusercontent.com/unal-edigital1-2019-2/work02-captura-datos-0v7670-grupo-02/master/docs/figs/Diagrama.jpeg)

Inicamos creando un mapa de flujo que describa el comportamiento del modulo, que se puede resumir de la siguiente manera: El modulo espera a que el usuario presione CBtn para luego prepararse para la captura en el proximo Vsync, una vez Vsync este en alto se encarga de resetear el addres. A partir de ese punto este se mantendra en espera hasta que Href se encuentre en alto, llevandolo a dos estados en bucle que son los encargados de realizar la captura y enviar a memoria. Este ciclo se repite hasta que Vsync este nuevamente en alto.

La idea funcional de CBtn que se tiene pensada, es que con pulsar el boton este realize una captura rapida, sin embargo si este se mantiene presionado este deberia poder actualizar constantemente la imagen mostrando video en pantalla.

A partir del mapa de flujo nos dimos cuenta de que el modulo fisicamente requeriria de un regitro con escritura serial para el almacenamiento de datos, el cual se le debia indicar cuando captruar los datos de rojo y verde, y cuando capturar los datos de azul. Igualmente se necesitaria de un contador para el addres, con una señal para añadir 1, y una señal para resetar el contador. Tambien notamos que podria necesitarse un registro para regwrite, sin embargo, este podia ser manejado directamente desde la UC de manera combinacional. Aunque estos componentes no es necesario añadirlos estructuralmente gracias al lenguaje de verilog, consideramos pertinente reconocer como podria ser modelado estructuralmente el modulo.

Realizamos el grafico de estados con un mapa de More para la Unicad de Control, y obtuvimos seis estados diferentes, asi que con un registro de tres bits,bastaria para almacenar el estado actual.

![alt text](https://raw.githubusercontent.com/unal-edigital1-2019-2/work02-captura-datos-0v7670-grupo-02/master/docs/figs/Diagrama%20de%20estados.png)

#### Codificacion e implementacion de la captura
Siendo asi, decidmos relizar la estructura del modulo como un conjunto de dos switch dentro de un unico always posedge, para que la primera parte se encargue de ajustar el estado de la UC, y el segundo, ejecutar los procesos que indica el estado actual. De utilizar un unico switch, la ejecucion se retrasaria un ciclo de reloj.

	always@(posedge PCLK) begin
     //Unidad de Control
		case(state)
			0: begin //Inactivo
			if (CBtn)	state <= 1;
			end
			1: begin //Iniciado
			if (VSYNC)	state <= 2;
			end
			2: begin //Reset
			if (~VSYNC)	state <= 3;
			end
			3: begin //Prepare
			if (VSYNC)	state <= 0;
			if (HREF)	state <= 4;
			end
			4: begin //Captura 1
			if (HREF)	state <= 5;
			else			state <= 3;
			end
			5: begin //Captura 2
			if (HREF)	state <= 4;
			else			state <= 3;
			end
		endcase
		//Ejecucion de Control
		case(state)
			2: begin //RESET
			addr <= 17'b111111111111111;
			end
			4: begin //CAPTURA 1
			regwrite <= 0;
			addr <= addr+1;
			data [7:2] <= {D[7:5],D[2:0]};
			end
			5: begin //CAPTURA 2;
			data [1:0] <= D[4:3];
			regwrite <= 1;
			end
			default: begin//DEFAULT
			regwrite <= 0;
			end
		endcase
	end
  
#### Primer test de pantalla
Para el testeo de pantalla se decidio cambiar no solamente a una pantalla monocolor, sino utilizar un archivo image men que contuviera las tres principales gamas de color, y blanco. Con ayuda de una programacion sencilla por c++ se creo una archivo de texto que repitiera un patron de 20 pixeles rojo, 20 verde, 20 azul y 20 blanco hasta llenar toda la matriz de (350x240). Para cada color se uso el siguiente codigo en hexadecimal:

* Blanco => FF
* Rojo => E0
* Verde => 1C
* Azul => 03

Los resultados depues de algunos ajustes con ensayo y error fueron los esperados, logramos que en la pantalla se proyectara adecuadamente sin pixeles corridos visibles en ningun punto. Vale la pena decir que curiosamente el resto de la pantalla parece adoptar el color del primer pixel de la memoria.

![alt text](https://raw.githubusercontent.com/unal-edigital1-2019-2/work02-captura-datos-0v7670-grupo-02/master/docs/figs/Pantalla1.jpg)
  
#### Observavion fallo de corrimiento
A partir del work package 3 se nos ofrecio una herramienta para poder probar una pantalla simulada, sin tener que sintetizar todo el bitstream, aun cuando al incio no fue clara la observacion del fallo en preubas anteriores por dos razones:

* Las preubas ofrecidas eran de barras horizontales de color, o de una sola pantalla de color
* La preuba anterior, coincidencialmente las barras tenian la longitud del dezplazamiento

Mas adelante, se crearon otras pruebas para la memoria y la capturadora, con barras de grosor desigual, y se detecto el fallo de un corrimiento a la derecha en la salida de la VGA. Con ayuda del profesor este error fue eliminado.

![alt text](https://github.com/unal-edigital1-2019-2/work04-proyectofinal-grupo-02-1/blob/master/docs/figs/Frame%20fallo.png)

#### Testeo de Pantalla RAM y Captura
Una vez corregido el error, se pudo continuar con el work pakage correspondiente a la prueba de capturadora, los resultados fueron buenos, y se ajusto un dato en el codigo, que buscaba una posicion de memoria por deafult, haciendo que el fondo cambiara de color, en vez de mantenerse en negro.

![alt text](https://github.com/unal-edigital1-2019-2/work04-proyectofinal-grupo-02-1/blob/master/docs/figs/Simulacion-02.png)

#### Configuracion de la Camara
La configuracion de la camara tambien debio sufrir un cambio minimo, pasando de usar un formato QCIF a CIF. Igualmente los registros explicados no fueron cambiados, y la configuracion fue exitosa al tercer intento, una vez se empalmaron las tierras y se conecto reset y powerdown a bajo y alto respectivamente

#### Pruebas finales y Resultados
Generamos el proyecto final, con su respectiva ucf, el bitstream fue cargado, y la camara fue configurada. Todos los elemntos fueron acoplados y configurados para probar la salida de la imagen. Las simulaciones correspondian con los resultados.

En mi opinion personal, supongo que las imagenes hablan por si solas.

![alt text](https://github.com/unal-edigital1-2019-2/work04-proyectofinal-grupo-02-1/blob/master/docs/figs/Proyecto-00.jpeg)
![alt text](https://github.com/unal-edigital1-2019-2/work04-proyectofinal-grupo-02-1/blob/master/docs/figs/Proyecto-02.jpeg)

## Resultados y obseravaciones
* Logramos cumplir con los objetivos del proyecto, y las imagenes fueron visualizadas de manera exitosa.
* Identificamos acertadamente la tarjeta a usar, con la mayor cantidad de memoria, y logramos genera una imagen con un area mayor a la requerida.
* Identificamos un fallo de corrimiento, alterando las simulaciones de fondos monocolor, o barras horizontales, un fallo que pudo pasar desapercibido si usasemos las pruebas estandar y no las personalizadas.
* Nuestra identificacion de los registros para configurar la camara fueron parcialmente acertados, especialmente en el COM7.
* Logramos adaptar los paquetes entregados a las necesidades de nuestro proyecto, y logramos incluir señales externas y cambios que originalmente no se nos indicaban.

