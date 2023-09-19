# LAB 4: PWM & ADCs

**OBJECTIVES**
-	Configure and implement Pulse Width Modulation (PWM) on RPi Pico
-	Configure and implement an Analog-to-Digital Converter (ADC) on RPi Pico
-	Understand how to configure the (line following) IR-based sensor & L298N Motor controller


**EQUIPMENT** 
1.	A laptop that has the Pico C/C++ SDK installed
2.	Raspberry Pico W
3.	Micro-USB Cable
4.	IR-based Line Sensor
5.	Motor Controller & DC Motor

> [NOTE]
> Only students wearing fully covered shoes are allowed in the SR6A lab due to safety.

## **INTRODUCTION** 

In the previous lab session, we learned that we can use interrupts to detect and react to requests from the GPIO and Timer. For GPIO, we can react when the GPIO pins change while for the Timer, we can invoke callback functions once the timer has lapsed. A timer can also be used as a counter that keeps track of the number of times a particular event or process occurred regarding a clock signal. It is used to count the events happening outside the microcontroller. However, this counter can also be configured to produce a pulse with a specific duty cycle known as pulse-width modulation (PWM). It is particularly useful for tasks like motor control, LED dimming, and generating analog-like signals. PWM works by varying the duty cycle (percentage of time the signal is ON) of a square wave signal while keeping its frequency constant. 

Typically, GPIO when configured as input is able to accept data as high, low, or edge-based signals. However, the world we live in doesn't always offer discrete data. Thus, we would also be looking at Analog-to-Digital Converter (ADC), which is an essential part of any embedded system. It helps to obtain **Analog** data (i.e. temperature, humidity, etc) that can’t be directly read on any microcontroller as they are in analog format. To read this type of analog data, we have to convert this analog data into a digital format via an ADC inside the microcontroller.

Lastly, we will understand how we can use PWM to adjust the speed of the motor via the motor controller and incorporate ADC for the IR-based sensor in measuring contrast on the surface.

## **PULSE-WIDTH MODULATION** 

The Raspberry Pi Pico, with its GPIO pins and the Pico C SDK, provides a convenient way to generate PWM signals for various applications. Configuring PWM on the Raspberry Pi Pico using the Pico C SDK involves several steps, including initializing the PWM hardware, setting the frequency, configuring channels, and controlling the duty cycle. PWM on the Raspberry Pi Pico is achieved through PWM slices. To understand how PWM works on the Pico, it's essential to understand the concept of PWM slices and how they generate PWM signals.

**What are PWM Slices?**

PWM slices are hardware modules on the Raspberry Pi Pico designed to generate PWM signals. These slices offer a flexible and efficient way to produce PWM waveforms with precise control over parameters like frequency, duty cycle, and polarity. The Pico has multiple PWM slices, and each slice can generate one or more PWM channels.

**Key Features and Concepts:**

The RP2040 PWM block has eight identical slices, where each slice can drive two PWM output signals or measure the frequency or duty cycle of an input signal. This gives a total of up to 16 controllable PWM outputs. All 30 GPIO pins can be driven by the PWM block, as shown below.

<img src="/img/pwmgpio.png" width=100% height=100%>

**How PWM Slices Work:**

1. **Initialization:** Before using PWM, initialize the PWM system using the `pwm_init()` function. This initializes all PWM slices and channels.
2. **Configuring Channels:** You configure the GPIO pins for PWM operation by setting their functions to `GPIO_FUNC_PWM` using `gpio_set_function()`. This allocates the pins to specific PWM channels.
3. **Frequency Setup:** You set the desired PWM frequency using the `pwm_set_wrap()` function. This determines how fast the PWM signal oscillates between high and low.
4. **Duty Cycle:** To control the duty cycle, you use the `pwm_set_chan_level()` function to specify when the signal transitions between high and low within each PWM cycle.
5. **Enabling PWM:** Once you've configured the PWM slices and channels, you enable the PWM signal using `pwm_set_enabled()`.
7. **Updating Parameters:** If needed, you can update the PWM parameters, such as frequency or duty cycle, at runtime to modify the PWM signal.
8. **Disabling PWM:** When you're done with PWM, disable it using `pwm_set_enabled()`.

