# Deploy Chatwoot Customizado no Coolify

Guia passo a passo para fazer deploy da sua versão customizada do Chatwoot no Coolify.

---

## 📋 Pré-requisitos

- [ ] Fork do Chatwoot configurado e modificado
- [ ] GitHub Actions publicando imagem no GHCR
- [ ] Servidor Coolify configurado
- [ ] Domínio apontado para o Coolify (opcional, mas recomendado)

---

## 🎯 Arquitetura do Deploy

```
┌─────────────────────────────────────────┐
│          GitHub Repository              │
│      github.com/admfizgo-spec/          │
│           chatwootfz                    │
└────────────┬────────────────────────────┘
             │
             │ Push to main
             ▼
┌─────────────────────────────────────────┐
│        GitHub Actions (CI/CD)           │
│   - Build Docker Image                  │
│   - Run Tests                           │
│   - Push to GHCR                        │
└────────────┬────────────────────────────┘
             │
             │ Publish
             ▼
┌─────────────────────────────────────────┐
│  GitHub Container Registry (GHCR)       │
│  ghcr.io/admfizgo-spec/chatwootfz      │
└────────────┬────────────────────────────┘
             │
             │ Pull image
             ▼
┌─────────────────────────────────────────┐
│            Coolify Server               │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │  Chatwoot Web (Rails)            │  │
│  │  - Serve requests                │  │
│  │  - Port 3000                     │  │
│  └──────────────────────────────────┘  │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │  Chatwoot Sidekiq (Workers)      │  │
│  │  - Background jobs               │  │
│  │  - Email sending                 │  │
│  └──────────────────────────────────┘  │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │  PostgreSQL (Database)           │  │
│  │  - pgvector extension            │  │
│  └──────────────────────────────────┘  │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │  Redis (Cache & Queue)           │  │
│  └──────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

---

## 🚀 Passo 1: Preparar o GitHub Container Registry

### 1.1. Habilitar Package no GitHub

1. Vá para o repositório: https://github.com/admfizgo-spec/chatwootfz
2. Navegue para **Settings** > **Actions** > **General**
3. Em **Workflow permissions**, selecione:
   - ✅ **Read and write permissions**
4. Salve as alterações

### 1.2. Verificar se o workflow está funcionando

```bash
# No seu repositório local
git add .
git commit -m "chore: configurar workflow de build"
git push origin main
```

Verifique em: https://github.com/admfizgo-spec/chatwootfz/actions

### 1.3. Tornar o Package público (recomendado)

1. Acesse: https://github.com/admfizgo-spec?tab=packages
2. Clique no package `chatwootfz`
3. **Package settings** > **Change visibility** > **Public**

---

## 🐘 Passo 2: Criar PostgreSQL no Coolify

### 2.1. Adicionar PostgreSQL

1. No Coolify, acesse seu projeto
2. **+ Add Resource** > **Database** > **PostgreSQL**
3. Configure:
   ```
   Name: chatwoot-postgres
   Version: 16 (com pgvector)
   Database: chatwoot_production
   Username: chatwoot
   Password: [gerar senha forte]
   ```
4. **Deploy**

### 2.2. Habilitar extensão pgvector

Após o deploy, execute no PostgreSQL:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

**Como executar:**
1. No Coolify, clique no PostgreSQL
2. **Terminal** ou **Execute Command**
3. Execute:
   ```bash
   psql -U chatwoot -d chatwoot_production -c "CREATE EXTENSION IF NOT EXISTS vector;"
   ```

### 2.3. Anotar credenciais

Copie as informações de conexão:
- Host: `chatwoot-postgres` (nome do serviço interno)
- Port: `5432`
- Database: `chatwoot_production`
- Username: `chatwoot`
- Password: `[sua senha]`

---

## 🔴 Passo 3: Criar Redis no Coolify

### 3.1. Adicionar Redis

1. **+ Add Resource** > **Database** > **Redis**
2. Configure:
   ```
   Name: chatwoot-redis
   Version: 7
   Password: [gerar senha forte]
   ```
3. **Deploy**

### 3.2. Anotar URL de conexão

A URL será algo como:
```
redis://:sua_senha@chatwoot-redis:6379
```

---

## 🌐 Passo 4: Deploy do Chatwoot Web

### 4.1. Criar Resource de Docker Image

1. **+ Add Resource** > **Docker Image**
2. Configure:

   **Basic Settings:**
   ```
   Name: chatwoot-web
   Image: ghcr.io/admfizgo-spec/chatwootfz:latest
   Port: 3000
   ```

### 4.2. Configurar Variáveis de Ambiente

Clique em **Environment Variables** e adicione:

#### **Rails Core**
```bash
RAILS_ENV=production
NODE_ENV=production
RAILS_LOG_TO_STDOUT=true
RAILS_SERVE_STATIC_FILES=true

