# Arquitetura Detalhada do Projeto Formula SAE App

## 1. Visão Geral

Este documento descreve a arquitetura do projeto Formula SAE App, um sistema composto por uma aplicação web e um backend API.

O fluxo geral da aplicação é o seguinte:

1.  O **Frontend Web**, construído com Next.js (React), é a interface com o usuário.
2.  A aplicação se comunica via requisições HTTP com uma **API Backend**.
3.  A **API Backend**, construída com NestJS, processa as requisições, aplica a lógica de negócio e interage com um banco de dados.
4.  Um banco de dados **PostgreSQL** é usado para a persistência dos dados, com o **Prisma ORM** gerenciando o acesso e as migrações do esquema.
5.  **Docker e Docker Compose** são utilizados para orquestrar os serviços de frontend, backend e banco de dados em um ambiente de desenvolvimento e produção consistente.

## 2. Arquitetura de Pastas e Componentes

A estrutura de pastas do projeto é organizada em um monorepositório com as seguintes pastas principais na raiz:

-   `api/`: Contém todo o código-fonte e configuração do backend.
-   `frontend-apuama/`: Contém todo o código-fonte e configuração da aplicação web frontend.
-   `db/`: Potencialmente para scripts de banco de dados ou backups (atualmente vazio).
-   `docker-compose.yml`: Arquivo de orquestração dos contêineres.
-   `README.md`: Documentação geral do projeto.
-   `db_model.mermaid`: Diagrama de Modelo Físico do Banco de Dados (ERD) em formato Mermaid.
-   `db_class_diagram.mermaid`: Diagrama de Classes UML do Backend em formato Mermaid.
-   `client_side_interactions.mermaid`: Diagrama de Interações Cliente-Lado (fluxograma de usuários) em formato Mermaid.

### 2.1. Arquitetura do Backend (`api/BackEnd_Appuama/`)

O backend segue a arquitetura modular padrão do NestJS, que promove uma forte organização e separação de conceitos.

**Módulos Principais:**
-   **`AppModule`**: Módulo raiz da aplicação.
-   **`AuthModule`**: Gerencia autenticação e autorização (login, JWT, guards).
-   **`UsersModule`**: Gerencia usuários (CRUD, aprovação).
-   **`CarsModule`**: Gerencia registros de carros.
-   **`BalanceModule`**: Gerencia dados de balanceamento de carros.
-   **`ReportsModule`**: Gerencia relatórios de testes.
-   **`ChecklistItemsModule`**: Gerencia itens de checklist.
-   **`PrismaModule`**: Encapsula a integração com o Prisma ORM.

**Estrutura de Exemplo (`users/`):**
-   **`users.controller.ts`**: Define os endpoints da API (ex: `GET /users`, `POST /users`). Recebe requisições HTTP e chama métodos do serviço.
-   **`users.service.ts`**: Contém a lógica de negócio. Realiza operações de banco de dados (via `PrismaService`), manipula dados e aplica regras de negócio.
-   **`dto/create-user.dto.ts`**: Define o formato dos dados de entrada. Utiliza DTOs com `class-validator` para validação automática.

**Diagrama de Classes:**
Para uma visão detalhada da estrutura de classes do backend, suas propriedades, métodos e relações, consulte o diagrama UML de classes gerado: `db_class_diagram.mermaid`.

### 2.2. Arquitetura do Frontend Web (`frontend-apuama/`)

O frontend é uma aplicação Next.js, utilizando o App Router para roteamento, React para a interface do usuário, Tailwind CSS para estilização e Shadcn UI para componentes.

**Estrutura de Pastas:**
-   **`app/`**: Contém as rotas e layouts da aplicação (Next.js App Router).
    -   **`login/page.tsx`**: Página de login.
    -   **`admin/page.tsx`**: Página do painel administrativo.
    -   **`reports/page.tsx`**: Página de visualização e criação de relatórios.
    -   **`layout.tsx`**: Layouts compartilhados para grupos de rotas ou para a aplicação inteira.
-   **`components/`**: Componentes React reutilizáveis, incluindo componentes de UI do Shadcn UI.
-   **`contexts/`**: Provedores de contexto React, como `AuthContext`, para gerenciamento de estado global.
-   **`hooks/`**: Hooks React personalizados para lógica reutilizável.
-   **`lib/`**: Utilitários e funções auxiliares.

**Interações Cliente-Lado (Fluxograma de Usuários):**
Para entender as interações e capacidades dos diferentes tipos de usuários na interface web, consulte o diagrama de interações cliente-lado: `client_side_interactions.mermaid`.

## 3. Esquema do Banco de Dados

O esquema físico do banco de dados PostgreSQL é definido usando Prisma ORM no arquivo `prisma/schema.prisma` do backend. Este arquivo descreve os modelos de dados (tabelas), seus campos e as relações entre eles.

**Modelo Físico (ERD):**
Para uma representação visual das tabelas do banco de dados, seus campos e relacionamentos, consulte o Diagrama de Entidade-Relacionamento (ERD) gerado: `db_model.mermaid`.

## 4. Orquestração com Docker e Docker Compose

O `docker-compose.yml` na raiz do projeto define e orquestra os serviços da aplicação, garantindo um ambiente consistente de desenvolvimento e produção.

```yaml
services:
  backend:
    build: ./api/BackEnd_Appuama
    ports:
      - "3000:3000"
    depends_on:
      - db
    # ... outras configurações do backend

  frontend:
    build: ./frontend-apuama
    ports:
      - "80:3000" # Mapeia a porta 3000 do contêiner para a porta 80 da máquina host
    depends_on:
      - backend
    # ... outras configurações do frontend

  db:
    image: postgres
    restart: always
    ports:
      - "5432:5432"
    # ... outras configurações do banco de dados
```

-   **`backend` service**: Constrói a imagem Docker a partir do `Dockerfile` em `api/BackEnd_Appuama`. Mapeia a porta 3000 do contêiner para a porta 3000 da máquina host e depende do serviço `db`.
-   **`frontend` service**: Constrói a imagem Docker a partir do `Dockerfile` em `frontend-apuama`. Mapeia a porta 3000 do contêiner para a porta 80 da máquina host e depende do serviço `backend`.
-   **`db` service**: Utiliza a imagem oficial do `postgres`. Mapeia a porta 5432 para a porta do host, permitindo a conexão direta para depuração. `restart: always` garante a resiliência do banco de dados.

Esta arquitetura modular e bem definida, juntamente com a orquestração via Docker, facilita a manutenção, o teste e a escalabilidade do projeto, permitindo que diferentes partes do sistema evoluam de forma independente.