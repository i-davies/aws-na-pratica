# Criação de uma Instância EC2

Este guia passo a passo ajudará os alunos a lançar sua primeira instância EC2 e a se conectar a ela.

## 1. Acessar o Console do EC2

1. Faça login no Console de Gerenciamento da AWS.
2. No campo de busca, digite "EC2" e selecione o serviço.
3. No painel do EC2, clique em "Launch instance" (Lançar instância).

## 2. Escolher uma Amazon Machine Image (AMI)

1. Dê um nome à sua instância, por exemplo, "Meu-Primeiro-Servidor".
2. Na seção "Application and OS Images (Amazon Machine Image)", selecione uma AMI. Para iniciantes, uma boa escolha é o **Amazon Linux 2** ou **Ubuntu Server**, que estão no "Free tier eligible" (qualificado para o nível gratuito).

## 3. Escolher um Tipo de Instância

1. Na seção "Instance type", selecione `t2.micro` ou `t3.micro`. Ambos são qualificados para o nível gratuito e suficientes para este tutorial.

## 4. Configurar um Par de Chaves (Key Pair)

1. Este é um passo crucial para a segurança do acesso. Na seção "Key pair (login)", clique em "Create new key pair".
2. Dê um nome ao seu par de chaves, por exemplo, "minha-chave-ssh".
3. Selecione o formato de arquivo de chave privada: **.pem** para usuários de macOS e Linux, ou **.ppk** para usuários de PuTTY no Windows.
4. Clique em "Create key pair". O arquivo da chave privada será baixado automaticamente. **Guarde este arquivo em um local seguro, pois você não poderá baixá-lo novamente.**

## 5. Configurar as Definições de Rede

1. Na seção "Network settings", clique em "Edit".
2. Para esta atividade inicial, você pode manter a VPC e a sub-rede padrão.
3. Deixe as configurações de armazenamento padrão.

## 6. Lançar a Instância

1. Revise o resumo das configurações no painel à direita.
2. Clique em "Launch instance".

## 7. Conectar-se à Instância via SSH

Após alguns minutos, sua instância estará no estado "Running".

1. Selecione a instância na lista e anote o seu **Public IPv4 address** (Endereço IPv4 Público).

### 7.1. Para macOS e Linux (usando o terminal):

1. Abra o terminal.
2. Navegue até o diretório onde você salvou o arquivo `.pem`:

   ```bash
   cd /caminho/para/sua/chave/
   ```

3. Altere as permissões do arquivo da chave para que apenas você possa lê-lo (isso é uma exigência do SSH):

   ```bash
   chmod 400 minha-chave-ssh.pem
   ```

4. Conecte-se à instância usando o seguinte comando, substituindo o endereço IP e o nome de usuário (para Amazon Linux é `ec2-user`, para Ubuntu é `ubuntu`):

   ```bash
   ssh -i "minha-chave-ssh.pem" ec2-user@SEU_ENDERECO_IP_PUBLICO
   ```

5. Quando solicitado, digite `yes` para confirmar a autenticidade do host.

### 7.2. Para Windows (usando PuTTY):

1. Se você baixou um arquivo `.ppk`, pode pular para o passo 3. Se baixou um `.pem`, precisa convertê-lo usando o PuTTYgen.
2. Abra o PuTTYgen, clique em "Load", selecione seu arquivo `.pem` e depois "Save private key".
3. Abra o PuTTY.
4. Em "Host Name (or IP address)", insira o endereço IP público da sua instância.
5. No menu à esquerda, vá para `Connection > SSH > Auth`.
6. Clique em "Browse..." ao lado de "Private key file for authentication" e selecione o seu arquivo `.ppk`.
7. Clique em "Open". Quando o terminal abrir, faça login com o nome de usuário apropriado (`ec2-user` ou `ubuntu`).

### 7.3. Para Acesso via Console da AWS (EC2 Instance Connect):

1. No Console do EC2, selecione sua instância.
2. Clique em "Connect" (Conectar).
3. Selecione "EC2 Instance Connect" e clique em "Connect" (Conectar).
4. Uma nova janela do navegador se abrirá com um terminal baseado na web, conectando você diretamente à instância sem precisar de chaves SSH ou ferramentas externas.
5. Faça login com o nome de usuário padrão (ex: ec2-user para Amazon Linux ou ubuntu para Ubuntu).
