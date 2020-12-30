
# Integrating Hive Active Heating SLR2/SLT2 - Domoticz, zigbee2MQTT and Node-RED - Working notes  28th Dec 2020

UPDATE - 30th Dec 2020 - Add some Domoticz dzVents code snippets

UPDATE - 28th Dec 2020 - flow.json - Improve readability/ Add comments. Version 0.3

UPDATE - 20th Dec 2020 - Water endpoint Issue fixed see https://github.com/Koenkk/zigbee2mqtt/issues/5357



## CH/HW Controller - node-RED screenshot

![CH-HW-Hive-Controller-node-RED-screenshot](https://user-images.githubusercontent.com/24318993/103233967-5a7af000-4936-11eb-81c0-5a5e522238ba.png)


## Background

My aim is to control my home Hive Active Central Heating/Hot Water system on my local network without requiring Internet access to the British Gas Hive 'cloud'. 

Until now, I have been reliant on controlling it using unofficial APIs via the British Gas Hive cloud infrastructure. Not ideal as BG change their APIs on occasions resulting in Hive CH/HW control downtime on my home automation platform, potentially resulting in a too hot or cold house!

Fortunately, support for the British Gas Hive SLR2 2-channel controller (Hot Water, Central heating) has been added recently to Koenkk's excellent zigbee2MQTT project (https://github.com/Koenkk/zigbee2mqtt). This should allow me achieve my aim of controlling my system 'locally'.

This is a repository of my node-RED flow and notes regarding my Hive Active controller testing and findings over the last couple of weeks. Some of it may be just plain wrong. However my test SLR2/SLT2, which has been shadowing my live Hive Active equipment connected to my Domoticz system, has been working fine for a few days using the mqtt publish messages found in the node-RED flow. (See message format below)

Feel free to re-use any of the information here if it helps, but be sure to run your own tests and ensure that any/all of this is 'fit for your purpose'.


## My Setup

My Home Automation (Domoticz) runs on a Raspberry Pi 4. In addition I run mosquitto message broker, node-RED and the aforementioned zigbee2mqtt. 

Zigbee2MQTT integration within Domoticz is taken care of by a Domoticz Python plugin - see https://github.com/stas-demydiuk/domoticz-zigbee2mqtt-plugin , however at this time the plugin doesn't currently support properly the Hive SLR2/SLT2 combination. 

As a result of this I am using mqqt publish/subscribe calls directly from Domoticz (dzVents) in order to control the SLR2/SLT2.

Status (ie the state of CH/HW relays, thermostat setpoint and temperature) **from** the SLR2/SLT2 is handled by a node-RED flow which publishes to **domoticz/in** topic thus updating devices in Domoticz. 


## Testing

Having procured a used Hive SLR2 controller and SLT2 thermostat (my test system), I placed zigbee2mqtt into pairing mode - then reset/added the Hive Controller/Thermostat. (Instructions for reset/pair may be found with a google search)

Zigbee2mqtt discovered the two devices and they were added to my Zigbee network. Both devices were next given 'friendly' zigbee names (Boiler Controller SLR2 and Boiler Thermostat SLT2)

I also enabled the newish zigbee2MQTT front end https://www.zigbee2mqtt.io/information/frontend.html to allow me to easily check out SLR2/SLT2 settings. Looking at the functions exposed for the controller/thermostat pair, the SLR2/SLT2 may be controlled by sending mqtt commands to the controller (SLR2) device only. Communication taking place over the zigbee network between the controller and thermostat, (eg. thermostat temperature), must be controlled by firmware local to the devices. 

After zigbee2MQTT discovered the SLR2/SLT2 pair, my home automation software, Domoticz utilising a zigbee2mqtt Python plugin, created three new Domoticz devices. However, the zigbee2mqtt plugin version I am running does not detect the SLR2/SLT2 properly so I am currently ignoring these devices. I guess support will be properly added in due course. Meanwhile I will control the SLR2 with my own mqtt commands (mosquitto_pub) within Domoticz using dzVents and check the status of the SLR2 with the help of a node-RED flow.


Initial testing with Zigbee2MQTT dev revision 1.16.2 initially threw up an issue with the 'water' endpoint (required for HW part of controller) being missing, requiring the addition of 'water', to the Const endpointNames section in utils.js

(UPDATE: This has since been fixed in a newer dev release of zigbee2MQTT)

Thus:

`const endpointNames = [ 'left', 'right', 'center', 'bottom_left', 'bottom_right', 'default', 'top_left', 'top_right', 'white', 'rgb', 'cct', 'system', 'top', 'bottom', 'center_left', 'center_right', 'ep1', 'ep2', 'row_1', 'row_2', 'row_3', 'row_4', 'relay', 'l1', 'l2', 'l3', 'l4', 'l5', 'l6', 'l7', 'l8', 'button_1', 'button_2', 'button_3', 'button_4', 'button_5', 'button_6', 'button_7', 'button_8', 'button_9', 'button_10', 'button_11', 'button_12', 'button_13', 'button_14', 'button_15', 'button_16', 'button_17', 'button_18', 'button_19', 'button_20', 'button_light', 'button_fan_high', 'button_fan_med', 'button_fan_low', 'heat', 'cool', ];`

becomes

`const endpointNames = [ 'left', 'right', 'center', 'bottom_left', 'bottom_right', 'default', 'top_left', 'top_right', 'white', 'rgb', 'cct', 'system', 'top', 'bottom', 'center_left', 'center_right', 'ep1', 'ep2', 'row_1', 'row_2', 'row_3', 'row_4', 'relay', 'l1', 'l2', 'l3', 'l4', 'l5', 'l6', 'l7', 'l8', 'button_1', 'button_2', 'button_3', 'button_4', 'button_5', 'button_6', 'button_7', 'button_8', 'button_9', 'button_10', 'button_11', 'button_12', 'button_13', 'button_14', 'button_15', 'button_16', 'button_17', 'button_18', 'button_19', 'button_20', 'button_light', 'button_fan_high', 'button_fan_med', 'button_fan_low', 'heat', 'cool', 'water', ];`

followed by Zigbee2MQTT restart.

## Controlling CH/HW Relays - Investigating MQTT messages

The next issue was working out which payloads written to which MQTT topic would allow me to switch the CH and HW relays from my dzVents heating controller script.

Each relay (CH and HW) has 3 modes: off, auto and heat.

- off:- self-explanatory. Relay is switched to off 
- auto:- relates to the relay being set to on/off according to a schedule set up on the thermostat (SLT2). I have no use for the 'auto' mode as my automation software (Domoticz) controls that. All I require is to be able to switch the two relays independently. Domoticz takes care of scheduling.
- heat:- turns 'on' the relevant relay dependant on the thermostat setpoint.  (ie demand for CH or HW, boiler comes on), although other settings appear to come into play

Initially I thought relay switching may be accomplished by just changing these heat/water modes, however, I discovered that a sequence of mqqt publishes including setting heat/water mode, thermostat setting (for CH, not valid for HW) and "temperature_setpoint_hold_heat/water" settings are actually required. (Unless someone else knows better).

In order to experiment easier and visualise the flow of commands to be issued to the MQTT broker, I created a 'quick and dirty' Node-RED flow. (flow.json in this repository) allowing me to easily try out combinations of MQTT messages to allow me to:-

1. Switch the HW and CH relays on/off from a virtual switch in Domoticz (or the node-RED flow ON/OFF buttons).

2. Set a CH thermostat setpoint controlled from a thermostat device in Domoticz (or node-RED flow) in order to toggle demand for heat on my system.

3. Read the status of the SLR2 CH/HW relays in node-RED and then send results via MQTT to virtual switches in Domoticz. (See 'Domoticz' nodes in attached flow).
 

## node-RED flow (flow.json)

This connects to my local mqtt broker - so if re-using this flow, be sure to change any mqtt publish/subscribe nodes to reflect your own mqtt broker IP and/or authorisation.

Also note that it incorporates flows to enable Domoticz thermostats/switches to be updated by node-RED. These nodes may be deleted if not using Domoticz. 


## Relay Control - Hot Water

Hot Water relay control is relatively simple. Just publish 3 mqqt messages in sequence for each state (Off/On). (The 'get' message is required):-

#### Switch off HW Relay mqtt message sequence (SLT2 displays 'Off'):-

1. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set` Message `{"system_mode_water": "off"}`

2. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/get` Message `{"system_mode_water": ""}`

3. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set` Message `{"temperature_setpoint_hold_water": "0"}`

#### Switch on HW Relay mqtt message sequence (SLT2 displays 'On'):-

1. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set` Message `{"system_mode_water": "heat"}`

2. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/get` Message `{"system_mode_water": ""}`

3. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set` Message `{"temperature_setpoint_hold_water": "1"}`

Note that the water thermostat **occupied_heating_setpoint_water** has no effect on this this function.


## Relay Control - Heating

Heating relay control is slightly different to the simple on/off Hot Water relay control. As well as publishing 3 mqqt messages for each CH relay state (Off/On). (The 'get' message is once again required). Relay control is also affected by **occupied_heating_setpoint_heat** (CH Thermostat SP). ie Setting this to a low value (less than thermostat temperature, will set the CH relay to 'off' OR setting this to a high value (greater than thermostat temperature, will set the CH relay to 'on'.

The message sequence to set up CH on/off mode is:-

#### Switch off CH Relay mqtt message sequence (SLT2 displays 'Off'):-

1. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set` Message `{"system_mode_water": "off"}`

2. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/get` Message `{"system_mode_water": ""}`

3. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set` Message `{"temperature_setpoint_hold_water": "0"}`

#### Switch on CH Relay mqtt message sequence (SLT2 displays 'Manual'):-

1. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set` Message `{"system_mode_heat": "heat"}`

2. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/get` Message `{"system_mode_heat": ""}`

3. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set` Message `{"temperature_setpoint_hold_heat": "1"}`

Now that CH Relay 'Manual' mode is selected, the relay may be switched by changing the thermostat SP value. To achieve this, send a thermostat set value message:-

Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set/occupied_heating_setpoint_heat` Message `VALUE` where VALUE is Thermostat setpoint in degrees C (eg. 20.5)

## SLR2 Status

Subscribing to MQTT topic **zigbee2mqtt/FRIENDLY_NAME** allows the SLR2 status to be read.

#### CH/HW Relay Status

The status of each relay, 'idle' == Off/'heat' == On, can be determined from **running_state_heat** and **running_state_water** status

#### CH Thermostat Setting

Can be read from **occupied_heating_setpoint_heat**

#### Thermostat (SLT2) Temperature

Can be read from **local_temperature_heat**

## Domoticz dzVents Code Snippets

The following code snippets can be used to format mqqt message to be published to MQTT Broker using **mosquitto_pub**

        local CHSwitch = dz.devices('CHSwitch')                         --Switch Hive CH on/off
        local HWSwitch = dz.devices('HWSwitch')                         --HW Switch
        local TestSwitch = dz.devices('TestSwitch')                     --Test switch to set Thermostat setpoint to a high value or low value       
    
        -- function to make an OS command to be passed to mosquitto_pub. Topic and Message passed to function
        function MakeMQTT(Topic, Message)
            MQTTStr = "mosquitto_pub -r -h 192.168.2.10 -t 'TOPIC' -m 'MESSAGE'"                            --make cli command template. (See mosquitto_pub for syntax)
            MQTTStr = string.gsub(MQTTStr, "TOPIC", Topic)                                                  --substitute TOPIC with Topic passed to function
            MQTTStr = string.gsub(MQTTStr, "MESSAGE", tostring(Message))                                    --same for Message, but ensure message is a string
            os.execute(MQTTStr)                                                                             --execute it
        end
    
        function CHSetTherm(Temp)                                                                           --set CH thermostat setpoint to value passed in Temp
            -- Topic zigbee2mqtt/FRIENDLY_NAME/heat/set/occupied_heating_setpoint      Message 23.5         --example thermostat setpoint topic/message
            MakeMQTT('zigbee2mqtt/Boiler Controller SLR2/heat/set/occupied_heating_setpoint',Temp)          --call function MakeMQTT passing topic/message (temperature SP)
        end
    
        -- device triggers...        
        if (item.isDevice) then 
            if (item.name == 'HWSwitch') then                                                                   --HW Switch - switches HW relay on/off
                if (HWSwitch.state == 'On') then                                                                --publish 3 messages to SLR2 to turn on HW relay
                    -- Topic zigbee2mqtt/FRIENDLY_NAME/heat/set      Message {"system_mode_water": "heat"}
                    -- Topic zigbee2mqtt/FRIENDLY_NAME/heat/get      Message {"system_mode_water": ""}
                    -- Topic zigbee2mqtt/FRIENDLY_NAME/heat/set      Message {"temperature_setpoint_hold_water": "1"}		            
                    MakeMQTT('zigbee2mqtt/Boiler Controller SLR2/heat/set','{\"system_mode_water\":\"heat\"}')
                    MakeMQTT('zigbee2mqtt/Boiler Controller SLR2/heat/get','{\"system_mode_water\":\"\"}')
                    MakeMQTT('zigbee2mqtt/Boiler Controller SLR2/heat/set','{\"temperature_setpoint_hold_water\":\"1\"}')
                else
                    -- Topic zigbee2mqtt/FRIENDLY_NAME/heat/set      Message {"system_mode_water": "off"}         --HW relay to 'off' send these three topic
                    -- Topic zigbee2mqtt/FRIENDLY_NAME/heat/get      Message {"system_mode_water": ""}
                    -- Topic zigbee2mqtt/FRIENDLY_NAME/heat/set      Message {"temperature_setpoint_hold_water": "0"}
                    MakeMQTT('zigbee2mqtt/Boiler Controller SLR2/heat/set','{\"system_mode_water\":\"off\"}')     --publish 3 messages to SLR2 to turn off HW relay
                    MakeMQTT('zigbee2mqtt/Boiler Controller SLR2/heat/get','{\"system_mode_water\":\"\"}')
                    MakeMQTT('zigbee2mqtt/Boiler Controller SLR2/heat/set','{\"temperature_setpoint_hold_water\":\"0\"}')
                end
            end
    
            if (item.name == 'CHSwitch') then                                                                       --CH Switch. On sets SLR2 to 'manual', Off to 'off'
                if (CHSwitch.state == 'On') then
                    MakeMQTT('zigbee2mqtt/Boiler Controller SLR2/heat/set','{\"system_mode_heat\":\"heat\"}')       --publish 3 messages to SLR2 to turn on CH relay
                    MakeMQTT('zigbee2mqtt/Boiler Controller SLR2/heat/get','{\"system_mode_heat\":\"\"}')
                    MakeMQTT('zigbee2mqtt/Boiler Controller SLR2/heat/set','{\"temperature_setpoint_hold_heat\":\"1\"}')
                else
                    MakeMQTT('zigbee2mqtt/Boiler Controller SLR2/heat/set','{\"system_mode_heat\":\"off\"}')        --publish 3 messages to SLR2 to turn off CH relay
                    MakeMQTT('zigbee2mqtt/Boiler Controller SLR2/heat/get','{\"system_mode_heat\":\"\"}')
                    MakeMQTT('zigbee2mqtt/Boiler Controller SLR2/heat/set','{\"temperature_setpoint_hold_heat\":\"0\"}')
                end
            end
    
            if (item.name == 'TestSwitch') then                                                                     --Test switch
                if (TestSwitch.state == 'On') then                                                                  --On... set CH thermostat to high setting.. say 24 deg C
                    CHSetTherm(24)
                else
                    CHSetTherm(14.5)                                                                                --Off... set CH thermostat to low setting.. say 14.5 deg C
                end
            end
        end


## NOTES on SLR2/SLT2 functionality

There is additional functionality built in to the Hive Active SLR2/SLT2 pair which at present cannot be overridden by external control.

The SLR2 controls the switch timing of the CH/HW relays in the case of sending rapid on/off switch commands to the controller. ie. In order to protect a connected boiler from rapid switching of these signals resulting in possible damage, a delay is built in to SLR2 firmware preventing such rapid toggling. This mean that the controller goes into a mode where the relay LED status indicators flash to indicate that there will be a delay before the physical activation/deactivation of the appropriate relay. The relay switch delay is around 30 seconds

The SLR2/SLT2 combination supports CH/HW scheduling that may be programmed into the thermostat by a sequence of button presses. This is not important to me as this is achieved using scripts in my home automation software. I also believe that zigbee2MQTT will require the addition of new 'endpopints' to allow the programming of this schedule from mqtt should this functionality be required at some point in the future.

When setting the heat control to 'off', the CH thermostat setpoint automatically switches to 1 deg C for the SLR2/SLT2 combination ('frost stat' setting).


## To Do List

Next steps are:-

1. Disconnect live Controller from my home system and replace with the 'test' controller (SLR2) along with the thermostat (SLT2)

2. Once disconnected. My current Controller and newer style thermostat will be reset then paired with my zigbee network to become a new test system. I am interested to find if there are any differences (functional and/or firmware) between older style Hive thermostat (SLT2) and newer style thermostat.

Watch this space :)



