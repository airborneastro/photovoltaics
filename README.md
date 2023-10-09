# photovoltaics
**SMA inverter readout + InfluxDB v2.6.1 + Grafana v9.4.7**

 The node red flow "Flow_SMA_v03_anonymous.json" extracts information from the web interface of an SMA Tripower Smart Energy hybrid inverter in combination with an SMA Sunny Home Manager (SHM). Essentially,the flow makes three http requests in sequence: 

 1. login and retrieve a session id (sid)
 2. using the sid, extract a json string with inverter information
 3. logout using the sid

This can be simulated with curl statements:

    curl -X POST https://sma<serialnumber>/dyn/login.json -k -H 'Content-Type: application/json' -d '{"right": "usr", "pass": "<yourpassword>"}'

The response is a json string containing a sid:

    {"result":{"sid":"A53TnUC-0MQ-dWCA"}}
With the sid, ask for all available data with:

    curl -X POST https://sma<serialnumber>/dyn/getAllOnlValues.json?sid=A53TnUC-0MQ-dWCA -k -H 'Content-Type: application/json' -d '{"destDev": []}'
    
The result is a (long) json string with the data as key: value pairs that can be extracted by parsing the json string. The file "valueID_STP10_SE.xlsx" contains all keys with their parameter descriptions that I found out by analyzing the web interface with the developer tools of a Firefox browser. The third step is to logout from the web interface with:

    curl -X POST https://sma<serialnumber>/dyn/logout.json?sid=A53TnUC-0MQ-dWCA -k -H 'Content-Type:application/json' -d '{}'

The red node flow then sends the requested data into an influxdb2 database. The flow also contains nodes to display data in "Homeassistant" (https://www.home-assistant.io/) and an alternate http request only retrieving a predetermined number of key: value pairs.

influxdb2 data organisation adapted from Matthias Kleine in
https://haus-automatisierung.com/software/2023/05/11/influxdb2-pv-dashboard.html

The organisation of the database with its buckets, measurements and fields as used in the following grafana dashboard is described in "influxdb2_data_organisation"

The data is visualized in a Grafana dashboard adapted from "JSAnyone" on
https://www.photovoltaikforum.com/thread/150542-umfangreiches-logging-mit-grafana-anleitung/
The queries for the grafana panels are written in the "flux" language for influxdb2. The dashboard json file is "SMA Smart Energy10.0-1696871662047.json"

All is "work in progress". The project runs in docker containers for node red, influxdb2, grafana and homeassistant using a Raspberry Pi 4 (4GB).








