# Criação de Skills — Refatoração Arquitetural Automatizada

Ao longo do curso você aprendeu o que são Skills e como elas permitem que um agente de IA atue como um especialista em tarefas específicas. Agora imagine o seguinte cenário: você herdou 3 projetos legados com problemas de arquitetura, segurança e qualidade de código. Revisar e corrigir tudo manualmente levaria dias.

Neste desafio, você vai criar uma Skill que automatiza esse processo — analisando, auditando e refatorando qualquer projeto para o padrão MVC, independente da tecnologia.

## Objetivo

Você deve entregar uma Skill capaz de:

- Analisar uma codebase detectando linguagem, framework e arquitetura atual
- Identificar anti-patterns e code smells, classificando por severidade com arquivo e linha exatos
- Gerar um relatório de auditoria estruturado com todos os achados
- Refatorar o projeto para o padrão MVC (Model-View-Controller), eliminando os problemas encontrados
- Validar o resultado garantindo que a aplicação continua funcionando após as mudanças

A skill deve ser agnóstica de tecnologia, funcionando com diferentes linguagens e frameworks.

## Contexto

### Definição de Severidades

Para padronizar a sua auditoria e os relatórios gerados pela IA, utilize a seguinte escala de classificação baseada em problemas de MVC e SOLID:

- **CRITICAL:** Falhas graves de arquitetura ou segurança que impedem o funcionamento correto, expõem dados sensíveis (ex: credenciais hardcoded, SQL Injection) ou violam completamente a separação de responsabilidades (ex: "God Class" contendo banco de dados, lógicas complexas e roteamento no mesmo arquivo).
- **HIGH:** Fortes violações do padrão MVC ou princípios SOLID que dificultam muito a manutenção e testes (ex: lógicas de negócio pesadas presas dentro de Controllers, forte acoplamento sem Injeção de Dependência, ou uso de estado global mutável em toda a aplicação).
- **MEDIUM:** Problemas de padronização, duplicação de código ou gargalos de performance moderada (ex: Queries N+1 no banco de dados, uso inadequado de middlewares, validações ausentes nas rotas).
- **LOW:** Melhorias de legibilidade, nomenclatura de variáveis ruins, ou "magic numbers" soltos pelo código.

### Exemplo de Uso no CLI

```bash
# Executar a skill no projeto com problemas
cd code-smells-project
claude "/refactor-arch"
```

```
================================
PHASE 1: PROJECT ANALYSIS
================================
Language:      Python
Framework:      Flask 3.1.1
Dependencies:  flask-cors
Domain:        E-commerce API (produtos, pedidos, usuários)
Architecture:  Monolítica — tudo em 4 arquivos, sem separação de camadas
Source files:  4 files analyzed
DB tables:     produtos, usuarios, pedidos, itens_pedido
================================
```

```
================================
ARCHITECTURE AUDIT REPORT
================================
Project: code-smells-project
Stack:   Python + Flask
Files:   4 analyzed | ~800 lines of code

## Summary
CRITICAL: 4 | HIGH: 5 | MEDIUM: 2 | LOW: 3

## Findings

### [CRITICAL] God Class / God Method
File: models.py:1-350
Description: Arquivo único contém toda lógica de negócio, queries SQL, validação e formatação para 4 domínios diferentes.
Impact: Impossível testar em isolamento, qualquer mudança afeta tudo.
Recommendation: Separar em models e controllers por domínio.

### [CRITICAL] Hardcoded Credentials
File: app.py:8
Description: SECRET_KEY hardcoded como 'minha-chave-super-secreta-123'
...

================================
Total: 14 findings
================================

Phase 2 complete. Proceed with refactoring (Phase 3)? [y/n]
> y
```

```
[... refatoração executada ...]

================================
PHASE 3: REFACTORING COMPLETE
================================
## New Project Structure
src/
├── config/settings.py
├── models/
│   ├── produto_model.py
│   └── usuario_model.py
├── views/
│   └── routes.py
├── controllers/
│   ├── produto_controller.py
│   └── pedido_controller.py
├── middlewares/error_handler.py
└── app.py (composition root)

## Validation
  ✓ Application boots without errors
  ✓ All endpoints respond correctly
  ✓ Zero anti-patterns remaining
================================
```

## Tecnologias obrigatórias

- **Ferramenta:** uma das três opções abaixo (não são aceitas outras ferramentas):
  - Claude Code
  - Gemini CLI
  - OpenAI Codex
- **Recurso:** Custom Skills (ou o equivalente na ferramenta escolhida)
- **Formato dos arquivos de referência:** Markdown
- **Projetos-alvo:** Python/Flask (2 projetos) e Node.js/Express (1 projeto) (fornecidos no repositório base)

> **Nota sobre a ferramenta:** Os exemplos deste documento usam o Claude Code (`.claude/skills/`) como referência, pois é a ferramenta utilizada no curso. Se você optar por Gemini CLI ou Codex, adapte o nome da pasta e o comando de invocação conforme a convenção dela — o conceito de skill e a estrutura interna (SKILL.md + arquivos de referência) permanecem os mesmos.

## Requisitos

### 1. Análise Manual dos Projetos

Antes de criar a skill, você deve entender os problemas que ela vai resolver.

**Tarefas:**

- Analisar o projeto `code-smells-project/` (Python/Flask — API de E-commerce)
- Analisar o projeto `ecommerce-api-legacy/` (Node.js/Express — LMS API com fluxo de checkout)
- Analisar o projeto `task-manager-api/` (Python/Flask — API de Task Manager)

Para cada projeto, identificar e documentar no mínimo 5 problemas, incluindo pelo menos:

- 1 de severidade CRITICAL ou HIGH
- 2 de severidade MEDIUM
- 2 de severidade LOW

Documentar os achados na seção "Análise Manual" do seu `README.md`

> **Dica:** Não precisa encontrar todos os problemas — foque nos que têm maior impacto arquitetural. Use os projetos como insumo para entender quais padrões sua skill precisa detectar.

> **Por que 3 projetos?** Dois são Python/Flask (com níveis de organização diferentes) e um é Node.js/Express. Sua skill precisa funcionar nos 3 para provar que é verdadeiramente agnóstica de tecnologia — lidando tanto com código completamente desestruturado quanto com projetos que já possuem alguma separação de camadas.

