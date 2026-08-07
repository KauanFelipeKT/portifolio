# Deploy — Portfólio Corvus (kauanfelipe.com)

Contexto: droplet (`198.199.65.100`) com **Docker + Caddy**. A ideia de segurança é
simples e forte: **raiz pública** (o portfólio, que você divulga) e **todo o resto
atrás de senha**, cada ferramenta num subdomínio próprio. Ninguém entra no que não deve.

Setup real deste droplet:
- Container do Caddy: `apps_caddy_1`
- Caddyfile: `/apps/Caddyfile` (no host)
- Arquivos do site: `/apps/portfolio` (no host), montado como `/var/www/portfolio`
  dentro do container do Caddy

---

## 1. Pasta do site no droplet

Já existe (`/apps/portfolio`). Só recriar se for um droplet novo:

```bash
ssh root@198.199.65.100
mkdir -p /apps/portfolio
exit
```

## 2. Subir os arquivos (do teu Linux)

Na pasta onde estão `index.html` e o `cv.pdf`:

```bash
scp index.html cv.pdf root@198.199.65.100:/apps/portfolio/
```

> O `cv.pdf` fica na **mesma pasta** do `index.html`. Assim ele abre em
> `kauanfelipe.com/cv.pdf` — que é exatamente pra onde o botão "Baixar CV" aponta.
> Enquanto você não subir o PDF, o botão dá 404; suba junto e resolve.

## 3. Caddyfile — raiz pública

```caddyfile
kauanfelipe.com {
    root * /var/www/portfolio
    file_server
    encode gzip
}
```

Isso serve o portfólio e o CV, com HTTPS automático. Nada além disso é público.

## 4. Ferramentas privadas (superapp, WMS) — subdomínio + senha

Gere o hash da senha (uma vez):

```bash
docker exec -it apps_caddy_1 caddy hash-password --plaintext 'SUA_SENHA_FORTE'
```

Copie o hash gerado e monte cada ferramenta assim:

```caddyfile
app.kauanfelipe.com {
    basic_auth {
        kauan COLE_O_HASH_AQUI
    }
    header X-Robots-Tag "noindex, nofollow"

    # se a ferramenta for um container (ex: superapp Next.js na porta 3000):
    reverse_proxy localhost:3000

    # OU, se for site estático:
    # root * /srv/superapp
    # file_server
}
```

Repita o bloco pra cada ferramenta (`app.`, `wms.` etc.), cada uma com o seu
`basic_auth`. Regras de ouro:

- **Nunca** linke esses subdomínios em lugar público.
- Todos com `basic_auth` — sem senha, não entra.
- `X-Robots-Tag noindex` pra não cair em busca.

> Caddy antigo (< v2.7) usa `basicauth` (sem underline). Se der erro no reload,
> troque `basic_auth` por `basicauth`.

## 5. Recarregar o Caddy

```bash
docker exec -w /etc/caddy apps_caddy_1 caddy reload
```

## 6. Checklist final

- [x] `index.html` e `cv.pdf` em `/apps/portfolio`
- [x] `kauanfelipe.com` abre o portfólio
- [x] `kauanfelipe.com/cv.pdf` abre/baixa o CV
- [ ] cada subdomínio de ferramenta **pede senha**
- [ ] nada além do portfólio abre sem senha
- [ ] cadeado (HTTPS) verde em tudo

---

### Fluxo pra atualizar depois

Toda vez que mudar o site, é só:

```bash
scp index.html cv.pdf root@198.199.65.100:/apps/portfolio/
```

Static file, sem rebuild. Caddy já serve na hora.
