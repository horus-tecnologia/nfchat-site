# Deploy automatizado — nfchat-site (padrão nfchat)

Fluxo: **push na `main` → GitHub Actions builda a imagem → publica no GHCR → Watchtower no servidor Linux atualiza sozinho.**

## Pipeline (CI)
- `.github/workflows/docker-publish.yml` builda e publica `ghcr.io/horus-tecnologia/nfchat-site:latest` a cada push na `main` (e `:vX.Y.Z` em tags).
- Usa o `GITHUB_TOKEN` (permissão `packages: write`) — não precisa de segredo extra.

## Servidor Linux (CD)
Todos os projetos nfchat convivem no **mesmo host**, cada um com seu `docker-compose.yml` e seu
Watchtower **com escopo** (atualização independente). Portas sugeridas:

| Projeto | Porta host |
|---|---|
| nfchat-api | 8080 |
| nfchat-app | 8081 |
| nfchat-sales-app | 8082 |
| nfchat-site | 8083 |

### Primeira vez
```bash
# 1. Docker + compose plugin instalados
# 2. Login no GHCR (PAT com read:packages, se a imagem for privada)
echo $GHCR_PAT | docker login ghcr.io -u <usuario> --password-stdin
# 3. Subir este projeto
git clone https://github.com/horus-tecnologia/nfchat-site.git
cd nfchat-site
docker compose up -d
```

### Depois
Nada manual: a cada push na `main`, a imagem nova sobe no GHCR e o Watchtower (intervalo 180s)
troca o container automaticamente. Para forçar: `docker compose pull && docker compose up -d`.

### Proxy reverso (opcional, recomendado)
Um nginx/Traefik na frente mapeia os domínios para as portas (ex.: `www → :8083`, `app → :8081`,
`api → :8080`). Fica a critério da infra.
