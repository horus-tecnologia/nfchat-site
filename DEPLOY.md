# Deploy automatizado — padrão nfchat (estilo odinchat-webhook)

O **pipeline faz o deploy** (push, não polling). Sem Watchtower.

```
push na main
   └─ job build  (runner GitHub)  -> builda a imagem -> publica no GHCR (ghcr.io/horus-tecnologia/<repo>)
   └─ job deploy (runner self-hosted no servidor Linux) -> docker compose pull + up -d (recria containers)
```

## Servidor Linux (uma vez)
Todos os projetos nfchat convivem no **mesmo host**, cada um com seu `docker-compose.yml` (atualização independente). Portas:

| Projeto | Porta host |
|---|---|
| nfchat-api | 8080 |
| nfchat-app | 8081 |
| nfchat-sales-app | 8082 |
| nfchat-site | 8083 |

### 1. Runner self-hosted (uma vez por servidor)
Registrar um GitHub Actions self-hosted runner no servidor, com os labels `self-hosted, linux, nfchat`
(em *Settings → Actions → Runners* de cada repo, ou um runner compartilhado na org):
```bash
# no servidor, conforme instruções do GitHub (./config.sh ... --labels nfchat)
# deixar como serviço: ./svc.sh install && ./svc.sh start
```

### 2. Login no GHCR + subir uma vez
```bash
echo $GHCR_PAT | docker login ghcr.io -u <usuario> --password-stdin
git clone https://github.com/horus-tecnologia/nfchat-site.git /opt/nfchat/nfchat-site
cd /opt/nfchat/nfchat-site && docker compose -p nfchat-site up -d
```

### 3. Depois
Nada manual: cada push na `main` builda a imagem e o job `deploy` (no runner self-hosted) roda
`docker compose pull` + `up -d`, recriando o container com a nova imagem.
Rollback: `docker compose -p nfchat-site pull <tag-antiga>` ou reexecutar um run anterior.

## Proxy reverso (opcional)
nginx/Traefik na frente mapeando domínios → portas (ex.: `www → :8083`).
