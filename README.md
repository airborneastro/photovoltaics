# photovoltaics
**SMA inverter readout + InfluxDB v2.6.1 + Grafana v9.4.7**

 The  [node-RED flow](https://github.com/airborneastro/photovoltaics/blob/main/Flow_SMA_v03_anonymous.json) (adapted from [Markus Geiger](https://github.com/mkgeiger/node-red-sma) ) extracts information from the web interface of an SMA Tripower Smart Energy hybrid inverter in combination with an SMA Sunny Home Manager (SHM) as the energy meter. Essentially, the flow makes three http requests in sequence every 5 seconds: 

 1. login and retrieve a session id (sid)
 2. using the sid, extract a json string with inverter information
 3. logout using the sid

This can be simulated with curl statements:

    curl -X POST https://sma<serialnumber>/dyn/login.json -k -H 'Content-Type: application/json' -d '{"right": "usr", "pass": "<yourpassword>"}'

The response is a json string containing a sid:

    {"result":{"sid":"A53TnUC-0MQ-dWCA"}}
With the sid, ask for all available data with:

    curl -X POST https://sma<serialnumber>/dyn/getAllOnlValues.json?sid=A53TnUC-0MQ-dWCA -k -H 'Content-Type: application/json' -d '{"destDev": []}'
    
The result is a (long) json string with the data as key: value pairs that can be extracted by parsing the json string. [This file](https://github.com/airborneastro/photovoltaics/blob/main/valueID_STP10_SE.xlsx)  contains all keys with their parameter descriptions that I found out by analyzing the web interface with the developer tools of a Firefox browser. The third step is to logout from the web interface with:

    curl -X POST https://sma<serialnumber>/dyn/logout.json?sid=A53TnUC-0MQ-dWCA -k -H 'Content-Type:application/json' -d '{}'

The node-RED flow then sends the requested data into an influxdb2 database. The flow also contains nodes to display data in  ["Homeassistant"](https://www.home-assistant.io/) and an alternate http request only retrieving a predetermined number of key: value pairs.



The organisation of my database with its buckets, measurements and fields as used in the following grafana dashboard is described in file
[influxdb2_data_organisation](https://github.com/airborneastro/photovoltaics/blob/main/influxdb2_data_organisation) (organisation adapted from [Matthias Kleine](https://haus-automatisierung.com/software/2023/05/11/influxdb2-pv-dashboard.html))

The data is visualized in a Grafana dashboard adapted from "JSAnyone"   on
[https://www.photovoltaikforum.com/thread/150542-umfangreiches-logging-mit-grafana-anleitung/](https://www.photovoltaikforum.com/thread/150542-umfangreiches-logging-mit-grafana-anleitung/)
The queries for the grafana panels are written in the "flux" language for influxdb2, partially adapted from Matthias Kleine (see above). The dashboard json file is ["SMA Smart Energy 10.0-1697030388119.json"](https://github.com/airborneastro/photovoltaics/blob/main/SMA%20Smart%20Energy%2010.0-1697030388119.json) To show the sun elevation angle, you need to install the [sun and moon plugin](https://grafana.com/grafana/plugins/fetzerch-sunandmoon-datasource/).

I added a node-RED flow to directly  [readout a BYD](https://github.com/airborneastro/photovoltaics/blob/main/BYD_readout.json) battery via its "WiFiAP" network address in the local network, based on the iobroker version of [christianh17](https://github.com/christianh17/ioBroker.bydhvs/blob/master/main.js). (No user/password and no static route needed). It will output voltage, current, SOC of the battery and many more parameters and can also display voltages and temperatures of all individual cells. It was tested with 2 Modules (5.1kWh) and should work up to 4 Modules. I did not (yet) implement more modules. The output is presently only in node-RED debug nodes, but could easily be used to send data to influxdb2.

All is "work in progress". The project runs in docker containers for node-RED, influxdb2, grafana and homeassistant using a Raspberry Pi 4 (4GB).

![Dashboard](https://github.com/airborneastro/photovoltaics/blob/main/Grafana_SMA_STP_SE10_part1.PNG)