The following code [hello_pwm](https://github.com/raspberrypi/pico-examples/blob/master/pwm/hello_pwm/hello_pwm.c) illustrates a simple example of how a PWM can be configured on the Raspberry Pi Pico. You could connect GP0 and GP1 to an oscilloscope to observe the signal. Alternatively, you may observe the effects of PWM on the motor controller by connecting, as described in the next section.

## **UNDERSTANDING THE L298N MOTOR CONTROLLER**

The L298N module is a high-power motor driver module for driving DC and stepper motors. This module comprises an L298 motor driver IC and a 78M05 5V regulator. This module can control up to two DC motors with directional and speed control. The module's datasheet can be found at the following [link](https://components101.com/modules/l293n-motor-driver-module). 

There are two ways for this module to control the speed of the motor. An easier way would be to use a jumper across ENA and ENB to fix the voltage to the maximum. However, removing the jumper and connecting the pin to a PWM source would facilitate controlling the motor’s speed. PWM is a widely used technique to control the speed of DC motors, including those used in robotics and other applications.

Again, we can use the [hello_pwm](https://github.com/raspberrypi/pico-examples/blob/master/pwm/hello_pwm/hello_pwm.c) code with some changes to demonstrate how PWM can control the motor speed using the L298N motor controller. Connect the motor controller as follows and observe how fast the motor turns.

The following changes are required:
1. include this in line #21: `pwm_set_clkdiv(slice_num, 100);`
2. change line #23: `pwm_set_wrap(slice_num, 3);` --> `pwm_set_wrap(slice_num, 12500);`
3. change line #25: `pwm_set_chan_level(slice_num, PWM_CHAN_A, 1);` --> `pwm_set_chan_level(slice_num, PWM_CHAN_A, 12500/2);`  
4. comment off line #27: `pwm_set_chan_level(slice_num, PWM_CHAN_B, 3);` --> `\\pwm_set_chan_level(slice_num, PWM_CHAN_B, 3);`

This modified code snippet configures the Raspberry Pi Pico to generate PWM (Pulse Width Modulation) signals on GP0 via the gpio_set_function function. It then obtains the PWM slice number associated with GP0 and sets the clock divisor to 100, reducing the main clock from 125Mhz to 1.25Mhz frequency. The pwm_set_wrap function sets the PWM wrap value, essentially determining the period of the PWM signal. Here, it's set to 12500, which corresponds to a period of 10ms. This is obtained as follows: ((1/1.25Mhz)*12500). pwm_set_chan_level sets the duty cycle of the PWM signal on channel A of the specified PWM slice to 50% (12500/2). Finally, pwm_set_enabled enables the PWM output on the specified slice. In summary, this code initializes PWM on GP0 with a 100Hz frequency and a 50% duty cycle, effectively generating a square wave output.

<img src="/img/motor.png" width=100% height=100%>

## **ANALOG-TO-DIGITAL CONVERTER**

An Analog-to-Digital Converter (ADC) on the Raspberry Pi Pico functions by converting continuous analog signals, such as voltage levels from sensors, into discrete digital values that the Pico can process. It does this through a multi-step process. First, the ADC captures the analog voltage from a sensor input pin and samples it at regular intervals. Then, it quantizes the sampled voltage, assigning it a digital value based on its magnitude. The resolution of the ADC determines the number of discrete values it can represent. The Raspberry Pi Pico's ADC typically provides a 12-bit resolution, allowing for 4,096 possible values. This digitized data can then be used in software applications, enabling the Pico to process and respond to analog inputs from the physical world.

The following code [adc_console](https://github.com/raspberrypi/pico-examples/blob/master/adc/adc_console/adc_console.c) illustrates a simple example of how an ADC can be configured on the Raspberry Pi Pico. This code creates a console interface for interacting with the Raspberry Pi Pico's ADC, allowing users to select channels, sample analog data, capture multiple samples, and perform GPIO operations for testing and experimentation.

## **CONFIGURING THE IR-BASED LINE SENSOR**

The IR line sensor is a reflective sensor that includes an infrared emitter and phototransistor in a leaded package which blocks visible light. The infrared emission diode constantly emits infrared rays. When the intensity of the reflection is not received or is not large enough, the light-activated diode will be in the off state. However, if the sensor detects an object within the field of view, infrared light is reflected back and activates the diode.

The IR sensor depicted below features both analog and digital outputs. The digital output alternates between high and low based on the infrared light's reflection, with a threshold set via a potentiometer determining the state. However, in this lab, we will exclusively utilize the Analog output (A), which provides a continuous signal proportional to the intensity of the reflected IR light. Subsequently, we will employ the ADC to convert this analog signal into a digital value, enabling precise measurement and offering a more comprehensive range of information compared to the binary high-low output of the digital mode.

<img src="/img/irlinesensor.png" width=100% height=100%>

Again, we can re-use the adc_console code to demonstrate how ADC can be used to obtain the analog signal from the IR-based sensor and convert it into a digital format. Connect the IR sensor as follows and observe the data received via the serial monitor.

## **EXERCISE**

To configure a PWM signal at 20 Hz with a 50% duty cycle on GP2 and feed it into an ADC at GP26 while sampling the ADC every 25 ms, you must use the Raspberry Pi Pico and its Pico C SDK. You will need to use a jumper wire to connect GP2 to GP26. You may use a timer interrupt. The output to look as follows:

<img src="/img/ex4.png" width=100% height=100%>
