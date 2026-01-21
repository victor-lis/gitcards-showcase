# 🔮 Git Cards - Showcase

> A plataforma definitiva para desenvolvedores criarem, personalizarem e compartilharem seus "Developer Cards" dinâmicos.

<div align="center">

<br />

<a href="https://gitcards.victorlisbronzo.me/">
  <img src="https://img.shields.io/static/v1?label=&message=Ver%20Demo%20Online&color=8257e5&style=for-the-badge&logo=vercel" alt="Ver Demo Online" />
</a>
&nbsp;
<a href="https://victorlisbronzo.me/">
  <img src="https://img.shields.io/static/v1?label=&message=Portf%C3%B3lio%20do%20Desenvolvedor&color=2e2e2e&style=for-the-badge&logo=github" alt="Portfólio" />
</a>
&nbsp;
<a href="https://linkedin.com/in/victor-lis-bronzo">
  <img src="https://img.shields.io/static/v1?label=&message=Entrar%20em%20Contato&color=0077b5&style=for-the-badge&logo=linkedin" alt="Entrar em Contato" />
</a>

</div>

## 🎯 Parte 1: O Produto

### O que é o GitCards?

O GitCards é uma aplicação Full-Stack SaaS que permite a desenvolvedores transformarem seus dados do GitHub e Tech Stack em cartões visuais de alto impacto. Esses "cards" são gerados dinamicamente e otimizados para serem compartilhados em redes sociais (LinkedIn, Twitter) ou incorporados diretamente no README.md de seus perfis.

### 🌟 Funcionalidades & Fluxo

A aplicação não é apenas um gerador de imagens; é um ecossistema completo com economia interna:

- **Marketplace de Templates:** Uma loja onde designers podem disponibilizar layouts e usuários podem adquiri-los.
- **Sistema de Créditos:** Integração financeira real. Usuários compram "Pacotes de Créditos" (microtransações) para desbloquear templates premium.
- **Editor WYSIWYG:** Customização em tempo real das cores, fontes, dados e visibilidade dos elementos do card.
- **Autenticação Moderna:** Login sem senha (Magic Link) e validação via códigos OTP enviados por e-mail.

<div align="center">
  <h3>✨ Experimente agora:</h3>
  <a href="https://gitcards.victorlisbronzo.me/">
    <img src="https://img.shields.io/badge/Acessar_Aplicação_Agora-8257e5?style=for-the-badge" alt="Link para o projeto" />
  </a>
</div>

## 🏗️ Parte 2: Engenharia & Arquitetura

> 💡 **Nota Técnica:** Como o código-fonte é proprietário, esta seção detalha as decisões arquiteturais de alto nível que garantem escalabilidade, segurança e manutenibilidade.

### ⚡ Tech Stack

| Camada | Tecnologia Principal | Por que foi escolhida? |
| :--- | :--- | :--- |
| **Core** | Turborepo (Monorepo) | Gerenciamento centralizado de pacotes, linting e build cacheado. |
| **Frontend** | Next.js 14 (App Router) | Renderização híbrida (SSR para SEO, CSR para o Editor) e performance. |
| **Backend** | Node.js + Fastify | Baixo overhead e alta taxa de requests por segundo (RPS) para a API. |
| **Database** | PostgreSQL + Prisma | Integridade relacional robusta e Developer Experience (DX) superior. |
| **Tipagem** | TypeScript + Zod | Validação de dados em runtime e time-to-market seguro. |
| **Infra** | Docker + Nginx | Containerização completa e proxy reverso para produção. |

### 📐 Arquitetura: End-to-End Type Safety

O diferencial técnico deste projeto é o compartilhamento estrito de contratos de dados entre Frontend e Backend.

```mermaid
graph LR
    A[Frontend Next.js] -- Importa Tipos --> B((Shared Packages))
    C[Backend Node.js] -- Importa Tipos --> B
    B -- Zod Schemas --> A
    B -- Zod Schemas --> C
    D[Database] -- Prisma Client --> C
```

### 📐 Arquitetura Antiga

