Homepoint
=============
#### An ESP32 based Smarthome Controller for MQTT

![Travis CI](https://travis-ci.org/sieren/Homepoint.svg?branch=master "Travis CI
Status")
[![Issues](https://img.shields.io/github/issues/sieren/Homepoint.svg
"Issues")](https://github.com/sieren/Homepoint/issues)

Homepoint is a screen-based interface for MQTT-connected Smarthome devices that runs
on the cheaply available ESP32 Chipset.
This project requires a MQTT-Broker to be running in your smarthome.

<img src="https://raw.githubusercontent.com/sieren/Homepoint/master/media/img1.jpg" width="400"><img src="https://raw.githubusercontent.com/sieren/Homepoint/master/media/img2.gif" width="400">  
[![HomePoint Youtube demo](https://raw.githubusercontent.com/sieren/Homepoint/master/media/youtube.png)](https://www.youtube.com/watch?v=bqzpkvtQSvY "HomePoint Youtube demo")


## Features

+ See and trigger scenes (groups of devices) on the front screen.
+ Trigger individual devices by diving into scenes (long press).
+ Support for temperature, humidity and air quality sensors.
+ Supports both Touch Screen or Button based navigation.
+ Screen updates automatically when devices are triggered from elsewhere.
+ Easy configuration through a JSON files.
+ Screensaver saves power by switching off screen after 10 minutes.
+ Statusbar shows connectivity for WiFi and MQTT as well as time.

#### Planned
+ HomeKit Button/Switch Support (for users without MQTT to setup Automations they can toggle from Homepoint)
+ Improved documentation  
  
Is there a feature missing? Open an issue, send me an [email](mailto:info@s-r-n.de) or fork this project and add it yourself.

## Requirements
Homepoint works with any ESP32 device and Touchscreen supported by [TFT_eSPI](https://github.com/Bodmer/TFT_eSPI), which are:  
ILI9341, ILI9163, ST7735, S6D02A1, ILI9481, ILI9486, ILI9488, HX8357D and ST7789 based TFT displays that support SPI.
  
It has been tested with  
+ [M5STACK](https://m5stack.com/collections/m5-core) 
+ [ArduiTouch](https://www.hwhardsoft.de/english/webshop/arduitouch/)  
+ [Adafruit Huzzah32](https://www.adafruit.com/product/3405) + [Adafruit IL9341](https://www.adafruit.com/product/2478)  


## Installation
In order to build Homepoint, you need the [ESP-IDF SDK](https://github.com/espressif/esp-idf).
### Requirements  

| Software       | Version                                                                                                                 |
| :------------- | :----------------------------------------------------------------------------------------------------------------------:|
|  ESP-IDF       | 3.3 (current [master](https://github.com/espressif/esp-idf))                                                            |
|  Toolchain     | [5.2.0](https://docs.espressif.com/projects/esp-idf/en/stable/get-started-cmake/index.html#step-1-set-up-the-toolchain) |
  
Installation is fairly straight forward, see [Setting up the Toolchain](https://docs.espressif.com/projects/esp-idf/en/stable/get-started-cmake/index.html#step-1-set-up-the-toolchain) in the Espressif Documentation.  
```
git clone https://www.github.com/sieren/homepoint.git
cd homepoint
git submodule update --init --recursive
mkdir build && cd build
cmake ../
make spiffs_spiffs_bin
make -j 8 flash
```  
For use with a Button-Based interface configure with
```
cmake -DBUTTONS=ON ../
```
and for out of box use with [M5STACK](https://m5stack.com/collections/m5-core) run
```
cmake -DM5STACK=ON ../
```
  
Otherwise the default configuration will be for touch-screens like [ArduiTouch](https://www.hwhardsoft.de/english/webshop/arduitouch/) or the widely used ILI9341 like the one offered by [Adafruit](https://www.adafruit.com/product/1770).
GPIOs for the screen may have to be configured in `main/libraries/TFT_ESPI/User_Setup.h`.  
  
Other Homepoint specific configuration settings like the time it takes before the Screensaver kicks in can be configured in `main/config/Config.h`.
  
## Configuration
See the `data/config.json` file as a reference.  
Each Scene consists of one or several devices. The scene can either be a `Switch`, `Light` or a `Sensor`.  
Do not put `Switch` devices into a `Sensor` scene.  
If Sensors transmit their information via JSON, set `jsonData` to true, and specify the key under which the data is sent (ie. `temperature`).  
Sensor can have up to two devices, in which case a combined view is shown (ie. for temperature and humidity).  
Wifi & MQTT credentials go in the top level.  
Homepoint gets its time from an NTP Server. In order to set the correct timezone, copy & paste your NTP TZ Setting from [here](https://remotemonitoringsystems.ca/time-zone-abbreviations.php).  

### Example
```
{
  "wifi": "MyWifiSSID",
  "password": "My Wifi Password",
  "mqttbroker": "mqtt://192.168.1.2",
  "mqttusername": "mqttusername",
  "mqttpasswd": "mymqttpassword",
  "timezone": "CET-1CEST-2,M3.5.0/02:00:00,M10.5.0/03:00:00",
  "scenes": [{
    "name": "Living Room",
    "type": "Light",
    "icon": "livingroom",
    "devices": [{
      "name": "Side Light",
      "setTopic": "lights/hue/00:17:88:01:02:3c:2a:6d-0b/set/on",
      "getTopic": "hue/status/lights/Hue color lamp 1",
      "onValue": "true",
      "offValue": "false"
    }]
  },
  {
    "name": "Bedroom",
    "type": "Sensor",
    "icon": "door",
    "devices": [{
      "name": "Temperature DHT Sensor",
      "type": "temperature",
      "jsondata": true,
      "temperatureKey": "temperature",
      "getTopic": "bedroom/esptemp"
    },
    {
      "name": "Humidity DHT Sensor",
      "type": "humidity",
      "jsondata": true,
      "humidityKey": "humidity",
      "getTopic": "bedroom/esphumi"
    }]
  }]
}
```
Switches can have custom icons, the icons of sensors are currently fixed and not customizable.
See the `data` folder for available icons or add your own following the naming scheme visible there.
