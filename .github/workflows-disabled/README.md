# Workflows Desabilitados

Estes workflows foram movidos para esta pasta porque não são necessários para o fork customizado.

## Workflows Ativos

- `docker-build.yml` - Build e publicação da imagem customizada no GHCR

## Workflows Desabilitados (movidos para esta pasta)

- `run_foss_spec.yml` - Testes do Chatwoot original
- `run_mfa_spec.yml` - Testes de MFA
- `frontend-fe.yml` - Testes de frontend
- `lint_pr.yml` - Lint de PRs
- `deploy_check.yml` - Checagem de deploy
- `publish_foss_docker.yml` - Publicação oficial do Chatwoot
- `publish_ee_docker.yml` - Publicação Enterprise
- `test_docker_build.yml` - Teste de build Docker
- `size-limit.yml` - Checagem de tamanho

## Por que desabilitar?

1. **Lint failures** - O código original pode ter erros de lint que não afetam sua customização
2. **Testes desnecessários** - Você não precisa rodar os testes do Chatwoot original
3. **Publicações duplicadas** - Você tem seu próprio workflow de build
4. **Performance** - Menos workflows = builds mais rápidos

## Como reativar

Se precisar de algum workflow, basta mover o arquivo de volta para `.github/workflows/`:

```bash
mv .github/workflows-disabled/nome-do-workflow.yml .github/workflows/
```