## Análise Manual

## Projeto 1 — code-smells-project (Python/Flask)

### 1. Identificação do projeto

| Item                | Informação                                                                                            |
| ------------------- | ----------------------------------------------------------------------------------------------------- |
| Linguagem           | Python                                                                                                |
| Framework           | Flask 3.1.1                                                                                           |
| Banco de dados      | SQLite                                                                                                |
| Dependências        | `flask==3.1.1`; `flask-cors==5.0.1`                                                                   |
| Domínio             | API de e-commerce, com produtos, pedidos e usuários                                                   |
| Arquivos analisados | `app.py`, `controllers.py`, `database.py` e `models.py`                                               |
| Arquitetura atual   | Monolítica, com responsabilidades concentradas em quatro arquivos e sem separação adequada de camadas |

### 2. Análise dos problemas encontrados

#### CRITICAL

**Achado 1 — Execução arbitrária de SQL**

- Arquivo e linhas: `app.py:59-78`.
- Evidência: A API executa diretamente uma instrução SQL recebida do cliente.
- Problema: A aplicação permite a execução de SQL fornecido externamente.
- Impacto: Dados podem ser consultados ou alterados sem controle adequado.
- Recomendação: Remover a execução arbitrária, restringir as operações ao domínio e aplicar autenticação e autorização adequadas.

**Achado 2 — Segredos expostos e armazenamento inseguro de senhas**

- Arquivo e linhas: `app.py:7-8`; `controllers.py:276-290`; `database.py:75-82`.
- Evidência: Há chave secreta fixa, debug habilitado, endpoint de saúde expondo chave secreta, caminho do banco e estado de debug, além de senhas armazenadas em texto puro.
- Problema: Informações sensíveis são mantidas ou divulgadas de forma insegura.
- Impacto: A exposição pode facilitar comprometimento da aplicação e das credenciais dos usuários.
- Recomendação: Externalizar segredos, desabilitar debug em produção, restringir o endpoint de saúde e armazenar senhas com hash seguro.

**Achado 3 — SQL Injection**

- Arquivo e linhas: `models.py:43-52`.
- Evidência: Valores recebidos do usuário são concatenados diretamente em consultas SQL; o mesmo padrão aparece em outras funções de manipulação e busca.
- Problema: A construção das consultas permite que a entrada controle partes do SQL.
- Impacto: Um atacante pode acessar ou modificar dados indevidamente.
- Recomendação: Usar consultas parametrizadas e centralizar o acesso ao banco em repositórios seguros.

#### HIGH

**Achado 4 — Controllers com excesso de responsabilidades**

- Arquivo e linhas: `controllers.py:188-220`.
- Evidência: O controller mistura HTTP, validação, regras de negócio e notificações.
- Problema: A implementação viola responsabilidade única e concentra decisões de camadas diferentes.
- Impacto: A manutenção e os testes ficam mais difíceis, e mudanças têm maior risco de regressão.
- Recomendação: Delegar regras de negócio a services, validação a schemas e notificações a um componente próprio.

**Achado 5 — Models com responsabilidades misturadas**

- Arquivo e linhas: `models.py:133-169`.
- Evidência: A função combina persistência, validação de estoque, cálculo de valores, criação de pedidos e montagem de respostas.
- Problema: O model acumula responsabilidades de domínio, persistência e apresentação.
- Impacto: O código fica fortemente acoplado e difícil de testar em isolamento.
- Recomendação: Separar entidades, repositórios, services e serialização de respostas.

#### MEDIUM

**Achado 6 — Risco de inconsistência em transações e concorrência**

- Arquivo e linhas: `models.py:139-168`.
- Evidência: A verificação e a atualização de estoque não são protegidas adequadamente contra concorrência.
- Problema: Operações relacionadas podem ocorrer sem proteção transacional suficiente.
- Impacto: Requisições simultâneas podem produzir estoque ou pedidos inconsistentes.
- Recomendação: Usar transações e mecanismos de controle de concorrência apropriados.

**Achado 7 — Consultas N+1 e duplicação de código**

- Arquivo e linhas: `models.py:171-233`.
- Evidência: Consultas adicionais são realizadas dentro de loops e há duplicação na montagem de pedidos.
- Problema: O fluxo repete acesso ao banco e lógica de construção de dados.
- Impacto: Há degradação de desempenho e maior custo de manutenção.
- Recomendação: Reestruturar as consultas para carregar os dados necessários em conjunto e extrair a lógica compartilhada.

**Achado 8 — Validação inconsistente**

- Arquivo e linhas: `controllers.py:24-58` e `controllers.py:64-93`.
- Evidência: As validações aplicadas na criação não são reproduzidas adequadamente na atualização.
- Problema: Operações equivalentes aceitam conjuntos diferentes de entradas válidas.
- Impacto: Dados inválidos podem entrar no sistema durante atualizações.
- Recomendação: Centralizar e reutilizar os mesmos schemas e regras de validação.

#### LOW

**Achado 9 — Números mágicos**

- Arquivo e linhas: `models.py:256-262`.
- Evidência: Valores e percentuais são usados sem constantes ou nomes explicativos.
- Problema: O significado das regras numéricas não fica claro no código.
- Impacto: Alterações e revisões ficam mais sujeitas a erro.
- Recomendação: Substituir os valores por constantes nomeadas ou configurações explícitas.

**Achado 10 — Nomenclatura e importação desnecessária**

- Arquivo e linhas: `models.py:1-2` e `models.py:24-28`.
- Evidência: Há importação não utilizada e nomes genéricos.
- Problema: A organização e a intenção do código ficam menos legíveis.
- Impacto: A compreensão e a manutenção do módulo são prejudicadas.
- Recomendação: Remover importações não utilizadas e adotar nomes que expressem a responsabilidade de cada elemento.

### 3. Resumo quantitativo

| Severidade | Quantidade |
| ---------- | ---------: |
| CRITICAL   |          3 |
| HIGH       |          2 |
| MEDIUM     |          3 |
| LOW        |          2 |
| **TOTAL**  |     **10** |

### 4. APIs deprecated e dependências obsoletas

Não foi identificada API deprecated com evidência suficiente na análise do projeto. Não foi registrado achado de dependência obsoleta.

