---
# Updates

UPDATE - 12th May 2021 - @russdan adds Hotwater boost / emergency_heating functionality. See https://github.com/roadsnail/Hive-SLR2-SLT2-Zigbee2MQTT-with-node-RED/issues/2 for more details if you wish to also add HW and probably a CH 'Boost' function. 

30th Apr 2021 - **Announcing Project Pi-ve.** 

Following on from this project, I now have a new Beta test Hive/Pi automation project - Project Pi-ve at https://github.com/roadsnail/Pi-ve

What is Pi-ve? - It is a standalone Hive Based Central Heating/Hot Water Controller controlled by a Raspberry Pi Zero W. 

Control your Central Heating/Hot Water boiler on your local network using Pi-ve from:-

* Node-RED Dashboard (in a Web browser)
* HTTP Requests
* Messages published to an onboard MQTT message broker

The Node-RED dashboard Web page offers full control of CH and HW with up to 8 programmable 'On' periods per day plus Override/Boost functions.

In addition, the Pi-ve may be controlled using MQTT commands OR HTTP Requests from an external system (eg. A Home Automation platform or phone/tablet App).


UPDATE - 1st Feb 2021 - Correct naming inconsistencies/errors

UPDATE - 30th Jan 2021 - Correct CH thermostat topic in write-up. dzVents is correct

UPDATE - 18th Jan 2021 - Update flow.json to v0.5 - Add support for SLR/SLT offline/online

UPDATE - 5th Jan 2021  -  Added NOTES on CC2531 and Zigbee2MQTT stability section

UPDATE - 1st Jan 2021  - flow.json - Modify MQQT input node and feed into json parser. Version 0.41

UPDATE - 31st Dec 2020 - Now testing Hive thermostat type SLT3 (shiny version with rotary encoder)

UPDATE - 31st Dec 2020 - Node-RED flow issue fixed. See https://github.com/roadsnail/Hive-SLR2-SLT2-Zigbee2MQTT-with-node-RED/issues/1 . Fixed> v0.4

UPDATE - 31st Dec 2020 - Add SLR2/SLT2 pairing instructions

UPDATE - 30th Dec 2020 - Add some Domoticz dzVents code snippets

UPDATE - 28th Dec 2020 - flow.json - Improve readability/ Add comments. Version 0.3

UPDATE - 20th Dec 2020 - Water endpoint Issue fixed see https://github.com/Koenkk/zigbee2mqtt/issues/5357


---




# Integrating Hive Active Heating SLR2/SLT2 & SLT3 - Domoticz, Zigbee2MQTT and Node-RED - A work in progress

## Break free from relying on Centrica's Hive Cloud for your Hive Active Heating/Hot water Controller/Thermostat and control it locally using Domoticz, Zigbee2MQTT, node-RED and MQTT. 


Hive Active CH/HW Controller - node-RED dashboard screenshot. Node-RED is used to visualise and test MQTT publish/subscribe topics during testing/development and is used to publish MQTT messages to Domoticz.

