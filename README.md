# portfolio-hub

Portfólio técnico com documentação viva, releases rastreáveis e automação via GitHub Actions.

## Começando

```bash
npm install
npm run dev
```

Acesse em `http://localhost:3000`

## Estrutura

```
portfolio-hub/
├── content/blog/        # Posts do blog (Markdown)
├── docs/                # Documentação dos projetos
├── projects/            # Metadados dos projetos (JSON)
├── src/
│   ├── components/      # Componentes Astro
│   ├── layouts/         # Layout global
│   └── pages/           # Páginas (index, blog, projects)
└── .github/workflows/   # CI/CD e automação
```

## Configuração de Projetos

Cada projeto é representado por um arquivo JSON em `projects/`:

```json
{
  "name": "seu-projeto",
  "display_name": "Seu Projeto",
  "description": "Descrição do projeto",
  "version": "1.0.0",
  "tags": ["Go", "Kubernetes"],
  "repo_url": "https://github.com/user/repo",
  "status": "active",
  "docs_updated_at": "2024-01-15T10:00:00Z",
  "changelog_updated_at": "2024-01-15T10:00:00Z"
}
```

## Blog

Posts ficam em `content/blog/` como arquivos Markdown com frontmatter:

```markdown
---
title: Título do post
description: Descrição breve
date: 2024-01-15
tags: [Go, GitOps]
---

Conteúdo aqui...
```

## Scripts

| Comando | Descrição |
|---------|-----------|
| `npm run dev` | Servidor de desenvolvimento |
| `npm run build` | Build para produção |
| `npm run preview` | Preview do build |
| `npm run changelog` | Atualiza `CHANGELOG.md` desde o último tag |
| `npm run changelog:all` | Regenera `CHANGELOG.md` do histórico completo |
| `npm run version:patch` | Bumpa versão patch (1.0.0 → 1.0.1) |
| `npm run version:minor` | Bumpa versão minor (1.0.0 → 1.1.0) |
| `npm run version:major` | Bumpa versão major (1.0.0 → 2.0.0) |
| `npm run release` | Gera changelog, commita e cria tag da versão |

## Documentação

Para integrar um novo projeto ao portfolio-hub, veja [docs/SETUP.md](./docs/SETUP.md).

## Licença

MIT
