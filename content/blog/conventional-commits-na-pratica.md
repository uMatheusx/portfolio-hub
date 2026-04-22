---
title: "Conventional Commits na prática: do commit ao changelog automático"
description: "Como padronizar mensagens de commit para gerar changelogs automaticamente, manter histórico legível e facilitar code review."
date: "2026-04-10"
tags: ["Git", "GitOps", "Automação", "Boas Práticas"]
---

# Conventional Commits na prática: do commit ao changelog automático

Mensagem de commit é documentação. Um histórico de git bem escrito conta a história do projeto — o que mudou, por que mudou, e quando. Um histórico mal escrito é ruído.

Conventional Commits é uma convenção simples que transforma mensagens de commit em dados estruturados. Com isso, é possível gerar changelogs automaticamente, determinar a próxima versão semântica e filtrar o histórico por tipo de mudança.

## A estrutura

```
<tipo>[escopo opcional]: <descrição>

[corpo opcional]

[rodapé opcional]
```

Os tipos principais:

| Tipo | Quando usar | Impacto na versão |
|------|-------------|-------------------|
| `feat` | Nova funcionalidade | Minor (1.X.0) |
| `fix` | Correção de bug | Patch (1.0.X) |
| `docs` | Documentação | Nenhum |
| `refactor` | Refatoração sem mudança de comportamento | Nenhum |
| `perf` | Melhoria de performance | Patch |
| `test` | Testes | Nenhum |
| `chore` | Build, dependências, configuração | Nenhum |
| `feat!` ou `BREAKING CHANGE` | Mudança incompatível | Major (X.0.0) |

## Exemplos reais

```bash
# Adicionando uma feature
git commit -m "feat(auth): add JWT token refresh endpoint"

# Corrigindo um bug com contexto
git commit -m "fix(api): resolve race condition in concurrent requests

The handler was not properly locking the shared state before
reading, causing intermittent 500 errors under high load.

Fixes #142"

# Breaking change
git commit -m "feat!: redesign authentication API

BREAKING CHANGE: /api/v1/auth endpoints removed.
Use /api/v2/auth instead. See migration guide in docs/migration.md"

# Documentação
git commit -m "docs: add architecture diagram to README"
```

## Do commit ao changelog

Com commits estruturados, ferramentas como `release-please` ou `auto-changelog` conseguem gerar changelogs automaticamente:

```markdown
## [2.1.0] - 2026-04-10

### Adicionado
- feat(auth): add JWT token refresh endpoint
- feat(api): support pagination in list endpoints

### Corrigido
- fix(api): resolve race condition in concurrent requests
- fix(db): handle null values in user profile query

### Documentação
- docs: add architecture diagram to README
- docs: update API reference with new endpoints
```

Isso é gerado automaticamente a partir das mensagens de commit. Sem trabalho manual.

## Automatizando com release-please

O `release-please` do Google é a ferramenta que uso. Ele monitora commits na branch `main` e, quando há commits relevantes, abre automaticamente uma PR com:

- `CHANGELOG.md` atualizado
- `package.json` (ou equivalente) com a versão bumped
- Tag de versão criada ao fazer merge

```yaml
# .github/workflows/release-please.yml
name: Release Please
on:
  push:
    branches: [main]
jobs:
  release-please:
    runs-on: ubuntu-latest
    steps:
      - uses: google-github-actions/release-please-action@v4
        with:
          release-type: simple
```

## O que muda no dia a dia

A mudança mais importante não é técnica — é de mentalidade. Antes de fazer commit, você precisa responder: *o que essa mudança faz?* Isso força clareza de pensamento e commits menores e mais focados.

Um commit que você não consegue descrever em uma linha provavelmente está fazendo coisas demais.

## Ferramentas úteis

- **[Commitizen](http://commitizen.github.io/cz-cli/)** — wizard interativo para criar commits no formato correto
- **[commitlint](https://commitlint.js.org/)** — valida mensagens de commit no pre-commit hook
- **[release-please](https://github.com/googleapis/release-please)** — automatiza releases e changelogs
- **[git-cliff](https://git-cliff.org/)** — gerador de changelog altamente configurável

O investimento inicial de configurar essas ferramentas se paga rapidamente. Depois de alguns meses, você tem um histórico que conta a história do projeto de forma clara e um processo de release que não depende de ninguém lembrar de atualizar o changelog manualmente.
