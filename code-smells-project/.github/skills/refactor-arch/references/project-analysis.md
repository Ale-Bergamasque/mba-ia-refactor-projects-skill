# Referência: Análise de Projetos

## 1. Objetivo

Orientar a análise inicial de qualquer codebase, identificando sua linguagem, framework, dependências, banco de dados, domínio e arquitetura atual.

A análise deve ser baseada em evidências encontradas nos arquivos do projeto, sem presumir tecnologias ou estruturas.

## 2. Identificação da linguagem de programação

Examine os arquivos e suas extensões para identificar as linguagens utilizadas.

Utilize como referência:

| Arquivo ou extensão | Linguagem provável |
| ------------------- | ------------------ |
| .py                 | Python             |
| .js, .mjs, .cjs     | JavaScript         |
| .ts, .tsx           | TypeScript         |
| .java               | Java               |
| .php                | PHP                |
| .rb                 | Ruby               |
| .go                 | Go                 |
| .cs                 | C#                 |
| .rs                 | Rust               |

### Regras

- Examine os arquivos de configuração e dependências para confirmar a linguagem.
- Considere que um projeto pode utilizar mais de uma linguagem.
- Identifique a linguagem predominante na aplicação.
- Identifique versões quando houver evidências, como arquivos de configuração ou declarações de runtime.
- Não determine a linguagem apenas pelo nome do projeto.

## 3. Identificação do framework

Examine os arquivos de dependências e configuração para identificar frameworks e bibliotecas principais.

Exemplos de evidências:

| Tecnologia     | Arquivos ou dependências indicativas          |
| -------------- | --------------------------------------------- |
| Python         | requirements.txt, pyproject.toml, Pipfile     |
| Flask          | Dependência flask                             |
| Django         | Dependência django                            |
| FastAPI        | Dependência fastapi                           |
| Node.js        | package.json                                  |
| Express        | Dependência express                           |
| NestJS         | Dependência @nestjs/core                      |
| React          | Dependência react                             |
| Java / Spring  | pom.xml, build.gradle e dependências Spring   |
| PHP / Laravel  | composer.json e dependência laravel/framework |
| Ruby / Rails   | Gemfile e dependência rails                   |
| Go             | go.mod e dependências importadas              |
| .NET / ASP.NET | Arquivos .csproj e referências ASP.NET        |
| Rust           | Cargo.toml e dependências correspondentes     |

### Regras

- Diferencie linguagem de framework e runtime.
- Não confunda bibliotecas auxiliares com frameworks principais.
- Identifique versões quando disponíveis.
- Se não houver evidências suficientes, informe que o framework não foi identificado.

## 4. Identificação de dependências

Examine os arquivos de gerenciamento de dependências.

Exemplos:

- Python: requirements.txt, pyproject.toml e Pipfile.
- JavaScript/Node.js: package.json e arquivos de lock.
- Java: pom.xml e build.gradle.
- PHP: composer.json.
- Ruby: Gemfile.
- Go: go.mod.
- Rust: Cargo.toml.
- .NET: arquivos .csproj.

Identifique:

1. Dependências principais.
2. Bibliotecas relacionadas ao banco de dados.
3. Bibliotecas de autenticação e segurança.
4. Ferramentas de testes.
5. Dependências relevantes para a arquitetura.

Não é necessário listar todas as dependências. Priorize as que ajudam a compreender o funcionamento e a estrutura da aplicação.

## 5. Identificação do banco de dados

Investigue possíveis tecnologias de armazenamento examinando:

- Arquivos de configuração.
- Variáveis de ambiente.
- Dependências de drivers e ORMs.
- Arquivos de conexão.
- Migrations e schemas.
- Scripts SQL.
- Modelos de dados.

Exemplos de evidências:

| Tecnologia | Indícios              |
| ---------- | --------------------- |
| PostgreSQL | psycopg, pg, Npgsql   |
| MySQL      | mysqlclient, mysql2   |
| SQLite     | Arquivos .db, sqlite3 |
| MongoDB    | pymongo, mongoose     |
| Redis      | redis, ioredis        |

### Regras

- Diferencie dependência instalada de banco efetivamente utilizado.
- Procure confirmar a tecnologia através das configurações e conexões.
- Identifique as principais entidades ou tabelas quando possível.
- Nunca exponha valores de credenciais ou segredos encontrados nos arquivos.
- Se não houver evidências suficientes, informe que o banco de dados não foi identificado.
- Não execute comandos destrutivos para investigar o banco.

## 6. Identificação do domínio da aplicação

O domínio representa o problema de negócio que o sistema resolve.