### 5. Arquitetura MVC recomendada

- **Models:** entidades e estruturas de domínio de produtos, pedidos e usuários.
- **Views/Routes:** definição dos endpoints Flask e serialização das respostas HTTP.
- **Controllers:** coordenação das requisições e delegação para os services.
- **Services:** regras de negócio, validações de fluxo, estoque, cálculo de valores e notificações.
- **Repositories:** consultas parametrizadas, persistência e transações SQLite.
- **Config:** configurações, segredos e seleção dos ambientes.
- **Middlewares:** CORS, autenticação, autorização e tratamento de requisições.
- **Tratamento centralizado de erros:** conversão consistente de exceções em respostas HTTP, sem expor dados sensíveis.

### 6. Observações da análise

- Os achados foram conferidos novamente no código original.
- Nenhum arquivo do projeto foi modificado durante a análise.
- O risco de concorrência é comprovável pela leitura do código, mas ainda não foi reproduzido por teste simultâneo.
- O resultado sobre APIs deprecated é limitado à evidência disponível nos arquivos analisados; não foi identificada API deprecated com evidência suficiente.

## Projeto 2 — ecommerce-api-legacy (Node.js/Express)

### 1. Identificação do projeto

| Item                | Informação                                                                                           |
| ------------------- | ---------------------------------------------------------------------------------------------------- |
| Linguagem           | JavaScript/Node.js                                                                                   |
| Framework           | Express                                                                                              |
| Banco de dados      | SQLite em memória                                                                                    |
| Dependências        | `express` e `sqlite3`                                                                                |
| Domínio             | LMS/e-commerce de cursos, com usuários, matrículas, pagamentos e relatórios financeiros              |
| Arquivos analisados | `app.js`, `AppManager.js`, `utils.js`, `package.json`, `package-lock.json`, `api.http` e `README.md` |
| Arquitetura atual   | Monolítica, com responsabilidades concentradas principalmente em `AppManager.js`                     |

### 2. Análise dos problemas encontrados

#### CRITICAL

**Achado 1 — Rotas administrativas sem autenticação e autorização**

- Arquivo e linhas: `AppManager.js:80-81` e `AppManager.js:131-137`.
- Evidência: Relatórios financeiros e exclusão de usuários podem ser acessados sem proteção adequada.
- Problema: Rotas administrativas não impõem autenticação e autorização suficientes.
- Impacto: Usuários não autorizados podem consultar informações financeiras ou excluir usuários.
- Recomendação: Adicionar middlewares de autenticação e autorização específicos para as operações administrativas.

**Achado 2 — Cartão exposto em logs e credenciais hardcoded**

- Arquivo e linhas: `AppManager.js:45` e `utils.js:1-6`.
- Evidência: O número completo do cartão é registrado nos logs e existem credenciais sensíveis diretamente no código.
- Problema: Dados de pagamento e credenciais são tratados de forma insegura.
- Impacto: Logs ou código expostos podem permitir uso indevido dessas informações.
- Recomendação: Remover dados completos de cartão dos logs e externalizar credenciais para configuração segura.

#### HIGH

**Achado 3 — Usuários existentes não são autenticados no checkout**

- Arquivo e linhas: `AppManager.js:40-46` e `AppManager.js:73-74`.
- Evidência: O sistema localiza o usuário pelo e-mail, mas não compara a senha informada.
- Problema: A identificação do usuário não comprova a posse da credencial.
- Impacto: O checkout pode ser executado em nome de outro usuário.
- Recomendação: Autenticar o usuário antes do checkout e aplicar autorização ao fluxo de matrícula e pagamento.

**Achado 4 — `AppManager` concentra responsabilidades**

- Arquivo e linhas: `AppManager.js:4-25` e `AppManager.js:28-137`.
- Evidência: A mesma classe mistura banco, SQL, rotas, checkout, pagamentos, relatórios e exclusão de usuários.
- Problema: A classe viola a separação de responsabilidades e concentra múltiplas camadas.
- Impacto: O acoplamento dificulta testes, manutenção e evolução da aplicação.
- Recomendação: Separar rotas, controllers, services, repositories e models por responsabilidade.

**Achado 5 — Armazenamento inseguro de senhas**

- Arquivo e linhas: `utils.js:17-23` e `AppManager.js:18`.
- Evidência: Base64 é usada para representar senhas e existe também senha inicial em texto puro.
- Problema: Base64 é apenas codificação e não oferece hash seguro.
- Impacto: As senhas podem ser recuperadas caso o armazenamento seja exposto.
- Recomendação: Usar algoritmo de hash apropriado para senhas e remover credenciais iniciais em texto puro.

#### MEDIUM

**Achado 6 — Checkout sem transação**

- Arquivo e linhas: `AppManager.js:50-63`.
- Evidência: Matrícula, pagamento e auditoria são operações independentes.
- Problema: O fluxo não garante atomicidade entre as operações relacionadas.
- Impacto: Uma falha intermediária pode deixar dados inconsistentes.
- Recomendação: Encapsular as operações do checkout em uma transação e tratar rollback.

**Achado 7 — Validação insuficiente de entradas**

- Arquivo e linhas: `AppManager.js:28-35`.
- Evidência: Faltam validações de senha obrigatória, e-mail, identificadores, cartão e tamanho do corpo JSON.
- Problema: Entradas inválidas ou excessivas podem chegar à lógica da aplicação.
- Impacto: A aplicação fica sujeita a falhas de integridade e comportamento inesperado.
- Recomendação: Aplicar schemas de validação nos endpoints e limitar o corpo das requisições.

**Achado 8 — Consultas N+1 no relatório**

- Arquivo e linhas: `AppManager.js:89-106`.
- Evidência: Consultas repetidas são realizadas dentro de loops.
- Problema: O relatório aumenta o número de acessos ao banco conforme a quantidade de registros.
- Impacto: O desempenho se degrada com o crescimento dos dados.
- Recomendação: Reestruturar as consultas para buscar os dados relacionados de forma agregada.

**Achado 9 — Tratamento incompleto de erros**

- Arquivo e linhas: `AppManager.js:104-106` e `AppManager.js:131-135`.
- Evidência: Erros de banco são ignorados e podem produzir respostas de sucesso incorretas.
- Problema: Falhas internas não são propagadas nem convertidas em respostas apropriadas.
- Impacto: Clientes podem receber confirmação de operações que falharam e o diagnóstico fica prejudicado.
- Recomendação: Propagar erros para um middleware centralizado e retornar códigos HTTP coerentes.

