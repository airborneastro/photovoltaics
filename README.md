## photovoltaics
**SMA inverter readout + InfluxDB v2.6.1 + Grafana v9.4.7**

 The  [node-RED flow](https://github.com/airborneastro/photovoltaics/blob/main/Flow_SMA_v03_anonymous.json) (adapted from [Markus Geiger](https://github.com/mkgeiger/node-red-sma) ) extracts information from the web interface of an SMA Tripower Smart Energy hybrid inverter in combination with an SMA Sunny Home Manager (SHM) as the energy meter. Essentially, the flow makes three http requests in sequence every 5 seconds: 

 1. login and retrieve a session id (sid)
 2. using the sid, extract a json string with inverter information
 3. logout using the sid

This can be simulated with curl statements (Linux):

    curl -X POST https://sma<serialnumber>/dyn/login.json -k -H 'Content-Type: application/json' -d '{"right": "usr", "pass": "<yourpassword>"}'
Replace \<serialnumber\> with your inverter serial number, as in sma1234567890 and "\<password\>" with your password in double quotes, as in "thisismypassword".
For Windows curl use

    curl -X POST https://sma<serialnumber>/dyn/login.json -k -H "Content-Type: application/json" -d "{\"right\": \"usr\", \"pass\": \"<yourpassword>\"}"

The response is a json string containing a sid (example only!):

    {"result":{"sid":"A53TnUC-0MQ-dWCA"}}
With the resulting sid, ask for all available data with:

    curl -X POST https://sma<serialnumber>/dyn/getAllOnlValues.json?sid=A53TnUC-0MQ-dWCA -k -H 'Content-Type: application/json' -d '{"destDev": []}'
(Windows:    

    curl -X POST https://sma<serialnumber>/dyn/getAllOnlValues.json?sid=A53TnUC-0MQ-dWCA -k -H "Content-Type: application/json" -d "{\"destDev\": []}"
)
 
The result is a (long) json string with the data as key: value pairs that can be extracted by parsing the json string. [This file](https://github.com/airborneastro/photovoltaics/blob/main/valueID_STP10_SE.xlsx)  contains all keys with their parameter descriptions that I found out by analyzing the web interface with the developer tools of a Firefox browser. The third step is to logout from the web interface with (again sample sid):

    curl -X POST https://sma<serialnumber>/dyn/logout.json?sid=A53TnUC-0MQ-dWCA -k -H 'Content-Type:application/json' -d '{}'
(Windows:

    curl -X POST https://sma<serialnumber>/dyn/logout.json?sid=A53TnUC-0MQ-dWCA -k -H "Content-Type:application/json" -d "{}"

)

The node-RED flow then sends the requested data into an influxdb2 database. The flow also contains nodes to display data in  ["Homeassistant"](https://www.home-assistant.io/) and an alternate http request only retrieving a predetermined number of key: value pairs.



The organisation of my database with its buckets, measurements and fields as used in the following grafana dashboard is described in file
[influxdb2_data_organisation](https://github.com/airborneastro/photovoltaics/blob/main/influxdb2_data_organisation) (organisation adapted from [Matthias Kleine](https://haus-automatisierung.com/software/2023/05/11/influxdb2-pv-dashboard.html))

The data is visualized in a Grafana dashboard adapted from "JSAnyone"   on
[https://www.photovoltaikforum.com/thread/150542-umfangreiches-logging-mit-grafana-anleitung/](https://www.photovoltaikforum.com/thread/150542-umfangreiches-logging-mit-grafana-anleitung/)
The queries for the grafana panels are written in the "flux" language for influxdb2, partially adapted from Matthias Kleine (see above). The dashboard json file is [here](https://github.com/airborneastro/photovoltaics/blob/main/SMA%20Smart%20Energy%2010.0.json). To show the sun elevation angle, you need to install the [sun and moon plugin](https://grafana.com/grafana/plugins/fetzerch-sunandmoon-datasource/).

I added a node-RED flow to directly  [readout a BYD](https://github.com/airborneastro/photovoltaics/blob/main/BYD_readout.json) battery via its "WiFiAP" network address in the local network, based on the iobroker version of [christianh17](https://github.com/christianh17/ioBroker.bydhvs/blob/master/main.js). It will output voltage, current, SOC of the battery and many more parameters and can also display voltages and temperatures of all individual cells. It was tested with 2 Modules (5.1kWh) and should work with up to three stacks with up to 5 Modules. The output is presently only in node-RED debug nodes, but could easily be used to send data to influxdb2. You can check with `echo -n -e "\x01\x03\x00\x00\x00\x66\xc5\xe0" | nc 192.168.xxx.xxx 8080` whether your BYD listens. You should see a reponse with the serial number in ASCII and some binary data.

I also added a node-RED flow to read and operate the SMA EV charger ("wallbox"). Mostly adapted/simplified code from [here](https://homematic-forum.de/forum/viewtopic.php?f=18&t=72536&sid=5cbddf649a40e787d9da95c92fee1a37) and [here](https://github.com/Nerdiyde/NodeRedSnippets/tree/master/snippets/sma_devices). Note that the charging (parameter setting) functions have NOT been tested!

Another node red flow for a simple [battery control](https://github.com/airborneastro/photovoltaics/blob/main/battcharge.json) was added to e.g. limit the SOC to 80% until a given time of day or to limit the charge power to leave power for other equipment such as a water heater for a given time. The flow has a link to the inverter data to input the present solar power ("generatorW") and the grid input and output power values ("meterInW", "meterOutW"). Note that after the recent (May 2024) update of the Sunny Home Manager you need your Grid Guard Code for this flow to work (set to 1234567890 here). I do not take any responsibility whatsoever for what you do with this code! There are nodes in the flow that periodically renew the GGC registers etc. because it has a timeout of 60min. (If it times out, you will have to restart the STP inverter (via its webinterface or by rewriting the GGC)). You have to enter the IP of your Sunny Home Manager in the Modbus configuration.

All is "work in progress". The project runs in docker containers for node-RED, influxdb2, grafana and homeassistant using a Raspberry Pi 4 (4GB).
![Dashboard](https://github.com/airborneastro/photovoltaics/blob/main/Grafana_SMA_STP_SE10_part1.PNG)









