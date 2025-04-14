# Connecting the Eleego 8-Channel Relay Module to a Raspberry Pi

The Eleego 8-Channel Relay Module is a relay interface board designed to work with the Arduino kits and the Raspberry Pi units.

## Overview of the Board 

The board uses optocouplers to control the relays. As the name implies, optocouplers transmit digital signals with light. This separates the interface to the controller (the Raspberry Pi) from the circuits that activate the relays. Since driving the relay coils can cause a large current draw, leading to fluctuations in the supply, this isolation design is appreciated in the module so as not to impact the Raspberry Pi's operation.

The relays have three screw terminals each. They are electrically isolated from the circuits to interface with the controller and the circuits to switch the relays. The relays are rated for 240V AC and 10A, which could theoretically handle controlling mains power. Depending on the load current for the appliance you are switching on or off, I'd be worried about the switch fusing, so be aware of this. The center terminal of the three-set for the relay is the **common** terminal (**C**). The relay controls the connectivity of the common terminal to the terminals on either side. One of these terminals is **normally closed** (**NC**), and the other is **normally open** (**NO**). The **normally open** terminal only closes when a current flows through the relay coil. Generally, you should use the **common** and **normally open** terminals so that the default state of your external circuit is not active. This will depend on your control scenario, so design your wiring accordingly.

## Physical Connection to a Raspberry Pi

The Raspberry Pi has a GPIO exposed via pins that allow the unit to connect to the Eleego 8-Channel Relay Module and control the relays. The GPIO has a maximum voltage of 3.3V. The Raspberry Pi GPIO header pins have a 3.3V supply for the Eleego board interface circuit. This allows the Eleego board to match a "high" signal from the Raspberry Pi GPIO. I'll be describing connectivity based on a physical Raspberry Pi 3B+. Adjust by referencing the documentation for your Raspberry Pi GPIO header.

The relays on the Eleego board require a 5V supply to switch the relays' state. The Raspberry Pi also has a 5V supply on the GPIO pinouts. However, you should only use this to power the Eleego board if your power supply for the Raspberry Pi can deliver 5V at 4A, particularly if you use the Raspberry Pi 3 and 4 series. You can buy power supplies for the Raspberry Pi 4 that will do the job, and if you have a Raspberry Pi 3B+ or earlier, just have a USB-C to USB-A converter. A converter was provided for the power supply I bought. Otherwise, just buy a separate 5V 2A power supply for the relays, but you'll need to work out how to hook them up yourself.

The first thing to do with the Eleego board is to remove the jumper for the relay circuit power that connects VCC to JD-VCC. When the board ships, the jumper is set to connect the relay circuit power rail to the controller interface power rail. This won't work with the Raspberry Pi. As stated, the Raspberry Pi GPIO power is 3.3V, and the relay circuit needs 5V. By the way, VCC refers to the power rail for the Common Collector. That dates to the time when discrete transistors were more common on boards. JD-VCC is also typical nomenclature for Jumper-Determined Voltage Controlled Circuit.

