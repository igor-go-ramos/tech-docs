Este trata-se de um compilado de informações úteis sobre o ecossistema UniFi. 

---
## :fontawesome-solid-download: Instalação
### :fontawesome-solid-clipboard-list: Pré-requisitos

Antes de iniciar, certifique-se de ter as seguintes dependências instaladas:

```bash
sudo apt update; sudo apt install ca-certificates wget -y
```

### :fontawesome-solid-list-check: Passo a Passo

=== ":fontawesome-solid-code: Script"
    Existem duas opções:

    Baixar uma versão específica ou a versão mais recente da controladora UniFi

    Se optar pela primeira:

    Baixe e excute a versão do script de instalação que você deseja, neste exemplo, a versão 8.0.28 está sendo baixada:

    ```bash
    wget https://get.glennr.nl/unifi/install/unifi-8.0.28.sh; bash unifi-8.0.28.sh
    ```

    Se optar pela segunda:

    ```bash
    rm unifi-latest.sh &> /dev/null; wget https://get.glennr.nl/unifi/install/install_latest/unifi-latest.sh && bash unifi-latest.sh
    ```

    A instalação irá iniciar, e você vai precisar responder, bastando confirmar e escolher o que será feito.

    Uma vez que a instalação for concluída, verifique se a controladora está efetivamente funcionando, para isso, digite o endereço ip da máquina em questão e a porta (`8443`) em um navegador (p. e.: `https://10.0.0.5:8443` ou `https://localhost:8443` se você está instalando diretamente na mesma máquina). Se abrir, a instalação foi concluída com sucesso e você pode fazer o setup inicial ou restaurar um backup.

=== ":fontawesome-brands-docker: Docker"

    **Passo 1 -** Crie um novo diretório em `/opt` chamado `unifi` e dentro deste diretório, crie o seguinte arquivo:

    ```yaml title="compose.yaml"
    services:
      app:
        image: lscr.io/linuxserver/unifi-network-application:latest
        restart: always
        depends_on:
          - db
        environment:
          PUID: 1000
          PGID: 1000
          TZ: America/Sao_Paulo
          # Config MongoDB
          MONGO_DBNAME: unifi
          MONGO_USER: unifi
          MONGO_PASS: senhadomongo
          MONGO_HOST: 127.0.0.1
          MONGO_PORT: 27017
        network_mode: host
        volumes:
          - ./app-data:/config

      db:
        image: mongo:7.0.8-jammy
        restart: unless-stopped
        environment:
          TZ: America/Sao_Paulo
        volumes:
          - ./db-data:/data/db
          - ./init-db.js:/docker-entrypoint-initdb.d/init.js:ro
        network_mode: host

      # updater:
      #   image: containrrr/watchtower
      #   hostname: unifi-updater
      #   restart: unless-stopped
      #   environment:
      #     - TZ=America/Sao_Paulo
      #     - WATCHTOWER_CLEANUP=true
      #   volumes:
      #     - /var/run/docker.sock:/var/run/docker.sock
    ```

    **Passo 2 -** Ainda no mesmo diretório, crie mais um arquivo chamado `init-db.js`, e dentro dele adicione o seguinte conteúdo:

    ```javascript title="init-db.js"
      db.getSiblingDB("unifi").createUser({
        user: "unifi",
        pwd: "senhadomongo",
        roles: [{ role: "dbOwner", db: "unifi" }],
      });
      db.getSiblingDB("unifi_stat").createUser({
        user: "unifi",
        pwd: "senhadomongo",
        roles: [{ role: "dbOwner", db: "unifi_stat" }],
      });
    ```

    **Passo 3 -** Execute o comando: `docker compose up -d`

    **Passo 4 -** O console do Unifi deve demorar aproximadamente 1 minuto para iniciar.

    **Passo 5 -** Acesse o console via `https://ip_da_maquina:8443`

    **Passo 6 -** [Opcional] Descomente as últimas linhas do arquivo `/opt/unifi/compose.yaml` para ativar o container que faz a atualização automática

=== ":fontawesome-brands-docker: Docker (servidores antigos)"
    
    **Passo 1 -** Crie um novo diretório em `/opt` chamado `unifi` e dentro deste diretório, crie o seguinte arquivo:

    ```yaml title="compose.yaml"
    services:
      unifi-network-application:
        image: lscr.io/linuxserver/unifi-network-application:latest
        depends_on:
          - unifi-db
        environment:
          - PUID=1000
          - PGID=1000
          - TZ=Etc/UTC
          - MONGO_USER=unifi
          - MONGO_PASS=mongopass
          - MONGO_HOST=127.0.0.1
          - MONGO_PORT=27017
          - MONGO_DBNAME=unifi
          - MEM_LIMIT=1024 #optional
          - MEM_STARTUP=1024 #optional
        volumes:
          - ./config:/config
        network_mode: host
        restart: unless-stopped

      unifi-db:
        image: docker.io/mongo:4.4
        environment:
          MONGO_INITDB_ROOT_USERNAME: root
          MONGO_INITDB_ROOT_PASSWORD: mongopass
          MONGO_DBNAME: unifi
          MONGO_USER: unifi
          MONGO_PASS: mongopass
        volumes:
          - ./data/db:/data/db
          - ./data/init-mongo.sh:/docker-entrypoint-initdb.d/init-mongo.sh:ro
        network_mode: host
        restart: unless-stopped
    ```

    **Passo 2 -** Dentro deste novo diretório `unifi`, crie outro chamado `data` e dentro dele adicione um arquivo chamado `init-mongo.sh`:

    ```bash title="/opt/unifi/data/init-mongo.sh"
    #!/bin/bash
    mongo <<EOF
    use admin
    db.auth("${MONGO_INITDB_ROOT_USERNAME}", "${MONGO_INITDB_ROOT_PASSWORD}")
    use ${MONGO_DBNAME}
    db.createUser({
      user: "${MONGO_USER}",
      pwd: "${MONGO_PASS}",
      roles: [
        { db: "${MONGO_DBNAME}", role: "dbOwner" },
        { db: "${MONGO_DBNAME}_stat", role: "dbOwner" }
        { db: "${MONGO_DBNAME}_audit", role: "dbOwner" }
      ]
    })
    EOF
    ```

    **Passo 3 -** Execute o comando: `docker compose up -d` (os logs da execução podem ser acessados com `docker compose logs`)
    
    **Passo 4 -** A interface web da controladora Unifi deve demorar aproximadamente 1 minuto para iniciar.
    
    **Passo 5 -** Acesse o console via `https://ip_da_maquina:8443`

### Pós instalação
Após a instalação, opcionalmente, você pode mudar o **inform host**, em razão da aplicação estar sendo executada dentro de um container e o endereçamento IP ser diferente do convencional, os dispositivos Unifi podem não conseguir se comunicar com a controladora. Portanto, nesta configuração, basta ir em Settings > System > Advanced e marcar override na opção de inform host, depois, adicionar o IP da máquina hospedeira e aplicar as alterações.

---

## :material-access-point: Redefinição de APs
Em alguns casos atípicos, em razão de algum problema ou configuração diferenciada, é necessário fazer a redefinição de APs via SSH. O comando utilizado é: `syswrapper.sh restore-default`.

Também é comum que seja necessário alterar o IP da controladora dos APs via SSH. O comando é: `set-inform http://ip-of-controller:8080/inform`. Quando o AP se situa em outra rede, por exemplo, separado por uma VLAN, este comando pode ser usado para que ele consiga se comunicar com a controladora, contanto que haja conectividade entre os dois.