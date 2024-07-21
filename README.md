<p align="center">
<img src="https://cwmkt.com.br/wp-content/uploads/2024/04/logo_github.png" width="240" />
<p align="center">Seja bem-vindo ao Guia de Instalação Docker N8N 🚀</p>
</p>
  
<p align="center"> 
<a href="https://hubconnect.top" target="_blank">👉 Participe da Comunidade HubConnect 👈</a>
</p>

<hr />
<hr />

### Caso não tenha Portainer e Traefix instalado, siga primeira etapa

<details>
<summary>Instalando Portainer e Traefix</summary>

### Atualizando Dependências

Atualize os repositórios do Ubuntu executando o seguinte comando:

```bash
sudo apt update && apt upgrade -y
```

----------------------------------------------------------------------------

**Instale o Docker em sua VPS**

```bash
sudo apt install docker.io -y
```

----------------------------------------------------------------------------

**Instalando Portainer**

```bash
docker swarm init
```

```bash
nano traefik.yml
```

```bash
version: "3.8"

services:

  traefik:
    image: traefik:v2.11.0
    command:
      - "--api.dashboard=true"
      - "--providers.docker.swarmMode=true"
      - "--providers.docker.endpoint=unix:///var/run/docker.sock"
      - "--providers.docker.exposedbydefault=false"
      - "--providers.docker.network=ecosystem_network"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.web.http.redirections.entryPoint.to=websecure"
      - "--entrypoints.web.http.redirections.entryPoint.scheme=https"
      - "--entrypoints.web.http.redirections.entrypoint.permanent=true"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.letsencryptresolver.acme.httpchallenge=true"
      - "--certificatesresolvers.letsencryptresolver.acme.httpchallenge.entrypoint=web"
      - "--certificatesresolvers.letsencryptresolver.acme.email=contato@seudominio.com.br"
      - "--certificatesresolvers.letsencryptresolver.acme.storage=/etc/traefik/letsencrypt/acme.json"
      - "--log.level=DEBUG"
      - "--log.format=common"
      - "--log.filePath=/var/log/traefik/traefik.log"
      - "--accesslog=true"
      - "--accesslog.filepath=/var/log/traefik/access-log"
    deploy:
      placement:
        constraints:
          - node.role == manager
      labels:
        - "traefik.enable=true"
        - "traefik.http.middlewares.redirect-https.redirectscheme.scheme=https"
        - "traefik.http.middlewares.redirect-https.redirectscheme.permanent=true"
        - "traefik.http.routers.http-catchall.rule=hostregexp(`{host:.+}`)"
        - "traefik.http.routers.http-catchall.entrypoints=web"
        - "traefik.http.routers.http-catchall.middlewares=redirect-https@docker"
        - "traefik.http.routers.http-catchall.priority=1"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "traefik_certificates_volume:/etc/traefik/letsencrypt"
    ports:
      - target: 80
        published: 80
        mode: host
      - target: 443
        published: 443
        mode: host
    networks:
      - ecosystem_network

volumes:
  traefik_certificates_volume:
    external: true
    name: traefik_certificates_volume

networks:
  ecosystem_network:
    external: true
    name: ecosystem_network
 ```

```bash
nano portainer.yml
```

```bash
version: "3.8"

services:

  agent:
    image: portainer/agent:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    networks:
      - ecosystem_network
    deploy:
      mode: global
      placement:
        constraints: [node.platform.os == linux]

  portainer:
    image: portainer/portainer-ce:latest
    command: -H tcp://tasks.agent:9001 --tlsskipverify
    volumes:
      - portainer_volume:/data
    networks:
      - ecosystem_network
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints: [node.role == manager]
      labels:
        - "traefik.enable=true"
        - "traefik.docker.network=ecosystem_network"
        - "traefik.http.routers.portainer.rule=Host(`painel.seudominio.com.br`)"
        - "traefik.http.routers.portainer.entrypoints=websecure"
        - "traefik.http.routers.portainer.priority=1"
        - "traefik.http.routers.portainer.tls.certresolver=letsencryptresolver"
        - "traefik.http.routers.portainer.service=portainer"
        - "traefik.http.services.portainer.loadbalancer.server.port=9000"

networks:
  ecosystem_network:
    external: true
    attachable: true
    name: ecosystem_network

volumes:
  portainer_volume:
    external: true
    name: portainer_volume
```

```bash
docker network create --driver=overlay ecosystem_network
```

```bash
docker stack deploy --prune --resolve-image always -c traefik.yml traefik
```

```bash
docker stack deploy --prune --resolve-image always -c portainer.yml portainer
```

Acesse URL de seu Site e Crie Usuario


</details>


Postgres

### Adicione Stack abaixo, Stack > add stack