#### LOW

**Achado 10 — Cache global e configuração acoplada**

- Arquivo e linhas: `utils.js:1-14`.
- Evidência: Há estado global mutável, configuração misturada e variável não utilizada.
- Problema: Configuração e estado compartilhado ficam acoplados ao módulo utilitário.
- Impacto: O comportamento pode variar entre requisições e os testes ficam mais difíceis.
- Recomendação: Isolar configuração, evitar estado global mutável e remover variáveis sem uso.

**Achado 11 — Nomenclatura abreviada e valores mágicos**

- Arquivo e linhas: `AppManager.js:29-33`, `AppManager.js:45-46` e `api.http:7-12`.
- Evidência: Há nomes pouco expressivos e valores fixos sem significado documentado.
- Problema: A intenção dos dados e das regras não é clara para quem mantém o código.
- Impacto: A leitura, a manutenção e a alteração dos fluxos ficam mais sujeitas a erro.
- Recomendação: Usar nomes descritivos e substituir valores fixos por constantes ou configuração nomeada.

### 3. Resumo quantitativo

| Severidade | Quantidade |
| ---------- | ---------: |
| CRITICAL   |          2 |
| HIGH       |          3 |
| MEDIUM     |          4 |
| LOW        |          2 |
| **TOTAL**  |     **11** |

### 4. APIs deprecated e dependências obsoletas

Não foi identificada chamada direta de API deprecated no código da aplicação. Foram identificadas dependências transitivas marcadas como deprecated no `package-lock.json`; isso é diferente do uso direto de uma API obsoleta pelo código da aplicação.

### 5. Arquitetura MVC recomendada

- **Models:** entidades de usuários, cursos, matrículas e pagamentos.
- **Views/Routes:** definição dos endpoints Express, parsing das requisições e serialização das respostas.
- **Controllers:** coordenação das requisições HTTP e delegação para os services.
- **Services:** checkout, matrícula, pagamentos, relatórios e demais regras de negócio.
- **Repositories:** persistência, consultas SQL e transações SQLite.
- **Config:** configurações e variáveis de ambiente, incluindo credenciais externas.
- **Middlewares:** autenticação, autorização, validação, limite de corpo e logging sem dados sensíveis.
- **Tratamento centralizado de erros:** captura de falhas dos controllers e repositories, com respostas HTTP consistentes.

Essa separação reduz o acoplamento: Controllers deixam de conhecer detalhes do banco, Services concentram as regras de negócio e Repositories isolam o SQL. Cada camada pode ser testada separadamente.

### 6. Observações da análise

- Os achados foram conferidos no código e nos arquivos de configuração analisados.
- Nenhum arquivo do projeto foi modificado durante as análises.
- A chave `pk_live` não deve ser classificada automaticamente como segredo privado sem documentação do gateway.
- A senha do banco presente no código é uma informação sensível.
- A existência de dependências transitivas deprecated foi observada no `package-lock.json`, mas não foi identificada chamada direta de API deprecated.
- Os riscos de autenticação, autorização, transação e tratamento de erros exigem testes adicionais para comprovar o comportamento em execução.

## Projeto 3 — task-manager-api (Python/Flask)

### 1. Identificação do projeto

| Item                | Informação                                                                                                    |
| ------------------- | ------------------------------------------------------------------------------------------------------------- |
| Linguagem           | Python                                                                                                        |
| Framework           | Flask 3.0.0                                                                                                   |
| ORM/banco           | Flask-SQLAlchemy 3.1.1 com SQLite                                                                             |
| Dependências        | Flask, Flask-SQLAlchemy, Flask-CORS, Marshmallow, Requests e python-dotenv                                    |
| Domínio             | Gerenciamento de tarefas, usuários, categorias, autenticação e relatórios                                     |
| Arquivos analisados | 17 arquivos, sendo 15 arquivos Python                                                                         |
| Arquitetura atual   | Parcialmente organizada em `models/`, `routes/`, `services/` e `utils/`, mas com responsabilidades misturadas |

### 2. Análise dos problemas encontrados

#### CRITICAL

**TM-001 — Senhas expostas e protegidas com MD5**

- Arquivos e linhas: `models/user.py:21`, `models/user.py:29`, `models/user.py:32`; `routes/user_routes.py:81`, `routes/user_routes.py:130` e `routes/user_routes.py:210`.
- Evidência: `to_dict()` inclui `'password': self.password`; `set_password()` armazena `hashlib.md5(pwd.encode()).hexdigest()`; `check_password()` compara usando MD5; o login retorna `user.to_dict()`.
- Problema: O hash da senha é exposto nas respostas HTTP e o armazenamento utiliza MD5, que não é apropriado para senhas.
- Impacto: Exposição dos hashes permite ataques offline, e as respostas de criação, atualização e login divulgam material sensível.
- Recomendação: Remover a senha de toda serialização HTTP e migrar para Argon2id, bcrypt ou PBKDF2 com salt e custo adequado.

#### HIGH

**TM-002 — Segredos hardcoded**

- Arquivos e linhas: `app.py:13`; `services/notification_service.py:7-10` e `services/notification_service.py:17`.
- Evidência: `SECRET_KEY` é definida como `'super-secret-key-123'`; o serviço contém `smtp.gmail.com`, porta `587`, usuário `taskmanager@gmail.com` e senha `'senha123'`, utilizados em `server.login(...)`.
- Problema: Segredos e credenciais externas estão diretamente no código-fonte.
- Impacto: Um repositório ou artefato exposto pode permitir comprometimento da aplicação ou da conta SMTP.
- Recomendação: Externalizar os valores para variáveis de ambiente ou gerenciador de segredos e rotacionar credenciais comprometidas.
- Ressalva: Não foi possível confirmar se as credenciais SMTP são reais; o hardcode, porém, está comprovado.

**TM-003 — Token fictício e ausência de autenticação/autorização**

