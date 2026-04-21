# Arquitetura

## Visão Geral

O portfolio-hub é um **agregador de documentação e changelogs**. Cada projeto mantém suas próprias docs — o hub centraliza tudo e publica um site estático no GitHub Pages.

```mermaid
flowchart TB
    subgraph Projetos["Repositórios de Projeto"]
        PA["repo-projeto-a"]
        PB["repo-projeto-b"]
        PN["repo-projeto-n"]
    end

    subgraph Hub["portfolio-hub"]
        RD["receive-docs.yml"]
        RR["receive-release.yml"]
        DEPLOY["deploy.yml"]
        DATA[("projects/*.json\ndocs/\nchangelogs/")]
    end

    PAGES["GitHub Pages\nhttps://usuario.github.io/portfolio-hub"]

    PA -->|"push em docs/\nupdate-docs"| RD
    PB -->|"push em docs/"| RD
    PN -->|"push em docs/"| RD

    PA -->|"git tag v*\nnew-release"| RR
    PB -->|"git tag v*"| RR

    RD --> DATA
    RR --> DATA
    DATA -->|"push na main"| DEPLOY
    DEPLOY --> PAGES
```

## Dois Fluxos Independentes

```mermaid
flowchart LR
    subgraph F1["Fluxo 1 — Documentação  (~1 min)"]
        direction TB
        A1["git push\n(alteração em docs/)"] --> B1["docs.yml\ndispatch: update-docs"]
        B1 --> C1["receive-docs.yml\nfetch + salva docs/"]
        C1 --> D1["deploy.yml\nrebuilda Astro"]
        D1 --> E1["GitHub Pages ✓\ndocs atualizadas"]
    end

    subgraph F2["Fluxo 2 — Changelog  (~1 min)"]
        direction TB
        A2["git tag v1.0.0"] --> B2["release.yml\ndispatch: new-release"]
        B2 --> C2["receive-release.yml\nfetch CHANGELOG.md"]
        C2 --> D2["deploy.yml\nrebuilda Astro"]
        D2 --> E2["GitHub Pages ✓\nchangelog atualizado"]
    end
```

## O que o Hub Sabe sobre os Projetos

O hub armazena apenas o necessário para exibir o portfolio. Não tem opinião sobre como o projeto roda.

```mermaid
classDiagram
    class Project {
        +string name
        +string display_name
        +string description
        +string version
        +string[] tags
        +string repo_url
        +datetime docs_updated_at
        +datetime changelog_updated_at
    }

    class DocsFolder {
        +README.md
        +architecture.md
        +usage.md
        +api.md
    }

    class Changelog {
        +CHANGELOG.md
    }

    Project "1" --> "1" DocsFolder : docs/projeto/
    Project "1" --> "1" Changelog : changelogs/projeto.md
```

## Sequência Completa de uma Atualização de Docs

```mermaid
sequenceDiagram
    actor Dev as Developer
    participant Repo as repo-meu-projeto
    participant Hub as portfolio-hub
    participant Pages as GitHub Pages

    Dev->>Repo: git push (docs/architecture.md)
    activate Repo
    Repo->>Repo: docs.yml detecta mudança em docs/
    Repo->>Hub: repository_dispatch: update-docs
    deactivate Repo

    activate Hub
    Hub->>Repo: fetch docs/architecture.md via API
    Repo-->>Hub: conteúdo do arquivo
    Hub->>Hub: salva em docs/meu-projeto/architecture.md
    Hub->>Hub: atualiza docs_updated_at no JSON
    Hub->>Hub: git commit + push
    deactivate Hub

    activate Pages
    Pages->>Pages: deploy.yml rebuild Astro
    Pages->>Pages: publica site atualizado
    deactivate Pages

    Note over Dev,Pages: ~1 minuto do push ao portfolio publicado
```

## Decisões de Design

### O hub não faz deploy de projetos

Cada projeto tem seu ciclo de vida próprio. O `release.yml` no projeto notifica o hub, mas o deploy do projeto para qualquer infraestrutura (Lambda, ECS, Kubernetes, VPS) é responsabilidade exclusiva do repo do projeto.

### Por que dois eventos separados?

| | `update-docs` | `new-release` |
|---|---|---|
| **Cadência** | Contínua, iterativa | Formal, versionada |
| **Gatilho** | Push em `docs/` | `git tag` |
| **O que atualiza** | `docs/projeto/` | `changelogs/projeto.md` + versão no JSON |
| **Cria versão nova?** | Não | Sim |

Separar os dois evita que uma melhoria de documentação precise criar uma release, e que uma release precise ter documentação perfeita.
