# Electrónica Digital - Grupo 02

* David Miguel García Palacios - 1001315936
* Javier Andres Africano Pachón - 1193381320
* David Ricardo Lugo Venegas - 1075872618


# TRABAJO FINAL

Se ejecuta una serie de actividades o “paquetes de trabajos” que tienen como  línea de enfoque  permitirnos desarrollar una cámara digital en su esencia más simple ayudados de herramientas tales como una tarjeta programable (Nexys 4 - FPGA),  Arduino y una cámara OV7670.


## Planteamiento del Problema
Realizar la síntesis de hardware (ayudados por la aplicación Xilinx) para recibir los datos de la cámara OV7670 con el protocolo I2C,  decodificarlos y proyectarlos en una pantalla con el estándar de video VGA. 

## Objetivos
Lograr la visualización en una pantalla con entrada VGA de una foto tomada por la cámara OV7670, haciendo uso de una FPGA y su respectivo código en Verilog.
Diseñar y crear un módulo de captura de la cámara que se encargue de recibir los datos de la imagen de la cámara, adecuarlos de formato RGB 565 a RGB 332, y guardarlos en la memoria RAM Dual Port
Configurar los registros de la la cámara OV7670, haciendo uso de una tarjeta programable Arduino, ajustando correctamente sus parámetros de escalado (Formato CIF) y formato del pixel (RGB 565).

### Elementos de Trabajo
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
* Descripción

Haciendo uso de una cámara y una tarjeta programable, la cual a partir de una señal de botón controlada por el usuario, realiza captura de una imagen de 320 x 240 pixeles de resolución, que se puede visualizar en una pantalla con conexión VGA, donde si dicho botón es oprimido por el usuario continuamente, la toma de imagen de igual manera se realiza constante dando opción de visualizar video en dicha pantalla.

### Camara OV7670

El módulo de cámara OV7670 está basado en el sensor OV7670 de omnivision. Es capaz de generar un máximo de 30 cuadros por segundo a una resolución de 640×480 pixeles (VGA). El OV7670 es un sistema en chip (SoC) por lo que además de ser un sensor de imagen, en su interior también encontramos circuitos destinados al procesamiento de la imagen capturada, por esta razón los procesos de control se realizan en el propio chip. Los parámetros de la cámara pueden configurarse a través del bus SCCB (Serial Camera Control Bus).En nuestro caso fue necesario modificar el parámetro com 7 en un valor de “0x24” para generar una imagen de tamaño 320x240  con formato RGB.

Las entradas de la cámara son: 
SIO-C y SIO-D: El cual  series un puerto serial que en nuestro caso se define por  el protocolo de comunicación I2C proviene de la tarjeta Arduino y nos permite configurar la cámara.
XCLK: Señal de reloj  24Mhz que necesita la cámara para su funcionamiento que proviene de módulo de clk_100MHZ_to_25M_24M.v
Reset: señal que se debe dejar a cero voltios.
PWDN: Señal que se debe dejar en 3.3 voltios.

Las salidas de la cámara son: 
VSYNC: Señal de sincronización vertical de la cámara.
HREF: Señal de sincronización horizontal de la cámara.
PCLK: Reloj para sincronización con la cámara.
[7:0] D: Datos de salida de la cámara.

### Módulo de Captura

Se trata de un circuito secuencial, una máquina de estados diseñada por nosotros para la captura de los datos que provienen de la Cámara OV7670. Este módulo trabaja con una una cantidad finita de estados (5), y el sincronizado con el reloj de salida de la cámara. El módulo está compuesto por el módulo de definición de estados, un contador, y un conjunto de flip flops que se encargan de guardar temporalmente los datos de entrada de la cámara, para ser enviados a la memoria RAM.

ENTRADAS:
VSYNC: Señal de sincronización vertical de la cámara.
HREF: Señal de sincronización horizontal de la cámara.
PCLK: Reloj para sincronización con la cámara.
[7:0] D: Datos de salida de la cámara.
CBtn: Señal botón de captura.

SALIDAS:
[7:0] Data: Datos de salida de pixel.
[16:0] Addr: Address de escritura para la memoria RAM.
Regwrite: Señal de escritura para la memoria RAM.


### Generador de Reloj

El sistema requiere además de los 32, 50 o 100 Mhz de entrada, generar dos señales de reloj de 25Mhz y 24 Mhz para la pantalla VGA y la Cámara respectivamente, para ello es necesario generar un nuevo PLL con Clocking Wizard en el entorno de ISE, tomando la siguiente ruta “tools -> Core Generator ... y general el ip con Clocking Wizard”. Una vez, generado el nuevo bloque de Clk se copia el archivo en la carpeta hdl/scr/PLL, reemplazando en el proyecto test_cam.xise el archivo clk_32MHZ_to_25M_24M.v por el generado, clk_100MHZ_to_25M_24M.v.

