# Portfólio — Kauan Felipe

Site pessoal de portfólio (Dados, BI & Automação).

## Stack

- HTML/CSS/JS puro (sem build, sem framework)
- Deploy: Docker + Caddy em VPS/droplet

## Estrutura

- `index.html` — o site
- `cv.pdf` — currículo (não versionado; ver `.gitignore`), servido em `/cv.pdf`
- `DEPLOY.md` — passo a passo de deploy/atualização no droplet

## Rodar localmente

```bash
python3 -m http.server 8000
```

Abra `http://localhost:8000`.

## Atualizar em produção

Ver `DEPLOY.md`. Resumo: `rsync -avz index.html cv.pdf root@SEU_IP:/srv/portfolio/`
— Caddy serve o arquivo estático na hora, sem rebuild.
