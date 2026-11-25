# nfchat-site

Site institucional/comercial (landing + blog) do nfchat. Estático, servido por nginx.

> **Marca provisória.** O nome visível ("nfchat") é um placeholder — fácil de trocar no `index.html`.

## Estrutura
- `index.html` — landing (hero, como funciona, benefícios, planos, FAQ, CTA).
- `blog/` — blog (listagem + posts em `blog/posts/`). Novos posts entram **via PR** (ver abaixo).
- `Dockerfile` — imagem nginx com o site.
- `.github/workflows/deploy.yml` — build + push da imagem no **GHCR** e **deploy pela pipeline** (job `deploy` em runner self-hosted no servidor: `docker compose pull` + `up -d`).
- `docker-compose.yml` — serviço do site (sem Watchtower; a atualização é feita pela pipeline).
- `DEPLOY.md` — procedimento de deploy no servidor Linux.

## Blog com aprovação (fluxo)
A cada 3 dias uma rotina gera um rascunho de post e abre um **PR** (branch `post/AAAA-MM-DD-...` → `main`).
Você recebe a notificação, **lê a matéria no PR** e, com seu **OK, faz o merge** → o CI/CD publica.
Nada vai ao ar sem sua aprovação.

## Rodar local
```bash
python3 -m http.server 8000   # abre http://localhost:8000
# ou: docker build -t nfchat-site . && docker run -p 8083:80 nfchat-site
```
