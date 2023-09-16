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


## **UNDERSTANDING THE L298N MOTOR CONTROLLER**

We will be exploring the [hello_gpio_irq.c](https://github.com/raspberrypi/pico-examples/blob/master/gpio/hello_gpio_irq/hello_gpio_irq.c) sample code designed for the Pico W. In this session, we'll merge our knowledge of GPIO with the concept of interrupts. Instead of the previous lab's approach, where we continuously polled the GPIO pin status using the `while(true)` statement, we'll now integrate interrupts. This will allow us to trigger the interrupts based on the desired state, whether edge-triggered or level-triggered. In this example, edge-triggered has been chosen. How would you change it to level-triggered? 

To test the code, you must connect the GP02 pin to 3.3V while observing the output on the Serial Monitor. In the [CMakeLists.txt](https://github.com/raspberrypi/pico-examples/blob/master/gpio/hello_gpio_irq/CMakeLists.txt), __ensure__ that the following line has been added `pico_enable_stdio_usb(hello_gpio_irq 1)`.

> [NOTE]
> Switching to trigger at a low-level (GPIO_IRQ_LEVEL_LOW) could lead to the software crashing (not working). Why?

## **IR-BASED WHEEL ENCODER**

<img src="https://rees52-fbcb.kxcdn.com/19851-thickbox_default/hc-020k-double-speed-measuring-module-with-photoelectric-encoders-for-experiment-rs019.jpg" width=50% height=50%>

The working principle of the encoder (shown above) is illustrated in the image below. It uses a slotted wheel with a single LED and photodetector pair that generate pulses as the wheel turns, and the speed of an object can be calculated by measuring the pulse duration Δti  (i.e. elapsed time or period of a pulse) between successive pulses. It comprises three connections: GND, VCC and OUT. GND and VCC supply power to the module (in our case, via the Pico's GND and 3.3V pins), while OUT generates the square-pulse signal. Connecting the GP02 from the Pico to the OUT of the IR-Sensor will allow the Pico to detect when the wheel is turning.

<img src="img/encoder.png" width=50% height=50%>

You may now change the callback function to convert this code into the wheel encoder driver. To measure distance, count each time the notch has been detected (Hint: Use edge-triggered). To measure speed, you will need to measure the pulse width.

<img src="img/connectIRsensor.png" width=70% height=70%>

## **TIMERS**

The RP2040 found in the Pico features a flexible timer system that can be used for various applications. Here is a brief description of its timer system, focusing on timer modes, input capture, and output compare:

1. **Timer Modes:**
    - **Free Running Mode:** In this mode, the timer simply counts from 0 to its maximum value and then wraps around to start counting again.
    - **Periodic Mode:** In this mode, the timer counts up to a predefined value (ALARM0, for instance) and then restarts from zero. This can be used to generate periodic interrupts or events.
    - **One-Shot Mode:** In this mode, the timer counts up to a predefined value and then stops. It's useful for creating a single delay or measuring an event of a known duration.

2. **Input Capture:**
    - Input capture is used to measure the time duration of an external event. For example, it can measure the duration of a pulse on a pin.
    - When the event occurs (like a rising or falling edge on a pin), the current timer value is 'captured' and stored in a register.
    - By comparing consecutive captured values, you can determine the duration of the event or the period between events.
    - The RP2040 timer can be configured to generate an interrupt when such capture occurs, which allows the CPU to process the captured value or take action based on the event.

3. **Output Compare:**
    - Output compare generates an event (like toggling a pin) at a specific timer value.
    - You set a value in a compare register, and an action can be triggered when the timer counts up to that value.
    - The action can be as simple as generating an interrupt or as complex as toggling a GPIO pin, generating PWM signals, etc.
    - This feature can be very useful in applications like motor control, where precise timing of events is crucial.

It's also worth noting that the RP2040 timer system provides multiple alarm (compare) registers, allowing multiple compare values to be active simultaneously. This multi-alarm capability can be particularly useful in applications requiring various events to happen simultaneously without constant CPU intervention. To make full use of the timer capabilities, you should take a look at the RP2040 datasheet and SDK, which will provide more detailed information and examples on how to configure and use the timers.

## **Periodic vs Single-shot**

This example [hello_timer.c](https://github.com/raspberrypi/pico-examples/blob/master/timer/hello_timer/hello_timer.c) illustrates how to configure a single-shot and a periodic-based timer that is used to trigger an interrupt. This interrupt will then trigger a function to perform the user-defined code. Changing the `delay_ms` parameter in the `add_repeating_timer_ms` API call can trigger different behaviours. Observe the starting time at each call.

The image below illustrates two different configurations for the periodic timer used in the sample code above.
<img src="img/periodic.png" width=70% height=70%>

## **HOW DOES THE ULTRASONIC HC-SR04P WORK**

<img src="img/ultrasonic.png" width=50% height=50%>

1. **Initiation**: A short high pulse, typically around 10 microseconds in duration, is applied to the "Trigger" pin of the HC-SR04 module to initiate a measurement.
2. **Ultrasonic Pulse Emission**: Once triggered, the HC-SR04 responds by emitting a burst of eight ultrasonic pulses at approximately 40 kHz. These sound waves travel through the air, radiating outward from the module's transmitter.
3. **Reflection and Reception**: If an object is present within the sensor's detection range, the emitted ultrasonic waves will bounce off that object's surface and reflect toward the module. The module's ultrasonic receiver, or the "Echo" pin, detects these reflected sound waves. The duration for which the "Echo" pin stays high is directly proportional to the time it takes for the emitted ultrasonic waves to hit an object and return.
4. **Distance Derivation**: The time measured from the emission of the ultrasonic pulse to its reception (the round trip) can be used to derive the distance to the object. Given that, we know the speed of sound in air (approximately 343 meters per second or 1125.33 feet per second at room temperature), the formula to calculate this distance is: <img src="img/distance.png" width=25% height=25%> The division by two accounts for the round trip of the sound waves; we need the time for just one way to determine the distance to the object.

<img src="img/connectultrasonic.png" width=70% height=70%>

## **SAMPLE CODE FOR ULTRASONIC HC-SR04P**

The following [example](https://github.com/KleistRobotics/Pico-Ultrasonic/blob/main/ultrasonic/ultrasonic.c) uses simple GPIO and delays to achieve the trigger and echo calculation to obtain the distance. However, this code is inefficient due to the use of block-waiting (lines #26 & #28). 


## **EXERCISE**

To develop a simple stopwatch that measures time intervals between button presses. On pressing the **START** pseudo-button (GP15), the stopwatch will begin, and the elapsed time will be displayed every second on the Serial Monitor (or equivalent). Releasing the **START** pseudo-button will stop the timer and reset the elapsed time to zero. The **START** pseudo-button must incorporate a debouncing algorithm. Please note that GPIO and Timer interrupts must be incorporated into this solution.
