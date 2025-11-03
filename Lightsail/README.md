# Amazon Lightsail na prática

## 1. Objetivo e escopo

- Aplicação web com FastAPI (rodando em instância Lightsail)
- Banco de dados gerenciado PostgreSQL (Lightsail Managed Database)
- CDN (Lightsail CDN) na frente de um bucket de Object Storage (estático)
- Limpeza para não gerar cobranças


## 2. Free Tier do Lightsail em 2025 (resumo verificado)

Conforme a página oficial de preços do Lightsail (2025):

- Instâncias Linux/Unix elegíveis: 3 meses grátis em bundles selecionados (ex.: $5, $7, $12 com IPv4; $3.50, $5, $10 com IPv6 apenas)
- Banco de dados gerenciado: 3 meses grátis no bundle de $15/mês
- Containers: 3 meses grátis no Micro ($10/mês) — alternativo à instância
- CDN: 12 meses grátis com 50 GB/mês; depois, $2.50/mês para 50 GB
- Object Storage: pacote de 5 GB + 25 GB de transferência grátis por 12 meses

Atenção: Load balancer, snapshots e block storage extras são cobrados — não usaremos.

Dica: Use apenas UM bundle gratuito por tipo (instância/DB/container) por conta, conforme elegibilidade. Acompanhe uso/transferência para evitar excedentes.


## 3. Arquitetura do projeto

- Lightsail Instance (Amazon Linux 2023) — roda FastAPI
- Lightsail Managed Database (PostgreSQL $15) — conexão via endpoint público TLS
- Lightsail Object Storage (5 GB) — armazena arquivos estáticos (CSS/html)
- Lightsail CDN Distribution (50 GB) — frente ao Object Storage


## 4. Passo a passo

### 4.1 Criar a instância (grátis por 3 meses)

1) Abra Lightsail → Create instance
- Platform: Linux/Unix
- Blueprint: Amazon Linux 2023
- Enable SSH key: use a padrão ou crie
- Plan: escolha um bundle elegível ao gratuito (ex.: $5 IPv4)
- Create instance

2) Networking
- Acesse a aba Networking da instância e libere portas:
  - 22 (SSH)
  - 8000 (para testes da API)
  - (Opcional) 80/443 se for usar Nginx ou servir na porta padrão


### 4.2 Criar o banco PostgreSQL (3 meses grátis no $15)

1) Lightsail → Databases → Create database
- Engine: PostgreSQL
- Plan: $15 (elegível aos 3 meses gratuitos)
- Version: padrão
- Master user/password: defina com segurança
- Public mode: habilite para conectar via internet (TLS)
- Create database e aguarde status Available

2) Anote credenciais e endpoint
- Host, Port, DB name, Username, Password, CA (se disponível)


### 4.4 Preparar a aplicação FastAPI (Amazon Linux 2023)

Na instância, execute:

- Atualize e instale Python 3.11 + ferramentas (Amazon Linux 2023):
```bash
sudo dnf install -y git 
```

- Clone o repositório:
```bash
git clone https://github.com/i-davies/lightsail-fastapi.git
cd lightsail-fastapi
```

- Crie o ambiente virtual com o Python 3.11 e ative:
```bash
python3 -m venv venv
source venv/bin/activate
```

- Instale dependências (fastapi[standard] já inclui uvicorn):
```bash
pip install -r requirements.txt
```

- Configure variáveis de ambiente:
```bash
cp .env.example .env
nano .env  # Preencha PGHOST, PGUSER, PGPASSWORD, etc.
```

- Teste local:
```bash
fastapi run
```

- Abra `http://<IP-da-instancia>:8000/docs` e teste os endpoints `/health` e `/todos`.


### 4.6 Object Storage + CDN (12 meses grátis)

O Object Storage do Lightsail é equivalente ao S3 simplificado e pode hospedar sites estáticos completos, incluindo landing pages, portfolios, documentação, etc.

#### 4.6.1 Criar o bucket e configurar para site estático

1) Lightsail → Object storage → Create bucket
- Escolha a região e o bundle gratuito (5 GB por 12 meses)
- Nome do bucket (ex.: `meu-site-lightsail`)
- Create bucket

2) Configure acesso público de leitura
- Abra o bucket → Permissions → Access → Enable public read access
- Isso permite que os arquivos sejam servidos publicamente (necessário para a CDN)
- Para ver a URL pública de um arquivo específico, use o botão "Copy URL" do objeto no console

