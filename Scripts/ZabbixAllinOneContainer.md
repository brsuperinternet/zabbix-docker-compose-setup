# Script em Shell Automatizando Zabbix em Container

Esse script ele instala o docker, depois cria o diretório conforme o padrão que realizo em produção, logo após isso ele cria o docker-compose.yaml e lembrando que tem campos no arquivo do docker compose que devem ser preenchido de acordo com seu ambiente, tais como:

DB_SERVER_HOST: Colocar o IP ou DNS do seu Zabbix

ZBX_PASSIVESERVERS:Colocar o IP ou DNS do seu Zabbix

Você deve salvar esse Script que está abaixo com o nome que desejar, recomendo como zabbix.sh pois é em shellscript e não pode esquecer o ".sh", você deve dar permissão a esse script.

## Comando para criar o arquivo

```sh
nano zabbix.sh
```
## Comando para dar permissão ao script

```sh
chmod +x zabbix.sh
```

# Abaixo conteúdo do script

```sh
#!/bin/bash

# Os três primeiros comando é para instalar o docker e update do SO
sudo curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo apt update

# Esse comando serve para criar o diretório onde irá ficar o docker-compose.yamlS
sudo mkdir -p /home/zabbix/

# Agora é a criação do arquivo docker-compose.yaml
cat <<EOF > /home/zabbix/docker-compose.yaml
version: '3.5'
services:
  zabbix-server:
    container_name: "zabbix-server"
    image: zabbix/zabbix-server-pgsql:alpine-trunk
    restart: always
    ports:
      - 10051:10051
    networks:
      - zabbix7
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro 
    environment:
      ZBX_CACHESIZE: 4096M
      ZBX_HISTORYCACHESIZE: 1024M
      ZBX_HISTORYINDEXCACHESIZE: 1024M
      ZBX_TRENDCACHESIZE: 1024M
      ZBX_VALUECACHESIZE: 1024M
      DB_SERVER_HOST: "SEUIP" 
      DB_PORT: 5432
      POSTGRES_USER: "zabbix"
      POSTGRES_PASSWORD: "zabbix123"
      POSTGRES_DB: "zabbix_db"
    stop_grace_period: 30s
    labels:
      com.zabbix.description: "Zabbix server with PostgreSQL database support"
      com.zabbix.company: "Zabbix LLC"
      com.zabbix.component: "zabbix-server"
      com.zabbix.dbtype: "pgsql"
      com.zabbix.os: "alpine"

  zabbix-web-nginx-pgsql:
    container_name: "zabbix-web"
    image: zabbix/zabbix-web-nginx-pgsql:alpine-trunk
    restart: always
    ports:
      - 8080:8080
      - 8443:8443
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./cert/:/usr/share/zabbix/conf/certs/:ro
    networks:
      - zabbix7
    environment:
      DB_SERVER_HOST: "SEUIP"
      DB_PORT: 5432
      POSTGRES_USER: "zabbix"
      POSTGRES_PASSWORD: "zabbix123"
      POSTGRES_DB: "zabbix_db"
      ZBX_MEMORYLIMIT: "1024M"
    depends_on:
      - zabbix-server
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/ping"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s
    stop_grace_period: 10s
    labels:
      com.zabbix.description: "Zabbix frontend on Nginx web-server with PostgreSQL database support"
      com.zabbix.company: "Zabbix LLC"
      com.zabbix.component: "zabbix-frontend"
      com.zabbix.webserver: "nginx"
      com.zabbix.dbtype: "pgsql"
      com.zabbix.os: "alpine"

  zabbix-db-agent2:
    container_name: "zabbix-agent2"
    image: zabbix/zabbix-agent2:alpine-trunk
    user: root
    depends_on:
      - zabbix-server
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - /run/docker.sock:/var/run/docker.sock
    environment:
      ZBX_HOSTNAME: "zabbix7"
      ZBX_SERVER_HOST: "127.0.0.1"
      ZBX_PASSIVE_ALLOW: "true"
      ZBX_PASSIVESERVERS: "SEUIP"
      ZBX_ENABLEREMOTECOMMANDS: "1"
      ZBX_ACTIVE_ALLOW: "false"
      ZBX_DEBUGLEVEL: "3"
    privileged: true
    pid: "host"
    ports:
      - 10050:10050
      - 31999:31999 
    networks:
      - zabbix7
    stop_grace_period: 5s
   
  db:
    container_name: "zabbix_db"
    image: postgres:15.6-bullseye
    restart: always
    volumes:
     - zbx_db15:/var/lib/postgresql/data
    ports:
     - 5432:5432
    networks:
     - zabbix7
    environment:
     POSTGRES_USER: "zabbix"
     POSTGRES_PASSWORD: "zabbix123"
     POSTGRES_DB: "zabbix_db"

networks:
  zabbix7:
   driver: bridge
volumes:
  zbx_db15:
EOF

# Agora irá para o diretório e dar o comando para criar os container
cd /home/zabbix/

# Esse comando serve para criar os container
sudo docker compose up -d

# Esse comando serve para ver o container
sudo docker ps

# Se for necessário ver os logs basta dar o comando baixo
# sudo docker logs nomedocontainer
``````

## Como Contribuir

Se você tem algo para contribuir, como novos templates, scripts ou guias de configuração, fique à vontade para enviar uma solicitação de pull. Contribuições são bem-vindas e apreciadas!

## Licença

Este repositório é fornecido sob a [Licença MIT](LICENSE). Sinta-se à vontade para usar, modificar e distribuir o conteúdo conforme necessário.

## Contato

Para perguntas, sugestões ou apenas para dizer olá, você pode entrar em contato com os mantenedores deste repositório através das issues ou por e-mail em fvcunhaa@gmail.com.