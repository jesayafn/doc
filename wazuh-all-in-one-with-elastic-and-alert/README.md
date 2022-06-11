# Wazuh All in One with Elastic and Alert

## A. Overview

### Components will be installed

1. Elasticsearch
1. Filebeat
1. Wazuh module for Filebeat
1. Wazuh Manager
1. Kibana
1. Wazuh plugin for Kibana
1. Wazuh Agent

Components number 1-5 will be installed on 1 node as a wazuh server and component number 6 will be installed on every node needed to monitor

### High-level architecture

![Wazuh All in One with Elastic high-level architecture](img/i-3-doc_wazuh-all-in-one-with-elastic-and-alert_0.png)

## B. Configurations steps

> **ℹ️ Operating system used is Ubuntu-based.**

### 1. Install Wazuh and Elastic Stack

> **⚠️ Attention: All commands run with privileges.⚠️**
>
> Run `sudo -i` before run any steps in this section and for exit from `sudo -i` session, run `exit` command or Ctrl+D

#### 1. Add Wazuh and Elastic Stack repository.

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

#### 2. Setup Elasticsearch

1. Install Elasticsearch

    ```bash
    apt install elasticsearch=7.17.4
    ```

1. Configure self-signed certificates for Elasticsearch

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
    /usr/share/elasticsearch/bin/elasticsearch-certutil cert ca --pem --in instances.yml --keep-ca-key --out ~/certs.zip
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
    
    curl -XGET https://[node-IP-address]:9200 -u elastic:[Elastic-password] -k
    ```

#### 3. Setup Wazuh Manager

1. Install Wazuh Manager

    ``` bash
    apt install wazuh-manager
    ```

1. Run the Wazuh Manager service

    ``` bash
    systemctl daemon-reload
    systemctl enable wazuh-manager
    systemctl start wazuh-manager
    ```

#### 4. Setup Filebeat

1. Install Filebeat

    ```bash
    apt install filebeat=7.17.4
    ```

1. Configure Filebeat

    ```bash
    vi /etc/filebeat/filebeat.yml
    ```

    ```yaml
    # Wazuh - Filebeat configuration file
    output.elasticsearch.hosts: [node-IP-address]:[elasticsearch-port]
    output.elasticsearch.password: [Elastic-password]

    filebeat.modules:
    - module: wazuh
        alerts:
        enabled: true
        archives:
        enabled: false

    setup.template.json.enabled: true
    setup.template.json.path: /etc/filebeat/wazuh-template.json
    setup.template.json.name: wazuh
    setup.template.overwrite: true
    setup.ilm.enabled: false

    output.elasticsearch.protocol: https
    output.elasticsearch.ssl.certificate: /etc/elasticsearch/certs/elasticsearch.crt
    output.elasticsearch.ssl.key: /etc/elasticsearch/certs/elasticsearch.key
    output.elasticsearch.ssl.certificate_authorities: /etc/elasticsearch/certs/ca/ca.crt
    output.elasticsearch.ssl.verification_mode: strict
    output.elasticsearch.username: elastic
    ```

1. Add alert template

    ```bash
    curl -so /etc/filebeat/wazuh-template.json https://raw.githubusercontent.com/wazuh/wazuh/4.3/extensions/elasticsearch/7.x/wazuh-template.json
    chmod go+r /etc/filebeat/wazuh-template.json
    ```

1. Configure Wazuh module for Filebeat

    ```bash
    curl -s https://packages.wazuh.com/4.x/filebeat/wazuh-filebeat-0.2.tar.gz | tar -xvz -C /usr/share/filebeat/module
    ```

1. Configure self-signed certificates for Filebeat

    ```bash
    cp -r /etc/elasticsearch/certs/ca/ /etc/filebeat/certs/
    cp /etc/elasticsearch/certs/elasticsearch.crt /etc/filebeat/certs/filebeat.crt
    cp /etc/elasticsearch/certs/elasticsearch.key /etc/filebeat/certs/filebeat.key
    ```

1. Run the Filebeat service

    ```bash
    systemctl daemon-reload
    systemctl enable filebeat
    systemctl start filebeat
    ```

1. Test the Filebeat service

    ```bash
    filebeat test output
    ```

    ```output
     elasticsearch: https://[node-IP-address]:[elasticsearch-port]...
      parse url... OK
      connection...
        parse host... OK
        dns lookup... OK
        addresses: 127.0.0.1
        dial up... OK
      TLS...
        security: server's certificate chain verification is enabled
        handshake... OK
        TLS version: TLSv1.3
        dial up... OK
      talk to server... OK
      version: 7.17.4
    ```

#### 5. Setup Kibana

1. Install Kibana

    ```bash
    apt install kibana=7.17.4
    ```

1. Configure self-signed certificates for Kibana

    ```bash
    mkdir /etc/kibana/certs/ca -p
    cp -R /etc/elasticsearch/certs/ca/ /etc/kibana/certs/
    cp /etc/elasticsearch/certs/elasticsearch.key /etc/kibana/certs/kibana.key
    cp /etc/elasticsearch/certs/elasticsearch.crt /etc/kibana/certs/kibana.crt
    chown -R kibana:kibana /etc/kibana/
    chmod -R 500 /etc/kibana/certs
    chmod 440 /etc/kibana/certs/ca/ca.* /etc/kibana/certs/kibana.*
    ```

1. Configure Kibana

    ```bash
    vi /etc/kibana/kibana.yml
    ```

    ```yaml
    server.host: [node-IP-address]
    server.port: [kibana-port]
    elasticsearch.hosts: https://[node-IP-address]:[elasticsearch-port]
    elasticsearch.password: [elasticsearch-password]

    # Elasticsearch from/to Kibana

    elasticsearch.ssl.certificateAuthorities: /etc/kibana/certs/ca/ca.crt
    elasticsearch.ssl.certificate: /etc/kibana/certs/kibana.crt
    elasticsearch.ssl.key: /etc/kibana/certs/kibana.key

    # Browser from/to Kibana
    server.ssl.enabled: true
    server.ssl.certificate: /etc/kibana/certs/kibana.crt
    server.ssl.key: /etc/kibana/certs/kibana.key

    # Elasticsearch authentication
    xpack.security.enabled: true
    elasticsearch.username: elastic
    uiSettings.overrides.defaultRoute: "/app/wazuh"
    elasticsearch.ssl.verificationMode: certificate
    telemetry.banner: false
    ```

    ```bash
    mkdir /usr/share/kibana/data
    chown -R kibana:kibana /usr/share/kibana
    ```

    > **⚠️ Attention ⚠️**
    >
    > The following command will needed only if you are using a [privileged ports or well-known ports (0-1023)](https://superuser.com/questions/710253/allow-non-root-process-to-bind-to-port-80-and-443) for Kibana service

    ```bash
    setcap 'cap_net_bind_service=+ep' /usr/share/kibana/node/bin/node
    ```

1. Configure Wazuh plugin for Kibana

    ```bash
    cd /usr/share/kibana
    sudo -u kibana /usr/share/kibana/bin/kibana-plugin install https://packages.wazuh.com/4.x/ui/kibana/wazuh_kibana-4.3.4_7.17.4-1.zip
    ```

1. Run the Kibana service

    ```bash
    systemctl daemon-reload
    systemctl enable kibana
    systemctl start kibana
    ```

1. Test the Kibana service

    1. Open any browser and visit `https://[node-IP-address]:[kibana-port]`

    1. Login to the Kibana with credential on below:

        ```info
        Username: elastic
        Password: [elasticsearch-password]
        ```

    1. Kibana will health check itself and make sure every component is  healthy ✔️ like screenshot below:

        ![Healthy Wazuh with Elastic](img/i-3-doc_wazuh-all-in-one-with-elastic-and-alert_1.jpeg)

## C. Appendix
