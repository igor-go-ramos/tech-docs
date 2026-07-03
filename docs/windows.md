# Windows
## Compartilhamento de arquivos
Em alguns momentos, o compartilhamento de arquivos pode parar de funcionar sem motivo aparente. Alguns comandos que podem ajudar são:

No servidor que armazena as pastas compartilhadas: # systemctl restart smbd.service

Na máquina do usuário que acessa o compartilhamento (terminal em modo administrador): net stop workstation. Depois: net start workstation 

Remover credenciais antigas pelo gerenciador de credenciais do Windows pode ajudar também.

No PowerShell (como admin), desativar a assinatura de segurança pode solucionar o problema em versões do Windows 11 e posteriores: Set-SmbClientConfiguration -RequireSecuritySignature $false