# Gerar com: openssl rand -hex 64
SECRET_KEY_BASE=seu_secret_key_base_aqui_64_caracteres
```

#### **Database**
```bash
POSTGRES_HOST=chatwoot-postgres
POSTGRES_PORT=5432
POSTGRES_DATABASE=chatwoot_production
POSTGRES_USERNAME=chatwoot
POSTGRES_PASSWORD=senha_do_postgres_que_voce_criou
```

#### **Redis**
```bash
REDIS_URL=redis://:senha_do_redis@chatwoot-redis:6379
```

#### **Chatwoot Settings**
```bash
INSTALLATION_NAME=Minha Empresa
FRONTEND_URL=https://seu-dominio.com
FORCE_SSL=true
ENABLE_ACCOUNT_SIGNUP=false
```

#### **Mailer (SMTP)**
```bash
MAILER_SENDER_EMAIL=noreply@seu-dominio.com
MAILER_INBOUND_EMAIL_DOMAIN=seu-dominio.com

SMTP_DOMAIN=seu-dominio.com
SMTP_ADDRESS=smtp.seu-provedor.com
SMTP_PORT=587
SMTP_USERNAME=seu_usuario_smtp
SMTP_PASSWORD=sua_senha_smtp
SMTP_AUTHENTICATION=plain
SMTP_ENABLE_STARTTLS_AUTO=true
SMTP_TLS=true
```

#### **Storage (S3 - Recomendado para produção)**
```bash
ACTIVE_STORAGE_SERVICE=amazon

AWS_ACCESS_KEY_ID=sua_access_key
AWS_SECRET_ACCESS_KEY=sua_secret_key
AWS_REGION=us-east-1
AWS_BUCKET_NAME=seu-bucket-chatwoot
```

**Alternativa - Storage Local (apenas para teste):**
```bash
ACTIVE_STORAGE_SERVICE=local
```

#### **Features Opcionais**
```bash
# Integração com S3 para avatars
S3_BUCKET_NAME=seu-bucket-chatwoot

# Habilitar logs detalhados
LOG_LEVEL=info

# Configuração de timezone
TZ=America/Sao_Paulo
```

### 4.3. Configurar Health Check

```bash
Health Check Path: /health
Health Check Interval: 30s
Health Check Timeout: 10s
Health Check Start Period: 60s
```

### 4.4. Configurar Domínio

1. Em **Domains**, adicione:
   ```
   seu-dominio.com
   ```
2. Coolify vai automaticamente:
   - Configurar Nginx como proxy reverso
   - Gerar certificado SSL com Let's Encrypt

### 4.5. Deploy

Clique em **Deploy**

---

## ⚙️ Passo 5: Deploy do Sidekiq (Workers)

### 5.1. Criar Service de Sidekiq

1. **+ Add Resource** > **Docker Image**
2. Configure:

   **Basic Settings:**
   ```
   Name: chatwoot-sidekiq
   Image: ghcr.io/admfizgo-spec/chatwootfz:latest
   ```

### 5.2. Override Command

Em **Command Override**, adicione:
```bash
bundle exec sidekiq -C config/sidekiq.yml
```

### 5.3. Variáveis de Ambiente

**Use as MESMAS variáveis do chatwoot-web** (copie e cole todas)

### 5.4. Deploy

Clique em **Deploy**

---

## 🗃️ Passo 6: Inicializar Database

### 6.1. Primeira inicialização

Acesse o terminal do container `chatwoot-web`:

```bash
# Criar database (se não existir)
bundle exec rails db:create

