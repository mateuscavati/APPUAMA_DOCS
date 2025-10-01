# Arquitetura Detalhada do Projeto Formula SAE App

## 1. Visão Geral

Este documento descreve a arquitetura do projeto Formula SAE App, um sistema composto por um aplicativo móvel e um backend.

O fluxo geral da aplicação é o seguinte:

1.  O **Aplicativo Móvel (Mobile App)**, construído com React Native (Expo), é a interface com o usuário.
2.  O aplicativo se comunica via requisições HTTP com uma **API Backend**.
3.  A **API Backend**, construída com NestJS, processa as requisições, aplica a lógica de negócio e interage com um banco de dados.
4.  Um banco de dados **PostgreSQL** é usado para a persistência dos dados, com o **Prisma ORM** gerenciando o acesso e as migrações do esquema.
5.  **Docker e Docker Compose** são utilizados para orquestrar os serviços de backend e banco de dados em um ambiente de desenvolvimento consistente.

## 2. Arquitetura de Pastas

A estrutura de pastas do projeto é organizada em um monorepositório com as seguintes pastas principais na raiz:

-   `api/`: Contém todo o código-fonte e configuração do backend.
-   `mobile/`: Contém todo o código-fonte e configuração do aplicativo móvel.
-   `db/`: Potencialmente para scripts de banco de dados ou backups (atualmente vazio).
-   `docker-compose.yml`: Arquivo de orquestração dos contêineres.
-   `README.md`: Documentação geral do projeto.

### 2.1. Arquitetura do Backend (`api/BackEnd_Appuama/`)

O backend segue a arquitetura modular padrão do NestJS, que promove uma forte organização e separação de conceitos.

```
api/BackEnd_Appuama/
├── prisma/
│   ├── schema.prisma      # Definição do esquema do banco de dados (modelos e relações)
│   └── migrations/        # Migrações geradas pelo Prisma
├── src/
│   ├── prisma/
│   │   ├── prisma.module.ts # Módulo para o serviço do Prisma
│   │   └── prisma.service.ts  # Serviço que encapsula o cliente Prisma
│   ├── users/
│   │   ├── dto/
│   │   │   └── create-user.dto.ts # Data Transfer Object para validação de dados de entrada
│   │   ├── users.controller.ts # Controla as rotas e requisições HTTP para o módulo de usuários
│   │   ├── users.module.ts     # Define o módulo de usuários, importando dependências
│   │   └── users.service.ts    # Contém a lógica de negócio para o módulo de usuários
│   ├── app.controller.ts    # Controlador principal da aplicação
│   ├── app.module.ts        # Módulo raiz que une todos os outros módulos
│   ├── app.service.ts       # Serviço principal da aplicação
│   └── main.ts              # Ponto de entrada da aplicação (inicializa o servidor)
├── test/
│   ├── app.e2e-spec.ts      # Testes end-to-end
│   └── jest-e2e.json        # Configuração do Jest para testes e2e
├── Dockerfile               # Define a imagem Docker para o backend
├── nest-cli.json            # Configurações da CLI do NestJS
├── package.json             # Dependências e scripts do projeto
└── tsconfig.json            # Configurações do TypeScript
```

-   **`src/`**: O coração da aplicação.
    -   **`main.ts`**: Inicializa o NestJS e o servidor HTTP.
    -   **`app.module.ts`**: É o módulo raiz. Ele importa outros "feature modules" (como `UsersModule`) e módulos de configuração (como `PrismaModule`).
    -   **`users/`**: Este é um "feature module" que encapsula tudo relacionado à entidade "User".
        -   **`users.controller.ts`**: Define os endpoints da API (ex: `GET /users`, `POST /users`). Ele recebe as requisições HTTP e chama os métodos apropriados no `users.service.ts`.
        -   **`users.service.ts`**: Contém a lógica de negócio. É aqui que as operações de banco de dados são executadas (através do `PrismaService`), os dados são manipulados e as regras de negócio são aplicadas.
        -   **`dto/create-user.dto.ts`**: Define a "forma" dos dados esperados no corpo de uma requisição. O NestJS usa DTOs com `class-validator` para validar automaticamente os dados de entrada.
    -   **`prisma/`**: Este módulo (`PrismaModule` e `PrismaService`) é uma boa prática para encapsular a conexão com o banco de dados e torná-la injetável em outros serviços.

