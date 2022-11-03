# Configuring Grafana, MQTT, Influx DB

## Useful links

- [BBQKees](https://bbqkees-electronics.nl/wiki/index.html)
- [Telegraf MQTT Consumer plugin repo](https://github.com/influxdata/telegraf/tree/release-1.24/plugins/inputs/mqtt_consumer)
- [EMS-ESP Docs](https://emsesp.github.io/docs/#/)
- [MQTT Topic parsing](https://www.influxdata.com/blog/mqtt-topic-payload-parsing-telegraf/)
- [Grafana EMS ESP Dashboard](https://grafana.com/grafana/dashboards/15940-ems-esp/) - probably not compatible



___
## Install Grafana
Download packages and install the *.deb
```
sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/enterprise/release/grafana-enterprise_9.2.3_arm64.deb
sudo dpkg -i grafana-enterprise_9.2.3_arm64.deb
```
After install run the following commands:
```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable grafana-server
sudo /bin/systemctl start grafana-server
```
The server should run on `127.0.0.1:3000`

Default credentials: admin/admin

---

## Install & Configure Mosquitto (MQTT)

```
sudo apt install mosquitto mosquitto-clients
sudo nano /etc/mosquitto/mosquitto.conf
```

Add the two following lines to the mosquitto.conf:

```
listener 1883
allow_anonymous true
```
Save and restart the service

```
sudo systemctl mosquitto restart
```

After restart, you should be able to Connect to the MQTT Service via the EMS-ESP Webinterface:

1. Enable MQTT
2. Set the Hostname to the address of the Raspberry
3. Save and Check the MQTT Status, should be `Connected`

---

## Install & Configure InfluxDB

```
sudo apt update && sudo apt upgrade -y

curl https://repos.influxdata.com/influxdb.key | gpg --dearmor | sudo tee /usr/share/keyrings/influxdb-archive-keyring.gpg >/dev/null

echo "deb [signed-by=/usr/share/keyrings/influxdb-archive-keyring.gpg] https://repos.influxdata.com/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

sudo apt update
sudo apt install influxdb
```

In `/etc/influxdb/influxdb.conf` uncomment the following lines: 
```
# Determines whether HTTP endpoint is enabled.
  enabled = true

# Determines whether the Flux query endpoint is enabled.
  flux-enabled = false
```

Open the InfluxDB-CLI with the command:
```
influx
```
and create an administrator user for Telegraf

```SQL
CREATE USER admin WITH PASSWORD '<password>' WITH ALL PRIVILEGES
```
and restart the service:

```
sudo systemctl unmask influxdb
sudo systemctl enable influxdb
sudo systemctl start influxdb
```
---

## Install & Configure Telegraf

```
sudo apt install telegraf
```

Uncomment & set the following lines in `/etc/telegraf/telegraf.conf`:

```
[[outputs.influxdb]]
    urls = [http://127.0.0.1:8086]
    //TODO: add telegraf adding to influxdb docs
    database_tag = "telegraf"
    ...
    username = "<InfluxDB username>"
    password = "<InfluxDB password>"

```
and

```
[[inputs.mqtt_consumer]]
    servers = ["tcp://127.0.0.1:1883"]
    topics = ["ems-esp/boiler_data_ww"]
    topic_tag = ""
    ...
    username = "<MQTT username>"
    password = "<MQTT password>"
    ...
    data_format = "json"

```

---
## Configure Grafana

On `localhost:3000`  log in, go to 

**Dashboards -> + Import**

Dashboard ID: **15940**
