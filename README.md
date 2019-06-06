## MQTT and TLS - A Configuration Guide
+ The aim of this repository is to aggregate all the `mosquitto` configuration files I will use for my secure MQTT broker implementation.

### Basic MQTT Configuration
#### Installing mosquitto from source
+ Following, the instructions to install `mosquitto`'s MQTT implementation from source.
```bash
# Get the source files
wget https://mosquitto.org/files/source/mosquitto-1.6.2.tar.gz
cd mosquitto-1.6.2
# Dependencies for compilation
sudo apt-get install docbook-xsl xsltproc
# Compile
make
# Install
sudo make install
# Expose the shared libs
sudo echo /usr/local/lib > /etc/ld.so.conf.d/local.conf
sudo ldconfig
# Create daemon for the broker
sudo groupadd mosquitto
sudo useradd -s /sbin/nologin mosquitto -g mosquitto -d /var/lib/mosquitto
# Create /etc/systemd/system/mosquitto.service
```
+ Once you have copied `mosquitto.service` to the before mentioned path, you need only to start it:
`sudo systemctl start mosquitto.service`

### Testing the Installation
+ It is easy to test this first installation:
```
mosquitto_sub -t 't'
mosquitto_pub -t 't' -m 'Hello World!'
```

### Setting Up MQTT with SSL/TLS and a cerificate signed by Let's Encrypt
#### Broker configuration
0. Have a pointable host for the broker using `duckdns.org`
1. Generate the certificate for the broker with Let's Encrypt
```
sudo certbot certonly --standalone --preferred-challenges http -d mqt-tz.duckdns.org
```
2. All the files are then stored at `/etc/letsencrypt/live/mqt-tz.duckdns.org/`. In order to have the `mqtt` broker correctly read the files, the `mosquitto` user must have permissions to read them. As a consequence I move them to my home folder. **NOTE: This should be eventually done in secure storage or preloaded**
```
sudo cp /etc/letsencrypt/live/mqt-tz.duckdns.org/cert.pem ~/mqt-tz/certs/
sudo cp /etc/letsencrypt/live/mqt-tz.duckdns.org/chain.pem ~/mqt-tz/certs/
sudo cp /etc/letsencrypt/live/mqt-tz.duckdns.org/privkey.pem ~/mqt-tz/certs/
sudo chown mosquitto:mosquitto ~/mqt-tz/certs/*
```
3. Use the `mosquitto.conf` file provided. Place it at `/etc/mosquitto/mosquitto.conf`
4. Restart the service `sudo systemctl restart mosquitto.service`
5. Allow incoming traffic on port 8883 `sudo ufw allow 8883`

#### Client Configuration
1. The client needs _a priori_ no configuration at all. To publish do:
`mosquitto_pub -h mqt-tz.duckdns.org -p 8883 -m 'Hello World!' -t 't' --capath /etc/ssl/certs/`
2. Similarly to subscribe:
`mosquitto_sub -h mqt-tz.duckdns.org -p 8883 -t 't' --capath /etc/ssl/certs/`

#### Debugging using Wireshark
+ We use Wireshark to make sure packets are indeed being encrypted. We first caputre the packets using `tcpdump` on the broker side:
`sudo tcpdump port 8883 -i eth0 -c 16 -w mqtt-ssl.log`
+ And then open the capture using Wireshark.
