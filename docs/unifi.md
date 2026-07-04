Este trata-se de um compilado de informações úteis sobre o ecossistema UniFi. 

---
## Instalação
### 📋 Pré-requisitos

Antes de iniciar, certifique-se de ter as seguintes dependências instaladas:

```bash
sudo apt update; sudo apt install ca-certificates wget -y
```

### 💻 Passo a Passo

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

---

## Redefinição de APs
Em alguns casos atípicos, em razão de algum problema ou configuração diferenciada, é necessário fazer a redefinição de APs via SSH. O comando utilizado é: `syswrapper.sh restore-default`.

Também é comum que seja necessário alterar o IP da controladora dos APs via SSH. O comando é: `set-inform http://ip-of-controller:8080/inform`. Quando o AP se situa em outra rede, por exemplo, separado por uma VLAN, este comando pode ser usado para que ele consiga se comunicar com a controladora, contanto que haja conectividade entre os dois.