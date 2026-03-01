# GitHub Actions – Introdução Técnica

## 1. O que é

**GitHub Actions** é o mecanismo nativo de CI/CD do GitHub, baseado em **event-driven workflows**, definidos como código em YAML dentro do próprio repositório.

Ele permite:

- Build automatizado
- Testes automatizados
- Análise estática
- Publicação de artefatos
- Deploy contínuo
- Orquestração de pipelines multi-stage

Documentação oficial:
- [https://docs.github.com/en/actions](https://docs.github.com/en/actions)
- [https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)

---

## 2. Conceitos Fundamentais
### 2.1 Workflow

Unidade principal de automação.
- Arquivo YAML
- Local obrigatório:
```
.github/workflows/*.yml
```
- Executado a partir de eventos

Exemplo mínimo:
```yaml
name: exemplo

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Hello"
```

Referência:
[https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

### 2.2 Event (`on:`)

Define o gatilho do workflow.

Principais eventos:
- `push`
- `pull_request`
- `workflow_dispatch` (manual)
- `schedule` (cron)
- `release`

Exemplo:
```yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
```
Referência:
[https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)

### 2.3 Job

Um workflow possui um ou mais jobs.

Características:
- Executa em runner isolado
- Pode depender de outro job (needs)
- Pode rodar em paralelo
- Compartilha dados via artifacts/outputs

Exemplo:
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: mvn clean package
```

### 2.4 Runner

Ambiente onde o job executa.

Tipos:
- GitHub-hosted (`ubuntu-latest`, `windows-latest`, `macos-latest`)
- Self-hosted

Exemplo:
```yaml
runs-on: ubuntu-latest
```
Referência:
[https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners)

### 2.5 Step

Unidade de execução dentro do job.

Pode ser:
- Script shell (`run`)
- Uso de Action pronta (`uses`)
- Docker
- Composite

Exemplo:
```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v4

  - name: Build
    run: mvn clean package
```
Referência:
[https://docs.github.com/en/actions/creating-actions](https://docs.github.com/en/actions/creating-actions)

### 2.6 Action

Bloco reutilizável.

Tipos:
- JavaScript action
- Docker action
- Composite action

Exemplo oficial:
```yaml
uses: actions/checkout@v4
```
Marketplace:
[https://github.com/marketplace?type=actions](https://github.com/marketplace?type=actions)

## 3. Estrutura de Execução Interna

Fluxo lógico:
```
Evento → Workflow → Jobs → Steps → Runner
```
Isolamento:
- Cada job executa em máquina limpa
- Sistema de arquivos efêmero
- Estado compartilhado apenas via:
  - artifacts
  - cache
  - outputs

## 4. Variáveis e Contextos

Principais contextos:
- github
- runner
- job
- steps
- secrets
- env

Exemplo:
```yaml
run: echo ${{ github.ref }}
```
Referência:
[https://docs.github.com/en/actions/learn-github-actions/contexts](https://docs.github.com/en/actions/learn-github-actions/contexts)

## 5. Secrets

Armazenamento seguro:
```
Settings → Secrets and variables → Actions
```
Uso:
```yaml
env:
  DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
```
Referência:
[https://docs.github.com/en/actions/security-guides/encrypted-secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

## 6. Artefatos

Permite compartilhar dados entre jobs ou baixar após execução.
```yaml
- uses: actions/upload-artifact@v4
  with:
    name: build
    path: target/*.jar
```
Referência:
[https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts)

## 7. Cache

Reduz tempo de build reutilizando dependências.
```yaml
- uses: actions/cache@v4
  with:
    path: ~/.m2
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
```
Referência:
[https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)


# Pipeline Real — Java + Maven (CI)

Objetivo: construir pipeline de CI para projeto Java com Maven executando:
- Checkout
- Setup JDK
- Cache do repositório .m2
- Build
- Testes
- Upload do artefato

Baseado na documentação oficial:

[https://docs.github.com/en/actions](https://docs.github.com/en/actions)

[https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-java-with-maven](https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-java-with-maven)

## 📂 Estrutura Esperada
```
project-root
 ├── pom.xml
 └── src/
```
Workflow deve estar em:
```
.github/workflows/ci.yml
```

## 🔎 Arquitetura do Pipeline

Fluxo técnico:
```
push/pull_request
      ↓
Runner ubuntu-latest
      ↓
checkout
      ↓
setup-java
      ↓
cache ~/.m2
      ↓
mvn verify
      ↓
upload artifact
```

## 🧩 Workflow Completo (ci.yml)
```yaml
name: CI - Java Maven

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout código
        uses: actions/checkout@v4

      - name: Setup JDK 21
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '21'
          cache: maven

      - name: Build e Test
        run: mvn -B verify

      - name: Upload artefato
        uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: target/*.jar
```

## ⚙️ Explicação Técnica Detalhada
### 🔹 `actions/checkout@v4`

Clona o repositório no runner.

Sem isso, não há código para build.

Doc:
[https://github.com/actions/checkout](https://github.com/actions/checkout)

### 🔹 `actions/setup-java@v4`

Provisiona JDK no runner.

Parâmetros relevantes:
- distribution: temurin (recomendado)
- java-version: 17, 21, etc
- cache: maven: ativa cache automático do ~/.m2/repository

Doc:
[https://github.com/actions/setup-java](https://github.com/actions/setup-java)

Internamente:
- Calcula hash do `pom.xml`
- Cria chave de cache
- Restaura dependências se houver correspondência

### 🔹 `mvn -B verify`

Modo batch (`-B`) evita interação.

Fases executadas:
```
validate
compile
test
package
verify
```

Se testes falharem → job falha → PR bloqueado.

### 🔹 `actions/upload-artifact@v4`

Persiste o `.jar` gerado.

Disponível na aba:
```
Actions → Run → Artifacts
```

Doc:
[https://github.com/actions/upload-artifact](https://github.com/actions/upload-artifact)

## 🔐 Controle de Branch

Estratégia recomendada:
```yaml
on:
  pull_request:
    branches: [ main ]
```
Garante que:
- Todo PR para main execute testes
- Pode ser usado com branch protection rules

Doc:
[https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches)

## 🚀 Boas Práticas Técnicas
### 1️⃣ Fixar versão das actions

Evitar `@main`, usar `@v4`.

### 2️⃣ Usar Java LTS (21 atualmente)
### 3️⃣ Sempre usar -B
### 4️⃣ Separar jobs se necessário

Exemplo futuro:
- build
- test
- quality
- publish

## 📊 Resultado Esperado

Ao fazer:
```bash
git push origin develop
```
Você verá:
```
Repository → Actions → CI - Java Maven
```
Com:

- ✔ Checkout
- ✔ Setup Java
- ✔ Restore cache
- ✔ Maven build
- ✔ Upload artifact


# Pipeline — Spring Boot + Docker (Build e Push de Imagem)

Objetivo: estender a CI para:
- Build da aplicação Spring Boot
- Build da imagem Docker
- Tag baseada em SHA / branch
- Push para registry (Docker Hub ou GHCR)

Referências oficiais:

[https://docs.github.com/en/actions/publishing-packages/publishing-docker-images](https://docs.github.com/en/actions/publishing-packages/publishing-docker-images)

[https://docs.docker.com/build/ci/github-actions/](https://docs.docker.com/build/ci/github-actions/)

[https://github.com/docker/build-push-action](https://github.com/docker/build-push-action)

## 📦 Pré-requisitos no Repositório
### 1️⃣ Dockerfile na raiz

Exemplo otimizado (multi-stage):
```dockerfile
# Stage 1 - build
FROM maven:3.9.6-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn -B -q -e -C -DskipTests dependency:go-offline
COPY src ./src
RUN mvn -B package -DskipTests

# Stage 2 - runtime
FROM eclipse-temurin:21-jre
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]
```
Boas práticas aplicadas:
- Cache de dependências
- Imagem final leve (JRE only)
- Multi-stage isolando build

## 🔐 Configuração de Secrets

Se for usar Docker Hub:
```
Settings → Secrets and variables → Actions
```
Criar:
```
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
```
Se usar GHCR:
```
GITHUB_TOKEN (já disponível automaticamente)
```

## 🧩 Workflow Completo — ci-docker.yml
```yaml
name: CI - Spring Boot Docker

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup JDK 21
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '21'
          cache: maven

      - name: Build aplicação
        run: mvn -B clean package -DskipTests

      - name: Login Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build e Push imagem
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: usuario/app:${{ github.sha }}
```

## 🔎 Explicação Técnica
### `🔹 docker/login-action@v3`

Realiza autenticação segura no registry.
- Internamente:
- Executa docker login
- Usa token armazenado como secret

Doc:
[https://github.com/docker/login-action](https://github.com/docker/login-action)

### 🔹 `docker/build-push-action@v5`

Action oficial baseada em BuildKit.

Características:
- Build multi-arch
- Cache remoto
- Tag dinâmica
- Inline cache

Parâmetros principais:
```yaml
with:
  context: .
  push: true
  tags: usuario/app:${{ github.sha }}
```
github.sha → garante rastreabilidade por commit.

Doc:
[https://github.com/docker/build-push-action](https://github.com/docker/build-push-action)

## 🏷 Estratégia de Tag Recomendada

Mais robusta:
```yaml
tags: |
  usuario/app:latest
  usuario/app:${{ github.ref_name }}
  usuario/app:${{ github.sha }}
```
Resultado:
- latest
- branch
- commit específico

## 🔄 Fluxo Final
```
push main
   ↓
checkout
   ↓
maven package
   ↓
docker build (multi-stage)
   ↓
docker push
   ↓
imagem disponível no registry
```

## 🔐 Segurança

Recomendações:
- Nunca usar senha, sempre token
- Definir permissões explícitas:
```yaml
permissions:
  contents: read
  packages: write
```
Ativar branch protection
