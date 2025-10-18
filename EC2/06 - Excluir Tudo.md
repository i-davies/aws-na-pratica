# Excluir Tudo

## 1. Como a Cobrança Funciona para Snapshots, AMIs e VPCs?

- **Snapshots (Backups do Disco):** **Sim, são cobrados.** A cobrança não é por hora, mas sim pelo espaço de armazenamento que eles ocupam, medido em Gigabytes por mês (GB/mês). Mesmo que sua instância esteja desligada, o Snapshot continua armazenado e gerando custos. **É crucial excluí-los.**
    
- **AMIs (Imagens/Moldes):** A AMI em si é apenas um registro, um "ponteiro", e é gratuita. **PORÉM**, toda AMI armazena um Snapshot por trás das cenas para guardar os dados do disco. Portanto, indiretamente, uma AMI **gera custos** por causa do Snapshot que ela mantém. Ao limpar, você precisa primeiro "desregistrar" a AMI e depois excluir o Snapshot associado.
    
- **VPCs (Redes Privadas):** A VPC e seus componentes básicos (Subnets, Tabelas de Rotas, Internet Gateways) são **gratuitos**. Você pode ter várias VPCs sem custo. No entanto, componentes **opcionais** dentro de uma VPC, como um **NAT Gateway** ou um **Elastic IP** que não está sendo usado, geram cobranças por hora. Como não usamos esses componentes pagos no nosso tutorial, a VPC em si não gerará custos, mas é uma excelente prática de organização excluí-la para manter a conta limpa.
    

---

## 2. ⚠️ Aviso Importante

Este processo é **destrutivo e irreversível**. Uma vez que um recurso é excluído, todos os dados contidos nele são perdidos permanentemente. Prossiga apenas quando tiver certeza de que não precisa mais da sua configuração.

---

## 3. Encerrar as Instâncias EC2 (Servidores Virtuais)

Esta é sempre a primeira etapa. Não é possível excluir uma rede se ainda houver máquinas funcionando dentro dela.

1. No Console da AWS, navegue até o serviço **EC2**.
    
2. No menu à esquerda, clique em **"Instâncias"**.
    
3. Selecione a caixa de seleção de **todas** as instâncias que você criou (a original, a pública, a privada, etc.).
    
4. Clique no botão **"Estado da instância"** (Instance state).
    
5. No menu suspenso, clique em **"Encerrar instância"** (Terminate instance). Confirme a ação.
    

O status mudará para "shutting-down" e depois para "terminated".

---

## 4. Excluir a VPC (Nossa Rede Privada)

Agora que a "casa" está vazia, podemos demolir o "condomínio".

1. Navegue até o serviço **VPC**.
    
2. No menu à esquerda, clique em **"Suas VPCs"** (Your VPCs).
    
3. Selecione a VPC que você criou (ex: `vpc-api-producao`).
    
4. Clique no botão **"Ações"** (Actions) e selecione **"Excluir VPC"** (Delete VPC).
    
5. Uma caixa de diálogo aparecerá. A AWS é inteligente e listará todos os recursos associados (Subnets, Internet Gateway, Tabelas de Rotas) que serão excluídos junto. Digite `delete` ou `excluir` no campo de confirmação e clique no botão para excluir.
    

> **Se ocorrer um erro:** O erro geralmente acontece se a AWS não conseguir excluir um recurso dependente automaticamente. O erro dirá qual recurso está impedindo a exclusão (ex: um "Internet Gateway" que precisa ser "desanexado" primeiro). Nesse caso, vá até o item mencionado no menu da VPC, desfaça a associação manualmente e tente excluir a VPC novamente.

---

## 5. Limpar Imagens e Backups (AMIs e Snapshots)

Mesmo com as instâncias e a rede removidas, os "moldes" e "backups" que geram custos ainda existem.

### 5.1. Desregistrar a AMI

1. No serviço **EC2**, vá para **"AMIs"** no menu à esquerda.
    
2. Filtre por **"Owned by me"**.
    
3. Selecione a AMI que você criou (ex: `AMI-API-FastAPI-v1-Pronta-Para-Clonar`).
    
4. Clique em **"Ações"** e selecione **"Desregistrar AMI"** (Deregister AMI). Confirme.
    

### 5.2. Excluir o Snapshot

1. Ainda no serviço **EC2**, vá para **"Snapshots"** na seção "Elastic Block Store".
    
2. Encontre o Snapshot associado à sua AMI (a descrição que você deu ajudará a identificá-lo).
    
3. Selecione o Snapshot, clique em **"Ações"** e **"Excluir snapshot"**. Confirme.
    

---

## 6. (Opcional, mas Recomendado) Limpeza Final

Os recursos abaixo geralmente são gratuitos, mas excluí-los mantém sua conta organizada.

- **Grupos de Segurança:** Vá para **EC2 > Grupos de segurança**. Selecione os grupos que você criou (ex: `sg-acesso-publico-api`) e use **Ações > Excluir grupos de segurança**. Você só pode excluir SGs que não estão mais associados a nenhuma instância.
    
- **Pares de Chaves:** Vá para **EC2 > Pares de chaves**. Selecione a chave que você usou e use **Ações > Excluir**. **Atenção:** Isso apaga a chave da AWS, mas não o arquivo `.pem` do seu computador.

