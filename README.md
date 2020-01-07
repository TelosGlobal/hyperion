**STOP AND READ!  This documentation is in progress.  Please submit comments/updates as a PR.**

# HYPERION INSTALL & RUNBOOK

This installation guide is tailored for Telos BPs who are interested in running the Hyperion v2 history tool on their telos BP node. 

## Helpful Resources:  
<https://github.com/eosrio/Hyperion-History-API>  

<https://github.com/eosrio/Hyperion-History-API/blob/master/INSTALL.md>


## Prerequisites
Start with a EOSIO node that is fully synced to the blockchain.  
**Note:** *This guide is based on EOSIO v1.8.4*

``sudo apt install gnupg gcc g++ yarn make tcl``

### ElasticSearch:
See: <https://www.elastic.co/guide/en/elasticsearch/reference/7.5/deb.html#deb-repo>  

``wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list``  

``sudo apt-get update && sudo apt-get install elasticsearch``
	
#### Setup to start automatically:  
``sudo update-rc.d elasticsearch defaults 95 10``
	
#### To Start/Stop ElasticSearch:  
``sudo -i service elasticsearch start``  
``sudo -i service elasticsearch stop``
	
#### To Check if ElasticSearch is running:  
``curl -X GET "localhost:9200/?pretty"``
	
#### Check Logs:  
``sudo journalctl -f``  
or  
``sudo journalctl --unit elasticsearch``  
or  
``sudo journalctl --unit elasticsearch --since  "2016-10-30 18:17:16"``
	
### RabbitMQ:
<https://www.rabbitmq.com/install-debian.html#supported-debian-distributions>  

	curl -fsSL https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc | sudo apt-key add -
	wget -O - "https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey" | sudo apt-key add -
		sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list <<EOF 
	
*The following installs the latest Erlang 22.x release.  Change component to "erlang-21.x" to install the latest 21.x version. "bionic" as distribution name should work for any later Ubuntu or Debian release.  See the release to distribution mapping table in RabbitMQ doc guides to learn more.*  

	deb https://dl.bintray.com/rabbitmq-erlang/debian bionic erlang  
	deb https://dl.bintray.com/rabbitmq/debian bionic main EOF  
	sudo apt-get update -y  
	sudo apt-get install rabbitmq-server -y --fix-missing 
	
#### To Start RabbitMQ:  
``service rabbitmq-server start``
	
#### Check Status:
``sudo service rabbitmq-server status``
	
#### Check logs:
``sudo journalctl --system | grep rabbitmq``
	
### REDIS:

	wget http://download.redis.io/redis-stable.tar.gz
	tar xvzf redis-stable.tar.gz
	cd redis-stable
	make
	make test
	make install
	
#### Start Redis:
``src/redis-server &``
	
	Test Redis:
	src/redis-cli
		set foo bar
		get foo

### NODE.JS v12
<https://github.com/nodesource/distributions/blob/master/README.md#installation-instructions>
	
	curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
	sudo apt-get install -y nodejs
	
	
### PM2:
<https://pm2.keymetrics.io/docs/usage/quick-start/>
	
``npm install pm2@latest -g``

####Show Logs:
	pm2 logs
	pm2 logs --lines 200


## HYPERION INSTALL
<https://github.com/eosrio/Hyperion-History-API>
	
	git clone https://github.com/eosrio/Hyperion-History-API.git
	cd Hyperion-History-API
	npm install

####EDIT CONFIGS:
	cp example-ecosystem.config.js ecosystem.config.js
	nano ecosystem.config.js

	# Enter connection details here (chain name must match on the ecosystem file)
	cp example-connections.json connections.json
	nano connections.json

###Installation COMPLETE

## RUNBOOK

###START HYPERION:
	pm2 start --only Indexer --update-env
	pm2 logs Indexer

###Stop reading and wait for queues to flush:
	pm2 trigger Indexer stop

###FORCE STOP:
	pm2 stop Indexer

###RESTART:
	pm2 start --only API --update-env
	pm2 logs API

## Feedback
**Telegram:** *kquainta*  
**Email:** *q at teleglobal dot io*
