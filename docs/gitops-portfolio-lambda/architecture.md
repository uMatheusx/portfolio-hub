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
        DATA[("projects - docs - changelogs")]
    end

    PAGES["GitHub Pages"]

    PA -->|"push em docs/"| RD
    PB -->|"push em docs/"| RD
    PN -->|"push em docs/"| RD

    PA -->|"git tag v*"| RR
    PB -->|"git tag v*"| RR

    RD --> DATA
    RR --> DATA
    DATA -->|"push na main"| DEPLOY
    DEPLOY --> PAGES
```

## Dois Fluxos Independentes

**Fluxo 1 — Documentação** (~1 min):

```mermaid
flowchart LR
    A1["git push em docs/"] --> B1["docs.yml"]
    B1 -->|"dispatch: update-docs"| C1["receive-docs.yml"]
    C1 --> D1["deploy.yml"]
    D1 --> E1["GitHub Pages atualizado"]
```

**Fluxo 2 — Changelog** (~1 min):

```mermaid
flowchart LR
    A2["git tag v1.0.0"] --> B2["release.yml"]
    B2 -->|"dispatch: new-release"| C2["receive-release.yml"]
    C2 --> D2["deploy.yml"]
    D2 --> E2["GitHub Pages atualizado"]
```

## O que o Hub Armazena por Projeto

```mermaid
classDiagram
    class Project {
        +string name
        +string display_name
        +string description
        +string version
        +string tags
        +string repo_url
        +string docs_updated_at
        +string changelog_updated_at
    }

    class DocsFolder {
        +string README
        +string architecture
        +string usage
        +string api
    }

    class Changelog {
        +string content
    }

    Project --> DocsFolder : docs
    Project --> Changelog : changelogs
```

## Sequência Completa de uma Atualização de Docs

```mermaid
sequenceDiagram
    actor Dev as Developer
    participant Repo as repo-meu-projeto
    participant Hub as portfolio-hub
    participant Pages as GitHub Pages

    Dev->>Repo: git push em docs/architecture.md
    activate Repo
    Repo->>Repo: docs.yml detecta mudanca em docs/
    Repo->>Hub: repository_dispatch: update-docs
    deactivate Repo

    activate Hub
    Hub->>Repo: fetch docs/ via GitHub API
    Repo-->>Hub: conteudo dos arquivos
    Hub->>Hub: salva em docs/meu-projeto/
    Hub->>Hub: atualiza docs_updated_at no JSON
    Hub->>Hub: git commit e push
    deactivate Hub

    activate Pages
    Pages->>Pages: deploy.yml rebuild Astro
    Pages->>Pages: publica site atualizado
    deactivate Pages

    Note over Dev,Pages: cerca de 1 minuto do push ao portfolio publicado
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
