
# Amazon S3 - Armazenamento de Objetos na AWS

## 1. Introdução ao Armazenamento de Objetos

**Características principais:**

- Dados são armazenados como **objetos**.
- Cada objeto tem: os **dados** (o arquivo em si), **metadados** (informações sobre o arquivo) e uma **chave única** (o "nome" do arquivo).
- Altamente escalável, durável e disponível.

### 1.1. Conceitos do S3: Buckets e Objetos

**Amazon S3 (Simple Storage Service):** O serviço da AWS para armazenamento de objetos.

**Buckets:**

- São os "contêineres" onde os objetos são armazenados.
- **Regra de Ouro:** O nome de um bucket é **globalmente único**. Nenhum outro usuário da AWS no mundo pode ter um bucket com o mesmo nome.
- São criados em uma **região específica** da AWS (ex: `us-east-1` em N. Virginia ou `sa-east-1` em São Paulo).

**Objetos:**

- São os arquivos que você armazena. Pode ser qualquer coisa: imagem, vídeo, PDF, backup de banco de dados, etc.
- Tamanho: de 0 bytes até 5 TB.
- A "chave" do objeto é seu identificador único dentro do bucket (ex: `relatorios/2025/janeiro.pdf`).

### 1.3. Casos de Uso Comuns

- **Armazenamento de Arquivos e Mídia:** Guardar imagens de um site, vídeos para streaming.
- **Backup e Recuperação de Desastres:** Salvar backups de bancos de dados e servidores. É barato e seguro.
- **Hospedagem de Sites Estáticos:** Hospedar sites feitos apenas com HTML, CSS e JavaScript, sem a necessidade de um servidor.
- **Data Lakes & Big Data:** Ponto central para armazenar grandes volumes de dados para análise posterior.

---

## 2. Criando e Gerenciando seu Primeiro Bucket


### 2.1. Preparação e Acesso ao Console

1. Faça login no Console de Gerenciamento da AWS.
2. Navegue até o serviço **S3**.

### 2.2. Criando um Bucket S3

1. Clique em **Criar bucket**.
2. Escolha o nome do bucket. O nome precisa ser único globalmente. Sugestão: `aula-s3-seunome-data`.
3. Escolha a Região. É importante escolher uma região próxima aos usuários para menor latência.
4. **Configurações de Acesso Público** - **Ponto crucial de segurança!** Por padrão, o S3 bloqueia todo o acesso público. Mantenha a configuração `Bloquear todo o acesso público` marcada por enquanto.
5. Clique em **Criar bucket**.

### 2.3. Upload e Acesso a Objetos

1. Entre no bucket recém-criado.
2. Clique em **Carregar** e escolha um arquivo simples (ex: uma imagem `.jpg` ou um `.txt`).
3. Após o upload, clique no objeto e copie a URL do objeto.
4. Tente abrir a URL no navegador. O resultado será um erro de **Acesso Negado** - isso é o esperado e correto! O bloqueio de acesso público está funcionando.

### 2.4. Tornando Objetos Públicos (de forma controlada)

1. Volte para o bucket e vá na aba **Permissões**.
2. Desmarque a opção **Bloquear todo o acesso público**. Salve as alterações.
3. Volte ao objeto, vá na aba **Permissões** e na seção **Lista de controle de acesso (ACL)**, conceda acesso de **Leitura** para **Todos (público)**.
4. Tente acessar a URL novamente - agora deve funcionar!

**Importante:** Mantenha o acesso bloqueado por padrão e abra acesso público apenas quando necessário (ex: arquivos de um site).

---

## 3. Recursos Avançados do S3: Custo e Segurança

### 3.1. Versionamento de Objetos

**Conceito:**  Quando ativado, o S3 não sobrescreve ou apaga um arquivo, ele cria uma nova versão.

**Utilidade:** Proteção contra exclusões acidentais ou sobrescritas indesejadas. É possível restaurar qualquer versão anterior de um objeto.

**Demonstração:** Ative o versionamento no bucket e faça o upload de um arquivo com o mesmo nome, mas conteúdo diferente. Você poderá visualizar as diferentes versões no console.

### 3.2. Classes de Armazenamento (Storage Classes)

**Conceito:** Nem todo dado é acessado com a mesma frequência. O S3 permite que você pague menos por dados que acessa raramente.

**Principais Classes:**

- **S3 Standard:** Para dados acessados frequentemente (sites, aplicações). Máxima performance e disponibilidade. Custo de armazenamento mais alto.
- **S3 Standard-IA (Infrequent Access):** Para dados acessados com menos frequência, mas que precisam estar disponíveis rapidamente quando solicitados (backups recentes). Armazenamento mais barato, mas cobra-se uma taxa por acesso.
- **S3 Glacier (Flexible Retrieval & Deep Archive):** Para arquivamento de longo prazo (dados de compliance, backups antigos). Armazenamento extremamente barato, mas o tempo para recuperar os dados pode levar de minutos a horas.
- **S3 Intelligent-Tiering:** Classe "inteligente" que move os dados automaticamente entre as camadas para otimizar custos com base no padrão de uso.

### 3.3. Políticas de Ciclo de Vida (Lifecycle Policies)

**Conceito:** Regras automáticas para gerenciar seus objetos e automatizar a mudança entre as classes de armazenamento.

**Exemplo de regra de ciclo de vida:**

1. Todo objeto no bucket...
2. ...depois de 30 dias, mover para **S3 Standard-IA**.
3. ...depois de 90 dias, mover para **S3 Glacier Deep Archive**.
4. ...depois de 7 anos, excluir permanentemente.

**Benefício:** Enorme economia de custos e automação de processos de governança de dados.

---

## 4. Hospedagem de Site Estático

### 4.1. Preparação dos Arquivos

Crie dois arquivos HTML simples:

- `index.html`: `<h1>Meu Site Estático na AWS!</h1>`
- `error.html`: `<h1>Página não encontrada!</h1>`

### 4.2. Configurando a Hospedagem

1. Faça o upload dos arquivos `index.html` e `error.html` para o bucket.
2. Vá em **Propriedades** do bucket, desça até o final e habilite a **Hospedagem de site estático**.
3. Especifique `index.html` como o documento de índice e `error.html` como o documento de erro.
4. Salve e anote a URL do endpoint do site (formato: `http://<bucket-name>.s3-website.<region>.amazonaws.com`).
5. Tente acessar a URL - não vai funcionar porque os objetos não são públicos.

### 4.3. Aplicando Política de Bucket

Para sites, a melhor prática não é tornar cada arquivo público, mas sim aplicar uma **Política de Bucket**.

1. Vá na aba **Permissões** do bucket e edite a **Política de bucket**.
2. Cole o seguinte JSON (substituindo `NOME-DO-SEU-BUCKET`):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::NOME-DO-SEU-BUCKET/*"
    }
  ]
}
```

3. Teste a URL do site novamente - agora deve funcionar!
        
