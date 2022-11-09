# A wake-up light using a dimmer wall module
Personal project to use a wall module behind a normal switch to turn on the ceiling light in the bedroom.

## Needed hardware

 <b>Wifi Wall Dimmer Module</b>
 In this case i had a "QS-WiFi-D01-TRIAC 150W Dimmer Module" laying around.
  
<div align="center">
  <kbd>
    <img src="images/qs-wifi_D01_dimmer.webp" />
  </kbd>
</div>

 - Pulse switch

<div align="center">
  <kbd>
    <img src="images/gira-015100-wipptaster-schliesser-einsatz_4.jpg" />
  </kbd>
</div>
 
 - normal wall mount
 - normal ceiling light (with a dimmable led)
 
## Needed software

- [Tasmota software](https://tasmota.github.io/docs/)
- [HomeAssistant software](https://www.home-assistant.io/)	


## Step 1: flash Wall dimmer module with Tasmota firmware

The wall module can not use the normal precompiled tasmota software.
At [the templates page of blakadder](https://templates.blakadder.com/qs-wifi_D01_dimmer.html) this is explained.
It needs a 'dimmer' script. To turn on [scripting in tasmota](https://tasmota.github.io/docs/Scripting-Language/) you have to compile this.
There is an easy manual how to compile tasmota software, completely in the cloud at a [Tasmota-github-page](https://tasmota.github.io/docs/Compile-your-build/).
I used [GitPod](https://www.gitpod.io/) to compile tasmota.

In ```user_config_override.h``` i added
```
#ifndef USE_SCRIPT
#define USE_SCRIPT  // adds about 17k flash size, variable ram size
#endif
#ifdef USE_RULES
#undef USE_RULES
#endif 

#ifdef SCRIPT_POWER_SECTION
#undef SCRIPT_POWER_SECTION
#endif 
```

The last section (defining SCRIPT_POWER_SECTION) is not clearly stated in the bladadder-template-page.
It is needed to control the device by WebGUI and HomeAssistant.

With the new .bin-file you can flash the firmware to the wall-module.
How to flash? It is explained in [the templates page of blakadder](https://templates.blakadder.com/qs-wifi_D01_dimmer.html), but i used [OTA and 'TuyaConvert'](https://tasmota.github.io/docs/Tuya-Convert/) and didn't have to open the device to put wires to it.

## Step 2: add template and script
Reading [the templates page of blakadder](https://templates.blakadder.com/qs-wifi_D01_dimmer.html) you have to add the template for the device to tasmota:
```
 {"NAME":"QS-WiFi-D01-TRIAC","GPIO":[0,3200,0,3232,0,0,0,0,0,352,416,0,0,0],"FLAG":0,"BASE":18}
```

And add the script found at a [a github subpage](https://gist.github.com/thxthx0/12074f1f5249e14b2a0aada75f590c9b).

## Step 3: Testing script and availability
I tested the device using a wooden board

<div align="center">
  <kbd>
    <img src="images/WifiDimmer testsetup - smaller.jpg" />
  </kbd>
</div>

----------------------------

## Step 4: Check availibilty of the Tasmota-Wifi-Dimmer in HomeAssistant

You can test the function as wel

<div align="center">
  <kbd>
    <img src="images/HomeAssistant Dimmer Device.jpg" />
  </kbd>
</div>

### Step 5: Script to wake-up the light

The transition-command seems not available, so we have to set a few brightness levels in series

```
alias: Wake-up light
sequence:
  - service: light.turn_on
    data:
      brightness_pct: 10
    target:
      entity_id: light.dimmer1
    alias: Dimmer 10%
  - delay:
      hours: 0
      minutes: 1
      seconds: 0
      milliseconds: 0
  - service: light.turn_on
    data:
      brightness_pct: 20
    target:
      entity_id: light.dimmer1
    alias: Dimmer 20%
  - delay:
      hours: 0
      minutes: 1
      seconds: 0
      milliseconds: 0
  - service: light.turn_on
    data:
      brightness_pct: 30
    target:
      entity_id: light.dimmer1
    alias: Dimmer 30%
  - delay:
      hours: 0
      minutes: 1
      seconds: 0
      milliseconds: 0
  - service: light.turn_on
    data:
      brightness_pct: 40
    target:
      entity_id: light.dimmer1
    alias: Dimmer 40%
  - delay:
      hours: 0
      minutes: 1
      seconds: 0
      milliseconds: 0
  - service: light.turn_on
    data:
      brightness_pct: 50
    target:
      entity_id: light.dimmer1
    alias: Dimmer 50%
mode: restart
icon: hass:weather-sunset
```

## Step 6: Helpers


| NAME | USAGE |
|-----------------------------|------------------------|
| input_boolean.wakeup_light_on  | Triggers the automation    |
| input_boolean.gebruik_wake_up_light | Overrule not to use the light (when on holiday e.g.) |


## Step 7: automation to wake up 

(the complete code is in the seperate file)

First, the trigger and action when the wakeup-light has to start

```
trigger:
  - platform: state
    entity_id:
      - input_boolean.wakeup_light_on
    to: "on"
    id: wakeup-start
	
condition:
  - condition: state
    entity_id: input_boolean.gebruik_wake_up_light
    state: "on"

action:
  - if:
      - condition: trigger
        id: wakeup-start
    then:
      - service: script.wake_up_light
        data: {}
    else:
      - service: script.turn_off
        data:
          entity_id: script.wake_up_light
      - service: light.turn_off
        data: {}
        target:
          entity_id: light.dimmer1
	
```

Second, when there is an schedule available the boolean-trigger is been turned off.
Also the script has to abort.

```
trigger:
  - platform: state
    entity_id:
      - input_boolean.wakeup_light_on
    to: "off"
    id: wakeup-einde

action:
  - if:
      - condition: trigger
        id: wakeup-einde
    then:
      - service: script.turn_off
        data:
          entity_id: script.wake_up_light
      - service: light.turn_off
        data: {}
        target:
          entity_id: light.dimmer1
```

And third, the same applies for any turn-off command (with the fysical switch on the wall or in HomeAssistant)

```
trigger:
  - platform: state
    entity_id:
      - light.dimmer1
    to: "off"
    id: dimmer gaat uit

action:
 - if:
      - condition: trigger
        id: dimmer gaat uit
    then:
      - service: script.turn_off
        data:
          entity_id: script.wake_up_light
      - service: light.turn_off
        data: {}
        target:
          entity_id: light.dimmer1
      - service: input_boolean.turn_off
        data: {}
        target:
          entity_id: input_boolean.wakeup_light_on

```

## Step 8: Using schedule to plan any wake-up

Although HomeAssistant currently has an function to create schedules i prefer another HomeAssistant-Addon
See (this site)[https://github.com/arthurdent75/SimpleScheduler] for more information how to install and use.
It uses my helper 'input_boolean.wakeup_light_on'

## Step 9: make nice dashboard-controls for the family

(again, the complete code is in a seperate file)

<div align="center">
  <kbd>
    <img src="images/HomeAssistant Dashboard.jpg" />
  </kbd>
</div>

I use [Custom Button cards](https://github.com/custom-cards/button-card) to make a button any way i like

Left button: start and (more important) stop wake-up sequence
```
type: custom:button-card
name: Wakeup-light
entity: input_boolean.wakeup_light_on
icon: hass:weather-sunset
tap_action:
  action: toggle
styles:
  card:
    - height: 170px
state:
  - value: 'off'
    name: Start wake-up
  - value: 'on'
    name: STOP en UIT
```

Middle button: control the light  (normal card so it is easy to dimm any direction)
```
type: light
entity: light.dimmer1
name: Plafondlamp
icon: mdi:ceiling-light-outline
```

Right button: turn wakeup-schedule on or off
```
type: custom:button-card
name: Wakeup-light
entity: input_boolean.gebruik_wake_up_light
tap_action:
  action: toggle
styles:
  card:
    - height: 170px
state:
  - value: 'off'
    name: Geen wake-up
    icon: mdi:sleep
  - value: 'on'
    name: Wake-up AAN
```


### License

This project is licensed under the [MIT License](LICENSE.md).

## Acknowledgments

Inspiration, code snippets, etc.
 * [README copied from Michael Stickels README template](https://github.com/MichaelStickels/README_Template)
 * [Adapted from TINY README](https://gist.github.com/noperator/4eba8fae61a23dc6cb1fa8fbb9122d45)


------------------------