- Arquivos e linhas: `routes/user_routes.py:210`; `routes/task_routes.py:85`, `routes/task_routes.py:156-159` e `routes/task_routes.py:225`; `routes/user_routes.py:42`, `routes/user_routes.py:92` e `routes/user_routes.py:134`.
- Evidência: O login retorna `'fake-jwt-token-' + str(user.id)`, sem assinatura, expiração ou validação. Os endpoints de criação, atualização e exclusão não verificam usuário autenticado, proprietário do recurso ou papel administrativo.
- Problema: O token é previsível e não existe mecanismo efetivo de autenticação ou autorização para os endpoints protegidos.
- Impacto: Clientes não autenticados podem criar, alterar e excluir tarefas e usuários, inclusive alterar dados sensíveis de usuários.
- Recomendação: Implementar autenticação real com sessão segura ou JWT assinado, middleware de autenticação e autorização por recurso e papel.

**TM-004 — Debug habilitado e aplicação exposta em todas as interfaces**

- Arquivo e linha: `app.py:34`.
- Evidência: `app.run(debug=True, host='0.0.0.0', port=5000)`.
- Problema: O modo debug está habilitado e o servidor escuta em todas as interfaces de rede.
- Impacto: Em produção ou em uma rede acessível, podem ser expostos detalhes internos e o debugger do Flask.
- Recomendação: Desabilitar debug fora do desenvolvimento, configurar o ambiente explicitamente e usar um servidor WSGI apropriado.
- Ressalva: A severidade é HIGH em produção; o impacto depende da exposição efetiva. Em ambiente exclusivamente local, classificar como MEDIUM.

#### MEDIUM

**TM-005 — Consultas N+1 na listagem de tarefas**

- Arquivo e linhas: `routes/task_routes.py:14`, `routes/task_routes.py:42` e `routes/task_routes.py:51`.
- Evidência: `Task.query.all()` carrega as tarefas e, dentro do loop, `User.query.get(t.user_id)` e `Category.query.get(t.category_id)` executam consultas adicionais.
- Problema: A quantidade de consultas cresce com a quantidade de tarefas.
- Impacto: Latência e carga no banco aumentam em listas maiores, agravadas pela ausência de paginação.
- Recomendação: Usar `JOIN` ou `joinedload`, reutilizar relacionamentos ORM e adicionar paginação.

**TM-006 — Relatórios com carregamento completo e consultas repetitivas**

- Arquivo e linhas: `routes/report_routes.py:19-30` e `routes/report_routes.py:53-56`.
- Evidência: O relatório executa várias contagens separadas, carrega todas as tarefas com `Task.query.all()`, carrega todos os usuários com `User.query.all()` e executa `Task.query.filter_by(user_id=u.id).all()` para cada usuário.
- Problema: Dados completos são carregados em memória e consultas repetitivas são executadas em loops.
- Impacto: O consumo de memória e o tempo de resposta crescem com o volume de dados.
- Recomendação: Usar agregações SQL, `GROUP BY` e `COUNT`, além de evitar carregar entidades completas quando somente contagens são necessárias.

#### LOW

**TM-007 — Uso de API legada `Query.get()`**

- Arquivos e linhas: `routes/task_routes.py:42`, `routes/task_routes.py:67` e `routes/task_routes.py:158`; `routes/user_routes.py:29` e `routes/user_routes.py:94`.
- Evidência: O código usa `User.query.get(...)` e `Task.query.get(...)`.
- Problema: `Query.get()` é uma API legada no padrão atual do SQLAlchemy 2.x.
- Impacto: Warnings de compatibilidade e maior custo de migração futura.
- Recomendação: Utilizar `db.session.get(Model, id)`.

**TM-008 — Tratamento genérico de exceções**

- Arquivos e linhas: `routes/task_routes.py:62` e `routes/task_routes.py:236`; `routes/user_routes.py:130` e `routes/user_routes.py:149`; `routes/report_routes.py:186`.
- Evidência: Existem blocos `except:` sem tipo que retornam mensagens genéricas.
- Problema: Qualquer exceção é capturada e a causa real não é registrada de forma adequada.
- Impacto: Erros de programação podem ser ocultados e o diagnóstico fica prejudicado.
- Recomendação: Capturar exceções específicas, registrar logs estruturados e centralizar o tratamento de erros HTTP.

### 3. Achados complementares

Os achados abaixo são reais, mas ficam separados dos oito principais para não misturar problemas arquiteturais e de qualidade com os achados prioritários de segurança, performance e compatibilidade.

**TM-009 — Rotas concentram responsabilidades excessivas**

- Severidade: MEDIUM.
- Referências: `routes/task_routes.py:85-152`, `routes/task_routes.py:156-220` e `routes/user_routes.py:42-89`.
- Evidência: As rotas misturam parsing HTTP, validação, regras de negócio, consultas, transações e serialização.
- Impacto: Maior acoplamento, dificuldade de teste e risco de regressões.
- Recomendação: Delegar regras a services, validação a schemas e persistência a repositories.

**TM-010 — Duplicação de código**

- Severidade: LOW.
- Referências: `routes/task_routes.py:27-38`, `routes/task_routes.py:69-78`, `routes/user_routes.py:165-179` e `routes/report_routes.py:32-43`.
- Evidência: O cálculo de tarefas atrasadas aparece repetido; validações de tarefas também são repetidas entre criação e atualização.
- Impacto: Correções podem ser aplicadas em um fluxo e esquecidas em outro.
- Recomendação: Extrair regras compartilhadas para services ou funções de domínio testáveis.

**TM-011 — Models acumulam responsabilidades**

- Severidade: LOW.
- Referências: `models/user.py:17-36` e `models/task.py:26-67`.
- Evidência: Os models combinam persistência ORM, serialização, hashing/verificação de senha e validações de domínio.
- Impacto: A separação entre entidade, segurança e representação HTTP fica prejudicada.
- Recomendação: Manter entidades e relacionamentos nos models e mover schemas, autenticação e serialização para componentes próprios.

**TM-012 — Serviço de notificações mistura responsabilidades**

- Severidade: LOW.
- Referências: `services/notification_service.py:4-10`, `services/notification_service.py:12-24` e `services/notification_service.py:27-42`.
- Evidência: O serviço configura SMTP, envia e-mail e mantém notificações em uma lista em memória.
- Impacto: O estado é perdido no reinício e responsabilidades externas ficam acopladas ao armazenamento temporário.
- Recomendação: Separar configuração, transporte de e-mail e persistência ou fila de notificações.

