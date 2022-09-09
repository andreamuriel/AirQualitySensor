# Sensor de Calidad de Aire

En este repositorio se muestra como configurar un microcontrolador con comunicación serial UART para monitorear la salida tanto digital como analógica del sensor MQ135. Su valor analógico se procesa mediante el ADC para obtener una medición digital. A esta medición se le aplica el filtro MAF (Mean Average Filter) para eliminar el ruido de las mediciones.

## Configuración Software

Para comenzar se utiliza el software STM32CubeIDE, se crea un proyecto nuevo y se selecciona el modelo del microcontrolador, en este caso es el STM32F103C8T6. Luego se cambian ciertas opciones en las categorías.

En la categoría “SYS mode and Configuration” se activa el modo de debug Serial Wire.

![Untitled](Sensor%20de%20Calidad%20de%20Aire%2061f12fecc8db4b9ab734fba239caac7a/Untitled.png)

 Después en la categoría “Analog” se activan los canales a utilizar de cualquiera de los ADC, en este caso se utiliza solo un canal del ADC1, se elige el canal 2 “IN2”, dentro de su configuración de parámetros se activa la configuración “Continous  Conversion Mode”. 

![Untitled](Sensor%20de%20Calidad%20de%20Aire%2061f12fecc8db4b9ab734fba239caac7a/Untitled%201.png)

En la categoría “Connectivity” se activa e modo asíncrono del “USART1”, en su configuración de parámetros se revisa la tasa de baudios (Baud Rate), en este caso no se modifica y se queda en 115200 Bits/s. 

![Untitled](Sensor%20de%20Calidad%20de%20Aire%2061f12fecc8db4b9ab734fba239caac7a/Untitled%202.png)

El último paso en la configuración es activar un pin como entrada, en este pin se asignará la salida digital del sensor. 

![Untitled](Sensor%20de%20Calidad%20de%20Aire%2061f12fecc8db4b9ab734fba239caac7a/Untitled%203.png)

## Código

Se oprime el botón para generar el código y dentro de este primero se definen las variables a utilizar:

```c
int main(void)
{
  /* USER CODE BEGIN 1 */
	uint16_t sensor_data;
	uint16_t ARRAY[60];
	int A1=0;
	int A2=0;
	int A3=0;
	int A4=0;
	int N=4;
	int Vfilt=0;
	int D=0;
  /* USER CODE END 1 */
```

Donde: sensor_data es el nombre dado al valor analógico medido. ARRAY es el arreglo que se utilizará para mostrar los valores obtenidos, este tendrá un tamaño de 60 espacios, para mostrar hasta 60 caracteres. A1, A2, A3 y A4 son los valores transformados a digital con el conversor ADC del microcontrolador. N es el número de datos a utilizar en cada muestra a filtrar. Vfilt es el resultado del filtro y D es la asignación de activación del valor digital del sensor.

Se activa el modo Convertidor análogo digital.

```c
/* USER CODE BEGIN 2 */
  HAL_ADC_Start(&hadc1);
/* USER CODE END 2 */
```

Dentro del ciclo while (1) se definen todas las acciones a realizar. En la variable sensor_data se obtiene una medición convertida a digital, Se define que cada variable A toma su valor anterior, siendo el primer dato el obtenido en la variable sensor_data.

Se aplica el filtro MAF, con los cuatro valores a evaluar y se guarda la medición en Vfilt.

También se define un condicional. Si el valor digital original medido por el sensor supera un cierto valor se activa el Pin B15 del microcontrolador, si no lo sobrepasa este se mantiene desactivado.

Finalmente se define la función sprintf, con esta se muestran los 3 datos a evaluar: El valor analógico convertido a digital, el valor filtrado de los 4 últimos valores medidos, y la activación del pin digital del sensor.

```c
/* USER CODE BEGIN WHILE */
  while (1)
  {
	  HAL_ADC_PollForConversion(&hadc1, 10);
	  		sensor_data = HAL_ADC_GetValue(&hadc1);
	  		         A4=A3;
	  		         A3=A2;
	  		         A2=A1;
	  		         A1=sensor_data;
	  		         Vfilt=(A1+A2+A3+A4)/N;
if (HAL_GPIO_ReadPin(GPIOB, GPIO_PIN_15)) {
	D=1;
}
else {
	D=0;
}

	  	  sprintf(ARRAY, "se ha leido el valor %d, filtrado  %d, digital %d\r\n ", sensor_data, Vfilt,D);
	  	  HAL_UART_Transmit(&huart1, ARRAY, 60, 100);
	  	  HAL_Delay(1500);
  }

    /* USER CODE END WHILE */
```

## Circuito

En el adaptador USB conectado al computador en el que se esté trabajando, se conectan los pines: 2. SWDIO, 4. GND, 6.SWCLK y 8. 3.3V a los pines respectivos del microcontrolador.

En la placa del UART se conecta con su cable de alimentación al computador, su pin a tierra (GND) se conecta con el respectivo del microcontrolador, y el pin Receptor (Rx) con el Pin Transmisor (Tx) del microcontrolador (Pin A9). 

Todos los pines del sensor se conectan al microcontrolador. A0, D0, GND y Vcc.

A0 es su salida análoga, este se conecta al pin A2, ya que este se configuró automáticamente al activar el ADC1. D0 es la salida digital, esta se conecta al pin B15, ya que este se configuró para como entrada de una señal digital. GND se conecta al pin GND  y Vcc se conecta al Pin 3.3V, ya que este pin entrega la alimentación para el funcionamiento del sensor.

## Resultados

Teniendo todo el circuito conectado y el código escrito, se abre el Software “PuTTY”, este es un emulador de terminal, al abrirlo se selecciona el puerto COM del computador en el que está conectada la placa del UART y se selecciona la tas de baudios anteriormente escogida (115200 Bits/s). Luego se corre el código para que el sensor empiece a medir y se abre la pantalla del emulador, en este se puede ver lo mostrado en la siguiente imagen:

![WhatsApp Image 2022-09-05 at 5.55.51 PM.jpeg](Sensor%20de%20Calidad%20de%20Aire%2061f12fecc8db4b9ab734fba239caac7a/WhatsApp_Image_2022-09-05_at_5.55.51_PM.jpeg)

Se logra el objetivo de medir partículas de gas en el ambiente y mostrar las mediciones trabajadas en la pantalla. El valor leído es el entregado por el ADC, el filtrado es el promedio del valor actual y los 3 valores anteriores, y el valor digital se mantiene en cero porque no se ha llegado al valor de activación.

## Autores

Todos los autores pertenecen al departamento de Ingeniería Electrica y Electrónica, Universidad del Bío-Bío, Concepción. 

Bastián Bascur (estudiante de pregrado) - responsable de diseño físico del circuito.

Manuel Oñate (estudiante de pregrado) - responsable de desarrollo del código.

Andrea Venegas (estudiante de pregrado) - responsable de verificación y documentación.