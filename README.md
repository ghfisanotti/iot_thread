# Investigation about Thread protocol

This project had the purpose to learn about the Thread protocol and its relation to HomeAssistant and IOT devices using ESPhome firmware.

## OpenThread

OpenThread is the open source implementation of Thread developed by Google.

Thread is a protocol designed to be used in IOT devices, particularly for battery-powered devices that spend most of their time deep-sleeping and require low latency and rapid connection with low bandwidth requirements. 

The Thread network is meshed and can have multiple border routers for augmented coverage and resilience. Some Thread devices (FTH: Full Thread Device) can act as routers too.

The Thread protocol use the same radio that WiFi (2.4GHz) but it does not use the WiFi protocol (no connection to AP, etc.). It uses IPv6 instead, this does not require DHCP, IPv6 addresses are automagically self-generated.
 
To interact with HomeAssistant a Thread Border Router (at least one) is required on the network. In this project I implemented my own TBR using a couple of ESP32 boards and a firmware provided by Espressif.

## The Thread Border Router

The TBR I built was based on the Espressif opentread example that is included with ESP-IDF (see examples/openthread/ot_rcp and examples/openthread/ot_br)

The hardware required are a couple of ESP32 boards:

- one to run the ot_br (the TBR code plus the WiFi interface), this can be any ESP32, I used an ESP32 Devkit V1.

- one to run the ot_rcp (the Radio Co-Processor, this handles the Thread network exclusively), this has to be any ESP32 whith support for the IEEE 802.15.4 standard. I used an ESP32-C6.

The instructions to configure, build and flash each of the boards is in the README.md of each component. Basically:

- For ot_br:

~~~
idf.py set-target esp32
idf.py menuconfig 
# configure the WiFi SSID and password
idf.py build
idf.py -p /dev/ttyACM0 flash
~~~

- For ot_rcp:

~~~
idf.py set-target esp32c6
idf.py build
idf.py -p /dev/ttyACM0 flash
~~~

The ot_br talks to the ot_rcp via an UART interface, this are the wiring required:

~~~
ot_br pin D4 to ot_rcp pin Tx
ot_br pin D5 to ot_rcp pin Rx
ot_br pin GND to ot_rcp pin GND
ot_br pin VIN to ot_rcp pin 5V
~~~

The cli console of the TBR is accessible thru the USB connector of the ESP32 (ot_br)

## HomeAssistant and Thread

In HA not much configuration is required, the Thread integration is automatically activated as soon as the TBR is detected. The only configuration required is to define the default Thread network, what requires the active dataset string obtained via the ot_br console cli:

~~~
dataset active -x
~~~

Once the default thread network is configured, the ESPhome thread devices are detected as usual.

## ESPhome and OpenThread

ESPhome now supports OpenThread on those ESP32 devices that have radios supporting the IEEE 802.15.4 standard, like the ESP32-C6 and ESP32-H2.

The only difference in configuration is the change of the wifi: and captive_portal: components for the following:

~~~
network:
  enable_ipv6: True
  
openthread:
  tlv: 0e080000000000.....
~~~

where tlv: is the active dataset of the thread network, which can be obtained thru the TBR console cli withe following command:

~~~
dataset active -x
~~~

### NOTES

- OTA over Thread is possible, but painfully slow, I waited for more than 10 minutes for un update. It may be due to the low bandwith of my DIY TBR, or it may be tuned for better performance...

### REFERENCES

- Obtaining ESP-IDF:

~~~
git clone -b v5.4.1 --recursive https://github.com/espressif/esp-idf.git esp-idf-v5.4.1
~~~
