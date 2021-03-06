# Projects
Most of these are based on existing examples from the ESPHome documentation and guides like [Frencks doorbell bloggpost](https://frenck.dev/diy-smart-doorbell-for-just-2-dollar/). Usually mix different parts from configs together to get the best results.

## Smart doorbell

- ESP32 Board with:
  - Camera module
  - Buttons
  - PIR-sensor
  - Display (not needed but it was bundled with the board)
- [ESP8266 ESP-01S](https://www.aliexpress.com/item/32880024822.html) - (Relay-board)
- [Wired Doorbell Chime 12V DC](https://www.aliexpress.com/item/32966895779.html)
- [BINZET DC 12V 24V to 5V 5A Converter Step Down Regulator](https://www.amazon.com/gp/aw/d/B00J3MHT1E) - 12V -> 5V
- [12V 2A Power Supply EU plug](https://www.aliexpress.com/item/4000102102421.html)

| | |
|----|----|
| ![Doorbell Chime](images/doorbell_chime.jpg) | ![ESP8266_ESP-01S](images/ESP8266_ESP-01S.jpg) |
| ![ttgocam](images/ttgocam_fisheye_lense.jpg) |  |

### Config-files

#### Doorbell-relay-chime config
```
---
esphome:
  name: doorbell
  platform: ESP8266
  board: esp01_1m

# WiFi connection, correct these
# with values for your WiFi.
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

# Enable logging.
logger:

# Enable Home Assistant API.
api:

# Enable over-the-air updates.
ota:

# Enable Web server.
web_server:
  port: 80

# Sync time with Home Assistant.
time:
  - platform: homeassistant
    id: homeassistant_time

# Text sensors with general information.
text_sensor:
  # Expose ESPHome version as sensor.
  - platform: version
    name: Doorbell ESPHome Version
  # Expose WiFi information as sensors.
  - platform: wifi_info
    ip_address:
      name: Doorbell IP
    ssid:
      name: Doorbell SSID
    bssid:
      name: Doorbell BSSID

# Sensors with general information.
sensor:
  # WiFi Signal sensor.
  - platform: wifi_signal
    name: Doorbell WiFi Signal
    update_interval: 60s

# Global to store the on/off state of the chime
globals:
  - id: chime
    type: bool
    restore_value: true
    initial_value: 'true'

# Exposed switches.
switch:
  # Switch to restart the doorbell.
  - platform: restart
    name: Doorbell Restart

  # Switch to turn on/off the chime.
  - platform: gpio
    id: relay
    inverted: true
    name: Doorbell Chime
    pin: GPIO0

  # Switch to turn on/off chime when
  # doorbell button is pushed.
  #
  # It creates a "virtual" switch based
  # on a global variable.
  - platform: template
    name: Doorbell Chime Active
    id: chime_active
    restore_state: false
    turn_on_action:
      - globals.set:
          id: chime
          value: 'true'
    turn_off_action:
      - globals.set:
          id: chime
          value: 'false'
    lambda: |-
      return id(chime);

# Binary sensor representing the
# Doorbell button push.
binary_sensor:
  - platform: gpio
    id: button
    name: Doorbell Button
    pin:
      # Connected to GPIO on the ESP-01S.
      number: GPIO2
      mode: INPUT_PULLUP
      inverted: true
    filters:
      # Small filter, to debounce the button press.
      - delayed_on: 25ms
      - delayed_off: 25ms
    on_press:
      # Only turn on the chime when it is active.
      then:
        if:
          condition:
            - switch.is_on: chime_active
          then:
            - switch.turn_on: relay
    on_release:
      # On release, turn of the chime.
      - switch.turn_off: relay
```

#### Ttgocam config
```
substitutions:
  devicename: ttgocam
  friendly_name: esp32cam
  ip_address: !secret ttgocam_static_ip_address

esphome:
  name: $devicename
  platform: ESP32
  board: esp-wrover-kit

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  manual_ip:
    static_ip: $ip_address
    gateway: !secret gateway_ip
    subnet: !secret subnet_ip


# Enable logging
logger:

# Enable Home Assistant API
api:

ota:

# Enable Web server.
web_server:
  port: 80

binary_sensor:
  - platform: gpio
    pin: GPIO33
    name: $friendly_name PIR
    device_class: motion

  - platform: gpio
    pin:
      number: GPIO34
      mode: INPUT_PULLUP
      inverted: True
    name: $friendly_name Button

  - platform: status
    name: $friendly_name Status

sensor:
  - platform: wifi_signal
    name: $friendly_name WiFi Signal
    update_interval: 10s

esp32_camera:
  name: $friendly_name Camera
  external_clock:
    pin: GPIO32
    frequency: 20MHz
  i2c_pins:
    sda: GPIO13
    scl: GPIO12
  data_pins: [GPIO5, GPIO14, GPIO4, GPIO15, GPIO18, GPIO23, GPIO36, GPIO39]
  vsync_pin: GPIO27
  href_pin: GPIO25
  pixel_clock_pin: GPIO19
  power_down_pin: GPIO26
  resolution: 800x600
  jpeg_quality: 10
  vertical_flip: false
  horizontal_mirror: false

i2c:
  sda: GPIO21
  scl: GPIO22

font:
  - file: "fonts/times-new-roman.ttf"
    id: tnr1
    size: 20
  - file: "fonts/times-new-roman.ttf"
    id: tnr2
    size: 35

time:
  - platform: homeassistant
    id: homeassistant_time

display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    address: 0x3C
    lambda: |-
      it.strftime(64, 0, id(tnr1), TextAlign::TOP_CENTER,"%d-%m-%Y", id(homeassistant_time).now());
      it.strftime(64, 64, id(tnr2), TextAlign::BASELINE_CENTER, "%H:%M", id(homeassistant_time).now());
```

#### Camera, PIR-sensor, Chime and doorbell-button
Todo, add schematics and automation rules.

## Bluetooth tracking hub (in planning stage)
- ESP32 Board with:
  - Bluetooth
  - WiFi

### Track presence

### Tracking devices

### Tracking sensors

#### Plantsensor (Done, need to update with config file)
| |
|----|
| ![Xiaomi MiFlora](images/Xiaomi_MiFlora.jpg) |

#### Software

[ESPHome](https://esphome.io/components/sensor/xiaomi_hhccjcy01.html)

#### Config-files
```
Todo
```

#### Temperature and Humidity
| |
|----|
| ![Xiaomi MiJia](images/Xiaomi_MiJia.jpg) |

#### Software

[ESPHome](https://esphome.io/components/sensor/xiaomi_lywsdcgq.html)

#### Config-files

```
esphome:
  name: 'tempsensors'
  platform: ESP32
  board: esp32dev

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  manual_ip:
    # Set this to the IP of the ESP
    static_ip: !secret esp_temphub_static_ip
    # Set this to the IP address of the router. Often ends with .1
    gateway: !secret gateway_ip
    # The subnet of the network. 255.255.255.0 works for most home networks.
    subnet: !secret subnet_ip
    dns1: 8.8.8.8 # google-dns
    dns2: 8.8.4.4 # google-dns

api:

web_server:
  port: 80

logger:

ota:

# Enable Bluetooth scanning for this ESP32
esp32_ble_tracker:

sensor:
  - platform: xiaomi_lywsdcgq
    mac_address: !secret bedroom_xiaomi_mijia_mac_address
    temperature:
      name: "Bedroom Xiaomi MiJia Temperature"
    humidity:
      name: "Bedroom Xiaomi MiJia Humidity"
    battery_level:
      name: "Bedroom Xiaomi MiJia Battery Level"
  - platform: xiaomi_lywsdcgq
    mac_address: !secret bathroom_xiaomi_mijia_mac_address
    temperature:
      name: "Bathroom Xiaomi MiJia Temperature"
    humidity:
      name: "Bathroom Xiaomi MiJia Humidity"
    battery_level:
      name: "Bathroom Xiaomi MiJia Battery Level"
  - platform: xiaomi_lywsdcgq
    mac_address: !secret livingroom_xiaomi_mijia_mac_address
    temperature:
      name: "Livingroom Xiaomi MiJia Temperature"
    humidity:
      name: "Livingroom Xiaomi MiJia Humidity"
    battery_level:
      name: "Livingroom Xiaomi MiJia Battery Level"
  - platform: xiaomi_lywsdcgq
    mac_address: !secret kitchen_xiaomi_mijia_mac_address
    temperature:
      name: "Kitchen Xiaomi MiJia Temperature"
    humidity:
      name: "Kitchen Xiaomi MiJia Humidity"
    battery_level:
      name: "Kitchen Xiaomi MiJia Battery Level"
  - platform: wifi_signal
    name: "TempHub WiFi Signal Sensor"
    update_interval: 60s
```

## Remotely controlled powerplugs

- Sonoff s20

| |
|--|
| ![Sonoff S20](images/sonoff_s20.jpg) |

### Software
[Sonoff HomeAssistant integration](https://github.com/AlexxIT/SonoffLAN/)

### Config-files
#### In configuration.yaml
```
# Enable sonoff integration
sonoff:
  username: !secret sonoff_user # mapped to variable in secrets.yaml
  password: !secret sonoff_pass # mapped to variable in secrets.yaml
#  reload: always  # update device list every time HA starts
```
#### In ui-lovelace.yaml

```
resources:
# Name: auto-entities-card
# Description: Automatically displays entities based on filters, makes it easy to show several entities with minimal config effort
# Used in config? Yes
# Maintainer: HACS
# source: https://github.com/thomasloven/lovelace-auto-entities/blob/master/README.md
  - url: /community_plugin/lovelace-auto-entities/auto-entities.js
    type: module

# Name: fold-entity-row
# Description: Enables collapsible/expandable cards.
# Used in config? Yes
# Maintainer: HACS
# source: https://github.com/thomasloven/lovelace-fold-entity-row/blob/master/README.md
  - url: /community_plugin/lovelace-fold-entity-row/fold-entity-row.js
    type: module

title: Home
views:

  - icon: mdi:toggle-switch
    title: Switches
    id: switches
    cards:

      - type: custom:auto-entities
        filter:
          include:
            - domain: switch
              entity_id: "switch.sonoff_*"
        card:
          type: custom:fold-entity-row
          head:
            type: section
            label: All Sonoff switches
```

## LED PSU Calculation
- Estiame 50 LED per 1 amp
- Add 1-2 amps for the controller

## LED
Parts:
- 2x [NodeMCU](http://s.click.aliexpress.com/e/pI2wgExm) - 
- 4x [Aluminum Channel (50cm)](http://s.click.aliexpress.com/e/sHXT8lzI) - 13m total
- 2x [1m 60px IP67, White PCB, WS2812B, DC5V](http://s.click.aliexpress.com/e/Kceu4gks) - 

### Software

[WLED](https://github.com/Aircoookie/WLED)

### Kitchen cabinet lights
| |
|--|
| ![Aluminum Channel (50cm)](images/Aluminum_Channel_50cm_part1.jpg) |
| ![Aluminum Channel (50cm)](images/Aluminum_Channel_50cm_part2.jpg) |

## LCD info-screens
Parts:
- 2x [0.96 inch oled IIC Serial Yellow-blue OLED Display Module 128X64](https://www.aliexpress.com/item/32896971385.html) - 
- 1x [ESP32 WROOM-32, 240MHz](https://m.aliexpress.com/item/32864722159.html) - 

| |
|--|
| ![Yellow-blue OLED Display](images/oled_yellow_blue.jpg) |

Description:
- One screen to show the time and temperatures inside+outside.
- One screen to show delays and when the next train is scheduled for the subway. Useful when commuting to work.

### Software
* [ESPHome](https://esphome.io/)
* [ESPHome Screen cookbook](https://esphome.io/cookbook/display_time_temp_oled.html)

#### Config-files

```
substitutions:
  devicename: esp32screens
  friendly_name: ESP Screens
  ip_address: !secret esp32screens_ip_address

esphome:
  name: $devicename
  platform: ESP32
  board: esp32dev

wifi:
  ap:
    ssid: $devicename
    password: !secret esp32screens_wifi_password
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  manual_ip:
    # Set this to the IP of the ESP
    static_ip: $ip_address
    # Set this to the IP address of the router. Often ends with .1
    gateway: !secret esp32screens_gateway
    # The subnet of the network. 255.255.255.0 works for most home networks.
    subnet: !secret esp32screens_subnet

# Enable logging.
logger:

# Enable Home Assistant API.
api:

# Enable over-the-air updates.
ota:

# Enable Web server.
web_server:
  port: 80

sensor:
  - platform: wifi_signal
    name: $friendly_name WiFi Signal
    update_interval: 60s

  - platform: homeassistant
    id: outside_temperature # internal id used by lambda
    entity_id: sensor.yr_dewpoint_temperature # homeassistant entity_id
    internal: true

  - platform: homeassistant
    id: inside_temperature # internal id used by lambda
    entity_id: sensor.home_temperature_average# homeassistant entity_id
    internal: true

binary_sensor:
  - platform: status
    name: $friendly_name Status

i2c:
  sda: GPIO21
  scl: GPIO22

font:
  - file: 'fonts/slkscr.ttf'
    id: font1
    size: 16

  - file: 'fonts/BebasNeue-Regular.ttf'
    id: font2
    size: 48

  - file: 'fonts/Arial.ttf'
    id: font3
    size: 14

# Sync time with Home Assistant.
time:
  - platform: homeassistant
    id: esptime

# Display #1
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    address: 0x3C
    lambda: |-
      // Print "Home" in top center.
      it.printf(64, 0, id(font1), TextAlign::TOP_CENTER, "Home");

      // Print time in HH:MM format
      it.strftime(0, 60, id(font2), TextAlign::BASELINE_LEFT, "%H:%M", id(esptime).now());

      // Print inside temperature (from homeassistant sensor)
      if (id(inside_temperature).has_state()) {
        it.printf(127, 23, id(font3), TextAlign::TOP_RIGHT , "%.1f°", id(inside_temperature).state);
      }

      // Print outside temperature (from homeassistant sensor)
      if (id(outside_temperature).has_state()) {
        it.printf(127, 60, id(font3), TextAlign::BASELINE_RIGHT , "%.1f°", id(outside_temperature).state);
      }

# Display #2
# TODO


```

##### Adding two screens (example)
Description:
TODO! Need to use two entries for i2c and specifying the mapping for each screen.
Example from https://esphome.io/components/i2c.html
```
Note
If the device can support multiple I²C buses (ESP32 has 2) these buses need to be defined as below and sensors need to be setup specifying the correct bus:

 # Example configuration entry
 i2c:
   - id: bus_a
     sda: 13
     scl: 16
     scan: True
   - id: bus_b
     sda: 14
     scl: 15
     scan: True
# Sensors should be specified as follows
- platform: bme680
  i2c_id: bus_b
  address: 0x76
  # ...
```


## Lightning sensor

Parts:
- 1x [AMS AS3935 Franklin Lightning Sensor](https://esphome.io/components/sensor/as3935.html)
- 1x [ESP32 WROOM-32, 240MHz](https://m.aliexpress.com/item/32864722159.html) - 


### Software
* [ESPHome](https://esphome.io/components/sensor/as3935.html)

#### Config-files
```
substitutions:
  devicename: esp32lightning
  friendly_name: ESP Lightning
  ip_address: !secret esp32lightning_ip_address

esphome:
  name: $devicename
  platform: ESP32
  board: esp32dev

wifi:
  ap:
    ssid: $devicename
    password: !secret esp32lightning_wifi_password
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  manual_ip:
    # Set this to the IP of the ESP
    static_ip: $ip_address
    # Set this to the IP address of the router. Often ends with .1
    gateway: !secret esp32lightning_gateway
    # The subnet of the network. 255.255.255.0 works for most home networks.
    subnet: !secret esp32lightning_subnet

# Enable logging.
logger:

# Enable Home Assistant API.
api:

# Enable over-the-air updates.
ota:

# Enable Web server.
web_server:
  port: 80

# Sync time with Home Assistant.
time:
  - platform: homeassistant
    id: esptime

# i2c-pin for the lightning sensor
as3935_i2c:
  irq_pin: GPIO12

# WiFi signal
sensor:
  - platform: wifi_signal
    name: $friendly_name WiFi Signal
    update_interval: 60s

# Configuration for creating sensors
  - platform: as3935
    lightning_energy:
      name: $friendly_name Lightning Energy
    distance:
      name: $friendly_name Distance Storm
binary_sensor:
  - platform: as3935
    name: $friendly_name Storm Alert

```

## Temperature and gas sensor
BME680 has the following sensors:
* Temperature
* Pressure
* Humidity
* Gas Sensor

Parts:
- 1x  [BME680](https://www.aliexpress.com/item/32961369966.html) - 
- 1x [ESP32 WROOM-32, 240MHz](https://m.aliexpress.com/item/32864722159.html) - 


### Software
* [ESPHome](https://esphome.io/components/sensor/bme680.html)

#### Config-files
```
substitutions:
  devicename: espmultisensor
  friendly_name: esp32multisensor
  ip_address: !secret espmultisensor_static_ip_address

esphome:
  name: $devicename
  platform: ESP32
  board: esp32dev

wifi:
  ap:
    ssid: $devicename
    password: !secret esp32multisensor_wifi_password
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  manual_ip:
    static_ip: $ip_address
    gateway: !secret gateway_ip
    subnet: !secret subnet_ip

# Enable logging
logger:

# Enable Home Assistant API
api:

# Enable over-the-air updates.
ota:

# Enable Web server.
web_server:
  port: 80

i2c:
  sda: GPIO21
  scl: GPIO22

sensor:
  - platform: wifi_signal
    name: $friendly_name WiFi Signal
    update_interval: 60s

  - platform: bme680
    temperature:
      name: $friendly_name Temperature
      oversampling: 16x
    pressure:
      name: $friendly_name Pressure
    humidity:
      name: $friendly_name Humidity
    gas_resistance:
      name: $friendly_name Gas Resistance
    address: 0x77
    update_interval: 60s
```