ENTRADAS:
Reloj 100Mhz: reloj de la tarjeta nexys 4.

SALIDAS:
Reloj 24Mhz:Señal de reloj para sincronizar la escritura.
Reloj 25Mhz:Señal de reloj para sincronizar la lectura.

### Dual Port RAM

Se crea un banco de memoria de lectura  y escritura simultánea con las siguientes dimensiones: 
 
Ancho de pixel: 8 bit
Cantidad de pixel: 2^17

Donde la velocidad de escritura es menor que la de lectura.


ENTRADAS:
DP_RAM_regW: Señal que proviene del modula de captura de datos y da permiso de escritura en la memoria.
DP_RAM_addr_in: Señal que proviene del modula de captura de datos e indica en qué posición de la memoria se va a guardar los datos.
DP_RAM_data_in: Señal que proviene del modula de captura en forma de un byte y representa un pixel.
DP_RAM_addr_out: Señal que proviene del conversor de dirección del módulo VGA 640x480 he indica que dirección o que pixel se va a leer. 
Reloj 24Mhz:Señal de reloj para sincronizar la escritura.
Reloj 25Mhz:Señal de reloj para sincronizar la lectura.

SALIDAS:
data_men: El valor de la posicion de lectura o  el valor del pixel solicitado por el módulo VGA 640x480 para visualizar en la pantalla.


### Módulo VGA

Serie de instrucciones que habilitar la  comunicación del sofware con el hardware. Los controladores VGA son usados para controlar información que es enviada a un monitor u otra forma de pantalla.
Nuestro módulo que es entregado por el profesor cuenta con un módulo central y dos módulos periféricos para codificar la dirección y convertir el formato RGB 322 al formato RGB444.

ENTRADAS:
data_men: El valor binario de la posicion  o el valor del pixel solicitado por el módulo VGA 640x480 para visualizar en la pantalla.
Reloj 25Mhz:Señal de reloj para sincronizar la lectura.

SALIDAS:
VGA_pos_X y VGA_pos_X: valores codificados de la posicion del pixel que se quiere leer, esta señal llegara al periférico Convert_ addr el cual solicitará la posición decodificada al modula DP_RAM. 
data_RGB322: Módulo que envía el valor del pixel en formato RGB322 al periférico de conversión de formato RGB322 to RGB444, donde este último envía el valor del pixel a la pantalla.
VGA_hsync_n y VGA_Vsync_n: señal de sincronizacion entre el módulo VGA640x480 y la pantalla.

## Funcionamiento
El proyecto final lleva a cabo el proceso de captura, guardado en memoria y envío de los datos a la salida VGA, haciendo uso de diferentes procesos que se llevan a cabo en cada uno de los respectivos módulos.

### Generador de Reloj
El módulo de generación de reloj, se encarga de recibir la señal de reloj nativa de la tarjeta de 100MHz para sincronizarse, luego esta genera dos señales de salida que serán usadas más adelante. La primera señal de reloj 24MHz es generada para la entrada de reloj para la cámara (Xclk), para que esta pueda funcionar correctamente. La siguiente señal de 25MHz es generada para el módulo VGA, para poder sincronizar correctamente con la pantalla.

### Módulo de captura
El funcionamiento de la cámara es independiente del proceso de captura, es precisamente para eso que fue creado este módulo. El módulo de captura se mantiene en estado inactivo mientras que la señal de Cbtn se mantiene en bajo. Al momento de recibir un alto, el módulo pasa a un estado de espera, a recibir un alto en VSync, en ese momento los registros para la señal de adress se resetean, y esperan a la bajada de VSync. Una vez pase esto, se esperará a una señal de Href, para capturar los bits correspondientes a Rojo y Verde, añadir 1 al conteo de Address, y mantener Regwrite en bajo, espera un ciclo de reloj para luego capturar los datos de los bits correspondientes a Azul, y enviar un alto a la señal de Regwrite. Este proceso se repite cíclicamente siempre y cuando Href se encuentre en alto. Finalmente, si en el estado de esperar a un alto de Href, recibe un alto de Vsync, este vuelve a estado inactivo y finaliza la captura de una imagen completa. Nótese que el método de captura no depende de las dimensiones de la imagen.

## Desarrollo
Durante todo el semestre se realizaron varias actividades que nos ayudaron a familiarizarnos con el entorno ISE xilinx herramienta con la cual aprendimos a sintetizamos hardware. Debido a que el profesor dividió el proyecto en paquetes de trabajo nos permitió  verla funcionalidad y por tanto el avance del proyecto en tiempo real, lo que significa que el paso a paso se convirtió en un mejor entendimiento del proyecto. Para realizar el módulo de captura de datos fue indispensable entender el diagrama de tiempo de las salidas de la cámara y contrastarlas con la funcionalidad de simulación en la aplicación de ISE.


