# Deploy — Portfólio Corvus (kauanfelipe.com)

Contexto: droplet com **Docker + Caddy**. A ideia de segurança é simples e forte:
**raiz pública** (o portfólio, que você divulga) e **todo o resto atrás de senha**,
cada ferramenta num subdomínio próprio. Ninguém entra no que não deve.

Ajuste caminhos e o nome do serviço do Caddy ao teu `docker-compose.yml`.
Onde aparecer `SEU_IP`, é o IP do droplet.

---

## 1. Pasta do site no droplet

```bash
ssh root@SEU_IP
mkdir -p /srv/portfolio
exit
```

## 2. Subir os arquivos (do teu Linux)

Na pasta onde estão `index.html` e o `cv.pdf`:

```bash
rsync -avz index.html cv.pdf root@SEU_IP:/srv/portfolio/
```

> O `cv.pdf` fica na **mesma pasta** do `index.html`. Assim ele abre em
> `kauanfelipe.com/cv.pdf` — que é exatamente pra onde o botão "Baixar CV" aponta.
> Enquanto você não subir o PDF, o botão dá 404; suba junto e resolve.

## 3. Caddyfile — raiz pública

```caddyfile
kauanfelipe.com {
    root * /srv/portfolio
    file_server
    encode gzip
}
```

Isso serve o portfólio e o CV, com HTTPS automático. Nada além disso é público.

## 4. Ferramentas privadas (superapp, WMS) — subdomínio + senha

Gere o hash da senha (uma vez):

```bash
# se o Caddy roda em container chamado "caddy":
docker exec -it caddy caddy hash-password --plaintext 'SUA_SENHA_FORTE'
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
# com container "caddy":
docker exec -w /etc/caddy caddy caddy reload

# se for systemd no host:
# sudo systemctl reload caddy
```

## 6. Checklist final

- [ ] `index.html` e `cv.pdf` em `/srv/portfolio`
- [ ] `kauanfelipe.com` abre o portfólio
- [ ] `kauanfelipe.com/cv.pdf` abre/baixa o CV
- [ ] cada subdomínio de ferramenta **pede senha**
- [ ] nada além do portfólio abre sem senha
- [ ] cadeado (HTTPS) verde em tudo

---

### Fluxo pra atualizar depois

Toda vez que mudar o site, é só:

```bash
rsync -avz index.html cv.pdf root@SEU_IP:/srv/portfolio/
```

Static file, sem rebuild. Caddy já serve na hora.
