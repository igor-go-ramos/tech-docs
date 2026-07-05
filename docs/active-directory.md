## Pastas Compartilhadas
Para configurar o compartilhamento de pastas no Active Directory, por meio das ferramentas RSAT, é necessário criar uma política de grupo. Depedendo de como a instalação do Active Directory foi feita, vale verificar se já não existem outras políticas pré-configuradas e também se não há nenhum erro ao aplicar uma política em uma determinada OU. Para fazer a listagem das políticas aplicadas, podemos usar o comando `gpresult /r` e para confirmar se não houve uma falha ao tentar aplicar uma política de grupo, usamos o comando `gpupdate /force` que deve retornar sucesso (é comum aparecer uma falha ou outra, mas a atualização precisa ser concluída com êxito). Caso ocorra algum problema para ler alguma política, uma opção que pode ajudar é criar uma nova OU e adicionar novas políticas para ela.


Uma vez que a questão das políticas e OUs já foram verificadas, podemos partir para as configurações do compartilhamento, dentro da OU de usuários (onde as contas dos usuários são armazenadas), clique com o botão direito em cima da OU de usuários e crie uma GPO no domínio. Depois, selecione essa nova GPO clicando com o botão direito novamente e, nas configurações de usuário, vá em preferências e selecione Mapas de Unidade. Crie uma nova Unidade Mapeada da seguinte forma:

* **Ação:** Atualizar
* **Local:** \\\ip-ou-url-do-compartilhamento\nome-do-compartilhamento
* **Reconectar:** ✅
* **Letra da unidade:** escolha uma letra ou deixe automático

Na aba Comum, configure algumas flags adicionais:

* **Executar no contexto de segurança do usuário conectado (opção de política do usuário):** marcado

* **Direcionamento de nível de item:** opcionalmente marcado (configure o direcionamento especificando um grupo de segurança para restringir quem verá o compartilhamento por padrão)