![alt text](https://raw.githubusercontent.com/BandedHawk/eleego-and-raspberry-pi/master/images/8-channel-relay.png "Eleego 8-Channel Relay Board View")

Having removed the jumper connecting the VCC supply to the JD-VCC rail, you must provide a 5V source and its corresponding ground (GND)  to the relay power pins. Do not connect anything to the VCC pin in that grouping, unless you want to connect the 3.3V source from the Raspberry Pi. If you do that, you don't need to connect the 3.3V source to the Interface set of the relay board from the Raspberry Pi. Refer to the Raspberry Pi GPIO header diagram for the GND, 5V, and 3.3V sources.  Remember to be careful with pin counting on the GPIO header when connecting wires. I prefer to keep the 3.3V power line with the rest of the GPIO connections.

![alt text](https://raw.githubusercontent.com/BandedHawk/eleego-and-raspberry-pi/master/images/RPI3_GPIO_card.png "Raspberry Pi 3B+ GPIO Pin Mapping - c/o freva.com")

I recommend pre-made female-to-female breadboard rainbow cables for wiring between the Raspberry Pi and the Eleego 8-channel relay board. Refer to the picture provided for an example. The female-to-female leads are the ones in the centre. You can peel them apart in the appropriate groupings. I used a strip of two for the relay power, and a group of ten for the GPIO connections.

![alt text](https://raw.githubusercontent.com/BandedHawk/eleego-and-raspberry-pi/master/images/breadboard-rainbow-cables.png "Breadboard Rainbow Cables")

For the GPIO connections, aside from the GND and 3.3V power, connect ot GPIO active pins to your liking. I used GPIO ports 25, 24, 23, 22, 27, 18, 17, and 4. These are reasonably clustered with the GND and 3.3V pins, which is ideal for keeping the wiring compact on the GPIO header. Again, be careful with pin counting and connection. Although the GPIO circuits are reasonably robust and don't blow up easily with wrong connections, you don't want to tempt fate.

After you have wired the connections between the Raspberry Pi and the Eleego board, try powering the Raspberry Pi and ensure both boards are fine. The Raspberry Pi should boot normally, the low power status does not come up on the desktop, and no smoke appears on either board. The red status LEDs for the relays on the Eleego board should be dimly lit. Yes. This is a real smoke test. That's how we got the phrase.

## Coding the GPIO Controls

The Raspberry Pi OS with desktop and recommended software will allow you to control the GPIO through Python. The GPIO on boot with Raspberry Pi OS is not configured so the GPIO ports are not set for input or output.

The GPIO software module must be initialized. It can be initialized by using the GPIO header pin numbering for reference or the Broadcom GPIO numbering. I prefer using the GPIO header pin numbering as that is less ambiguous when debugging physical connections.

```python
GPIO.setmode(GPIO.BOARD)
```

Next, you must define the GPIO ports as either inputs or outputs. To control the Eleego board, we want the ports to be set up as outputs. We should also define the state of the output port. The relay is on or energized when the output port state is **low**. In this case, we want to define the relevant GPIO port initialized in a **high** state. If we wanted to set GPIO header pin 22 (GPIO25) as an output for the Eleego board, we would do the following to initialize the output port.

```python
GPIO.setup(22, GPIO.OUT)
GPIO.output(22, GPIO.HIGH)
```
Note that if you specify a header pin number that is not a port, you will get an error message when running the Python program. This will help you with debugging.

The other thing to remember is to clean up the GPIO state before exiting the Python program. This is accomplished by the following:

```python
GPIO.cleanup()
```
The GPIO clean up will reset the state of the port outputs, so be aware that the final state after the Python program has exited may not be what you desire. 

The program below shows an example of all these elements being incorporated to randomly turn on and off relay switches in short bursts until the program is interrupted by a key press.

```python
## Import modules necessary to control the Raspberry Pi GPIO and drive the port status
import RPi.GPIO as GPIO
import time
import random

# Define GPIO pin numbers participating in the relay control
pins = [22, 18, 16, 15, 13]
# Define GPIO pin numbers wired for relay control but not participating 
extras = [12, 11, 7]

## Set the mode for GPIO reference using the Raspberry Pi header pin numbers
GPIO.setmode(GPIO.BOARD)
## Set each port for output and initialize it so the relay is in an "off" state.
for x in pins:
    GPIO.setup(x, GPIO.OUT)
    GPIO.output(x, GPIO.HIGH)
for x in extras:
    GPIO.setup(x, GPIO.OUT)
    GPIO.output(x, GPIO.HIGH)

## Loop until there is a keyboard interrupt
try:
    while True:
        ## Randomly choose an output port to control
        pin = random.randint(0, 4)
        print(pin)
        ## Switch on the relay for 100 milliseconds 
        GPIO.output(pins[pin], GPIO.LOW)
        time.sleep(0.1)
        ## Switch off the relay for 750 milliseconds 
        GPIO.output(pins[pin], GPIO.HIGH)
        time.sleep(0.75)
        ## Switch on the relay for 100 milliseconds 
        GPIO.output(pins[pin], GPIO.LOW)
        time.sleep(0.1)
        GPIO.output(pins[pin], GPIO.HIGH)
        time.sleep(0.75)
except KeyboardInterrupt:
    print("exiting")
    ## Turn off all the relays on interrupt in case any were on when interrupted 
    for x in pins:
        GPIO.output(x, GPIO.HIGH)
print("cleanup")
## Clean up the GPIO state so it is clean for the next use of the GPIO 
GPIO.cleanup()
```

That's it. Hopefully, this article has provided some tips for people using the Eleego relay board or programming the GPIO on the Raspberry Pi for the first time.


