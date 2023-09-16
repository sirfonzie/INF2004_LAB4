# LAB 4: PWM & ADCs

**OBJECTIVES**
-	Configure and implement Pulse Width Modulation (PWM) on RPi Pico
-	Configure and implement an Analog-to-Digital Converter (ADC) on RPi Pico
-	Understand how to configure the (line following) IR-based sensor & L298N Motor controller


**EQUIPMENT** 
1.	A laptop that has the Pico C/C++ SDK installed
2.	Raspberry Pico W
3.	Micro-USB Cable

> [NOTE]
> Only students wearing fully covered shoes are allowed in the SR6A lab due to safety.

## **INTRODUCTION** 

In the previous lab session, we learned that we can use interrupts to detect and react to requests from the GPIO and Timer. For GPIO, we can react when the GPIO pins change while for the Timer, we can invoke callback functions once the timer has lapsed. A timer can also be used as a counter that keeps track of the number of times a particular event or process occurred regarding a clock signal. It is used to count the events happening outside the microcontroller. However, this counter can also be configured to produce a pulse with a specific duty cycle known as pulse-width modulation (PWM). It is particularly useful for tasks like motor control, LED dimming, and generating analog-like signals. PWM works by varying the duty cycle (percentage of time the signal is ON) of a square wave signal while keeping its frequency constant. 

Typically, GPIO when configured as input is able to accept data as high, low, or edge-based signals. However, the world we live in doesn't always offer discrete data. Thus, we would also be looking at Analog-to-Digital Converter (ADC), which is an essential part of any embedded system. It helps to obtain **Analog** data (i.e. temperature, humidity, etc) that can’t be directly read on any microcontroller as they are in analog format. To read this type of analog data, we have to convert this analog data into a digital format via an ADC inside the microcontroller.

Lastly, we will understand how we can use PWM to adjust the speed of the motor via the motor controller and incorporate ADC for the IR-based sensor in measuring contrast on the surface.

## **PULSE-WIDTH MODULATION** 

The Raspberry Pi Pico, with its GPIO pins and the Pico C SDK, provides a convenient way to generate PWM signals for various applications. Configuring PWM on the Raspberry Pi Pico using the Pico C SDK involves several steps, including initializing the PWM hardware, setting the frequency, configuring channels, and controlling the duty cycle. PWM on the Raspberry Pi Pico is achieved through PWM slices. To understand how PWM works on the Pico, it's essential to understand the concept of PWM slices and how they are used to generate PWM signals.

**What are PWM Slices?**

PWM slices are hardware modules on the Raspberry Pi Pico designed specifically for generating PWM signals. These slices offer a flexible and efficient way to produce PWM waveforms with precise control over parameters like frequency, duty cycle, and polarity. The Pico has multiple PWM slices, and each slice can generate one or more PWM channels.

**Key Features and Concepts:**

1. **Multiple PWM Slices:** The Pico typically has multiple PWM slices, each capable of generating one or more PWM channels. The actual number of slices and channels may vary depending on the specific microcontroller model used in the Pico.
2. **PWM Channels:** Each PWM slice can generate one or more PWM channels. A PWM channel corresponds to a specific GPIO pin that can output a PWM signal. This allows you to control multiple devices or components independently.
3. **Frequency:** PWM slices allow you to set the frequency of the PWM signal. Frequency is the number of PWM cycles (periods) per second. Higher frequencies can provide smoother control but may require more CPU resources.
4. **Duty Cycle:** The duty cycle represents the proportion of time during one PWM cycle that the signal is high (on). It is usually expressed as a percentage. A 50% duty cycle means the signal is high for half of the cycle.
5. **Polarity:** PWM slices often support polarity settings, allowing you to specify whether the PWM signal starts high or low at the beginning of each cycle. Polarity settings can affect how devices respond to the PWM signal.

**How PWM Slices Work:**

Here's a step-by-step explanation of how PWM slices work on the Raspberry Pi Pico:

1. **Initialization:** Before using PWM, you initialize the PWM system using the `pwm_init()` function. This initializes all PWM slices and channels.
2. **Configuring Channels:** You configure the GPIO pins for PWM operation by setting their functions to `GPIO_FUNC_PWM` using `gpio_set_function()`. This allocates the pins to specific PWM channels.
3. **Frequency Setup:** You set the desired PWM frequency using the `pwm_set_wrap()` function. This determines how fast the PWM signal oscillates between high and low.
4. **Duty Cycle:** To control the duty cycle, you use the `pwm_set_chan_level()` function to specify when the signal transitions between high and low within each PWM cycle.
5. **Polarity Control:** If necessary, you can set the polarity of the PWM signal using `pwm_set_polarity()` to determine whether the signal starts high or low.
6. **Enabling PWM:** Once you've configured the PWM slices and channels, you enable the PWM signal using `pwm_set_enabled()`.
7. **Generation of PWM Signals:** Once enabled, the hardware takes care of generating the PWM signals based on your settings. The PWM signal is typically a square wave that repeats according to the configured frequency and duty cycle.
8. **Updating Parameters:** If needed, you can update the PWM parameters, such as frequency or duty cycle, at runtime to modify the PWM signal.
9. **Disabling PWM:** When you're done with PWM, you can disable it using `pwm_set_enabled()`.

The following code [hello_pwm](https://github.com/raspberrypi/pico-examples/blob/master/pwm/hello_pwm/hello_pwm.c) illustrates a simple example of how a PWM can be configured on the Raspberry Pi Pico.

## **UNDERSTANDING THE L298N MOTOR CONTROLLER**

The L298N module is a high-power motor driver module for driving DC and stepper motors. This module consists of an L298 motor driver IC and a 78M05 5V regulator. This module can control up to two DC motors with directional and speed control. The datasheet of the module can be found at the following [link](https://components101.com/modules/l293n-motor-driver-module). 

There are two ways for this module to control the speed of the motor. An easier way would be to use a jumper across ENA and ENB to fix the voltage to the maximum. However, removing the jumper and connecting the pin to a PWM source would facilitate controlling the motor’s speed. PWM is a widely used technique to control the speed of DC motors, including those used in robotics and other applications.

Again, we can re-use the hello_pwm code to demonstrate how PWM can be used to control the motor speed using the L298N motor controller. Connect the motor controller as follows and observe how the left or right wheel turns.

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

YYY
