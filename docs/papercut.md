Instalação do Cliente de Impressão
Abra a URL de instalação (p. e.: 10.10.14.2:9163/setup), faça o download e instale.

A instalação requer a inserção do e-mail e da senha do Google.

Após a instalação, você pode confirmar se a impressora foi configurada corretamente se ela estiver aparecendo na listagem de dispositivos.

Para Chromebooks, baixe e abra a extensão Mobility Print e faça login para liberar a opção de imprimir.

Setup de Rede
Como o servidor do Papercut é compartilhado, precisamos ter faixas de rede distintas para que a conexão entre as unidades não conflite. Isso se aplica tanto para os clientes como para as impressoras, pois ambos precisam conversar com o software de impressão.

Outro aspecto importante que vale ressaltar é uma particularidade do Papercut que precisa ser considerada na hora de fazer a configuração de rede. O IP configurado inicialmente é mantido em todas as URLs geradas pelo servidor, então é necessário adicionar redirecionamentos apontando para o IP do Papercut na rede local de cada unidade. Por exemplo, se o Papercut foi instalado inicialmente em uma VPN configurada na faixa de IP 10.10.14.0/24, com IP 10.10.14.2, precisa haver uma regra NAT que troque o IP do Papercut para o IP configurado na VPN local (ex.: 10.10.16.2 -> 10.10.14.2).

Instalação nas Impressoras
Obs.: em caso de dúvida, é possível abrir a interface de uma impressora já configurada para conferir os detalhes e fazer comparações.

Passo 1 - acessar o site de registro da impressora e informar o número de série junto com as informações de região e o software utilizado (Papercut MF). Link: https://openplatform.epson.biz/license-op/inputInformation.html

Passo 2 - na impressora, entrar na seção "Epson Open Platform" e preencher o campo Chave do produto com a informação do site anterior e prosseguir.

Passo 3 - após inserir a chave, ainda na mesma seção, abra a opção Sistema de Autenticação > Básico e insira dois links (lembre-se de trocar o "papercut-mf-IP" pelo IP do Papercut: 

https://<papercut-mf-IP>:11260/epson-ops-papercut -> URL da página Web antes do Início de sessão

https://<papercut-mf-IP>:11260/epson-ops-papercut-notification-sfp-configure -> URL de notificação

Passo 5 - ainda na mesma área, ative a Gestão de quota e por fim, dê OK.

Passo 6 - acesse a URL: https://<papercut-mf-IP>:11260/epson-ops-papercut

Passo 7 - verifique na interface web do Papercut, na área de Devices, se o dispositivo já apareceu na listagem, caso não, ative a impressora manualmente navegando na interface física (tocar na tela dela força uma atualização). Se ainda assim não houver comunicação, verifique se não existe algum bloqueio de firewall.

Passo 8 - selecione a impressora na lista de dispositivos e marque "Identity number", "Mask identity number", "Track & control scanning", "Enable print release", "Enable Find-Me printing support", dê um check na fila virtual (que concentra todas as outras) e por último na região onde tem os radio buttons, marque a fila da impressora que está sendo configurada. Se uma mensagem de alerta aparecer, certifique-se de que a fila virtual está agregando as outras (Printers > Printer List > Fila Virtual > Job Redirection Settings).

Passo 9 - ainda na mesma área de configuração da impressora, abra a seção de Configurações avançadas ("Advanced Config") e atualize o campo "ext-device.epson.admin-password" com a senha da interface web da impressora sendo configurada (geralmente é o número de série).