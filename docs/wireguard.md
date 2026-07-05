## Instalação
A instalação pode ser encontrada no site oficial, mas resumidamente, é comum encontrá-lo nas principais distribuições Linux, nos repositórios oficiais. Por exemplo:

Debian: `apt install wireguard`

Fedora: `dnf install wireguard-tools`

No Windows, basta acessar o site oficial, baixar e usar o `.exe`.

## Geração de Chaves
O par de chaves privada/pública deve ser gerado para cada peer que for ser conectado; posteriormente, essas chaves serão usadas no arquivo de configuração do WireGuard. Por exemplo:

```bash
wg genkey | tee privatekey | wg pubkey > publickey
```

O comando anterior gera dois arquivos: "privatekey" e "publickey", sendo que a chave pública usa o resultado do comando de geração da chave privada (que é a própria chave privada) para ser produzida.

## Exemplo de Configuração
Dentro de `/etc/wireguard` é possível criar um arquivo de configuração para estabelecer a conexão entre dispositivos (peers):

```conf title="/etc/wireguard/wg0.conf"
[Interface]
PrivateKey = <informe a chave privada aqui>
Address = <aqui vai o endereço deste dispositivo no túnel junto com a máscara (ex.: 10.20.30.40/32)>

[Peer]
PublicKey = <informe a chave pública aqui>
AllowedIPs = <aqui vão os IPs que serão roteados pela interface do WireGuard (wg0) (ex.: 192.168.1.0/24, 10.0.0.1/24), 0.0.0.0/0 faz com que todo o tráfego seja direcionado pela VPN>
Endpoint = <neste campo colocamos o endereço IP do outro dispositivo com o qual faremos a conexão (geralmente o IP público) e a porta (ex.: 123.210.012.111:51820). Também é possível usar um nome de domínio seguido da porta (ex.: meudns.com.br:51820)>
PersistentKeepalive = 20
PresharedKey = <chave compartilhada entre o servidor e o peer> (opcional)
```

Uma vez configurado, basta executar o arquivo por meio do comando: `wg-quick up wg0`

Para parar a execução, o comando é: `wg-quick down wg0`

Outras conexões podem ser definidas, por exemplo `/etc/wireguard/wg1.conf`.