```mermaid
sequenceDiagram
    autonumber
    actor User as 👤 Usuário
    box "Frontend (Next.js)" #000000
        participant Page as Page/Component
        participant Hook as React Query Hook<br/>(useGetUserCard)
        participant Client as API Client<br/>(Axios)
    end
    box "Backend (Node.js/Fastify)" #333333
        participant Route as Fastify Route<br/>(Controller)
        participant Service as Business Service<br/>(UserCardService)
        participant Prisma as Prisma ORM
    end
    participant DB as PostgreSQL

    Note over User, DB: 🚀 Início do Fluxo de Leitura (Ex: Get Card)

    User->>Page: Acessa a página de detalhes
    Page->>Hook: Chama hook useGetUserCard(id)
    activate Hook
    Hook->>Client: Executa função de fetch
    Client->>Route: HTTP GET /user-cards/:id
    activate Route

    Note right of Client: Validação de Auth (JWT)<br/>via Middleware

    Route->>Route: Valida Params com Zod
    Route->>Service: Chama UserCardService.findById(id)
    activate Service

    Service->>Prisma: prisma.userCard.findUnique()
    activate Prisma
    Prisma->>DB: Query SQL
    DB-->>Prisma: Retorna dados brutos
    deactivate Prisma

    Service->>Service: Valida retorno com Zod Schema
    Service-->>Route: Retorna Objeto UserCard
    deactivate Service

    Route-->>Client: HTTP 200 OK (JSON)
    deactivate Route

    Client-->>Hook: Resolve Promise com dados
    Hook-->>Page: Atualiza estado (data, isLoading)
    deactivate Hook
    
    Page-->>User: Renderiza o Card SVG na tela
```

### 🔄 Arquitetura e Fluxo de Dados

```mermaid
sequenceDiagram
    autonumber
    actor User as 👤 Usuário
    box "Client Side (Browser)" #000000
        participant UI as 🖥️ UI Component
    end
    box "Server Side (Next.js Runtime)" #222222
        participant Action as ⚡ Server Action
        participant Service as 🧠 Service (Importado)
        participant ORM as 🔌 Prisma Client
    end
    participant DB as 🐘 PostgreSQL

    Note over User, DB: ⚡ Fluxo Otimizado (Zero HTTP Interno)

    User->>UI: Interação (Click / Submit)
    UI->>Action: Invoca Server Action (RPC)
    activate Action

    Note right of Action: 🔒 Auth Check (Session)<br/>✅ Validação Zod (Input)

    Action->>Service: await UserCardService.get(id)
    activate Service

    Note right of Service: 📦 Execução direta em memória<br/>(Sem serialização HTTP JSON)

    Service->>ORM: prisma.userCard.findUnique()
    activate ORM
    ORM->>DB: SQL Query
    DB-->>ORM: Result Set
    deactivate ORM

    Service-->>Action: Retorna Objeto/DTO
    deactivate Service

    Note left of Action: 💾 Opcional: Cachear com<br/>unstable_cache() aqui

    Action-->>UI: Retorna Payload Serializado
    deactivate Action

    UI-->>User: Atualiza Interface
```

### 📂 Estrutura do Projeto (Detalhada)

- `/nextjs`: Aplicação Frontend (Loja e Dashboard).
- `/nodejs`: API RESTful (Gerenciamento de usuários, compras e entrega de assets).
- `/packages`: Bibliotecas compartilhadas (Schemas Zod, Tipos).
- `/nginx`: Configurações de proxy reverso.


## 📸 Galeria (Mockups)

### Marketplace
> Vitrine de cards disponíveis para compra e personalização.

![MarketPlace](./src/marketplace.png)

### Customização
> Editor visual com preview em tempo real e ajustes finos.

![Customization](./src/customization.png)

## 👨‍💻 Autor

<div align="center">

<img src="https://github.com/victor-lis.png" width="100px;" alt=""/>
<br />
<sub><b>Victor Lis</b></sub>
<br />

Software Engineer & Content Creator
<br />
Focado em criar experiências digitais de alta performance e arquiteturas escaláveis.

<br />

<a href="https://linkedin.com/in/victor-lis-bronzo" target="_blank">
<img src="https://img.shields.io/badge/-LinkedIn-%230077B5?style=for-the-badge&logo=linkedin&logoColor=white" target="_blank"></a> 
<a href="https://github.com/victor-lis" target="_blank">
<img src="https://img.shields.io/badge/-GitHub-gray?style=for-the-badge&logo=github&logoColor=white" target="_blank"></a>

<br />
<br />
<p>
  Feito com 💜 e TypeScript.
</p>

</div>