### 2.2. Arquitetura do Aplicativo Móvel (`mobile/mobile/`)

O aplicativo móvel usa a estrutura de roteamento baseada em arquivos do Expo Router, que simplifica a navegação.

```
mobile/mobile/
├── app/
│   ├── (tabs)/              # Define um grupo de rotas com layout de abas
│   │   ├── _layout.tsx      # Layout para as abas (define o TabBar)
│   │   ├── index.tsx        # Tela da primeira aba (Home)
│   │   └── explore.tsx      # Tela da segunda aba
│   ├── _layout.tsx          # Layout principal da aplicação (ex: provedores de tema)
│   └── +not-found.tsx       # Tela para rotas não encontradas
├── assets/
│   ├── fonts/
│   └── images/
├── components/
│   ├── ui/                  # Componentes de UI mais genéricos
│   ├── Collapsible.tsx
│   ├── ExternalLink.tsx
│   └── ThemedText.tsx       # Componentes que usam o tema da aplicação
├── constants/
│   └── Colors.ts            # Define a paleta de cores para os temas claro e escuro
├── hooks/
│   ├── useColorScheme.ts    # Hook para detectar o tema do dispositivo
│   └── useThemeColor.ts     # Hook para obter cores baseadas no tema atual
├── app.json                 # Configurações do projeto Expo
├── package.json             # Dependências e scripts
└── tsconfig.json            # Configurações do TypeScript
```

-   **`app/`**: O diretório mais importante para o Expo Router. A estrutura de arquivos aqui define a estrutura de navegação do aplicativo.
    -   **`(tabs)`**: O parêntese cria um "grupo de rotas" que não afeta a URL, mas permite agrupar rotas sob um layout comum.
    -   **`_layout.tsx`**: Arquivos de layout definem a UI que "envolve" um conjunto de rotas. O `app/(tabs)/_layout.tsx` define o `TabLayout`, enquanto o `app/_layout.tsx` pode definir provedores globais que envolvem toda a aplicação.
-   **`components/`**: Contém componentes React reutilizáveis. A separação entre componentes de UI genéricos (`ui/`) e componentes mais complexos é uma boa prática. O uso de `ThemedText` e `ThemedView` demonstra a implementação de um sistema de temas.
-   **`constants/`**: Usado para armazenar valores que não mudam, como a paleta de cores (`Colors.ts`), facilitando a manutenção de um design consistente.
-   **`hooks/`**: Para lógica reutilizável que precisa de estado ou ciclo de vida do React. `useColorScheme` e `useThemeColor` são exemplos perfeitos de como encapsular a lógica de tema.

## 3. Orquestração com Docker

O `docker-compose.yml` define e orquestra os serviços da aplicação.

```yaml
services:
  api:
    build: ./api/BackEnd_Appuama
    ports:
      - "3000:3000"
    depends_on:
      - db
    # ...
  db:
    image: postgres
    restart: always
    ports:
      - "5432:5432"
    # ...
```

-   **`api` service**: Constrói a imagem Docker a partir do `Dockerfile` localizado em `api/BackEnd_Appuama`. Ele mapeia a porta 3000 do contêiner para a porta 3000 da máquina host e depende do serviço `db`.
-   **`db` service**: Usa a imagem oficial do `postgres`. Mapeia a porta 5432 para a porta do host, permitindo a conexão direta com o banco de dados para fins de depuração. A configuração `restart: always` garante que o banco de dados seja reiniciado em caso de falha.

Esta arquitetura modular e bem definida facilita a manutenção, o teste e a escalabilidade do projeto, permitindo que diferentes partes do sistema evoluam de forma independente.
