# :material-google: Google Workspace

## :material-code-braces: Apps Script
A ferramenta Apps Script do Google pode ser usada para automatizar muitas tarefas cotidianas.

Alguns exemplos de uso:

* Ajuste informações de Chromebooks
* Busca de e-mails
* Automatização de planilhas

!!!note "Nota"
    Para alguns projetos, dependendo do objetivo, podemos usar serviços do Google para obter recursos mais avançados.

???tip "Exemplo prático: busca de todos os e-mails de um domínio"
    
    ```javascript
    // Neste exemplo, AdminDirectory é um serviço provido pela Admin SDK API do Google
    const domains = [
      'example.com',
      'another.example.com'
    ];

    const my_customer = getCustomerId('admin@example.com')
    function getEmails() {
      let emails = [], email, pageToken, page, count = 1;
      domains.forEach(domain => {
        do {
          page = AdminDirectory.Users.list({
            orgUnitPath: '/',
            includeChildOrgunits: true,
            customer: my_customer,
            domain,
            pageToken: pageToken,
            maxResults: 50,
            orderBy: 'givenName',
            query: "isSuspended=false"
          });
          
          const users = page.users;
          if (!users) {
            Logger.log('No users found.');
            return;
          }

          users.forEach(user => {
            email = user.primaryEmail;
            emails.push(email);
          });

          console.log(emails);

          count += users.length;
          emails = [];

          pageToken = page.nextPageToken;
        } while (pageToken);
      });
    }

    function getCustomerId(adminEmail) {
      const { customerId } = AdminDirectory.Users.get(adminEmail);
      return customerId;
    }
    ```