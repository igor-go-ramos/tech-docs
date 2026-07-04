## BTRFS
Alguns comandos úteis para verificar a integridade do sistema de arquivos BTRFS são:

Verificação em busca erros (/dev/sda pode ser substituído pelo volume):

```bash
btrfs scrub start -Bd /dev/sda
```

Checagem do status da última verificação (pode ser executado em paralelo ao comando anterior):

```bash
btrfs scrub status /dev/sda
```

Busca de logs:

```bash
journalctl --output=cat --grep='BTRFS.* i/o error' | sort | uniq | less
```

!!!tip "Dica"
    Para conferir os detalhes dos arquivos corrompidos (se houverem) o comando `dmesg` pode ser usado.

!!!note "Nota"
    As informações obtidas dependem da execução do scrub, é através dessa operação que o sistema de arquivos é analisado.

## VLANS
Para fazer o teste de VLANs no Linux, com o NetworkManager, existem alguns comandos úteis:

```bash
nmcli connection add type vlan con-name vlan20-test dev enp2s0 id 20
```

```bash
nmcli connection up vlan20-test
```

Os comandos acima criam uma VLAN chamada vlan20-test em cima da interface "enp2s0" e ativam a conexão. Essa VLAN é criada com a tag 20.

Para deletar essa nova VLAN, basta executar:

```bash
nmcli connection delete vlan20-test
```

Vale destacar que geralmente também é possível fazer isso pela interface gráfica do sistema.

Outro ponto de atenção na hora dos testes é que a VLAN é uma interface virtual e é adicionada em cima da interface física, logo, se existe uma outra conexão ativa, a interface física possuirá dois IPs atrelados a ela. Isso pode ser alterado se a outra conexão for desativada.

## Fixação de IPs - Debian
Existem diferentes formas de fixar o endereço de IP no Debian. Uma delas se dá utilizando o systemd-networkd:

Crie o arquivo de configuração e adicione o seguinte conteúdo:

```bash
nano /etc/systemd/network/10-static.network
```
```bash title="10-static.network"
[Match]

Name=eth0

[Network]

Address=192.168.0.2/24

Gateway=192.168.0.1

DNS=8.8.4.4 1.1.1.1
```

!!!note "Observação"
    altere o endereço de IP da interface e do gateway conforme sua rede e não se esqueça de alterar a interface caso necessário (`eth0`)

Depois, remova a configuração de DHCP padrão (caso o IP não esteja fixado no servidor DHCP) comentando a linha que ativa a atribuição automática:

```bash
nano /etc/network/interfaces
```
```bash title="interfaces"
#iface eth0 inet dhcp # conteúdo do arquivo
```

Obs.: `eth0` - é o nome da interface que pode ser diferente dependendo da máquina

Reinicie o serviço:

```bash
systemctl restart systemd-networkd
```
Por fim, certifique-se de que o serviço está habilitado:

```bash
systemctl enable systemd-networkd
```

## PDFs
### Como juntar dois ou mais arquivos em PDF?
O utilitário **pdfunite** pode ser usado para a junção de arquivos em PDF. Geralmente ele já vem instalado no Linux.

Exemplo de uso para junção de todos os arquivos do diretório atual em um único "`arquivo-mesclado.pdf`":

```bash
pdfunite *.pdf arquivo-mesclado.pdf
```

### Como comprimir arquivos PDF?
Alguns arquivos no formato PDF podem ser extremamente pesados, o que dificulta o processo de impressão, portanto o comando `gs` (também é comum de vir instalado no Linux) pode ser usado para fazer uma compressão do arquivo, por exemplo:

```bash
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS=/prepress -dNOPAUSE -dQUIET -dBATCH -sOutputFile=arquivo-comprimido.pdf arquivo-original.pdf
```