O Samba AD DC é um serviço que se assemelha com o Active Directory da Microsoft.

Neste [link](https://youtu.be/KT6O-TfJ41g?si=R7DPS4_01D_pCXVs), temos um tutorial que mostra a configuração do Samba como Domain Controller no Ubuntu. No Debian, a configuração é bem similar.

Alguns exemplos de comandos para gerenciamento do Samba são:

* Adicionar novo usuário

```bash
samba-tool user add username password@123 --must-change-at-next-login --userou="ou=users,ou=seesaw"
```

* Remover usuário

```bash
samba-tool user delete username
```

* Alterar senha de usuário

```bash
samba-tool user setpassword username
```

* Alterar senha de múltiplos usuários

```bash
for user in username1 username2 username3 ; samba-tool user setpassword $user --newpassword=newpassword@123 --must-change-at-next-login
```

* Adicionar múltiplos usuários à um grupo

```bash
for user in username1 username2 username3 ; samba-tool group addmembers groupname $user
```

* Adicionar novo computador no domínio (PowerShell)

```powershell
Add-Computer -DomainName "dominio.lan" -Restart
```

* Mostrar configurações de senhas

```bash
samba-tool domain passwordsettings show
```

* Mudar a política padrão de expiração de senha para nunca expirar

```bash
samba-tool domain passwordsettings set --max-pwd-age=0
```
---
## :material-folder-arrow-up-down: Compartilhamento de Arquivos
As permissões devem ser configuradas de modo que os usuários do domínio consigam acessar os arquivos.

Primeiro, é interessante alterar as permissões globais:

```bash
chmod 775 -R /data
```

```bash
chown -R root:"domain users" /data
```

Obs.: "`-R`"  significa que o comando irá ser executado de forma recursiva. "`/data`" é a pasta de exemplo onde se encontram os dados.

Depois, as configurações mais específicas de permissões podem ser feitas por algum computador Windows, tudo pelo próprio compartilhamento de arquivos do Samba, através da seleção do diretório e modificação de suas propriedades.