![image](https://github.com/cwmkt/dockerquepasa/assets/91642837/623a6dc6-c231-4105-9a02-3070d894adb8)

```bash
version: "3.8"

services:

  postgresql:
    image: bitnami/postgresql:15
    restart: always
    environment:
      POSTGRESQL_USERNAME: postgres
      POSTGRESQL_PASSWORD: r45796yv3bhub9w4f3ga3ikxmxos648r
    networks:
      - ecosystem_network
    ports:
      - 5432:5432
    volumes:
      - postgresql_volume:/bitnami/postgresql
      - postgresql_config_volume:/opt/bitnami/postgresql/conf
    deploy:
    

volumes:
  postgresql_volume:
    external: true
    name: postgresql_volume
  postgresql_config_volume:
    external: true
    name: postgresql_config_volume
    
networks:
  ecosystem_network:
    external: true
    name: ecosystem_network
```

Depois clique em DEPLOY

![image](https://github.com/cwmkt/dockerquepasa/assets/91642837/bdc62781-993a-4d31-b8cd-5cd6466900f5)

Acesse portainer, vá até contaneir Postgres, crie banco de dados

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/503bf33f-ff42-4fe5-9a8f-a98e2d80d6e4)

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/67eb98c2-f7e7-4f27-ae9d-1befc672edcf)


```bash
psql -U postgres
```

```bash
create database n8n;
```

### Adicione Stack abaixo, Stack > add stack

![image](https://github.com/cwmkt/dockerquepasa/assets/91642837/623a6dc6-c231-4105-9a02-3070d894adb8)

Lembre de Alterar os dados 

seuemail@seuemail.com.br<br>


```bash
version: "3.8"

services:

  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    restart: always
    networks:
      - ecosystem_network
    volumes:
      - n8n_cwmkt_volume:/home/node/.n8n      
    deploy:

      labels:
        - traefik.enable=true
        - traefik.docker.network=ecosystem_network
        - traefik.http.routers.n8n.rule=Host(`${N8N_HOST}`)
        - traefik.http.routers.n8n.tls=true
        - traefik.http.routers.n8n.entrypoints=web,websecure
        - traefik.http.routers.n8n.tls.certresolver=letsencryptresolver
        - traefik.http.routers.n8n.service=n8n_cwmkt
        - traefik.http.routers.n8n.priority=1      
        - traefik.http.middlewares.n8n.headers.SSLRedirect=true
        - traefik.http.middlewares.n8n.headers.STSSeconds=315360000
        - traefik.http.middlewares.n8n.headers.browserXSSFilter=true
        - traefik.http.middlewares.n8n.headers.contentTypeNosniff=true
        - traefik.http.middlewares.n8n.headers.forceSTSHeader=true
        - traefik.http.middlewares.n8n.headers.SSLHost=${N8N_HOST}
        - traefik.http.middlewares.n8n.headers.STSIncludeSubdomains=true
        - traefik.http.middlewares.n8nt.headers.STSPreload=true
        - traefik.http.services.n8n.loadbalancer.server.port=5678
        - traefik.http.services.n8n.loadbalancer.passHostHeader=true            
    environment:
      - N8N_HOST=https://${N8N_HOST}
      - N8N_PROTOCOL=https
      - N8N_EDITOR_BASE_URL=https://${N8N_HOST}
      - NODE_ENV=production
      - WEBHOOK_URL=https://${N8N_HOST}
      - GENERIC_TIMEZONE=America/Sao_Paulo
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_HOST=postgresql
      - DB_POSTGRESDB_USER=postgres
      - DB_POSTGRESDB_PASSWORD=senhaaqui
      - C8Q_SINGLETHREAD=false
      - C8Q_QUEPASAINBOXCONTROL=1001
      - C8Q_GETCHATWOOTCONTACTS=1002
      - C8Q_QUEPASACHATCONTROL=1003
      - C8Q_CHATWOOTPROFILEUPDATE=1004
      - C8Q_POSTTOWEBCALLBACK=1005
      - C8Q_POSTTOCHATWOOT=1006
      - C8Q_CHATWOOTTOQUEPASAGREETINGS=1007
      - C8Q_CW_PUBLIC_URL=${C8Q_CW_PUBLIC_URL}
      - C8Q_QP_DEFAULT_USER=${C8Q_QP_DEFAULT_USER}
      - C8Q_QP_BOTTITLE=${C8Q_QP_BOTTITLE}
      - C8Q_QP_CONTACT=${C8Q_QP_CONTACT}
      - EXECUTIONS_DATA_PRUNE=true
      - EXECUTIONS_DATA_MAX_AGE=168
      - EXECUTIONS_DATA_PRUNE_MAX_COUNT=5000
      - C8Q_CW_HOST=${C8Q_CW_HOST}
      - C8Q_N8N_HOST=${C8Q_N8N_HOST}
      - C8Q_QUEPASA_HOST=${C8Q_QUEPASA_HOST}

networks:
  ecosystem_network:
    external: true
    attachable: true
    name: ecosystem_network

volumes:
  n8n_cwmkt_volume:
    external: true
    name: n8n_cwmkt_volume
```

Depois clique em DEPLOY

![image](https://github.com/cwmkt/dockerquepasa/assets/91642837/bdc62781-993a-4d31-b8cd-5cd6466900f5)


Acesse: seudominio.com.<br>
Faça seu cadastro

**Pronto tudo Funcionando** ✅😎
