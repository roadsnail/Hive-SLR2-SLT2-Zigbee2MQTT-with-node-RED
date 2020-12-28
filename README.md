
# Integrating Hive Active Heating SLR2/SLT2 - Domoticz, zigbee2MQTT and Node-RED - Working notes  28th Dec 2020


UPDATE - 28th Dec 2020 - flow.json - Improve readability/ Add comments.

UPDATE - 20th Dec 2020 - Water endpoint Issue fixed see https://github.com/Koenkk/zigbee2mqtt/issues/5357



## CH/HW Controller - node-RED screenshot

![CH-HW-Hive-Controller-node-RED-screenshot](https://user-images.githubusercontent.com/24318993/103233967-5a7af000-4936-11eb-81c0-5a5e522238ba.png)


Background
----
My aim is to control my home Hive Active Central Heating/Hot Water system on my local network without requiring Internet access to the British Gas Hive 'cloud'. 

Until now, I have been reliant on controlling it using undocumented APIs via the British Gas Hive cloud infrastructure. Not ideal as BG change their APIs on occasions resulting in Hive downtime within my home automation platform.

Fortunately, support for the British Gas Hive SLR2 2-channel controller (Hot Water, Central heating) has been added recently to Koenkk's excellent zigbee2MQTT project (https://github.com/Koenkk/zigbee2mqtt). This should allow me achieve my aim of controlling my system 'locally'.

This is a repository of my node-RED flow and notes regarding my Hive Active controller testing and findings over the last couple of weeks. Some of it may be just plain wrong. However my Domoticz system has been working fine for a few days using the mqtt publish messages found in the node-RED flow.

Feel free to re-use any of the information here if it helps, but be sure to run your own tests and ensure that any of this is 'fit for your purpose'.


My Setup
----
My Home Automation (Domoticz) runs on a Raspberry Pi 4. In addition I use mosquitto message broker, node-RED and the aforementioned zigbee2mqtt. 

Zigbee2MQTT integration within Domoticz is taken care of by a Domoticz plugin - see https://github.com/stas-demydiuk/domoticz-zigbee2mqtt-plugin , however this plugin doesn't currently support properly the Hive SLR2/SLT2 combination. Therefore I am currently using mqqt publish/subscribe calls directly from Domoticz (dzVents) in order to control the SLR2/SLT2.

Status (ie the state of CH/HW relays, thermostat setpoint and temperature) are handled by a node-RED flow which publishes to domoticz/in topic which updates devices in Domoticz. 


Testing
----
Having procured a used Hive SLR2 controller and SLT2 thermostat (my test system), I placed zigbee2mqtt into pairing mode - then reset/added the Hive Controller/Thermostat. (Instructions for reset/pair may be found with a google search)

Zigbee2mqtt discovered the two devices and they were added to my Zigbee network. Both devices were next given 'friendly' zigbee names (Boiler Controller SLR2 and Boiler Thermostat SLT2)

I also enabled the newish zigbee2MQTT front end https://www.zigbee2mqtt.io/information/frontend.html to allow me to easily check out SLR2/SLT2 settings. You will see from looking at the functions exposed for the controller/thermostat pair that the SLR2/SLT2 may be controlled by sending mqtt commands to the controller (SLR2) device. 2-way communication between the controller and thermostat must be by zigbee under local firmware control?

After discovering the SLR2/SLT2, my home automation software, Domoticz utilising the zigbee2mqtt plugin, detected three new Domoticz devices. However, the zigbee2mqtt version I am running does not detect the SLR2/SLT2 properly. I guess support will be properly added in due course. Meanwhile I will control the SLR2 with my own mqtt commands within Domoticz using dzVents and a node-RED flow.


Initial testing with Zigbee2MQTT dev revision 1.16.2 initially threw up an issue with the 'water' endpoint (required for HW part of controller) being missing, requiring the addition of 'water', to the Const endpointNames section in utils.js

(UPDATE: This has since been fixed in a newer dev release of zigbee2MQTT)

Thus:

`const endpointNames = [ 'left', 'right', 'center', 'bottom_left', 'bottom_right', 'default', 'top_left', 'top_right', 'white', 'rgb', 'cct', 'system', 'top', 'bottom', 'center_left', 'center_right', 'ep1', 'ep2', 'row_1', 'row_2', 'row_3', 'row_4', 'relay', 'l1', 'l2', 'l3', 'l4', 'l5', 'l6', 'l7', 'l8', 'button_1', 'button_2', 'button_3', 'button_4', 'button_5', 'button_6', 'button_7', 'button_8', 'button_9', 'button_10', 'button_11', 'button_12', 'button_13', 'button_14', 'button_15', 'button_16', 'button_17', 'button_18', 'button_19', 'button_20', 'button_light', 'button_fan_high', 'button_fan_med', 'button_fan_low', 'heat', 'cool', ];`

becomes

`const endpointNames = [ 'left', 'right', 'center', 'bottom_left', 'bottom_right', 'default', 'top_left', 'top_right', 'white', 'rgb', 'cct', 'system', 'top', 'bottom', 'center_left', 'center_right', 'ep1', 'ep2', 'row_1', 'row_2', 'row_3', 'row_4', 'relay', 'l1', 'l2', 'l3', 'l4', 'l5', 'l6', 'l7', 'l8', 'button_1', 'button_2', 'button_3', 'button_4', 'button_5', 'button_6', 'button_7', 'button_8', 'button_9', 'button_10', 'button_11', 'button_12', 'button_13', 'button_14', 'button_15', 'button_16', 'button_17', 'button_18', 'button_19', 'button_20', 'button_light', 'button_fan_high', 'button_fan_med', 'button_fan_low', 'heat', 'cool', 'water', ];`

followed by Zigbee2MQTT restart.

Controlling CH/HW Relays - Investigating MQTT messages
----
The next issue was working out which payloads written to which MQTT topic allowed me to switch the CH and HW relays.

Each relay (CH and HW) has 3 modes: off, auto and heat.

- off:- self-explanatory. Relay is switched to off 
- auto:- relates to the relay being set to on/off according to a schedule set up on the thermostat (SLT2). I have no use for the 'auto' mode as my automation software (Domoticz) controls that. All I require is to be able to switch the two relays independently.
- heat:- turns 'on' the relevant relay dependant on the thermostat setpoint.  (ie demand for CH or HW, boiler comes on), although other settings appear to come into play

Initially I thought this may be accomplished by just changing these heat/water modes, however, I discovered that a sequence of mqqt publishes including setting heat/water mode, thermostat setting (for CH, not valid for HW) and "temperature_setpoint_hold_heat/water" settings are actually required. (Unless someone else knows better).

In order to experiment easier and visualise the flow of commands to be issued to MQTT, I created a 'quick and dirty' Node-RED flow. (see flow below). 

node-RED flow (flow.json)
----
This connects to a local mqtt broker - so if re-using this flow, be sure to change any mqtt publish/subscribe nodes to reflect your own mqtt broker IP and/or authorisation.

Also note that it incorporates flows to enable Domoticz thermostats/switches to be updated by node-RED. These nodes may be deleted if not using Domoticz. 


## Relay Control

#### Switch off HW Relay mqtt message sequence (SLT2 displays 'Off'):-

Topic `zigbee2mqtt/Boiler Controller SLR2/heat/set` Message `{"system_mode_water": "off"}`

Topic `zigbee2mqtt/Boiler Controller SLR2/heat/get` Message `{"system_mode_water": ""}`

Topic `zigbee2mqtt/Boiler Controller SLR2/heat/set` Message `{"temperature_setpoint_hold_water": "0"}`

#### Switch on HW Relay mqtt message sequence (SLT2 displays 'On'):-

Topic `zigbee2mqtt/Boiler Controller SLR2/heat/set` Message `{"system_mode_water": "heat"}`

Topic `zigbee2mqtt/Boiler Controller SLR2/heat/get` Message `{"system_mode_water": ""}`

Topic `zigbee2mqtt/Boiler Controller SLR2/heat/set` Message `{"temperature_setpoint_hold_water": "1"}




## NOTES on SLR2/SLT2 functionality

There is additional functionality built in to the Hive Active SLR2/SLT2 pair which at present cannot be overridden by external control.

The SLR2 controls the switch timing of the CH/HW relays in the case of rapid CH/HW command switching. ie. In order to protect a connected boiler from rapid switching of these signals resulting in possible damage, a delay is built in to the SLR2. This mean that the controller goes into a mode where the relay LED status indicators flash to indicate that there will be a delay before the physical activation/deactivation of the appropriate relay. The relay switch delay is around 30 seconds

The SLR2/SLT2 combination supports CH/HW scheduling that may be programmed into the thermostat by a sequence of button presses. This functionality is not important to me as this is done from my home automation software (Domoticz). I also believe that zigbee2MQTT would require the addition of new 'endpopints' to allow the programming of this schedule from mqtt.

When setting the heat control to 'off', the CH thermostat setpoint automatically switches to 1 deg C for the SLR2/SLT2 combination. I believe this is a 'frost stat' function automatically supported by the SLR2/SLT2 combination.