**TM-013 — Constantes centralizadas não utilizadas**

- Severidade: LOW.
- Referências: `utils/helpers.py`; `routes/task_routes.py:109`; `routes/user_routes.py:72`.
- Evidência: `utils/helpers.py` define constantes para status, papéis, limites e prioridade, mas as rotas repetem os valores diretamente.
- Impacto: Alterações podem produzir validações inconsistentes.
- Recomendação: Centralizar as regras e reutilizar as constantes ou schemas.

**TM-014 — Dependências declaradas sem uso evidente**

- Severidade: LOW.
- Dependências: `marshmallow`, `requests` e `python-dotenv`.
- Evidência: Não há uso evidente dessas dependências nos arquivos analisados.
- Impacto: Aumentam a superfície de manutenção e podem gerar confusão sobre as capacidades realmente utilizadas.
- Recomendação: Confirmar o uso em toda a aplicação e remover dependências não utilizadas.
- Ressalva: Achado parcialmente confirmado; a análise estática dos arquivos presentes não comprova ausência de uso externo.

### 4. Resumo quantitativo

#### Achados principais

| Severidade | Quantidade |
| ---------- | ---------: |
| CRITICAL   |          1 |
| HIGH       |          3 |
| MEDIUM     |          2 |
| LOW        |          2 |
| **TOTAL**  |      **8** |

#### Achados complementares

6 achados adicionais:

- 1 MEDIUM.
- 5 LOW.

### 5. Limitações e riscos não comprovados

- **SQL Injection:** não confirmado; as consultas analisadas usam SQLAlchemy e não foi encontrada construção de SQL bruto com entrada do usuário.
- **Brute force:** risco plausível no login, mas exige testes adicionais de taxa e comportamento.
- **CORS excessivamente permissivo:** `CORS(app)` existe, mas o impacto depende da configuração efetiva e do ambiente.
- **Integridade referencial na exclusão de categorias:** exige testes e confirmação da configuração de foreign keys do SQLite.
- **Dependências transitivas vulneráveis:** não foram auditadas.
- **Debug:** o risco é condicionado ao ambiente de execução e à exposição efetiva da aplicação.

### 6. APIs deprecated e dependências obsoletas

- Foi identificado uso direto de `Model.query.get()`, API legada do SQLAlchemy 2.x; a recomendação é `db.session.get(Model, id)`.
- `datetime.utcnow()` aparece em vários módulos e merece revisão para compatibilidade com versões recentes do Python. Não foi classificado automaticamente como API deprecated sem verificar a versão e a documentação aplicáveis.
- Não foram auditadas dependências transitivas quanto a vulnerabilidades ou obsolescência.

### 7. Arquitetura MVC recomendada

- **Models:** entidades e relacionamentos ORM de usuários, tarefas e categorias.
- **Views/Routes:** endpoints HTTP, parsing das requisições e serialização das respostas.
- **Controllers:** coordenação das requisições e delegação para os services.
- **Services:** regras de negócio, autenticação, autorização, tarefas, usuários e relatórios.
- **Repositories:** consultas, agregações e persistência SQLAlchemy.
- **Schemas:** validação e serialização com Marshmallow.
- **Config:** configurações por ambiente e segredos externos ao código.
- **Middlewares:** autenticação, autorização, CORS e tratamento centralizado de erros.
- **Tests:** validação dos endpoints, regras de negócio, segurança e relatórios.

Essa organização aproveita as pastas já existentes: `models/` continua responsável pelas entidades, `routes/` permanece como camada HTTP, `services/` recebe as regras de negócio e `utils/` pode concentrar helpers realmente compartilhados. Repositories, schemas, configuração e middlewares devem ser adicionados somente onde removem responsabilidades das rotas e melhoram a testabilidade, sem criar camadas artificiais.

### 8. Observações da análise

- Os oito achados principais foram conferidos novamente no código original.
- Os achados complementares são problemas reais de arquitetura ou qualidade, mas não devem ser somados aos oito principais.
- As ressalvas sobre credenciais SMTP, debug, CORS, brute force, integridade referencial e dependências transitivas foram preservadas.
- Nenhum arquivo do projeto foi modificado durante a análise manual.

### 2. Criação da Skill

Agora que você conhece os problemas, crie uma skill que os detecte, gere um relatório de auditoria e corrija automaticamente.

**Tarefas:**

Criar a skill dentro do projeto `code-smells-project/` e implementar o SKILL.md com 3 fases sequenciais:

- **Fase 1 — Análise:** Detectar stack, mapear arquitetura atual, imprimir resumo
- **Fase 2 — Auditoria:** Cruzar código contra catálogo de anti-patterns, gerar relatório, pedir confirmação
- **Fase 3 — Refatoração:** Reestruturar para o padrão MVC, validar que funciona

Criar arquivos de referência em Markdown que forneçam à skill o conhecimento necessário para executar as 3 fases. Os arquivos devem cobrir **obrigatoriamente** as seguintes áreas de conhecimento:

| Área de conhecimento      | O que deve conter                                                                                     |
| ------------------------- | ----------------------------------------------------------------------------------------------------- |
| Análise de projeto        | Heurísticas para detecção de linguagem, framework, banco de dados e mapeamento de arquitetura         |
| Catálogo de anti-patterns | Anti-patterns com sinais de detecção e classificação de severidade                                    |
| Template de relatório     | Formato padronizado do relatório de auditoria (Fase 2)                                                |
| Guidelines de arquitetura | Regras do padrão MVC alvo (camadas Models, Views/Routes e Controllers, responsabilidades de cada uma) |
| Playbook de refatoração   | Padrões concretos de transformação para cada anti-pattern (com exemplos de código)                    |

> **Nota:** Você tem liberdade para organizar os arquivos de referência como preferir — pode usar os nomes e a quantidade de arquivos que fizer sentido para sua skill. O importante é que todas as 5 áreas de conhecimento estejam cobertas. O nome da skill (`refactor-arch`) e o arquivo `SKILL.md` são obrigatórios e não devem ser alterados. O path da skill segue a convenção da ferramenta escolhida (no Claude Code, por exemplo, é `.claude/skills/refactor-arch/`).

