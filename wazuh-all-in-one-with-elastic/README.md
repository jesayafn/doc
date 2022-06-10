# Wazuh All in One with Elastic

## A. Overview

### Components will be installed

1. Elasticsearch
1. FileBeat
1. Wazuh Manager
1. Kibana
1. Wazuh plugin for Kibana
1. Wazuh Agent

Components number 1-5 will be installed on 1 node as a wazuh server and component number 6 will be installed on every node needed to monitor

### High-level architecture

![Wazuh All in One with Elastic high-level architecture](img/i-3-doc_wazuh-all-in-one-with-elastic_0.png)

## B. Installation steps

> Operating system used is Ubuntu-based.

### 1. Install Wazuh and Elastic Stack

> All commands run with privileges.
>
> Run the following command before run any steps in this section:
>
> ```bash
> sudo -i
> ```
>
1. Add Wazuh and Elastic Stack repository.

    1. Install prerequisites

        ```bash
        apt-get install apt-transport-https zip unzip lsb-release curl gnupg
        ```

    1. Add  repository

        ```bash
        curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
        echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list
        
        curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | apt-key add -
        echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list

        apt update
        ```

1. Setup Elasticsearch
    1. Install Elasticsearch

        ```bash
        apt install elasticsearch=7.17.4
        ```

    1. Configure self-signed for Elasticsearch

        ```bash
        vi /usr/share/elasticsearch/instances.yml
        ```

        ```yaml
        instances:
          - name: [node-hostname]
            ip:
            - [node-IP-address]
        ```

        ```bash
        /usr/share/elasticsearch/bin/elasticsearch-certutil cert ca \
        --pem --in instances.yml --keep-ca-key --out ~/certs.zip
        unzip ~/certs.zip -d ~/certs
        mkdir /etc/elasticsearch/certs/ca -p
        cp -R ~/certs/ca/ ~/certs/elasticsearch/* /etc/elasticsearch/certs/
        chown -R elasticsearch: /etc/elasticsearch/certs
        chmod -R 500 /etc/elasticsearch/certs
        chmod 400 /etc/elasticsearch/certs/ca/ca.* /etc/elasticsearch/certs/elasticsearch.*
        rm -rf ~/certs/ ~/certs.zip
        ```

    1. Configure Elasticsearch

        ```bash
        vi /etc/elasticsearch/elasticsearch.yml
        ```

        ```yaml
        network.host: [node-IP-address]
        node.name: [hostname]
        cluster.initial_master_nodes: [hostname]

        # Transport layer
        xpack.security.transport.ssl.enabled: true
        xpack.security.transport.ssl.verification_mode: certificate
        xpack.security.transport.ssl.key: /etc/elasticsearch/certs/elasticsearch.key
        xpack.security.transport.ssl.certificate: /etc/elasticsearch/certs/elasticsearch.crt
        xpack.security.transport.ssl.certificate_authorities: /etc/elasticsearch/certs/ca/ca.crt

        # HTTP layer
        xpack.security.http.ssl.enabled: true
        xpack.security.http.ssl.verification_mode: certificate
        xpack.security.http.ssl.key: /etc/elasticsearch/certs/elasticsearch.key
        xpack.security.http.ssl.certificate: /etc/elasticsearch/certs/elasticsearch.crt
        xpack.security.http.ssl.certificate_authorities: /etc/elasticsearch/certs/ca/ca.crt

        # Elasticsearch authentication
        xpack.security.enabled: true

        path.data: /var/lib/elasticsearch
        path.logs: /var/log/elasticsearch
        ```

    1. Run the Elasticsearch service

        ```bash
        systemctl daemon-reload
        systemctl enable elasticsearch
        systemctl start elasticsearch
        ```

    1. Generate built-in user password for elasticsearch check elastic user password

        ```bash
        /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
        
        ---output ommited--
        Changed password for user elastic
        PASSWORD elastic = [Elastic-password]
        ```

        ```bash
        curl -XGET https://[node-IP-address]:9200 -u elastic:[Elastic-password] -k
        ```
        