# Carregar schema
bundle exec rails db:schema:load

# Popular dados iniciais (cria conta admin padrão)
bundle exec rails db:seed
```

### 6.2. Credenciais padrão

Após o seed, você pode fazer login com:
```
Email: admin@example.com
Password: Password1!
```

**⚠️ IMPORTANTE:** Mude a senha imediatamente após primeiro login!

---

## ✅ Passo 7: Verificações Pós-Deploy

### 7.1. Verificar se os services estão rodando

```bash
# No Coolify, verificar logs de cada service:
- chatwoot-web: deve mostrar "Listening on tcp://0.0.0.0:3000"
- chatwoot-sidekiq: deve mostrar "Sidekiq starting"
- chatwoot-postgres: "ready to accept connections"
- chatwoot-redis: "Ready to accept connections"
```

### 7.2. Testar acesso

1. Acesse: `https://seu-dominio.com`
2. Você deve ver a tela de login do Chatwoot
3. Faça login com as credenciais padrão

### 7.3. Criar primeira conta

1. Após login, você será direcionado para criar uma conta (workspace)
2. Preencha as informações
3. Crie seu primeiro inbox

---

## 🔄 Passo 8: Configurar Auto-Deploy (Webhook)

### 8.1. Gerar Webhook no Coolify

1. No `chatwoot-web`, vá em **Settings**
2. Copie a **Webhook URL**

### 8.2. Configurar no GitHub

1. Vá para: https://github.com/admfizgo-spec/chatwootfz/settings/hooks
2. **Add webhook**
3. Configure:
   ```
   Payload URL: [URL do webhook do Coolify]
   Content type: application/json
   Events: Just the push event
   Active: ✅
   ```

### 8.3. Testar Auto-Deploy

```bash
# Fazer uma modificação
echo "# Test" >> README.md
git add README.md
git commit -m "test: auto-deploy"
git push origin main

# Aguardar:
# 1. GitHub Actions fazer build (2-5 min)
# 2. Coolify detectar webhook e fazer redeploy (1-2 min)
```

---

## 🔐 Passo 9: Configurações de Segurança

### 9.1. Secrets Management

No Coolify, marque como **Secret** as seguintes variáveis:
- `SECRET_KEY_BASE`
- `POSTGRES_PASSWORD`
- `REDIS_URL` (contém senha)
- `AWS_SECRET_ACCESS_KEY`
- `SMTP_PASSWORD`

### 9.2. Firewall

Certifique-se que apenas as portas necessárias estão abertas:
- `80` (HTTP - redireciona para HTTPS)
- `443` (HTTPS)
- `22` (SSH - apenas para seu IP)

### 9.3. Backups

Configure backup automático no Coolify:
1. Vá em `chatwoot-postgres` > **Backups**
2. Configure:
   ```
   Frequency: Daily
   Retention: 7 days
   ```

---

## 📊 Passo 10: Monitoramento

### 10.1. Health Checks

Coolify vai automaticamente monitorar:
- `/health` endpoint do Chatwoot
- Status dos containers
- Uso de recursos

### 10.2. Logs

Acesse logs em tempo real:
```bash
# No Coolify, clique em cada service > Logs
```

### 10.3. Alertas (opcional)

Configure notificações no Coolify:
1. **Settings** > **Notifications**
2. Adicione webhook/email para alertas

---

## 🛠️ Troubleshooting

### Problema: Container não inicia

**Solução:**
```bash
# Verificar logs
Coolify > chatwoot-web > Logs

# Erros comuns:
# - SECRET_KEY_BASE vazio → gerar novo
# - Database não conecta → verificar POSTGRES_HOST
# - Redis não conecta → verificar REDIS_URL
```

