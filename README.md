<span style="color:red">**STOP AND READ!  This documentation is in progress.  Please submit comments/updates as a PR.**</span>

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

#### Configure ElasticSearch
Edit ``/etc/elasticsearch/elasticsearch.yml``
	
	cluster.name: myCluster
	bootstrap.memory_lock: true
Edit ``/etc/elasticsearch/jvm.options``
	
	# Set your heap size, avoid allocating more than 31GB, even if you have enought RAM.
	# Test on your specific machine by changing -Xmx32g in the following command:
	# java -Xmx32g -XX:+UseCompressedOops -XX:+PrintFlagsFinal Oops | grep Oops
	-Xms16g
	-Xmx16g
	
Disable swap on /etc/fstab
	
##### Allow memlock:  
run ``sudo systemctl edit elasticsearch``  

and add the following lines
	
	[Service]
	LimitMEMLOCK=infinity

Start elasticsearch and check the logs  
(verify if the memory lock was successful)

	sudo service elasticsearch start
	sudo less /var/log/elasticsearch/myCluster.log
	sudo systemctl enable elasticsearch

Test the REST API ``curl http://localhost:9200``

	{
	  "name" : "ip-172-31-5-121",
	  "cluster_name" : "hyperion",
	  "cluster_uuid" : "....",
	  "version" : {
	    "number" : "7.1.0",
	    "build_flavor" : "default",
	    "build_type" : "deb",
	    "build_hash" : "606a173",
	    "build_date" : "2019-05-16T00:43:15.323135Z",
	    "build_snapshot" : false,
	    "lucene_version" : "8.0.0",
	    "minimum_wire_compatibility_version" : "6.8.0",
	    "minimum_index_compatibility_version" : "6.0.0-beta1"
	  },
	  "tagline" : "You Know, for Search"
	}

### Kibana Installation
	wget https://artifacts.elastic.co/downloads/kibana/kibana-7.4.0-amd64.deb
	sudo apt install ./kibana-7.4.0-amd64.deb
	sudo systemctl enable kibana
	sudo service kibana start
Open and test Kibana: ``curl http://localhost:5601``
	
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

	sudo rabbitmq-plugins enable rabbitmq_management
	sudo rabbitmqctl add_vhost /hyperion
	sudo rabbitmqctl add_user my_user my_password
	sudo rabbitmqctl set_user_tags my_user administrator
	sudo rabbitmqctl set_permissions -p /hyperion my_user ".*" ".*" ".*"
	
#### To Start RabbitMQ:  
``service rabbitmq-server start``
	
#### Check Status:
``sudo service rabbitmq-server status``

#### Check access to the WebUI  
	curl http://localhost:15672
	
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
``sudo pm2 startup``

#### Show Logs:
	pm2 logs
	pm2 logs --lines 200


## HYPERION INSTALL
<https://github.com/eosrio/Hyperion-History-API>
	
	git clone https://github.com/eosrio/Hyperion-History-API.git
	cd Hyperion-History-API
	npm install

#### EDIT CONFIGS:
	cp example-ecosystem.config.js ecosystem.config.js
	nano ecosystem.config.js

	# Enter connection details here (chain name must match on the ecosystem file)
	cp example-connections.json connections.json
	nano connections.json


### Hyperion Indexer
	git clone https://github.com/eosrio/Hyperion-History-API.git
	cd Hyperion-History-API
	npm install
	cp example-ecosystem.config.js ecosystem.config.js

### Setup Indices and Aliases
Load templates first by starting the Hyperion Indexer in preview mode

	PREVIEW: 'true'

Indices and aliases are created automatically using the ``CREATE_INDICES``option (set it to your version suffix e.g, v1, v2, v3) If you want to create them manually, use the commands bellow on the kibana dev console

	PUT mainnet-action-v1-000001
	PUT mainnet-abi-v1-000001
	PUT mainnet-block-v1-000001
	
	POST _aliases
	{
	  "actions": [
	    {
	      "add": {
	        "index": "mainnet-abi-v1-000001",
	        "alias": "mainnet-abi"
	      }
	    },
	    {
	      "add": {
	        "index": "mainnet-action-v1-000001",
	        "alias": "mainnet-action"
	      }
	    },
	    {
	      "add": {
	        "index": "mainnet-block-v1-000001",
	        "alias": "mainnet-block"
	      }
	    }
	  ]
	}
Before indexing actions into elasticsearch its required to do a ABI scan pass

Start with

	ABI_CACHE_MODE: 'true',
	FETCH_BLOCK: 'false',
	FETCH_TRACES: 'false',
	INDEX_DELTAS: 'false',
	INDEX_ALL_DELTAS: 'false',
When indexing is finished, change the settings back and restart the indexer. In case you do not have much contract updates, you do not need to run a full pass.

Tune your configs to your specific hardware using the following settings:

	BATCH_SIZE
	READERS
	DESERIALIZERS
	DS_MULT
	ES_IDX_QUEUES
	ES_AD_IDX_QUEUES
	READ_PREFETCH
	BLOCK_PREFETCH
	INDEX_PREFETCH

### Installation COMPLETE

## RUNBOOK

### START HYPERION:
	pm2 start --only Indexer --update-env
	pm2 logs Indexer

### Stop reading and wait for queues to flush:
	pm2 trigger Indexer stop

### FORCE STOP:
	pm2 stop Indexer

### RESTART:
	pm2 start --only API --update-env
	pm2 logs API

## Feedback
**Telegram:** *kquainta*  
**Email:** *q at teleglobal dot io*