Investigue:

- README e documentação.
- Nomes de módulos e diretórios.
- Rotas e endpoints.
- Modelos e entidades.
- Serviços e regras de negócio.
- Nomes de tabelas e campos.

Exemplos:

- E-commerce: produtos, usuários, pedidos e pagamentos.
- Task Manager: tarefas, usuários, projetos e status.
- LMS: cursos, alunos, matrículas e checkout.

Apresente uma descrição curta do propósito da aplicação e suas principais funcionalidades.

Não deduza funcionalidades que não tenham evidências no código.

## 7. Mapeamento da arquitetura atual

Examine a organização dos diretórios e identifique onde estão as principais responsabilidades.

Procure por:

| Responsabilidade    | Possíveis localizações                 |
| ------------------- | -------------------------------------- |
| Modelos e entidades | models/, entities/, domain/            |
| Rotas e endpoints   | routes/, routers/, endpoints/, views/  |
| Controllers         | controllers/, handlers/                |
| Regras de negócio   | services/, usecases/, domain/          |
| Acesso a dados      | repositories/, database/, persistence/ |
| Configurações       | config/, settings/                     |
| Middlewares         | middlewares/, middleware/              |
| Tratamento de erros | errors/, exceptions/, handlers/        |
| Ponto de entrada    | app.py, main.py, server.js, index.js   |

### Regras de análise

1. Identifique a localização das responsabilidades.
2. Verifique se existem separações reais entre as camadas.
3. Investigue se arquivos concentram responsabilidades excessivas.
4. Identifique dependências entre módulos.
5. Verifique se regras de negócio estão misturadas com rotas ou acesso a dados.
6. Identifique pontos de entrada e inicialização da aplicação.
7. Observe se existe uma organização arquitetural definida ou apenas separação de pastas.

### Classificação da arquitetura

Descreva a arquitetura encontrada utilizando categorias como:

- Monolítica sem separação clara de responsabilidades.
- Parcialmente organizada em camadas.
- MVC.
- Arquitetura em camadas.
- Modular.
- Outra arquitetura identificada.

A classificação deve considerar o comportamento e as responsabilidades do código, não somente os nomes dos diretórios.

## 8. Estratégia de inspeção dos arquivos

Para evitar análises incompletas ou desperdício de recursos:

1. Comece pela estrutura de diretórios.
2. Leia os arquivos de configuração e dependências.
3. Identifique os pontos de entrada.
4. Examine rotas, controllers, models e serviços.
5. Investigue os módulos responsáveis por persistência e regras de negócio.
6. Amplie a inspeção quando encontrar indícios de problemas ou dependências importantes.

### Arquivos que devem ser ignorados na contagem de código-fonte

Não contabilize como arquivos-fonte analisados:

- node_modules/
- .venv/
- venv/
- **pycache**/
- .git/
- dist/
- build/
- Diretórios de dependências externas.

Arquivos gerados automaticamente também devem ser excluídos quando não forem relevantes para a análise.

Informe separadamente limitações de acesso ou arquivos que não puderam ser inspecionados.

## 9. Formato da saída da Fase 1

Apresente os resultados utilizando a seguinte estrutura:

### PROJECT ANALYSIS

**Language:** Linguagem identificada e versão.

**Framework:** Framework identificado e versão.

**Dependencies:** Principais dependências.

**Database:** Banco de dados identificado ou não identificado.

**Domain:** Descrição do domínio da aplicação.

**Architecture:** Classificação e justificativa da arquitetura atual.

**Entry Points:** Principais arquivos de inicialização.

**Source Files:** Quantidade de arquivos-fonte analisados.

### Architecture Overview

Descreva resumidamente:

- Como o projeto está organizado.
- Onde estão as principais responsabilidades.
- Quais separações de camadas existem.
- Quais problemas estruturais aparentes merecem investigação na Fase 2.

### Analysis Limitations

Informe arquivos inacessíveis, informações não identificadas e limitações encontradas.

## 10. Princípio de neutralidade tecnológica

As heurísticas apresentadas são exemplos, não uma lista fechada de tecnologias suportadas.

A análise deve:

- Adaptar-se à linguagem e ao framework encontrados.
- Consultar os arquivos e convenções próprios de cada tecnologia.
- Não exigir uma estrutura de diretórios específica.
- Não pressupor que todo projeto utiliza MVC.
- Não considerar a ausência de uma pasta como evidência isolada de um problema.
- Priorizar evidências concretas do código.

O objetivo é compreender a arquitetura existente antes de recomendar qualquer transformação.
