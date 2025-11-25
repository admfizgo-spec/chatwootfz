# Chatwoot Customizado - admfizgo-spec

> **Fork customizado do Chatwoot com modificações no frontend**

[![Docker Build](https://github.com/admfizgo-spec/chatwootfz/actions/workflows/docker-build.yml/badge.svg)](https://github.com/admfizgo-spec/chatwootfz/actions/workflows/docker-build.yml)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

---

## 📦 Imagem Docker

```bash
ghcr.io/admfizgo-spec/chatwootfz:latest
```

Esta é uma versão customizada do Chatwoot com as seguintes modificações:

- Logo customizada removida/alterada
- Toggle bar adicionada na sidebar
- Tema e cores personalizadas
- [Adicione suas customizações aqui]

---

## 🚀 Quick Start

### Docker Compose (Desenvolvimento)

```bash
# Clonar o repositório
git clone https://github.com/admfizgo-spec/chatwootfz.git
cd chatwootfz

# Copiar e configurar .env
cp .env.example .env

# Subir ambiente de desenvolvimento
docker compose up
```

Acesse: http://localhost:3000

### Docker Compose (Produção Local)

```bash
# Build e subir ambiente de produção
docker compose -f docker-compose.production.local.yaml up -d

# Inicializar database (primeira vez)
docker compose -f docker-compose.production.local.yaml exec chatwoot-web \
  bundle exec rails db:create db:schema:load db:seed
```

---

## 📚 Documentação

- **[CUSTOMIZATION_GUIDE.md](CUSTOMIZATION_GUIDE.md)** - Como customizar o frontend
- **[COOLIFY_DEPLOY.md](COOLIFY_DEPLOY.md)** - Deploy completo no Coolify
- **[Documentação Oficial](https://www.chatwoot.com/docs)** - Docs do Chatwoot

---

## 🛠️ Desenvolvimento

### Estrutura do Projeto

```
chatwootfz/
├── app/javascript/dashboard/          # Frontend Vue.js
│   ├── components/                    # Componentes
│   ├── assets/                        # Assets
│   └── routes/                        # Rotas
├── Dockerfile.production              # Build de produção
├── docker-compose.yaml                # Dev environment
└── .github/workflows/                 # CI/CD
```

### Comandos Úteis

```bash
# Instalar dependências
bundle install
pnpm install

# Rodar servidor de desenvolvimento
pnpm dev

# Lint
pnpm eslint:fix
bundle exec rubocop -a

# Tests
pnpm test
bundle exec rspec
```

---

## 🏗️ Build e Deploy

### Build Manual

```bash
docker build -f Dockerfile.production -t chatwoot-custom .
```

### CI/CD Automático

Push para `main` → GitHub Actions → GHCR → Deploy Automático

Acompanhe em: https://github.com/admfizgo-spec/chatwootfz/actions

---

## 🔗 Links

- **Repositório Original**: [chatwoot/chatwoot](https://github.com/chatwoot/chatwoot)
- **Documentação**: [chatwoot.com/docs](https://www.chatwoot.com/docs)
- **Comunidade**: [GitHub Discussions](https://github.com/chatwoot/chatwoot/discussions)

---

## 📄 Licença

Este projeto mantém a mesma licença do projeto original: [MIT License](LICENSE)

---

## 🤝 Contribuindo

1. Fork este repositório
2. Crie uma branch para sua feature (`git checkout -b feature/nova-feature`)
3. Commit suas mudanças (`git commit -m 'feat: adicionar nova feature'`)
4. Push para a branch (`git push origin feature/nova-feature`)
5. Abra um Pull Request

---

## 🙏 Créditos

Este é um fork do excelente projeto [Chatwoot](https://github.com/chatwoot/chatwoot) desenvolvido pela equipe Chatwoot.

Todas as modificações customizadas são mantidas neste fork para fins específicos de uso.
