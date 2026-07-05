## Instalação
Primeiro precisamos iniciar o container:

```bash
docker run -p 7000:80 -itd binwiederhier/ntfy serve
```

Para verificar se a instalação foi bem sucedida, você pode acessar o serviço digitando: `ip-da-maquina:7000`, onde "ip-da-maquina" é o endereço IP onde o container está rodando.

O próximo passo é parar a execução do container e adicionar um arquivo de configuração:

```bash
docker stop id-do-container; mkdir /etc/ntfy; nano /etc/ntfy/server.yml
```

Dentro do arquivo, adicione o conteúdo do próprio Github do projeto, depois vá até o início e altere a seguinte linha que está comentada (descomente):

```bash
- # base url:

+ base url: http://ip-da-maquina
```

Depois de salvar o arquivo, execute o comando:

```bash
docker run -v /var/cache/ntfy:/var/cache/ntfy -v /etc/ntfy:/etc/ntfy -p 7000:80 -itd binwiederhier/ntfy serve --cache-file /var/cache/ntfy/cache.db
```

Agora o container do NTFY estará configurado para usar cache também, além de montar um volume na máquina hospedeira.

Com isso, a instalação foi concluída.

## Envio de mensagens
Para enviar uma mensagem, você pode usar o utilitário `curl`, conforme o exemplo:

```bash
curl -d "Hello, world!" localhost:7000/testing
```

No comando anterior, uma mensagem será enviada com o conteúdo "Hello, world!", para a máquina local, no tópico "testing", qualquer aplicação que esteja inscrita neste tópico e que tenha acesso à essa máquina irá receber esse texto.

Também é possível utilizar o próprio serviço do site ntfy.sh, da seguinte forma:

```bash
curl -d "Hello" ntfy.sh/topic
```

Deste modo, quem estiver inscrito no tópico "topic" irá receber a notificação, sem a necessidade de qualquer tipo de instalação de container. Porém é importante escolher o nome com sabedoria, pois é um serviço aberto.

## Exemplo Prático: Envio de Log de Backup (RClone)
```bash
#! /bin/bash

rclone sync /home/user/Documents/ drive:Backups/ -v --log-file /tmp/rclone-log.txt;

bkpLog=$(tail -n 6 /tmp/rclone-log.txt);

curl -d "Backup: $bkpLog" ntfy.sh/backup-topic
```