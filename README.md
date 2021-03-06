Arcade PI RTSP server and NeoPixel based MQTT alerting / DHT!! MQTT posting. 

#Fork of "saito_mqtt_bed_neopixels" 

# Arcade Pi

Atempt to create a arcade style cabinet that connects to an MQTT broker for publishing temps and reading alert to publish on a 8LED neopixel light. I am going with a Pi as I also want a RTSP livecam for watching him at night (he is a toddler and having some sleep issues), but he is terrifed of Wyze cams. 

There are various guides on RTSP and Pi's so I won't cover that here. 

Makes use of Jeremy Garffs neopixel lib for the Rpi. More info at:
* https://learn.adafruit.com/neopixels-on-raspberry-pi?view=all
* https://github.com/jgarff/rpi_ws281x

## Installing rpi_ws281x

```shell
sudo apt-get update
sudo apt-get install build-essential python-dev git scons swig
cd ; git clone https://github.com/jgarff/rpi_ws281x.git
cd rpi_ws281x
scons
cd python
sudo python setup.py install
```

## Other requirements

You also need jsonschema and paho mqtt libs. *Updated to pip3

```shell
sudo pip3 install jsonschema
sudo pip3 install paho-mqtt
```

## Hardware

Currently library defaults to GPIO18 (pin 12) on http://www.raspberry-pi-geek.com/howto/GPIO-Pinout-Rasp-Pi-1-Model-B-Rasp-Pi-2-Model-B

## Home Assistant

Just add an MQTT JSON Light with following config:

```
# Enable mqtt
mqtt:
  broker: xxxxxxxxxxxxx
  port: 8000
  client_id: xxxxxxxxxxxxxxxxx
  keepalive: 60

# Example configuration.yaml entry
light:
  platform: mqtt_json
  name: "Saito RGB Light"
  command_topic: "saito/bed/neopixels/set"
  state_topic: "saito/bed/neopixels"
  brightness: true
  rgb: true
  effect: true
  effect_list: [rainbow, rainbowcycle, theaterchaserainbow, colorwipe, theaterchase]
```

and you are good to go.

State and commands are passed using json:

```json
{
  "brightness": 255,
  "color": {
    "g": 255,
    "b": 255,
    "r": 255
  },
  "effect": "rainbow",
  "transition": 2,
  "state": "ON"
}
```

## Autostart on Raspberry Pi

Just use pm2 to start the script

```shell
sudo su
pm2 startup
pm2 start app.py --name saito_bed_neopixels
pm2 save
```

Or systemd

```shell
chmod +x app.py

sudo su
cp neopixel.service /etc/systemd/system
systemctl enable neopixel.service
systemctl start neopixel.service
```
