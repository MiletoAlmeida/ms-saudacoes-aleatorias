# Deploy ms-saudacoes-aleatorias

Este repositório contém uma API simples em Go para gerenciar saudações aleatórias, mas o **principal objetivo deste projeto foi a implementação e demonstração de uma pipeline de CI/CD utilizando GitHub Actions e Terraform para deploy contínuo em uma plataforma como serviço (PaaS) como o Koyeb.**

A API em si serve como um exemplo prático para validar o funcionamento do pipeline de entrega contínua, garantindo que o código seja testado, empacotado em um contêiner Docker e, finalmente, implantado automaticamente.

Este projeto faz parte do desafio de CI/CD do Bootcamp de DevOps da Atlantico Avanti. 

## Visão Geral da API (Apenas para Contexto)

A API fornece dois endpoints básicos:

* **POST `/api/saudacoes`**: Para adicionar uma nova saudação ao banco de dados.
* **GET `/api/saudacoes/aleatorio`**: Para obter uma saudação aleatória do banco de dados.

A aplicação utiliza [Gin Gonic](https://gin-gonic.com/) como framework web e [SQLite](https://www.sqlite.org/index.html) como banco de dados embarcado.

## O Foco Principal: Pipeline de CI/CD com GitHub Actions e Terraform

O coração deste projeto reside na sua automação de CI/CD. O pipeline é projetado para garantir que qualquer alteração no código seja validada, construída e implantada de forma eficiente e confiável.

### Componentes Principais do Pipeline:

* **GitHub Actions**: Orquestra todo o fluxo de CI/CD, desde a validação do código até o deploy.
* **Go**: Linguagem de programação da API.
* **Docker**: Utilizado para empacotar a aplicação em um contêiner leve e portátil.
* **Terraform**: Gerencia a infraestrutura como código (IaC) no Koyeb, garantindo que o deploy e o destroy do serviço sejam automatizados e versionados.
* **Koyeb**: Plataforma como Serviço (PaaS) onde a aplicação é implantada.

### Estágios do Pipeline:

O workflow de CI/CD é composto pelos seguintes estágios (Jobs):

1.  **`lint`**:
    * **Propósito**: Garante a qualidade e conformidade do código-fonte Go.
    * **Ações**: Executa `go fmt`, `go vet` e `golangci-lint`.
    * **Condição**: Roda em todas as branches, exceto `main`.

2.  **`test`**:
    * **Propósito**: Valida a lógica de negócios da aplicação através de testes unitários.
    * **Ações**: Executa os testes Go e gera um relatório JUnit.
    * **Dependência**: Depende do sucesso do estágio `lint`.
    * **Condição**: Roda em todas as branches, exceto `main`.

3.  **`build-and-push`**:
    * **Propósito**: Constrói a imagem Docker da aplicação e a publica no Docker Hub.
    * **Ações**: Utiliza Docker Buildx para builds multi-plataforma e faz login no Docker Hub para o push da imagem.
    * **Condição**: Executa apenas em pushes para a branch `main`.

4.  **`deploy`**:
    * **Propósito**: Implanta o serviço no Koyeb usando Terraform.
    * **Ações**: Inicializa, valida e aplica a configuração Terraform, criando ou atualizando o serviço no Koyeb.
    * **Dependência**: Depende do sucesso do estágio `build-and-push`.
    * **Condição**: Executa apenas em pushes para a branch `main`.
    * **Ambiente**: Configurado para o ambiente `staging` (no GitHub Actions).

5.  **`destroy`**:
    * **Propósito**: Desprovisiona a infraestrutura (serviço Koyeb) gerenciada pelo Terraform.
    * **Ações**: Inicializa e executa o `terraform destroy -auto-approve`.
    * **Condição**: **Acionamento manual** via `workflow_dispatch` no GitHub Actions. Isso permite controlar quando a infraestrutura é removida, sem que cada commit a exclua automaticamente.

### Como Acionar o Pipeline (Deploy e Destroy)

* **Deploy**: Faça um `git push` para a branch `main`. O pipeline detectará a alteração, rodará os testes/lints (se aplicável), construirá e fará o push da imagem Docker e, em seguida, executará o deploy no Koyeb.
* **Destroy**:
    1.  Navegue até a aba **"Actions"** no seu repositório GitHub.
    2.  Selecione o workflow **"CI/CD Pipeline"** no painel esquerdo.
    3.  Clique no botão **"Run workflow"** (geralmente no canto superior direito).
    4.  Confirme a branch (`main`) e clique em **"Run workflow"** novamente.
    5.  O job `destroy` será acionado e removerá o serviço no Koyeb.

### Requisitos para Execução

Para que este pipeline funcione, você precisará configurar os seguintes segredos no seu repositório GitHub (Settings > Secrets and variables > Actions):

* `DOCKER_PASS`: Senha do Docker Hub (para o `DOCKER_USER` definido nas variáveis de ambiente).
* `KOYEB_TOKEN`: Token de API do Koyeb com permissões para gerenciar aplicações/serviços.

## Estrutura do Projeto

```mermaid
graph TD
    A[ms-saudacoes-aleatorias/] --> B[.github/]
    B --> B1[workflows/]
    B1 --> B2[main.yml - Definição do pipeline de CI/CD GitHub Actions]

    A --> C[infra/]
    C --> C1[main.tf - Configuração principal do Terraform para o Koyeb]
    C --> C2[variables.tf - Definição de variáveis do Terraform]

    A --> D[main.go - Ponto de entrada da aplicação Go]
    A --> E[go.mod - Módulos Go]
    A --> F[go.sum - Checksums dos módulos Go]

    A --> G[handlers/]
    G --> G1[handlers.go - Lógica dos handlers da API]

    A --> H[database/]
    H --> H1[database.go - Lógica de conexão e operações com o banco de dados]

    A --> I[Dockerfile - Definição da imagem Docker da aplicação]
    A --> J[README.md - Este arquivo]