### Problema: Assets não carregam

**Solução:**
```bash
# Verificar se RAILS_SERVE_STATIC_FILES=true
# Verificar se public/packs existe na imagem

# Recompilar assets manualmente (se necessário):
docker exec -it chatwoot-web bundle exec rails assets:precompile
```

### Problema: Emails não enviam

**Solução:**
```bash
# Testar SMTP no terminal do container:
docker exec -it chatwoot-web rails runner "ActionMailer::Base.mail(from: 'test@example.com', to: 'você@example.com', subject: 'test', body: 'test').deliver_now"

# Verificar logs do Sidekiq
Coolify > chatwoot-sidekiq > Logs
```

### Problema: Database migration pendente

**Solução:**
```bash
# Executar migrations
docker exec -it chatwoot-web bundle exec rails db:migrate

# Ou via Coolify Terminal
bundle exec rails db:migrate
```

### Problema: Imagem não atualiza

**Solução:**
```bash
# Forçar pull da nova imagem
Coolify > chatwoot-web > Settings > Force Redeploy

# Verificar se GitHub Actions completou
https://github.com/admfizgo-spec/chatwootfz/actions
```

---

## 📈 Otimizações de Performance

### 1. Aumentar workers do Sidekiq

Editar comando do `chatwoot-sidekiq`:
```bash
bundle exec sidekiq -C config/sidekiq.yml -c 10
```

### 2. Configurar CDN para assets

Use CloudFlare ou similar na frente do Coolify.

### 3. Tuning do PostgreSQL

Ajustar `postgresql.conf`:
```conf
shared_buffers = 256MB
effective_cache_size = 1GB
maintenance_work_mem = 64MB
```

### 4. Redis persistence

Para produção, configure RDB snapshots:
```bash
# No chatwoot-redis, adicionar comando:
redis-server --save 60 1000 --appendonly yes
```

---

## 🔄 Workflow de Atualização

```bash
# 1. Fazer modificações locais
vim app/javascript/dashboard/components/SeuComponente.vue

# 2. Testar localmente
docker compose up

# 3. Commit e push
git add .
git commit -m "feat: nova funcionalidade"
git push origin main

# 4. Aguardar pipeline
# - GitHub Actions: ~3-5 minutos
# - Coolify webhook: ~1-2 minutos

# 5. Verificar deploy
curl https://seu-dominio.com/health
```

---

## 📚 Comandos Úteis

### Rails Console
```bash
docker exec -it chatwoot-web bundle exec rails console
```

### Verificar versão
```bash
docker exec -it chatwoot-web cat VERSION_CW
```

### Limpar cache
```bash
docker exec -it chatwoot-web bundle exec rails cache:clear
```

### Resetar senha de admin
```bash
docker exec -it chatwoot-web bundle exec rails runner "User.find_by(email: 'admin@example.com').update(password: 'NovaSenh@123')"
```

---

## 🎉 Checklist Final

- [ ] PostgreSQL rodando e conectado
- [ ] Redis rodando e conectado
- [ ] chatwoot-web respondendo em https://seu-dominio.com
- [ ] chatwoot-sidekiq processando jobs
- [ ] SSL/TLS configurado (cadeado verde)
- [ ] Login funcionando
- [ ] Emails enviando (testar recuperação de senha)
- [ ] Upload de arquivos funcionando
- [ ] Webhook configurado para auto-deploy
- [ ] Backups configurados
- [ ] Customizações visíveis (logo removida, toggle sidebar, etc)

---

## 🆘 Suporte

Se encontrar problemas:

1. **Verificar logs** (primeira coisa sempre!)
2. **Pesquisar na documentação oficial**: https://www.chatwoot.com/docs
3. **Abrir issue no fork**: https://github.com/admfizgo-spec/chatwootfz/issues
4. **Comunidade Chatwoot**: https://github.com/chatwoot/chatwoot/discussions

---

**Versão:** 1.0.0
**Última atualização:** 2025-11-25
**Compatível com:** Chatwoot v3.14.0+
