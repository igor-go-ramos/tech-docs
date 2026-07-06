# :material-home-assistant: Home Assistant
## MQTT
### :material-message-outline: Tópicos
É possível adicionar tópicos no MQTT, por exemplo, para enviar alguma informação sobre um dispositivo (como a porcentagem da bateria). Porém, para que funcione, o serviço do MQTT precisa estar configurado para ouvir em uma porta específica - por padrão: `1883` - e no IP `0.0.0.0` (e não apenas `localhost` - `127.0.0.1`), pois a resposta vai ser enviada de volta para o dispositivo. Para tanto, o arquivo de configuração pode ser configurado desta maneira:

```conf title="mosquitto/config/mosquitto.conf"
password_file /mosquitto/config/pwfile
persistence true
persistence_location /mosquitto/data
listener 1883 0.0.0.0
protocol mqtt
allow_anonymous false
```

Para testar, podemos usar o comando:
```bash
mosquitto_pub -h <BROKER_IP> -u <USER> -P <PASS> \
  -t linux/mydevice/battery \
  -m "55"
```

Onde,
`BROKER_IP`: endereço IP do host que roda o MQTT
`USER`: usuário do MQTT
`PASS`: senha do MQTT
`linux/mydevice/battery`: tópico de publicação da mensagem para o MQTT
`“55”`: mensagem sendo publicada

No Home Assistant, para verificar se os dados estão sendo recebidos, podemos ir em *Settings > Devices & services > MQTT > Settings (engrenagem) > Listen to a topic* e nessa seção especificar o tópico, neste exemplo: `linux/#`

!!!note "Nota"
    Na dúvida sobre o usuário e senha do MQTT, é possível verificar no arquivo de configuração do `zigbee2mqtt` (`configuration.yaml`).

???tip "Exemplo: Script de Envio da Capacidade da Bateria"

    ```bash
    #!/bin/bash
    BAT=$(cat /sys/class/power_supply/BAT0/capacity)
    STATUS=$(cat /sys/class/power_supply/BAT0/status)
    MQTT_USER="mqttuser"
    MQTT_PASS="password"
    TOPIC="living_room/screen/"
    mosquitto_pub -h 192.168.0.50 -u mqttuser -P "$MQTT_PASS" -t "${TOPIC}battery" -m "$BAT"
    mosquitto_pub -h 192.168.0.50 -u "$MQTT_USER" -P "$MQTT_PASS" -t "${TOPIC}charging" -m "$STATUS"
    ```

### :material-lock: pwfile
Se estiver tendo problemas para rodar o container do MQTT por conta do arquivo de senha, é importante mudar o usuário e o grupo para o que representa o `mosquitto` (`1883`):

```bash
chown 1883:1883 mosquitto/config/pwfile
```

Lembrando que este diretório “`mosquitto/config`” geralmente é montado no container, então ele costuma ficar na máquina rodando o docker.

## :material-view-dashboard: Dashboard
A dashboard pode ser configurada em um equipamento com o Chromium instalado, neste caso, existe uma função de Kiosk no próprio browser que permite este tipo de setup. Se configurarmos o auto login e deixarmos o navegador como aplicativo de inicialização, conseguimos alcançar um esquema funcional para interação com o Home Assistant.

???tip "Exemplo: Script que Executa o Navegador Chromium em modo Kiosk"

    ```bash title="kiosk.sh"
    #!/bin/bash

    # Hide mouse cursor after inactivity
    unclutter &

    # Launch Chromium in kiosk mode
    chromium \
      --app=http://home-assistant-address:8123 \
      --start-fullscreen \
      --restore-last-session
      --password-store=basic
      --disable-pinch \
      --noerrdialogs \
      --overscroll-history-navigation=0 \
      --disable-session-crashed-bubble \
      --check-for-update-interval=31536000 \
      --touch-events=enabled \
      --enable-features=OverlayScrollbar
    ```

## :material-remote-desktop: Remote Access
Existem diferentes formas de acessar a interface web do Home Assistant remotamente. Por exemplo, com o uso de uma VPN (Tailscale é uma boa opção). Porém, uma das formas mais simples - possuindo um domínio registrado - é utilizar um serviço da Cloudflare que permite criar um túnel de acesso de forma simples e rápida. O link da documentação oficial que mostra um passo a passo é: https://developers.cloudflare.com/learning-paths/replace-vpn/connect-private-network/cloudflared/
Uma vez que o túnel foi criado, basta adicionar um container que receba as informações de conexão e o acesso já é liberado (incluindo um certificado TLS).

Exemplo: arquivo `.yaml` (pode ser incluído em conjunto com o `compose.yaml` do Home Assistant)

```yaml
cloudflared:
  container_name: cloudflared
  image: cloudflare/cloudflared
  restart: unless-stopped
  network_mode: host
  hostname: cloudflared
  command: "tunnel --no-autoupdate run --token token-do-túnel”
  entrypoint: "cloudflared --no-autoupdate"
```

Por conta do modo de acesso ser diferente do que teríamos em uma rede local, é interessante editar o arquivo de configuração do Home Assistant (`configuration.yaml`) adicionando as seguintes linhas:

```bash
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 192.168.1.0/24
  ip_ban_enabled: true
  login_attempts_threshold: 5
```