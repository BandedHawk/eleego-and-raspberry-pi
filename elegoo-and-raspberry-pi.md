# Connecting the Eleego 8-Channel Relay Module to a Raspberry Pi

The Eleego 8-Channel Relay Module is a relay interface board designed to work with the Arduino kits and the Raspberry Pi units.

The board uses optocouplers to control the relays. This separates the interface to controller (the Raspberry Pi) from the circuits that activate trhe relays. Since driving the relays can cause a large current draw and fluctuations in the supply due to the coils, this isolation design is appreciated in the module.

The Raspberry Pi has a GPIO exposed via a set of pins that allow the unit to connect to the Eleego 8-Channel Relay Module and control the relays.