# Publicando sua API FastAPI na Nuvem AWS (EC2)

**Tecnologias que vamos usar:**

- **AWS EC2:** Nosso servidor virtual na nuvem.
    
- **Python 3:** A versão padrão que vem com o servidor.
    
- **FastAPI:** O framework web da nossa API.
    
- **Git:** Para baixar o código da aplicação.
    

## 1. Pré-requisitos

1. Uma conta na AWS.
    
2. Uma instância EC2 (servidor) já criada (usando Amazon Linux 2).
    
3. O arquivo de chave `.pem` para acessar sua instância.
    

---

## 2. Conexão e Preparação da Instância EC2

O primeiro passo é conectar ao nosso servidor e instalar as ferramentas essenciais para nosso projeto.

1. Conecte-se à sua instância via SSH:
    
    Abra seu terminal e use o comando ssh. O usuário padrão do Amazon Linux é ec2-user.
    
    ```bash
    # Substitua o caminho para sua chave e o IP da sua instância
    ssh -i /caminho/para/sua/chave.pem ec2-user@SEU_ENDERECO_IP_PUBLICO
    ```
    
2. Atualize o Sistema e Instale as Ferramentas:
    
    Vamos garantir que o sistema está atualizado e instalar o git (para baixar nosso código) e o pip (para instalar as dependências do Python).
    
    ```bash
    # Atualiza todos os pacotes existentes
    sudo yum update -y
    
    # Instala o Git e o gerenciador de pacotes do Python (pip)
    sudo yum install git python3-pip -y
    ```
    
    > O `python3` já vem instalado na imagem do Amazon Linux, então só precisamos do `pip` para gerenciar as bibliotecas.
    

---

## 3. Baixando e Configurando o Projeto da API

Com o ambiente preparado, vamos baixar o código da API e configurar suas dependências em um ambiente isolado.

1. **Clone o repositório do GitHub:**
    
    ```bash
    git clone https://github.com/i-davies/fastapi-example.git
    ```
    
2. **Navegue para a pasta do projeto:**
    
    ```bash
    cd fastapi-example
    ```
    
3. Crie um Ambiente Virtual (venv):
    
    É uma prática essencial criar um ambiente isolado para as dependências de cada projeto. Este comando usará o python3 padrão do sistema.
    
    ```bash
    python3 -m venv venv
    ```
    
4. **Ative o Ambiente Virtual:**
    
    ```bash
    source venv/bin/activate
    ```
    
    > Você notará que `(venv)` aparece no início do seu terminal. Isso indica que o ambiente está ativo!
    
5. Instale as dependências do projeto:
    
    O arquivo requirements.txt contém a lista de todas as bibliotecas que o projeto precisa (FastAPI, Uvicorn, etc.). O pip irá baixar e instalar todas elas.
    
    ```bash
    pip install -r requirements.txt
    ```
    

---

## 4. Executando a Aplicação com Uvicorn

Tudo está instalado e configurado. Agora, vamos iniciar o servidor!

1. **Execute o servidor Fastapi:**
    
    ```bash
    fastapi run
    ```
    

---

## 5. Liberando o Acesso Externo (AWS Security Group)

Nossa API está rodando, mas a AWS possui um firewall, chamado **Security Group**, que bloqueia conexões externas por padrão. Precisamos criar uma regra para liberar o acesso à porta 8000.

1. No painel da **AWS**, navegue até sua instância **EC2**.
    
2. Selecione a instância e clique na aba **"Segurança"**.
    
3. Clique no nome do **"Grupo de segurança"** associado a ela.
    
4. Na página do grupo de segurança, clique em **"Editar regras de entrada"**.
    
5. Clique em **"Adicionar regra"** e preencha os campos:
    
    - **Tipo:** `TCP personalizado`
        
    - **Intervalo de portas:** `8000`
        
    - **Origem:** `Qualquer lugar-IPv4` (que preencherá com `0.0.0.0/0`).
        
6. Clique em **"Salvar regras"**.
    

Esta regra diz para a AWS: "Qualquer pessoa (`0.0.0.0/0`) pode tentar se conectar a esta máquina na porta `8000`."

---

## 6. Acessando sua API no Navegador!

Este é o momento da verdade!

1. **Encontre o IP Público** da sua instância EC2 no painel da AWS (como visto na Parte 1).
    
2. Abra uma nova aba no seu navegador e digite o endereço da sua API:
    
    ```
    http://SEU_IP_PUBLICO:8000
    ```
    
3. **Explore a documentação interativa automática**, uma das melhores funcionalidades do FastAPI:
    
    ```
    http://SEU_IP_PUBLICO:8000/docs
    ```
    
    Nesta página, você pode ver todos os _endpoints_ da sua API e até mesmo testá-los diretamente do navegador.
