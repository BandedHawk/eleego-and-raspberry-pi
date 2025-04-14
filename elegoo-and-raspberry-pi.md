# Connecting the Eleego 8-Channel Relay Module to a Raspberry Pi

The Eleego 8-Channel Relay Module is a relay interface board designed to work with the Arduino kits and the Raspberry Pi units.

The board uses optocouplers to control the relays. As the name implies, optocouplers transmit digital signals through the use of light. This separates the interface to the controller (the Raspberry Pi) from the circuits that activate the relays. Since driving the relay coils can cause a large current draw leading to fluctuations in the supply, this isolation design is appreciated in the module so as not to impact the Raspberry Pi.

The Raspberry Pi has a GPIO exposed via a set of pins that allow the unit to connect to the Eleego 8-Channel Relay Module and control the relays. The GPIO has a maximum voltage of 3.3V. The Raspberry Pi GPIO pinout has a 3.3V supply for use by the Eleego board interface circuit. This allows the Eleego board to match a "high" signal from the Raspberry Pi GPIO.

The relays on the Eleego board require a 5V supply to be able to switch the state of the relays. The Raspberry Pi also has a 5V supply on the GPIO pinouts. However, you should only use this to power the Eleego board if your power supply for the Raspberry Pi can deliver 5V at 4A, particularly if you are using the Raspberry Pi 3 and 4 series. You can buy power supplies for the Raspberry Pi 4 that will do the job, and if you have a Raspberry Pi 3B+ or earlier, just have a USB-C to USB-A converter. The power supply I bought had a converter provided. Otherwise, just buy a separate 5V 2A power supply for the relays but you'll need to work out how to hook them up yourself.

![alt text](https://raw.githubusercontent.com/BandedHawk/eleego-and-raspberry-pi/master/images/8-channel-relay.png "Eleego 8-Channel Relay Board View")
![alt text](https://raw.githubusercontent.com/BandedHawk/eleego-and-raspberry-pi/master/images/RPI3_GPIO_card.png "Raspberry Pi 3B+ GPIO Pin Mapping - c/o freva.com")