**Requisitos da skill:**

- Deve ser agnóstica de tecnologia — deve funcionar corretamente nos 3 projetos fornecidos, independente da stack ou nível de organização
- O catálogo de anti-patterns deve conter no mínimo 8 anti-patterns com severidade distribuída (CRITICAL, HIGH, MEDIUM, LOW)
- O catálogo deve incluir detecção de APIs deprecated — identificar uso de APIs obsoletas e recomendar o equivalente moderno
- O playbook deve ter no mínimo 8 padrões de transformação com exemplos de código antes/depois
- A Fase 2 deve pausar e pedir confirmação antes de modificar qualquer arquivo
- A Fase 3 deve validar o resultado (boot da aplicação + endpoints funcionando)

### 3. Execução da Skill

Execute sua skill nos 3 projetos e valide que ela funciona em todas as stacks.

#### Projeto 1 — code-smells-project (Python/Flask)

Invocar a skill no Claude Code:

```bash
claude "/refactor-arch"
```

> **Nota:** O comando acima é o exemplo com Claude Code. Se você estiver usando Gemini CLI ou Codex, utilize o comando equivalente para invocar uma skill na sua ferramenta.

- Verificar que a Fase 1 detecta corretamente a stack e imprime o resumo
- Verificar que a Fase 2 encontra no mínimo 5 dos problemas documentados na sua análise manual
- Confirmar a execução da Fase 3
- Verificar que a Fase 3:
  - Cria a estrutura de diretórios baseada em MVC
  - A aplicação inicia sem erros
  - Os endpoints originais continuam respondendo
- Salvar o relatório de auditoria (output da Fase 2) em `reports/audit-project-1.md`
- Commitar o código refatorado do projeto no repositório

#### Projeto 2 — ecommerce-api-legacy (Node.js/Express)

Prove que sua skill é reutilizável em outro projeto de backend, mas com stack diferente.

- Copiar a pasta `.claude/skills/refactor-arch/` para dentro de `ecommerce-api-legacy/`
- Invocar a skill:

```bash
cd ../ecommerce-api-legacy
claude "/refactor-arch"
```

- Verificar que as 3 fases executam corretamente neste projeto
- Salvar o relatório em `reports/audit-project-2.md`
- Commitar o código refatorado do projeto no repositório

#### Projeto 3 — task-manager-api (Python/Flask)

Agora o teste com um projeto Python/Flask que já possui alguma organização de camadas (models, routes, services, utils).

- Copiar a pasta `.claude/skills/refactor-arch/` para dentro de `task-manager-api/`
- Invocar a skill:

```bash
cd ../task-manager-api
claude "/refactor-arch"
```

- Verificar que:
  - A Fase 1 detecta corretamente Python/Flask como stack e identifica o domínio de Task Manager
  - A Fase 2 identifica problemas mesmo em um projeto parcialmente organizado
  - A Fase 3 melhora a estrutura sem quebrar a aplicação (todos os endpoints devem continuar respondendo)
- Salvar o relatório em `reports/audit-project-3.md`
- Commitar o código refatorado do projeto no repositório

> **Nota:** Este projeto já possui alguma separação de camadas, mas isso não significa que a arquitetura está adequada. A skill deve identificar tanto problemas de código (segurança, performance, qualidade) quanto oportunidades de melhoria arquitetural. Se houver mudanças estruturais necessárias, a skill deve propô-las e executá-las.

#### Validação

Para cada projeto refatorado, valide o seguinte checklist:

```markdown
## Checklist de Validação

### Fase 1 — Análise

- [ ] Linguagem detectada corretamente
- [ ] Framework detectado corretamente
- [ ] Domínio da aplicação descrito corretamente
- [ ] Número de arquivos analisados condiz com a realidade

### Fase 2 — Auditoria

- [ ] Relatório segue o template definido nos arquivos de referência
- [ ] Cada finding tem arquivo e linhas exatos
- [ ] Findings ordenados por severidade (CRITICAL → LOW)
- [ ] Mínimo de 5 findings identificados
- [ ] Detecção de APIs deprecated incluída (se aplicável)
- [ ] Skill pausa e pede confirmação antes da Fase 3

### Fase 3 — Refatoração

- [ ] Estrutura de diretórios segue padrão MVC
- [ ] Configuração extraída para módulo de config (sem hardcoded)
- [ ] Models criados para abstrair dados
- [ ] Views/Routes separadas para visualização ou roteamento
- [ ] Controllers concentram o fluxo da aplicação
- [ ] Error handling centralizado
- [ ] Entry point claro
- [ ] Aplicação inicia sem erros
- [ ] Endpoints originais respondem corretamente
```

> **Dica:** Se a skill não detectou problemas suficientes ou a refatoração falhou, ajuste os arquivos de referência e execute novamente. É normal precisar de 2-4 iterações.

## Entregável

Repositório público no GitHub (fork do repositório base) contendo:

- Skill completa em `.claude/skills/refactor-arch/` (dentro dos 3 projetos)
- Código refatorado dos 3 projetos (resultado da execução da Fase 3, commitado no repositório)
- Relatórios de auditoria em `reports/` (3 arquivos)
- `README.md` atualizado

### Estrutura do repositório

Faça um fork do repositório base contendo os três projetos com code smells.

> **Nota:** A estrutura abaixo usa Claude Code como exemplo (`.claude/skills/`). Se estiver usando outra ferramenta, adapte os caminhos conforme a convenção dela.

```
desafio-skills/
├── README.md                              # Sua documentação
│
├── code-smells-project/                   # Projeto 1 — Python/Flask (API de E-commerce)
│   ├── .claude/
│   │   └── skills/
│   │       └── refactor-arch/             # ← SUA SKILL AQUI
│   │           ├── SKILL.md
│   │           └── (arquivos de referência)
│   ├── app.py
│   ├── controllers.py
│   ├── models.py
│   ├── database.py
│   └── requirements.txt
│
├── ecommerce-api-legacy/                  # Projeto 2 — Node.js/Express (LMS API com checkout)
│   ├── .claude/
│   │   └── skills/
│   │       └── refactor-arch/             # ← CÓPIA DA SKILL
│   │           └── ...
│   ├── src/
│   │   ├── app.js
│   │   ├── AppManager.js
│   │   └── utils.js
│   ├── api.http
│   └── package.json
│
├── task-manager-api/                      # Projeto 3 — Python/Flask (API de Task Manager)
│   ├── .claude/
│   │   └── skills/
│   │       └── refactor-arch/             # ← CÓPIA DA SKILL
│   │           └── ...
│   ├── app.py
│   ├── database.py
│   ├── seed.py
│   ├── requirements.txt
│   ├── models/
│   ├── routes/
│   ├── services/
│   └── utils/
│
└── reports/                               # Relatórios gerados
    ├── audit-project-1.md                 # Saída da Fase 2 no projeto 1
    ├── audit-project-2.md                 # Saída da Fase 2 no projeto 2
    └── audit-project-3.md                 # Saída da Fase 2 no projeto 3
```

