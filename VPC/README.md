## # Criando a VPC

## 1. Criando a VPC e as Subnets com o Assistente


### 1.1 Navegue até o Painel da VPC

- No Console da AWS, procure e vá para o serviço **VPC**.
- No painel da VPC, clique no botão laranja **"Criar VPC"**.

### 1.2 Use o Assistente "VPC e mais"

- Na tela de criação, certifique-se de que a opção **"VPC e mais"** (VPC and more) está selecionada. Isso automatizará todo o processo.

#### 1.2.1 Configurações

- **Nome:** `vpc-api-producao` (dá um nome para todo o conjunto de recursos).
- **Bloco CIDR IPv4:** Deixe o padrão `10.0.0.0/16`. Este é o intervalo de IPs da sua rede privada.
- **Número de zonas de disponibilidade (AZs):** Para simplificar, selecione `1`.
- **Número de sub-redes públicas:** `1`.
- **Número de sub-redes privadas:** `1`.
- **Gateways NAT:** Selecione `Nenhum`. Um gateway NAT permite que instâncias privadas acessem a internet para atualizações, mas adiciona custo. Para este exercício de isolamento total, não precisamos dele.
- **Endpoints da VPC:** Deixe como `Nenhum`.

- Observe o diagrama de arquitetura à direita. Ele mostra exatamente o que será criado.
- Clique em **"Criar VPC"**.

Em alguns minutos, a AWS criará para você:

- Sua VPC (`vpc-api-producao`).
- Uma subnet pública e uma privada.
- Um **Internet Gateway** (o portão de saída para a internet) e uma tabela de rotas que o conecta à subnet pública.
- Uma tabela de rotas para a subnet privada que **não** tem saída para a internet.

---

## 2. Lançando as Instâncias em Suas Respectivas Subnets

Agora que a rede está pronta, vamos colocar nossos servidores nela. Usaremos a AMI que você já criou.

### **A. Lançando a Instância Pública (Servidor Web)**

1. Vá para o painel do **EC2** e clique em **"Lançar instâncias"**.
    
2. Selecione sua AMI customizada em **"Minhas AMIs"**.
    
3. **Tipo de Instância:** `t3.micro`.
    
4. **Par de Chaves:** Selecione seu arquivo `.pem`.
    
5. **Configurações de Rede (Etapa mais importante):**
    
    - Clique em **"Editar"**.
        
    - **VPC:** Selecione a VPC que acabamos de criar (`vpc-api-producao`).
        
    - **Sub-rede (Subnet):** Selecione a **subnet pública** da lista (o nome dela terá `public` no final).
        
    - **Atribuir IP público automaticamente:** **Habilitar**. Isso é o que conecta a instância à internet.
        
    - **Security Group:** Crie um novo grupo de segurança com as seguintes regras de entrada (inbound rules):
        
        - `SSH` (porta 22) de `Meu IP` (para sua segurança).
            
        - `TCP Personalizado` (porta 8000) de `Qualquer lugar-IPv4` (`0.0.0.0/0`) para a API ser acessível.
            
    
6. Clique em **"Lançar instância"**.
    

### **B. Lançando a Instância Privada (Servidor de Backend)**

1. Clique em **"Lançar instâncias"** novamente.
    
2. Use a **mesma AMI**.
    
3. **Tipo de Instância:** `t3.micro`.
    
4. **Par de Chaves:** O mesmo arquivo `.pem`.
    
5. **Configurações de Rede:**
    
    - **VPC:** `vpc-api-producao`.
        
    - **Sub-rede (Subnet):** Desta vez, selecione a **subnet privada**.
        
    - **Atribuir IP público automaticamente:** **Desabilitar**. Isso a torna inacessível pela internet.
        
    - **Security Group:** Crie um **novo** grupo de segurança para a camada privada. Configure as regras de entrada para permitir acesso **apenas de dentro da sua VPC**.
        
        - `SSH` (porta 22) da **Origem (Source)** `10.0.0.0/16` (o CIDR da sua VPC).
            
        - `Todo o tráfego ICMP - IPv4` da **Origem (Source)** `10.0.0.0/16` (isso permite "pingar" a máquina a partir da instância pública, útil para testes).
	    - `ICMP` para Tudo e origem 10.0.0.0/16
6. Clique em **"Lançar instância"**.
    

---

## **Parte 3: Verificando o Acesso**

#### **Testando a Instância Pública**

1. Vá para a lista de instâncias, selecione sua instância pública e copie o **Endereço IPv4 público**.
    
2. Acesse no navegador: `http://SEU_IP_PUBLICO:8000/docs`. **Deve funcionar!**
    

#### **Testando a Instância Privada**

1. Selecione a instância privada. Note que ela **não tem um Endereço IPv4 público**, apenas um IP privado (algo como `10.0.x.x`).
    
2. **Você não consegue se conectar a ela diretamente do seu computador.** Tentar o comando `ssh` com o IP privado vai falhar.
    

#### **Como Acessar a Instância Privada? (Usando um Bastion Host)**

A maneira correta de acessar uma instância privada é "pulando" através de uma instância pública. A instância pública funciona como um portão de entrada seguro, também conhecido como **Bastion Host** ou **Jump Server**.

1. **Primeiro, conecte-se à sua instância PÚBLICA:**
    
    Bash
    
    ```
    ssh -i /caminho/para/sua/chave.pem ec2-user@IP_DA_INSTANCIA_PUBLICA
    ```
    
2. **Agora, de dentro da instância pública, conecte-se à instância PRIVADA:**
    
    - Você precisará da sua chave `.pem` dentro da máquina pública para fazer o pulo. A forma mais segura (e avançada) de fazer isso é com uma técnica chamada "SSH Agent Forwarding".
        
    - Mas, para um teste simples, você pode simplesmente usar o `ssh` de dentro da máquina pública para a privada, usando o IP privado dela. (Isso requer que a chave .pem também esteja na máquina pública, ou que se use o Agent Forwarding).
        
    - Vamos fazer um teste mais simples: um **ping**. De dentro da instância pública, execute:
        
        Bash
        
        ```
        ping IP_DA_INSTANCIA_PRIVADA
        ```
        
    - Se o ping responder, isso prova que as duas máquinas conseguem se comunicar dentro da sua rede privada, e que a máquina privada está isolada do mundo exterior.
        
