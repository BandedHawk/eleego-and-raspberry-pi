# Connecting the Eleego 8-Channel Relay Module to a Raspberry Pi

The Eleego 8-Channel Relay Module is a relay interface board designed to work with the Arduino kits and the Raspberry Pi units.

## Overview of the Board 

The board uses optocouplers to control the relays. As the name implies, optocouplers transmit digital signals with light. This separates the interface to the controller (the Raspberry Pi) from the circuits that activate the relays. Since driving the relay coils can cause a large current draw, leading to fluctuations in the supply, this isolation design is appreciated in the module so as not to impact the Raspberry Pi's operation.

The relays have three screw terminals each. They are electrically isolated from the circuits to interface with the controller and the circuits to switch the relays. The relays are rated for 240V AC and 10A, which could theoretically handle controlling mains power. Depending on the load current for the appliance you are switching on or off, I'd be worried about the switch fusing, so be aware of this. The center terminal of the three-set for the relay is the **common** terminal (**C**). The relay controls the connectivity of the common terminal to the terminals on either side. One of these terminals is **normally closed** (**NC**), and the other is **normally open** (**NO**). The **normally open** terminal only closes when a current flows through the relay coil. Generally, you should use the **common** and **normally open** terminals so that the default state of your external circuit is not active. This will depend on your control scenario, so design your wiring accordingly.

## Physical Connection to a Raspberry Pi

The Raspberry Pi has a GPIO exposed via pins that allow the unit to connect to the Eleego 8-Channel Relay Module and control the relays. The GPIO has a maximum voltage of 3.3V. The Raspberry Pi GPIO header pins have a 3.3V supply for the Eleego board interface circuit. This allows the Eleego board to match a "high" signal from the Raspberry Pi GPIO. I'll be describing connectivity based on a physical Raspberry Pi 3B+. Adjust with documentation for your Raspberry Pi GPIO header.

The relays on the Eleego board require a 5V supply to switch the relays' state. The Raspberry Pi also has a 5V supply on the GPIO pinouts. However, you should only use this to power the Eleego board if your power supply for the Raspberry Pi can deliver 5V at 4A, particularly if you use the Raspberry Pi 3 and 4 series. You can buy power supplies for the Raspberry Pi 4 that will do the job, and if you have a Raspberry Pi 3B+ or earlier, just have a USB-C to USB-A converter. A converter was provided for the power supply I bought. Otherwise, just buy a separate 5V 2A power supply for the relays, but you'll need to work out how to hook them up yourself.

The first thing to do with the Eleego board is to remove the jumper for the relay circuit power that connects VCC to JD-VCC. When the board ships, the jumper is set to connect the relay circuit power rail to the controller interface power rail. This won't work with the Raspberry Pi. As stated, the Raspberry Pi GPIO power is 3.3V, and the relay circuit needs 5V. By the way, VCC refers to the power rail for the Common Collector. That dates to the time when discrete transistors were more common on boards. JD-VCC is also typical nomenclature for Jumper-Determined Voltage Controlled Circuit.

![alt text](https://raw.githubusercontent.com/BandedHawk/eleego-and-raspberry-pi/master/images/8-channel-relay.png "Eleego 8-Channel Relay Board View")

Having removed the jumper connecting the VCC supply to the JD-VCC rail, you must provide a 5V source and its corresponding ground (GND)  to the relay power pins. Do not connect anything to the VCC pin in that grouping, unless you want to connect the 3.3V source from the Raspberry Pi. If you do that, you don't need to connect the 3.3V source to the Interface set of the relay board from the Raspberry Pi. Refer to the Raspberry Pi GPIO header diagram for the GND, 5V, and 3.3V sources.  Remember to be careful with pin counting on the GPIO header when connecting wires. I prefer to keep the 3.3V power line with the rest of the GPIO connections.

![alt text](https://raw.githubusercontent.com/BandedHawk/eleego-and-raspberry-pi/master/images/RPI3_GPIO_card.png "Raspberry Pi 3B+ GPIO Pin Mapping - c/o freva.com")

I recommend pre-made female-to-female breadboard rainbow cables for wiring between the Raspberry Pi and the Eleego 8-channel relay board. Refer to the picture provided for an example. The female-to-female leads are the ones in the centre. You can peel them apart in the appropriate groupings. I used a strip of two for the relay power, and a group of ten for the GPIO connections.

![alt text](https://raw.githubusercontent.com/BandedHawk/eleego-and-raspberry-pi/master/images/breadboard-rainbow-cables.png "Breadboard Rainbow Cables")

For the GPIO connections, aside from the GND and 3.3V power, connect ot GPIO active pins to your liking. I used GPIO ports 24, 23, 22, 27, 18, 17, 15, and 14. These are reasonably clustered with the GND and 3.3V pins, which is ideal for keeping the wiring compact. Again, be careful with pin counting and connection. Although the GPIO circuits are reasonably robust and don't blow up easily with wrong connections, you don't want to tempt fate.

## Coding the GPIO Controls