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

## Particionamento (RAID)
Particionamento (RAID)

O Particionamento via RAID é interessante para aumentar a confiabilidade do sistema em caso de falha nos dispositivos de armazenamento. Existem diferentes formas de configurar o RAID, a mais simples é via md.

Para fazer a configuração usando o md é possível usar a interface ou a linha de comando. O objetivo é juntar os discos e, no cenário mais comum de dois HDs/SSDs, ativar um RAID 1 entre eles. Após a configuração em RAID, usamos o LVM para criar partições adicionais no espaço disponível das unidades de armazenamento. Por fim, seguimos com a 	instalação em cima do LVM que está sobre o RAID.

Tanto para UEFI quanto para BIOS Legacy, é importante lembrar de criar partições de inicialização para instalação do Grub. A diferença geralmente é só o tamanho da partição e a forma como ela é instalada. No caso de UEFI, deixamos um 1 GB, já para BIOS Legacy, bastam alguns megas.

É interessante destacar também que no momento de particionamento, devemos evitar a configuração de duas partições do tipo UEFI, pois isso pode causar problema na instalação do Grub (como tem duas partições que podem ser usadas é confuso). Além disso, as partições UEFI não devem fazer parte do RAID.

Por fim, podemos copiar a partição EFI de um dos discos para o outro.

Link útil: https://youtu.be/ykzR3i-pToY?si=CAdgDheA8SQuNLev

No vídeo, a configuração é feita no formato de BIOS Legacy, porém esse procedimento pode ser adaptado para UEFI, simplesmente precisaríamos mudar o tipo de partição para EFI e aumentar o tamanho para 1 GB (+1G). Importante notar que não usaríamos o comando dpkg-reconfigure e sim, faríamos a cópia da partição /boot/efi para a mesma partição EFI do outro HD/SSD. Os passos detalhados para isso seriam:

✅ Passos detalhados

1 - Monte ambas as partições EFI:

```bash
sudo mkdir -p /mnt/efi-sda
sudo mkdir -p /mnt/efi-sdb
sudo mount /dev/sda1 /mnt/efi-sda
sudo mount /dev/sdb1 /mnt/efi-sdb
```

2 - Copie o conteúdo da partição EFI principal para a secundária:

```bash
sudo cp -r /mnt/efi-sda/* /mnt/efi-sdb/
```

Isso copiará as pastas como EFI/debian, EFI/BOOT, etc., para que o segundo disco também tenha um ambiente de boot funcional.

3 - Instale o GRUB apontando para o segundo disco:
Agora que /dev/sdb1 tem o conteúdo correto, diga ao GRUB para instalar o bootloader EFI nele:

```bash
sudo grub-install --target=x86_64-efi --efi-directory=/mnt/efi-sdb --bootloader-id=debian --recheck --no-nvram /dev/sdb
```

🔧 Importante: O `--no-nvram` evita sobrescrever a ordem de boot no firmware, e o `--bootloader-id` pode ser mantido como Debian se quiser que o nome apareça igual na BIOS/UEFI.

4 - Atualize o GRUB

Mesmo que os arquivos já estejam sincronizados, é bom garantir que tudo esteja em ordem: `sudo update-grub`