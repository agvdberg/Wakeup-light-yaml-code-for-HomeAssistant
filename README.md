# A wake-up light using a dimmer wall module
Personal project to use a wall module behind a normal switch to turn on the ceiling light in the bedroom.

## Needed hardware

 - Wifi Wall Dimmer Module
 
<div align="center">
  <kbd>
    <img src="images/qs-wifi_D01_dimmer.webp" />
  </kbd>
</div>

 - Pulse switch

 (plaatje puls switch)
 
 - normal wall mount
 - normal ceiling light (dimmable led)
 
## Needed software

- Tasmota
- [HomeAssistant software](https://www.home-assistant.io/)	

## Steps 

### Program Wall dimmer module with Tasmota firmware

- compile tasmota software 
- burn tasmota software
- add wall module

(plaatje montage)

## Not covered but links where to find

- how to compile tasmota software
- how to burn tasmota software on wall-module
- HomeAssistant. [Starting point](https://www.home-assistant.io/)
- Building User Interfaces and automations in HomeAssistant. [A lot on YouTube](https://www.youtube.com/results?search_query=home+assistant+beginners+guide)

### License

This project is licensed under the [MIT License](LICENSE.md).

## Acknowledgments

Inspiration, code snippets, etc.
 * [README copied from Michael Stickels README template](https://github.com/MichaelStickels/README_Template)
 * [Adapted from TINY README](https://gist.github.com/noperator/4eba8fae61a23dc6cb1fa8fbb9122d45)


------------------------

# Examples to use in editing

## level 2 header
### level 3 header

- opsomming 1
- opsomming 2
- [ESPHome software](https://esphome.io/)  		(to program the ESP8266)
 
 Plaatje

Code
```
# button functionality
# Button is pushed low for 0,5 sec
button:
  - platform: template
    name: ${devicename} Button-CHANNEL-RX-IO3
    id: wemosd1_rx_button
    icon: "mdi:gesture-tap-button"
    on_press:
      - logger.log: "Knop CHANNEL ingedrukt = output RX even LOW = CHANNEL"
      - switch.turn_off: pin3   # negative: pin=off => output low
      - delay: 500ms 
      - switch.turn_on: pin3    # pin=on => output high
  - platform: template
    name: ${devicename} Button-UP-D5-IO14
    id: wemosd1_d5_button
    icon: "mdi:gesture-tap-button"
    on_press:
      - logger.log: "Knop UP ingedrukt = output D5 even LOW = UP"
      - switch.turn_off: pin14   # negative: pin=off => output low
      - delay: 500ms 
      - switch.turn_on: pin14    # pin=on => output high
  - platform: template
    name: ${devicename} Button-STOP-D6-IO12
    id: wemosd1_d6_button
    icon: "mdi:gesture-tap-button"
    on_press:
      - logger.log: "Knop STOP ingedrukt = output D6 even LOW = STOP"
      - switch.turn_off: pin12   # negative: pin=off => output low
      - delay: 500ms 
      - switch.turn_on: pin12    # pin=on => output high
  - platform: template
    name: ${devicename} Button-DOWN-D7-IO13
    id: wemosd1_d7_button
    icon: "mdi:gesture-tap-button"
    on_press:
      - logger.log: "Knop DOWN ingedrukt = output D7 even LOW = DOWN"
      - switch.turn_off: pin13   # negative: pin=off => output low
      - delay: 500ms 
      - switch.turn_on: pin13    # pin=on => output high
```

Tabel

| RGB LED (PL9823)            | ESP - WemosD1 mini Pro |
|-----------------------------|------------------------|
| (short) pin 1 Data In (DI)  | TGPIO00 = D3           |
| (short) pin 2 V+ (5V)       | 5V                     |
| (long)  pin 3 GND           | GND                    |
| (long)  pin 4 Data Out (DO) | not connected          |

Bullet's

### To-do

- [x] Working with a prototype-board
- [ ] ~~Redesign and use an dedicated pcb~~
- [ ] ~~Temperature & Humidity sensor~~
- [ ] Design and print a nice 3D case