**O que você vai criar:**

- `.claude/skills/refactor-arch/` — A skill completa (SKILL.md + arquivos de referência)
- Código refatorado dos 3 projetos — resultado da execução da Fase 3, commitado no repositório
- `reports/audit-project-{1,2,3}.md` — Relatório de auditoria de cada projeto
- `README.md` — Documentação do seu processo

**O que já vem pronto:**

- `code-smells-project/` — API de E-commerce Python/Flask com code smells intencionais
- `ecommerce-api-legacy/` — LMS API Node.js/Express (com fluxo de checkout) e problemas de implementação
- `task-manager-api/` — API de Task Manager Python/Flask com organização parcial e problemas de segurança/qualidade

> **Dica:** Cada projeto contém problemas intencionais de diferentes severidades (CRITICAL, HIGH, MEDIUM, LOW), incluindo falhas de segurança, violações arquiteturais e problemas de qualidade de código. Parte do desafio é identificá-los por conta própria através da análise manual do código.

### README.md deve conter

**A) Seção "Análise Manual":**

- Lista dos problemas identificados manualmente em cada projeto
- Classificação por severidade
- Justificativa de por que cada problema é relevante

**B) Seção "Construção da Skill":**

- Decisões de design: como estruturou o SKILL.md e os arquivos de referência
- Quais anti-patterns incluiu no catálogo e por quê
- Como garantiu que a skill é agnóstica de tecnologia
- Desafios encontrados e como resolveu

**C) Seção "Resultados":**

- Resumo dos relatórios de auditoria dos 3 projetos (quantos findings por severidade em cada)
- Comparação antes/depois da estrutura de cada projeto
- Checklist de validação preenchido para cada projeto
- Screenshots ou logs mostrando as aplicações rodando após refatoração
- Observações sobre como a skill se comportou em stacks diferentes

**D) Seção "Como Executar":**

- Pré-requisitos (a ferramenta escolhida — Claude Code, Gemini CLI ou Codex — instalada e configurada)
- Comandos para executar a skill em cada projeto
- Como validar que a refatoração funcionou

### Ordem de execução sugerida

**1. Analisar os projetos manualmente**

Leia o código dos três projetos e documente os problemas encontrados.

**2. Criar a skill**

Escreva o SKILL.md e os arquivos de referência.

**3. Executar nos 3 projetos**

```bash
# Projeto 1
cd code-smells-project
claude "/refactor-arch"

# Projeto 2
cd ../ecommerce-api-legacy
claude "/refactor-arch"

# Projeto 3
cd ../task-manager-api
claude "/refactor-arch"
```

Salve a saída da Fase 2 de cada projeto em `reports/audit-project-{1,2,3}.md`.

**4. Iterar**

Se a skill não detectou problemas suficientes ou a refatoração falhou, ajuste os arquivos de referência e execute novamente. É normal precisar de 2-4 iterações.

## Critérios de Aceite

A skill deve atingir os seguintes mínimos em **todos os 3 projetos**:

| Critério                                    | Requisito                  |
| ------------------------------------------- | -------------------------- |
| Fase 1 detecta stack corretamente           | OBRIGATÓRIO (3/3 projetos) |
| Fase 2 encontra >= 5 findings               | OBRIGATÓRIO (3/3 projetos) |
| Fase 2 inclui pelo menos 1 CRITICAL ou HIGH | OBRIGATÓRIO (3/3 projetos) |
| Fase 3 aplicação funciona após refatoração  | OBRIGATÓRIO (3/3 projetos) |

**IMPORTANTE:** Todos os critérios devem ser atingidos nos 3 projetos, não apenas em um!

> **Sobre o projeto 3 (task-manager-api):** Este projeto já possui alguma organização. "aplicação funciona" significa que a API inicia sem erros e todos os endpoints continuam respondendo corretamente.

## Referências

- [Claude Code: Skills](https://docs.anthropic.com/en/docs/claude-code/skills) — Documentação oficial sobre como criar e estruturar Skills
- [Claude Code: Overview](https://docs.anthropic.com/en/docs/claude-code/overview) — Visão geral do Claude Code e suas capacidades
- [The Complete Guide to Building Skills for Claude (PDF)](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf) — Guia completo da Anthropic sobre construção de Skills
- [Equipping Agents for the Real World with Agent Skills](https://claude.com/blog/equipping-agents-for-the-real-world-with-agent-skills) — Blog oficial da Anthropic sobre Agent Skills

---

## Dicas Finais

- **Comece pela análise manual** — entender os problemas profundamente é essencial para criar uma skill que os detecte.
- **O SKILL.md é um prompt** — ele instrui o agente sobre o que fazer, enquanto os arquivos de referência fornecem o conhecimento de domínio.
- **Seja específico nos sinais de detecção** — "código ruim" não ajuda; "query SQL dentro de loop for" é acionável.
- **Teste incrementalmente** — não tente criar a skill perfeita de primeira.
- **A skill deve ser copiável** — se ela só funciona em um projeto específico, está acoplada demais. Teste nos 3 projetos para validar.
- **Projetos diferentes exigem adaptação** — a Fase 3 de um projeto já parcialmente organizado não vai ter as mesmas transformações de um monolito. Sua skill deve se adaptar ao contexto.
- **Pedir confirmação na Fase 2 é obrigatório** — o humano deve revisar o relatório antes de qualquer modificação.
- **Consulte as referências do curso** — revise a documentação oficial da ferramenta escolhida e os materiais das aulas para relembrar a estrutura e anatomia de uma skill.
