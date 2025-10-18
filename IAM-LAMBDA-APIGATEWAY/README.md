# AWS IAM, Lambda e API Gateway 


## 1. Introdução ao IAM (Identity and Access Management)

O IAM é o coração da segurança na AWS. Ele permite controlar quem (usuários e serviços) pode fazer o quê (permissões) nos recursos da sua conta.

### 1.1. O Princípio do Menor Privilégio

A regra de ouro na AWS é **não usar o usuário root para tarefas diárias**. O usuário root tem acesso irrestrito a todos os serviços, incluindo as configurações de faturamento e o encerramento da conta.

Para as atividades cotidianas, a prática recomendada é criar usuários com permissões limitadas apenas ao necessário para executar suas funções.

### 1.2. Entendendo a Diferença entre Usuários e Funções (Roles)

-   **Usuários (Users):** Representam uma entidade que interage com a AWS, seja uma pessoa ou uma aplicação. Eles possuem credenciais de longo prazo, como senha para acesso ao console ou chaves de acesso (`Access Key`) para interações via API.

-   **Funções (Roles):** Não possuem credenciais próprias. São um conjunto de permissões que podem ser "assumidas" temporariamente por uma entidade confiável (um usuário, uma aplicação ou um serviço da AWS como EC2 ou Lambda).
    -   **Exemplo prático:** Uma instância EC2 assume uma *Role* que lhe permite acessar e gerenciar outras instâncias EC2, sem precisar armazenar credenciais de acesso permanentes.

## 2. Criando e Gerenciando Usuários no IAM

Vamos colocar em prática a criação de usuários com diferentes níveis de acesso.

### 2.1. Criando um Usuário Administrador com Acesso ao Faturamento

1.  No console da AWS, navegue até o serviço **IAM**.
2.  No menu lateral, selecione **Usuários** (Users) e clique em **Criar usuário** (Create user).
3.  **Nome do usuário:** Defina um nome, como `admin-principal`.
4.  Marque a opção **Fornecer ao usuário acesso ao Console de Gerenciamento da AWS** (Provide user access to the AWS Management Console).
5.  Selecione **Quero criar um usuário do IAM** (I want to create an IAM user) e defina uma senha.
6.  Na tela de permissões, escolha **Anexar políticas diretamente** (Attach policies directly).
7.  Procure e anexe as seguintes políticas (policies):
    -   `AdministratorAccess`: Concede acesso total a todos os recursos da AWS.
    -   `Billing`: Permite visualizar informações de faturamento.
8.  Revise e conclua a criação do usuário.

> 🔑 **Acesso ao Billing:** Por padrão, usuários IAM não podem ver a área de Faturamento. Para habilitar, você deve fazer login com o **usuário root**, ir em **Conta** (Account) (no canto superior direito) e editar a seção **Acesso de usuário e perfil do IAM às informações de faturamento** (IAM User and Role Access to Billing Information) para permitir o acesso. Após isso, acesse com o novo usuário em uma **aba anônima** para verificar.

### 2.2. Criando um Usuário com Permissões Limitadas (Lambda)

1.  Crie um novo usuário (ex: `dev-lambda`).
2.  Anexe a política gerenciada pela AWS chamada `AWSLambda_FullAccess`.
3.  **Teste prático:** Tente criar uma nova função Lambda com este usuário. Você encontrará um erro de permissão, como `iam:CreateRole`.
    -   **Motivo:** O serviço Lambda precisa criar uma "Função de Execução" (Execution Role) para poder interagir com outros serviços (como o CloudWatch para logs). A política `AWSLambda_FullAccess` não concede permissão para criar *Roles* no IAM.
    -   **Solução:** Seria necessário criar e anexar uma política adicional que permita a ação `iam:CreateRole`, seguindo o princípio de conceder permissões apenas conforme a necessidade.

## 3. Trabalhando com AWS Lambda

AWS Lambda permite executar código em resposta a eventos, sem a necessidade de provisionar ou gerenciar servidores.

1.  **Criar a Função:** No console do Lambda, crie uma nova função.
2.  **Testar o Código:** Teste a função com o **código padrão** (template) fornecido pela AWS e, em seguida, altere para um **código customizado** para atender a um requisito específico.
3.  **Habilitar URL da Função:** Para um acesso rápido e direto, vá em **Configuração** (Configuration) -> **URL da função** (Function URL) e crie uma URL.

## 4. Integrando Lambda com API Gateway

O API Gateway é um serviço mais robusto para criar, gerenciar e proteger APIs em escala. Vamos criar uma API REST que se integra com nossa função Lambda e possui dois métodos com diferentes tipos de autorização.

1.  **Criar o Gatilho:** Na página da sua função Lambda, vá em **Adicionar gatilho** (Add trigger) e selecione **API Gateway**. Crie uma nova **API REST** (REST API).

### 4.1. Método 1: Acesso via Chave de API (API Key)

Este método será acessível por qualquer cliente que apresente uma chave de API válida no cabeçalho da requisição.

1.  **Configurar o Método:**
    -   No console do API Gateway, acesse sua API recém-criada.
    -   Em **Recursos** (Resources), selecione o método (ex: `ANY` ou crie um `GET`) e vá em **Solicitação de método** (Method Request).
    -   Defina **Autorização** (Authorization) como `Nenhum` (None).
    -   Defina **Chave de API necessária** (API Key Required) como `true`.
    -   Em **Validador de solicitações** (Request Validator), configure a validação do corpo, parâmetros e cabeçalhos.
2.  **Criar Chave de API:**
    -   No menu lateral do API Gateway, vá em **Chaves de API** (API Keys) e crie uma nova chave. Anote o valor.
3.  **Criar Plano de Uso (Usage Plan):**
    -   Vá em **Planos de uso** (Usage Plans), crie um novo plano.
    -   **Associe a API:** Adicione seu API e o *stage* (estágio, ex: `prod`) ao plano.
    -   **Associe a Chave:** Adicione a chave de API criada no passo anterior a este plano.
4.  **Executar no Postman:**
    -   Faça uma requisição para a URL do seu método.
    -   Na aba **Cabeçalhos** (Headers), adicione o header `x-api-key` com o valor da chave que você gerou.

### 4.2. Método 2: Acesso via Autenticação IAM

Este método exigirá que a requisição seja assinada digitalmente com as credenciais de um usuário IAM autorizado.

1.  **Criar um Novo Método:**
    -   Em **Recursos** (Resources), crie um novo recurso (ex: `/private`) e um novo método dentro dele (ex: `GET`).
    -   Integre este método com a mesma função Lambda.
    -   Em **Solicitação de método** (Method Request), defina **Autorização** (Authorization) como `AWS_IAM`.
2.  **Criar Usuário Programático:**
    -   No IAM, crie um usuário **sem acesso ao console** (ex: `api-invoker`).
    -   Anexe a política `AmazonAPIGatewayInvokeFullAccess`.
    -   Salve o **ID da chave de acesso** (Access Key ID) e a **Chave de acesso secreta** (Secret Access Key).
3.  **Executar no Postman:**
    -   Use a URL do novo método (`/private`).
    -   Na aba **Autorização** (Authorization), selecione o tipo **Assinatura da AWS** (AWS Signature).
    -   Preencha os campos:
        -   `AccessKey`: Seu ID da chave de acesso.
        -   `SecretKey`: Sua Chave de acesso secreta.
        -   `AWS Region`: A região da sua API (ex: `us-east-2`).
        -   `Service Name`: `execute-api`.
    -   Envie a requisição. O Postman irá assiná-la corretamente para a autenticação IAM.