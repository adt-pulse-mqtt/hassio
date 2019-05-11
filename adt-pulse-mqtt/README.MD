# adt-pulse-mqtt
ADT Pulse bridge for MQTT. 

This add-on for [Hass.io](https://www.home-assistant.io/hassio/) uses MQTT to create a bridge between Home Assistant and ADT Pulse website. It will update the alarm panel status, allow the alarm to be set from Home Assistant, and update the status for each zone. It can also be used as a standalone Docker container without Hass.io.

An MQTT broker is required and the Mosquitto broker is recommended:
* Hass.io add-on - https://www.home-assistant.io/addons/mosquitto/
* Standalone Docker container - https://hub.docker.com/_/eclipse-mosquitto

For Smartthings support, see https://github.com/haruny/adt-pulse-mqtt

## Hassio Setup
* Add Craig's Addon repository (https://github.com/digitalcraig/hassio) to Hass.io.
* Install the add-on
* Set the add-on configuration options
* Start the add-on 

## Standalone Setup
### Docker Componse
<pre>
   pulse-adt-mqtt:
      container_name: pulse-adt-mqtt
      image: digitalcraig/adt-pulse-mqtt:latest-{amd64|armhf}
      restart: always
      volumes:
       - /path/to/options.json:/data
       - /path/to/ssl:/ssl
</pre>

## Configuration
### Sample options.json
<pre>
{
      "pulse_login" : {
        "username": "",
        "password": ""
      },
      "mqtt_host" : "core-mosquitto",
      "mqtt_connect_options" :  {
        "username" : "",
        "password" : "",
    		"clientId" : "",
  	  	"protocolId" : "MQIsdp",
  	  	"protocolVersion" : "3",
    		"connectTimeout" : "1000",
    		"debug" : "true"
      },
      "alarm_state_topic": "home/alarm/state",
      "alarm_command_topic": "home/alarm/cmd",
      "zone_state_topic": "adt/zone"
}
</pre>

### Add-on Configuration Options
* ssl - Use SSL? Default is false.
* certfile - Name of SSL certfile. Default is fullchain.pem
* keyfile - Name of keyfile Default is privkey.pem
* Pulse login
  * username - Pulse login 
  * password - Pulse password 
* mqtt_host - Hostname of MQTT server Default is core-mosquitto.
* mqtt_url - Use instead of mqtt_host to specify advanced connect options (see https://www.npmjs.com/package/mqtt#connect)
* mqtt_connect_options - Options to use when connecting to MQTT (see https://www.npmjs.com/package/mqtt#connect)
  * username - MQTT username
  * password - MQTT password
* alarm_state_topic - MQTT topic to post the alarm state. Default is home/alarm/state
* alarm_command_topic - MQTT topic to subscribe to receive commands. Default is home/alarm/cmd
* zone_state_topic - MQTT topic to post zone status Default is adt/zone

### Home Assistant Configuration
In configuration.yaml, add MQTT alarm control panel:

<pre>alarm_control_panel:
  - platform: mqtt
    name: "ADT Pulse"
    state_topic: "home/alarm/state"
    command_topic: "home/alarm/cmd"
    payload_arm_home: "arm_home"
    payload_arm_away: "arm_away"
    payload_disarm: "disarm"
</pre>

Once you've successfully started the add-on and connect to MQTT, list the topics in the zone_state_topic using an app such as as [MQTT Snooper for Android](https://play.google.com/store/apps/details?id=mqttsnooper.mqttsnooper) or use mosquitto from the command line:

<pre>
# mosquitto_sub -h YOUR_MQTT_IP -v -t "adt/zone/#"
</pre>

For each zone, add each of them as an MQTT binary sensor:

<pre>
binary_sensor:
  - platform: mqtt
    name: "Kitchen Door"
    state_topic: "adt/zone/Kitchen Door/state"
    payload_on: "devStatOpen" 
    payload_off: "devStatOK"  
    device_class: door
    retain: true
    value_template: '{{ value_json.status }}' 
</pre>

Note: The payload values vary depending on zone type. Topic names vary depending on your Pulse configuration. You may need to experiment to find the correct values.