![2021-01-28 13_48_01-Node-RED Dashboard](https://user-images.githubusercontent.com/24318993/106148552-15362200-6171-11eb-81b3-7acd9f413b3f.png)


## Background

My aim is to control my home Hive Active Central Heating/Hot Water system on my local network without requiring Internet access to the Centrica Hive 'cloud'. 

Until now, I have been reliant on controlling it using unofficial APIs via the Centrica Hive cloud infrastructure using a Domoticz plugin. Not ideal as they change their APIs on occasions resulting in Hive CH/HW control downtime on my home automation platform, potentially resulting in a too hot or cold house until the changes have been reverse-engineered and changes made to accomodate Centrica's API changes.

Fortunately, support for the Hive SLR2 2-channel controller (Hot Water, Central heating) has been added recently to Koenkk's excellent Zigbee2MQTT project (https://github.com/Koenkk/zigbee2mqtt). This should allow me achieve my aim of controlling my system 'locally'.

This is a repository of my node-RED flow, Domoticz dzVents code snippets (using mosquitto pub/sub) and notes regarding my Hive Active controller testing and findings over the last couple of months. 

Some of it may be just plain wrong. However my test SLR2/SLT2, which has been shadowing my live Hive Active equipment connected to my Domoticz system, has been working fine for a few days using the MQTT publish messages found in the node-RED flow. (See message format below). 

UPDATE 1st Feb 2021 - My Hive hardware has been working consistently on my Zigbee network without issues for more than a month now.

Feel free to re-use any of the information here if it helps, but be sure to run your own tests and ensure that any/all of this is 'fit for your purpose'.

UPDATE 11th Apr 2021 - What a difference a dongle makes! I finally managed to procure a zig-a-zig-ah Zigbee dongle from Electrolama based on the TI CC2652R controller and what a difference it has made to my Zigbee network. Gone are the intermittent issues with the former CC2531 controller much improving reliability of my whole Zigbee network including, of course, the SLR/SLT devices.


## Setup

My Home Automation (Domoticz) software runs on a Raspberry Pi 4. In addition I run mosquitto message broker, mosquitto-clients (pub and sub), node-RED and Zigbee2MQTT. 

Zigbee2MQTT integration within Domoticz is taken care of by a Domoticz Python plugin - see https://github.com/stas-demydiuk/domoticz-zigbee2mqtt-plugin , however at this time the plugin doesn't appear to support properly the Hive SLR2/SLT2 or SLT3 combination.

As a result of this I am using MQQT publish/subscribe calls directly from Domoticz (dzVents) in order to control the SLR2/SLT2 (and SLR2/SLT3). (See https://github.com/roadsnail/Hive-SLR2-SLT2-Zigbee2MQTT-with-node-RED#domoticz-dzvents-code-snippets )

Status (ie the state of CH/HW relays, thermostat setpoint, Controller On/Offline and temperature) **from** the SLR2/SLT2 is handled by a node-RED flow https://github.com/roadsnail/Hive-SLR2-SLT2-Zigbee2MQTT-with-node-RED/blob/main/flow.json which publishes to **domoticz/in** topic thus updating devices in Domoticz. 

## Devices

![SLT3](https://user-images.githubusercontent.com/24318993/103422589-26076e00-4b9a-11eb-87cf-bd28548f8012.jpg) **SLT3 Thermostat** 

![SLR2](https://user-images.githubusercontent.com/24318993/103422602-2f90d600-4b9a-11eb-8436-d0608720e210.jpg) **SLR2 CH/HW Controller**

![SLT2](https://user-images.githubusercontent.com/24318993/103422610-3a4b6b00-4b9a-11eb-8f6d-a858fa836211.jpg) **SLT2 Thermostat**


## Testing (SLR2 and SLT2 combination)

Having procured a used Hive SLR2 controller and SLT2 thermostat (my test system), I placed Zigbee2MQTT into pairing mode - then reset/added the Hive Controller/Thermostat to my Zigbee network. (Instructions for reset/pair below https://github.com/roadsnail/Hive-SLR2-SLT2-Zigbee2MQTT-with-node-RED#pairing-instructions)

Zigbee2MQTT discovered the two devices and they were added to my Zigbee network. Both devices were next given 'friendly' Zigbee names (Boiler Controller SLR2 and Boiler Thermostat SLT2)

I also enabled the Zigbee2MQTT front end https://www.zigbee2mqtt.io/information/frontend.html to allow me to easily check out SLR2/SLT2 settings. Looking at the functions exposed for the controller/thermostat pair, the SLR2/SLT2 may be controlled by sending MQTT commands to the controller (SLR2) device only. Communication taking place over the Zigbee network between the controller and thermostat, (eg. thermostat temperature), must be controlled by proprietary firmware local to the devices. 

After Zigbee2MQTT discovered the SLR2/SLT2 pair, my home automation software, Domoticz utilising the Zigbee2MQTT Python plugin, created three new Domoticz devices. However, the Zigbee2MQTT (Domoticz) plugin version I am running does not detect the SLR2/SLT2 properly so I am currently ignoring these autodiscovered Domoticz devices. 

I guess support will be properly added in due course. Meanwhile I will control the SLR2 with my own MQTT commands (via mosquitto_pub) within Domoticz using dzVents with manually created virtual switches and thermostat devices. I will check the status of the SLR2 with the help of a node-RED flow which creates an MQTT message published to Domoticz MQTT domoticz/in. 


Initial testing with Zigbee2MQTT dev revision 1.16.2 initially threw up an issue with the 'water' endpoint (required for Hot Water part of controller) being missing, requiring the addition of 'water', to the Const endpointNames section in utils.js

(UPDATE: This has since been fixed in a newer dev release of Zigbee2MQTT, therefore updating to at least v 1.17.x is recommended)


## Pairing Instructions:

1. Switch off Hive bridge (usually connected to home router)
2. Remove a battery from the thermostat (SLT2 or SLT3)
3. Enable Zigbee2MQTT to allow it to accept new devices. (Logs will show pairing activity as it happens later, hopefully)
4. On the heating controller SLR2, press and hold 'Central Heating' button until it flashes pink. Release then press and hold it again. It will flash amber and the controller should join the network. With Zigbee2MQTT still in pairing mode:-
5. Replace batteries in the thermostat (SLT2 or SLT3) **while** pressing 'back' and 'menu' buttons on the SLT3 OR the '+' and '-' buttons on the SLT" to perform a reset. It will, reset, reboot and join the network (check logs).

## Testing (SLR2 and SLT3 combination)

Following on from testing the older Hive Active SLT2 Thermostat working with the SLR2 Controller. I have done some preliminary testing of the newer Hive Active Thermostat, the SLT3 (shiny with rotary encoder) and SLR2 combination.

Pairing of these is carried out in a similar manner to the SLR2/SLT2 versions.

However, there are some differences in setup between the older SLT2 and newer SLT3 thermostats. 

I observed that following a factory reset of the SLT3 - It joined my Zigbee network and then went into setup mode - requesting a HW/CH schedule to be setup.

This may be skipped and the thermostat subsequently displays the current temperature and status of CH/HW to both be 'Sch'

At this point the controller does not publish a full list of MQTT messages on the its root topic zigbee2mqtt/FRIENDLY_NAME

By connecting a 'debug' node to the output of the node-RED MQTT input node you can see that fewer messages are being sent from the Hive test system.

To enable the missing MQTT messages relating to CH and HW relay control - 

On the SLT3 thermostat:-
Press **MENU** button
Select **'Heat'** and set to **'Manual'** - Then press confirm button

Similarly (for Hot Water):-

Press **MENU** button
Select **'Hot Water'** and set to **'Always On'** - Then press confirm button

On the SLT2 thermostat:-
Press top right **'menu'** button marked **+** to enable other switches
Press **Heat** function button (marked right arrow) until the status changes to **Manual**
Press **Hot Water** funtion button (marked left arrow) until the status changes to **Off**


## Controlling CH/HW Relays - Investigating MQTT messages

The next issue was working out which payloads written to which MQTT topic would allow me to switch the CH and HW relays from my dzVents heating controller script.

Each relay (CH and HW) has 3 modes: off, auto and heat.

- off:- self-explanatory. Relay is switched to off 
- auto:- relates to the relay being set to on/off according to a schedule set up on the thermostat (SLT2). I have no use for the 'auto' mode as my automation software (Domoticz) controls that. All I require is to be able to switch the two relays independently. Domoticz takes care of scheduling.
- heat:- turns 'on' the relevant relay dependant on the thermostat setpoint.  (ie demand for CH or HW, boiler comes on), although other settings appear to come into play

Initially I thought relay switching may be accomplished by just changing these heat/water modes, however, I discovered that a sequence of MQTT publishes including setting heat/water mode, thermostat setting (for CH, not valid for HW) and "temperature_setpoint_hold_heat/water" settings are actually required. (Unless someone else knows better).

In order to experiment easier and visualise the flow of commands to be issued to the MQTT broker, I created a 'quick and dirty' Node-RED flow. (flow.json in this repository) allowing me to easily try out combinations of MQTT messages to allow me to:-

1. Switch the HW and CH relays on/off from a virtual switch in Domoticz (or the node-RED flow ON/OFF buttons).

2. Set a CH thermostat setpoint controlled from a thermostat device in Domoticz (or node-RED flow) in order to toggle demand for heat on my system.

3. Read the status of the SLR2 CH/HW relays in node-RED and then send results via MQTT to virtual switches in Domoticz. (See 'Domoticz' nodes in attached flow).

4. Read Zigbee offline/online status of controllers. (See offline/online part of flow.json - omit Domoticz nodes if not required)
 

## node-RED flow (flow.json)

This connects to my local MQTT broker - so if re-using this flow, be sure to change any MQTT publish/subscribe nodes to reflect your own MQTT broker IP and authorisation (if used).

Also note that it incorporates flows to enable Domoticz thermostats/switches to be updated by node-RED. These nodes may be deleted if not using Domoticz. 

#### Issue: 

node-RED dashboard does not display value. See fix https://github.com/roadsnail/Hive-SLR2-SLT2-Zigbee2MQTT-with-node-RED/issues/1


## Relay Control - Hot Water

Hot Water relay control is relatively simple. Just publish 3 MQTT messages in sequence for each state (Off/On). Although the controller publishes a 'water' temperature and thermostat value, these are not used. The HW Relay can be simply switched using the below sequence. (The 'get' message (second one below) is required):-

#### Switch off HW Relay MQTT message sequence (SLT2 displays 'Off'):-

1. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set` Message `{"system_mode_water": "off"}`

2. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/get` Message `{"system_mode_water": ""}`

3. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set` Message `{"temperature_setpoint_hold_water": "0"}`

#### Switch on HW Relay MQTT message sequence (SLT2 displays 'On'):-

1. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set` Message `{"system_mode_water": "heat"}`

2. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/get` Message `{"system_mode_water": ""}`

3. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set` Message `{"temperature_setpoint_hold_water": "1"}`

Note that the water thermostat **occupied_heating_setpoint_water** has no effect on this this function.


## Relay Control - Heating

Heating relay control is slightly different to the simple on/off Hot Water relay control. As well as publishing 3 MQTT messages for each CH relay state (Off/On). (The 'get' message is once again required). Relay control is also affected by **occupied_heating_setpoint** (CH Thermostat SP). ie Setting this to a low value (less than thermostat temperature, will set the CH relay to 'off' OR setting this to a high value (greater than thermostat temperature, will set the CH relay to 'on'. 

The message sequence to set up CH on/off mode is:-

#### Switch off CH Relay MQTT message sequence (SLT2 displays 'Off' and thermostat setting changes to 1deg C). Publish:-

1. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set` Message `{"system_mode_heat": "off"}`

2. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/get` Message `{"system_mode_heat": ""}`

3. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set` Message `{"temperature_setpoint_hold_heat": "0"}`

#### Switch on CH Relay 'manual' mode MQTT message sequence (SLT2 displays 'Manual'). Publish:-

1. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set` Message `{"system_mode_heat": "heat"}`

2. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/get` Message `{"system_mode_heat": ""}`

3. Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set` Message `{"temperature_setpoint_hold_heat": "1"}`

Now that CH Relay 'Manual' mode is selected, the relay may be switched by changing the thermostat setpoint value. To achieve this, send a thermostat set value message:-

Topic `zigbee2mqtt/FRIENDLY_NAME/heat/set/occupied_heating_setpoint` Message `VALUE` where VALUE is Thermostat setpoint in degrees C (eg. 20.5)

## SLR2 Status

Subscribing to MQTT topic **zigbee2mqtt/FRIENDLY_NAME** allows the SLR2 status to be read.

#### CH/HW Relay Status

The status of each relay, 'idle' == Off/'heat' == On, can be determined from **running_state_heat** and **running_state_water** status

#### CH Thermostat Setting

Can be read from **occupied_heating_setpoint_heat**

#### Thermostat (SLT2) Temperature

Can be read from **local_temperature_heat**

## SLR2 Off/Online Status

Subscribing to topic **zigbee2mqtt/FRIENDLY_NAME/availability** allows the SLR2 offline/online status to be read (see flows.json). For this to work ensure that the 'Availability' feature in Zigbee2MQTT is enabled (see https://github.com/Koenkk/zigbee2mqtt/issues/775 ). 


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
                    MakeMQTT('zigbee2mqtt/Boiler Controller SLR2/heat/set','{\"system_mode_heat\":\"heat\"}')       --publish 3 messages to SLR2 to turn on CH 'heat' function
                    MakeMQTT('zigbee2mqtt/Boiler Controller SLR2/heat/get','{\"system_mode_heat\":\"\"}')
                    MakeMQTT('zigbee2mqtt/Boiler Controller SLR2/heat/set','{\"temperature_setpoint_hold_heat\":\"1\"}')
                else
                    MakeMQTT('zigbee2mqtt/Boiler Controller SLR2/heat/set','{\"system_mode_heat\":\"off\"}')        --publish 3 messages to SLR2 to turn off CH to 'off'
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

The SLR2 controls the switch timing of the CH/HW relays in the case of sending rapid on/off switch commands to the controller. ie. In order to protect a connected boiler from rapid switching of these signals resulting in possible damage, a delay is built in to SLR2 firmware preventing such rapid toggling. This means that the controller goes into a mode where the relay LED status indicators flash to indicate that there will be a delay before the physical activation/deactivation of the appropriate relay. The relay switch delay is around 30 seconds

The SLR2/SLT2 combination supports CH/HW scheduling that may be programmed into the thermostat by a sequence of button presses. This is not important to me as this is achieved using scripts in my home automation software. I also believe that zigbee2MQTT will require the addition of new 'endpopints' to allow the programming of this schedule from MQTT should this functionality be required at some point in the future.

When setting the heat control to 'off', the CH thermostat setpoint automatically switches to 1 deg C for the SLR2/SLT2 combination ('frost stat' setting).

## NOTES on CC2531 and zigbee2MQTT stability

Since setting out with my Zigbee network implimentation, I have kept to the CC2531 Zigbee USB dongle (without antenna) running Koenkk's coordinator software version 20190608 from https://github.com/Koenkk/Z-Stack-firmware/raw/master/coordinator/Z-Stack_Home_1.2/bin/default/CC2531_DEFAULT_20190608.zip

Initial results were promising, however Zigbee range was poor. After some experimentation, including trying a CC2531 with antenna, I returned to the original CC2531 with its onboard antenna connecting it to the USB port of my Raspberry Pi with a 1 metre USB extension lead.

This improved range as I presume the CC2531 receiver was not being overloaded with spurious RF generated by the RPi but I was still seeing CC2531 'lockup' issues every few days, or even few hours, where I would lose connection to the Zigbee network, requiring unplugging/replugging the CC2531 in order to re-establish operation. This problem would make my Hive local control very unreliable, therefore a fix was required.

![cc2531](https://user-images.githubusercontent.com/24318993/103656165-2c159a00-4f60-11eb-9ee3-92964fdb687a.png)

I think I have traced the 'lockup' issue to a 'noisy' 5V supply voltage at the CC2531 probably due to a poor quality USB extension cable. I am guessing that the power wires are quite thin causing issues on the CC2531 when the coordinator is transmitting. In order to test this theory, I have soldered a 1000uF/10V electrolytic capacitor directly across the power rails at the USB connector. (See picture). Why 1000uF? Well that was what I had to hand and it physically fits into the USB dongle enclosure.

Since adding the capacitor, I have seen 15 days of continuous Zigbee network uptime without experiencing a single 'lockup'. (My Zigbee network availability is being monitored using a Domoticz 'watchdog' device that pings me a pushover message whenever the network appears to be down).