3) Torne os objetos públicos (leitura)
- Permissions → Access → Enable public read access
- Isso permite que o bucket sirva arquivos diretamente pela URL pública ou CDN

#### 4.6.2 Faça upload da landing page

Você já tem exemplos prontos em `Lightsail/static/`:
- `index.html` — página principal
- `styles.css` — estilos

**Upload pelo console:**
- Objects → Upload → selecione todos os arquivos de `Lightsail/static/`
- Aguarde o upload

Após o upload, você pode testar acessando a URL do arquivo exibida no console (Copy URL) ou, preferencialmente, pela URL da CDN quando a distribuição estiver pronta.

Nota: No Lightsail Object Storage não há a configuração de "Website hosting" igual ao S3 clássico (com "index document" e "error document"). Para servir a página inicial, acesse explicitamente `.../index.html` ou configure a CDN e use `.../index.html` na URL da distribuição.

#### 4.6.3 CDN para melhor performance e cache

1) Lightsail → Networking → Create distribution (CDN)
- Origin: selecione seu Object Storage bucket
- Default behavior: Cache all
- Create distribution (primeiros 50 GB/mês por 12 meses grátis)

2) Aguarde o deploy (pode levar alguns minutos)

3) Teste a landing page via CDN
- Copie a URL da distribuição (ex.: `https://abc123.cloudfront.net/`)
- Abra no navegador: `https://abc123.cloudfront.net/index.html`
- A CDN cacheia globalmente e acelera o carregamento

#### 4.6.4 Exemplo de uso: landing page + API

- Landing page (HTML/CSS/JS) → Object Storage + CDN (global, rápido, barato)
- API dinâmica (FastAPI) → Lightsail Instance (porta 8000 ou 80)
- Frontend chama a API via `fetch()` ou `axios` usando o IP público da instância

Arquitetura comum:
```
Usuário → CDN (landing page) → JavaScript faz requisição → API (instância Lightsail) → PostgreSQL
```

## 5. Usar no AWS Lightsail Container Service (gratuito)

No console do Lightsail:
1. Crie um Container Service (escolha o menor plano disponível; verifique as condições do período gratuito na página da AWS).
2. Adicione um deployment usando a imagem do Docker Hub (ex.: `docker.io/<usuario>/lightsail-fastapi:latest`).
3. Configure o container:
   - Porta de escuta: 8000 (o Dockerfile já inicia `uvicorn` nessa porta)
   - Endpoint público: habilitado na porta 8000
   - Health check HTTP: path `/health`
4. Configure as variáveis de ambiente do banco gerenciado (Lightsail Managed Database):
   - `PGHOST` = endpoint do banco
   - `PGPORT` = `5432`
   - `PGDATABASE` = nome do seu DB
   - `PGUSER` = usuário
   - `PGPASSWORD` = senha
   - `PGSSLMODE` = `require` (recomendado para banco gerenciado)
5. Salve e faça o deploy. Acesse a URL pública do serviço (exposta pelo Lightsail) e teste `/health` e `/docs`.

> Observações:
> - O arquivo `.env` não vai dentro da imagem; defina as variáveis no Lightsail.
> - Para desenvolvimento local, use `PGSSLMODE=disable`; em produção (Lightsail DB), use `require`.

## 6. Aplicação de exemplo (FastAPI + PostgreSQL)

Endpoints principais:
- `GET /health` — status da API e do banco
- `GET /todos` — lista tarefas
- `POST /todos` — cria tarefa `{title}`
- `PATCH /todos/{id}` — atualiza `done`
- `DELETE /todos/{id}` — remove

A aplicação cria automaticamente a tabela `todos` no startup, se não existir.


## 7. Comparativo rápido: Lightsail x AWS (EC2, RDS, CloudFront, S3)

- Custos: Lightsail usa bundles simples e previsíveis, com free tier específico; no AWS “clássico”, custos são separados por serviço (EC2, EBS, RDS, etc.)
- Facilidade: Console simplificado (instância, DB, object storage, CDN)
- Limitações: Menos flexível/avançado que EC2+RDS+S3+CloudFront; ideal para POCs, ensino e workloads simples


## 8. Limpeza (evitar cobranças)

Ao finalizar a prática, EXCLUA:
- Instância (Instance → Delete)
- Banco (Database → Delete)
- CDN Distribution (Networking → Delete)
- Object Storage bucket (apague objetos e depois o bucket)

Isso encerra o uso antes do fim do trial e evita custos após 3 meses (instância/db) e após 12 meses (CDN/objeto).
