---
title: "Home assistant - cycle scenes with a single button"
date: 2023-09-12
published: true
tags: home-assistant
category:
---

When I set up our lounge fan I had an excellent idea to add a physical switch that could control it. Initially this was setup to turn it on and off, but, given that this exists to parent and visitor proof the house I decided I would make it cycle through the fan speeds. It turns out that this seemingly simple request is a little more complex than I anticipated.

Outcome: a single button that will (in my case) cycle through fan speeds: fast, medium, slow, off.

To achieve this you need to add the following:

- two helpers
- some scenes
- the actual automations

Instructions are provided for both the GUI and direct construction in the `conf` files.

## Helpers

Go to `Settings > Devices & Services > Helpers > Create Helper`

You're going to need two here, one to cycle through the fan speeds and the other to help keep the first in sync.

### Fan speeds

For the fan speeds you need a `Dropdown`

![](/assets/2023-09-12-dropdown.png)<br>
Fan speed automation boolean

Add another automation, this time of type `toggle`:

![](/assets/2023-09-12-toggle.png)

## Scenes

Next set up some scenes that correspond to the various states via `Settings > Automation & Scenes > Scenes > Add Scene`

Name your scene and add the entities you care about and the states they want to be in. Under entities you also want your automation flag set.

Once you've set your scene up you can duplicate it for the other states. For this case I didn't need an "off" state but depending on what you want to achieve you might.

![](/assets/2023-09-12-scene.png)

Click on your devices and entities in order to edit their states:

{:.gallery.two-by-one}
![](/assets/2023-09-12-lounge-fan-states.png)
![](/assets/2023-09-12-fan-speed-button.png)

## Automations

This is the bit that makes it all work. You need four state automations (high, med, low and off) and one to allow the switch to control the states.

Go to `Settings > Automations & Scenes > Automations > Create Automation`

For the state automations you need the trigger to be the "Fan Speed" dropdown taking a specific value and the action to be the corresponding scene.

As for the scenes you can duplicate this for the other states.

![](/assets/2023-09-12-automation-update-fan-state.png)

The button automation needs to be triggered by the button press and have an action of select next on the helper. You also want this to cycle through so that it goes back to the start.

![](/assets/2023-09-12-automation-toggle-fan-states.png)

Finally you want some more complex logic to make sure that when you change the state via another route the speed helper is kept up to date. I'm going to pseudo-code this as the whole thing doesn't fit on one screen in the GUI.

The trigger for this one is the fan changing state or any attributes.

The first action is to work out the state of the fan if this hasn't been triggered by the automation:

```
if fan_speed_automation is "off":
    if lounge_fan is "off":
        lounge_fan_speed = "off"
    else:
        if lounge_fan.speed > 80:
            lounge_fan_speed = "high"
        else if lounge_fan.speed > 50:
            lounge_fan_speed = "med"
        else if lounge_fan.speed > 10:
            lounge_fan_speed = "low"
```

The second is to set the fan_speed_automation to off.

Done.

## Setup Via configuration

### Helpers

```yaml
input_select:
    lounge_fan_speed:
        name: Lounge Fan Speed
        icon: mdi:fan
        options:
            - high
            - med
            - low
            - off

input_boolean:
    fan_speed_automation:
        name: Fan Speed Automation
        icon: mdi: fan
```

### Scenes

There's no "off" here as I didn't need it but for more complex scenes you might.

```yaml
scene:
    - name: Lounge Fan Low
    - entities:
        fan.lounge_fan:
            percentage: 33
            state: 'on'
        input_boolean.fan_speed_automation:
            state: 'off'
    - name: Lounge Fan Med
    - entities:
        fan.lounge_fan:
            percentage: 66
            state: 'on'
        input_boolean.fan_speed_automation:
            state: 'off'
    - name: Lounge Fan High
    - entities:
        fan.lounge_fan:
            percentage: 100
            state: 'on'
        input_boolean.fan_speed_automation:
            state: 'off'
```

### Automations

Create an automation for each state:

```yaml
alias: Lounge Fan High
description: ""
trigger:
  - platform: state
    entity_id:
      - input_select.lounge_fan_speed
    to: high
condition: []
action:
  - service: scene.turn_on
    data: {}
    target:
      entity_id: scene.lounge_fan_high
mode: single

alias: Set lounge fan to medium
description: ""
trigger:
  - platform: state
    entity_id:
      - input_select.lounge_fan_speed
    to: med
    id: lounge_fan_speed_automation
condition: []
action:
  - service: scene.turn_on
    data: {}
    target:
      entity_id: scene.lounge_fan_med_2
mode: single

alias: Set lounge fan to low
description: ""
trigger:
  - platform: state
    entity_id:
      - input_select.lounge_fan_speed
    to: low
condition: []
action:
  - service: scene.turn_on
    data: {}
    target:
      entity_id: scene.lounge_fan_low
  - service: input_boolean.turn_on
    data: {}
    target:
      entity_id: input_boolean.fan_speed_automation
mode: single

alias: Set lounge fan to off
description: ""
trigger:
  - platform: state
    entity_id:
      - input_select.lounge_fan_speed
    to: "off"
condition: []
action:
  - type: turn_off
    device_id: 0cc997d5e37e5ab0155d4f742a131521
    entity_id: fan.lounge_fan
    domain: fan
mode: single
```

Button automation:

```yaml
alias: Toggle lounge fan states
description: ""
trigger:
  - platform: device
    domain: mqtt
    device_id: d1555d371666a72c69bc6126e7a9492a
    type: action
    subtype: single_right
    discovery_id: 0x00158d0008cf5d84 action_single_right
action:
  - service: input_select.select_next
    data:
      cycle: true
    target:
      entity_id: input_select.lounge_fan_speed
mode: single
```

State automation:

```yaml
alias: Ensure fan speed select matches fan speed
description: ""
trigger:
  - platform: state
    entity_id:
      - fan.lounge_fan
condition: []
action:
  - if:
      - condition: state
        entity_id: input_boolean.fan_speed_automation
        state: "off"
    then:
      - if:
          - condition: state
            entity_id: fan.lounge_fan
            state: "off"
        then:
          - service: input_select.select_option
            data:
              option: "off"
            target:
              entity_id: input_select.lounge_fan_speed
        else:
          - choose:
              - conditions:
                  - condition: numeric_state
                    entity_id: fan.lounge_fan
                    attribute: percentage
                    above: 80
                sequence:
                  - service: input_select.select_option
                    data:
                      option: high
                    target:
                      entity_id: input_select.lounge_fan_speed
              - conditions:
                  - condition: numeric_state
                    entity_id: fan.lounge_fan
                    attribute: percentage
                    above: 50
                sequence:
                  - service: input_select.select_option
                    data:
                      option: med
                    target:
                      entity_id: input_select.lounge_fan_speed
              - conditions:
                  - condition: numeric_state
                    entity_id: fan.lounge_fan
                    attribute: percentage
                    above: 10
                sequence:
                  - service: input_select.select_option
                    data:
                      option: low
                    target:
                      entity_id: input_select.lounge_fan_speed
  - service: input_boolean.turn_off
    data: {}
    target:
      entity_id: input_boolean.fan_speed_automation
mode: single
```

## Sources

- <https://community.home-assistant.io/t/cycle-through-scenes-with-a-button/41362/9>