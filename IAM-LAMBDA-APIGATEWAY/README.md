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

O API Gateway permite criar, gerenciar e proteger APIs em escala. Em 2025, o console foi atualizado e o fluxo recomendado para usar chaves de API e planos de uso continua sendo a criação de uma **REST API** (não confundir com **HTTP API**). HTTP APIs são mais simples/rápidas, mas não oferecem API Keys/Usage Plans.

1.  **Criar a REST API (novo console):**
    -   Acesse o console: API Gateway → APIs → REST APIs → **Create API** → **New API**.
    -   Defina um nome (ex.: `lambda-demo-api`) e mantenha o tipo de endpoint como **Regional**. Clique em **Create**.
2.  **Criar recurso e método:**
    -   Em **Resources**, clique em **Create Resource** (ex.: nome: `private`, path: `/private`).
    -   Selecione o recurso criado e clique em **Create Method** (ex.: `GET`).
3.  **Anexar integração Lambda:**
    -   Em **Integration type**, selecione **Lambda Function**.
    -   Escolha a região e a sua função Lambda. Deixe habilitado o proxy (Lambda proxy integration) quando disponível.
    -   Salve. Se solicitado, permita que o API Gateway invoque a função (permissão `lambda:InvokeFunction`).

    - Configurações de solicitação de método -> clique em `editar`
    - Autorização: selecione `Nenhuma`
    - Validador de solicitação: selecione `Validar parâmetros de string de consulta e cabeçalhos`
    - Marque a opção `Chaves de API obrigatória`
    - Salve as alterações.
    - Clique em implantar API (Deploy) para aplicar as mudanças, selecione o estágio (ex.: `prod`).

4.  **Implantar a API (Deploy):**
    -   Vá em **Implantar API** → **Novo estágio** → nome do estágio (ex.: `prod`) → **Deploy**.
    -   Copie a **Invoke URL** do estágio (formato `https://{apiId}.execute-api.{regiao}.amazonaws.com/prod`).

### 4.1. Método 1: Acesso via Chave de API (API Key)

Este método exige que o cliente envie um cabeçalho `x-api-key` válido. Disponível apenas em REST APIs com Usage Plans.

1.  **Marcar o método como “API Key Required”:**
    -   Em **Resources**, selecione o método (ex.: `GET`) → **Method Request** (Solicitação de método).
    -   Defina **Authorization** como `NONE` (Nenhum) e **API Key Required** como `true`. Salve.
    -   Reimplante a API em **Stages** (qualquer mudança de método requer novo deploy).
2.  **Criar a Chave de API:**
    -   Em **API Keys** → **Create API key** → gere a chave. Anote o valor.
3.  **Criar o Plano de Uso (Usage Plan):**
    -   Em **Usage Plans** → **Create** → defina limites (opcional).
    -   **Add API Stage:** associe sua API e o estágio (ex.: `prod`).
    -   **Add API Key:** vincule a chave criada ao plano de uso.

5.  **Testar:**
    -   Faça uma requisição para a Invoke URL do método.
    -   Inclua o cabeçalho `x-api-key: <SUA_CHAVE>`. Sem esse cabeçalho, o API Gateway retorna `403 Forbidden`.

### 4.2. Método 2: Acesso via Autenticação IAM

Neste método, as chamadas devem ser assinadas com SigV4 usando credenciais AWS válidas (usuário/role com permissão de invocar a API).

1.  **Configurar o método para IAM:**
    -   Em **Resources**, selecione o método → **Method Request**.
    -   Defina **Authorization** como `AWS_IAM`. Salve e faça deploy novamente do estágio.
2.  **Criar credenciais programáticas e permissões mínimas:**
    -   No IAM, crie um usuário programático (ex.: `api-invoker`) ou use uma role.
    -   Conceda a permissão de invocação (exemplo de política mínima):
        ```json
        AmazonAPIGatewayInvokeFullAccess
        ```
    - Crie Chaves de acesso em Credenciais de segurança.
    - Selecione a opção `Outros
Seu caso de uso não está listado aqui.`
    - Anote a Chave de acesso e a Chave secreta.
3.  **Assinar a requisição (Postman ou código):**
    -   No Postman, escolha **Authorization: AWS Signature** e informe `AccessKey`, `SecretKey`, `AWS Region` (ex.: `us-east-1`) e `Service Name` = `execute-api`.
    -   Envie a requisição para a Invoke URL. Sem assinatura correta, o retorno será `403`.