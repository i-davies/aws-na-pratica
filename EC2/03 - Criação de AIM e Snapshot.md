# Clonando e Lançando Instâncias de API na AWS

## 1. Pré-requisitos

- Você tem uma instância EC2 original, totalmente configurada, com a API funcionando.
    
- Os arquivos do projeto estão no local correto: `/home/ec2-user/fastapi-example`.
    

---

## 2. Criando o Molde Perfeito (Snapshot + AMI)

O objetivo aqui é criar um "molde" (AMI) à prova de falhas da sua instância principal, garantindo que todas as configurações e arquivos sejam copiados.

### 2.1. Etapa A: Criar o Snapshot (A Cópia Fiel do Disco)

1. No Console da AWS, navegue até **EC2 > Elastic Block Store > Volumes**.
    
2. Encontre e selecione o disco (Volume) que está anexado à sua instância principal.
    
3. Clique em **Ações > Criar snapshot**.
    
4. Dê uma descrição clara (ex: `Snapshot-API-v1-Pronto`) e clique em **Criar snapshot**.
    
5. Vá para a seção **Snapshots** e aguarde o status mudar para **`completed`**.
    

### 2.2. Etapa B: Criar a Imagem (A Planta da Máquina)

1. Ainda na seção **Snapshots**, selecione o snapshot que você acabou de criar.
    
2. Clique em **Ações > Criar imagem do snapshot**.
    
3. Preencha os detalhes:
    
    - **Nome da imagem:** Dê um nome claro (ex: `AMI-API-FastAPI-v1-Pronta-Para-Clonar`).
        
    - **Nome do dispositivo raiz:** Verifique o nome na sua instância original (aba "Armazenamento"), mas geralmente é `/dev/xvda`.
        
4. Clique em **Criar imagem**.
    

---

## 3. Lançando um Clone (Nova Instância)

Agora vamos usar nossa AMI para criar um servidor novo e idêntico, que já iniciará com a API funcionando.

1. Navegue até **EC2 > AMIs**.
    
2. Filtre por **"Owned by me"** (De minha propriedade), selecione a AMI que você acabou de criar e clique em **"Lançar instância da AMI"**.
    
3. Configure o lançamento:
    
    - **Número de instâncias:** `1` (ou quantas você quiser).
        
    - **Tipo de instância:** `t2.micro`.
        
    - **Par de chaves (Key pair):** Selecione seu arquivo `.pem` de sempre.
        
    - **Configurações de Rede > Security Group:** Escolha **"Selecionar grupo de segurança existente"** e selecione o grupo que já permite acesso na porta `8000`.
        
4. Clique em **"Lançar instância"**.
    

### 3.1. Verificação Final

Após alguns minutos, a nova instância estará no estado "Running".

1. Selecione a nova instância na lista para ver seus detalhes.
    
2. Copie o **"Endereço IPv4 público"** (Public IPv4 address).
    
3. Cole no seu navegador, adicionando a porta `:8000` no final:
    
    `http://SEU_NOVO_IP_PUBLICO:8000`
    
    E para ver a documentação:
    
    `http://SEU_NOVO_IP_PUBLICO:8000/docs`
