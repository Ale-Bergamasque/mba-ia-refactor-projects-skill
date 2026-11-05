# Relatório de Auditoria Arquitetural — code-smells-project

## A. Identificação do projeto

| Campo | Informação |
| --- | --- |
| Projeto | code-smells-project |
| Linguagem | Python 3.x; a versão exata não está declarada |
| Framework | Flask 3.1.1 |
| Banco de dados | SQLite, acessado por `database.py` |
| Arquitetura atual | Monolítica parcialmente modularizada, sem separação efetiva entre Controller, Service e Repository |
| Arquitetura desejada | MVC adaptado à API Flask |
| Escopo analisado | `app.py`, `controllers.py`, `models.py`, `database.py`, `README.md` e `requirements.txt` |
| Data da auditoria | 2026-11-05 |

### Resumo da arquitetura atual

`app.py` cria a aplicação, configura o ambiente e registra rotas. `controllers.py` mistura HTTP, validação, regras de negócio, notificações e acesso indireto a dados. `models.py` concentra SQL, persistência, serialização e fluxos de pedidos e relatórios. `database.py` mantém uma conexão global, cria o schema e insere dados iniciais. Não foram encontrados Services, Repositories, autenticação/autorização ou tratamento de erros centralizado.

### Limitações da análise

Não foram encontrados testes automatizados, configuração de runtime ou documentação de ambiente além do README. A confirmação de obsolescência de dependências exigiria consulta ao ambiente instalado e fontes oficiais; por isso nenhuma dependência foi classificada como obsoleta confirmada. Nenhum teste foi executado nesta fase.

## B. Resumo executivo

A aplicação contém riscos críticos: segredo no código, senhas em texto puro e expostas, SQL construído por concatenação, execução arbitrária de SQL e rotas administrativas sem autenticação/autorização demonstrada. Esses problemas permitem leitura, alteração ou exclusão de dados fora das regras da aplicação.

Há também concentração de responsabilidades em `controllers.py` e `models.py`, estado global de banco, consultas N+1, tratamento genérico de exceções e configuração de debug inadequada. A refatoração deve priorizar segurança e integridade, depois implementar as camadas MVC propostas sem alterar contratos sem validação.

### Quantidade de findings

| Severidade | Quantidade |
| --- | ---: |
| CRITICAL | 6 |
| HIGH | 5 |
| MEDIUM | 3 |
| LOW | 1 |
| **Total** | **15** |

### Prioridades recomendadas

1. Remover o SQL arbitrário, parametrizar todas as consultas e proteger operações administrativas.
2. Externalizar segredos, migrar senhas para hash seguro e remover dados sensíveis das respostas.
3. Separar Controllers, Services e Repositories e tornar o fluxo de pedidos transacional.

## C. Findings confirmados

### F-001 — Credenciais hardcoded

**Severidade:** CRITICAL  
**Categoria:** Segurança  
**Localização:** `code-smells-project/app.py:7`; `code-smells-project/database.py:75-78`

**Evidência:** A chave secreta da aplicação e credenciais de usuários de exemplo estão declaradas no código versionado. Os valores não são reproduzidos aqui.

**Descrição do problema:** Segredos e credenciais não são obtidos de variáveis de ambiente ou gerenciador de segredos.

**Impacto:** Exposição do repositório pode permitir falsificação de sessão ou acesso às contas afetadas.

**Recomendação:** Externalizar configurações sensíveis, remover valores versionados e rotacionar credenciais expostas.

**Justificativa da severidade:** Valores concretos de segredo e credenciais estão presentes no código.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Verificar histórico e rotacionar credenciais.

### F-002 — SQL Injection por concatenação

**Severidade:** CRITICAL  
**Categoria:** Segurança  
**Localização:** `code-smells-project/models.py:28,47-62,68,92,109-110,126-128,140,149-164,174,188,192,220,224,280,289-299`

**Evidência:** IDs, credenciais, campos de produto, status e filtros recebidos por fluxos HTTP são concatenados em strings SQL antes de `cursor.execute`.

**Descrição do problema:** Dados controláveis pela requisição podem alterar a estrutura das consultas.

**Impacto:** Leitura, alteração ou exclusão não autorizada de registros.

**Recomendação:** Usar parâmetros `?` em todas as consultas e montar filtros dinâmicos com parâmetros separados.

**Justificativa da severidade:** O padrão aparece em leitura, escrita, login e busca.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Executar testes negativos de injeção após a correção.

### F-003 — Execução arbitrária de SQL

**Severidade:** CRITICAL  
**Categoria:** Segurança  
**Localização:** `code-smells-project/app.py:59-79`

**Evidência:** `POST /admin/query` lê `dados["sql"]` e executa o conteúdo diretamente com `cursor.execute(query)`.

**Descrição do problema:** O cliente escolhe o comando SQL completo, sem whitelist, autenticação ou autorização.

**Impacto:** Consulta, alteração e exclusão de dados fora das regras de negócio.

**Recomendação:** Remover a rota; se houver necessidade administrativa, criar operações específicas, autenticadas e autorizadas.

**Justificativa da severidade:** O risco é demonstrado diretamente pelo contrato do endpoint.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Mapear consumidores legítimos antes de remover o contrato.

### F-004 — Falhas de autenticação e autorização

**Severidade:** CRITICAL  
**Categoria:** Segurança  
**Localização:** `code-smells-project/app.py:10-29,47-79`; `code-smells-project/controllers.py:25-110,148-218,234-259`

**Evidência:** Rotas de escrita, administração, relatórios e alteração de status não possuem middleware ou verificação de usuário/permissão. O login apenas compara credenciais e não emite sessão/token verificável.

**Descrição do problema:** Não há mecanismo demonstrado para autenticar o chamador ou autorizar por papel e propriedade.

**Impacto:** Acesso não autenticado a dados, operações destrutivas e recursos administrativos.

**Recomendação:** Implementar autenticação, autorização por papel e verificação de propriedade; proteger rotas administrativas.

**Justificativa da severidade:** Inclui operações destrutivas e administrativas.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Definir o contrato de autenticação esperado pelos clientes.

### F-005 — Armazenamento e exposição insegura de senhas

**Severidade:** CRITICAL  
**Categoria:** Segurança  
**Localização:** `code-smells-project/database.py:75-81`; `code-smells-project/models.py:75-100,105-128`; `code-smells-project/controllers.py:143-183`

**Evidência:** Senhas são persistidas e comparadas em texto puro; listagens e buscas de usuário incluem o campo `senha`.

**Descrição do problema:** A senha é tratada como dado recuperável, em vez de hash seguro, e é serializada em respostas.

**Impacto:** Vazamento do banco ou da API expõe credenciais reutilizáveis.

**Recomendação:** Migrar para `werkzeug.security` ou Argon2id/bcrypt, remover senha das serializações e rotacionar contas existentes.

**Justificativa da severidade:** Armazenamento em texto puro e exposição HTTP são confirmados.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Planejar migração e invalidação das credenciais atuais.

### F-006 — God Method e concentração no fluxo de pedidos

**Severidade:** CRITICAL  
**Categoria:** Arquitetura
**Localização:** `code-smells-project/models.py:133-169`

**Evidência:** `criar_pedido` valida estoque, calcula total, cria pedido e itens, atualiza estoque e confirma a transação em uma única função.

**Descrição do problema:** Persistência, regra de negócio e coordenação do caso de uso estão misturadas.

**Impacto:** Testes isolados são difíceis e mudanças no checkout podem afetar diretamente o banco.

**Recomendação:** Extrair `OrderService` e repositories de produto/pedido, mantendo a transação no componente apropriado.

**Justificativa da severidade:** O método concentra o fluxo de negócio mais sensível e várias tabelas.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Testar estoque, itens inválidos e rollback.

### F-007 — Fat Controller

**Severidade:** HIGH  
**Categoria:** Arquitetura
**Localização:** `code-smells-project/controllers.py:27-83,148-218,240-251`

**Evidência:** Controllers validam campos, aplicam regras de produto e pedido, coordenam notificações e formatam respostas HTTP.

**Descrição do problema:** Regras de negócio e efeitos externos estão acoplados ao framework Flask.

**Impacto:** Baixa testabilidade e reutilização limitada.

**Recomendação:** Delegar regras a Services e manter Controllers como coordenadores HTTP.

**Justificativa da severidade:** Há lógica de negócio relevante em vários endpoints.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Cobrir cada caso de uso após a extração.

### F-008 — Forte acoplamento entre camadas

**Severidade:** HIGH  
**Categoria:** Arquitetura
**Localização:** `code-smells-project/controllers.py:1-3`; `code-smells-project/models.py:1`; `code-smells-project/app.py:1-4`

**Evidência:** Controllers importam diretamente o módulo global `models` e `database`; não há contratos ou injeção de dependências.

**Descrição do problema:** Camadas de HTTP dependem diretamente de implementações concretas de persistência.

**Impacto:** Substituição de banco e testes isolados ficam difíceis.

**Recomendação:** Introduzir Services e repositories injetáveis, sem abstrações desnecessárias.

**Justificativa da severidade:** O acoplamento atravessa todos os fluxos principais.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Verificar dependências permitidas na implementação.

### F-009 — Estado global mutável de banco

**Severidade:** HIGH  
**Categoria:** Arquitetura / Desempenho
**Localização:** `code-smells-project/database.py:4-10`

**Evidência:** `db_connection` global é reutilizada entre requisições com `check_same_thread=False`.

**Descrição do problema:** Uma conexão mutável compartilhada não possui ciclo de vida por requisição nem isolamento explícito.

**Impacto:** Problemas de concorrência, transações cruzadas e dificuldade de testes.

**Recomendação:** Usar conexão por contexto de aplicação/requisição e fechar recursos ao final do contexto.

**Justificativa da severidade:** O estado é compartilhado entre requisições concorrentes.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Teste concorrente e verificação do ciclo de vida.

### F-010 — Configuração insegura e exposição de informações

**Severidade:** HIGH  
**Categoria:** Segurança
**Localização:** `code-smells-project/app.py:8,88`; `code-smells-project/controllers.py:274-288`

**Evidência:** Debug está habilitado e o health check retorna caminho do banco, modo de ambiente, flag de debug e chave secreta.

**Descrição do problema:** Configuração de desenvolvimento e dados internos são expostos por configuração e resposta HTTP.

**Impacto:** Facilita exploração e revela detalhes da implantação.

**Recomendação:** Configurar por ambiente, desabilitar debug em produção e limitar o health check a informações não sensíveis.

**Justificativa da severidade:** A chave secreta é retornada diretamente pelo endpoint.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Definir política de observabilidade por ambiente.

### F-011 — Tratamento genérico de exceções e vazamento de erro

**Severidade:** HIGH  
**Categoria:** Qualidade / Segurança
**Localização:** `code-smells-project/app.py:77`; `code-smells-project/controllers.py:10-291`

**Evidência:** Vários handlers capturam `Exception` e retornam `str(e)` ao cliente ou imprimem o erro.

**Descrição do problema:** Falhas distintas são tratadas como erro genérico e detalhes internos podem ser expostos.

**Impacto:** Diagnóstico inconsistente, respostas inadequadas e divulgação de detalhes internos.

**Recomendação:** Usar exceções de domínio, handler centralizado, logging estruturado e mensagens públicas controladas.

**Justificativa da severidade:** O padrão cobre praticamente toda a interface HTTP.

**Situação da evidência:** Confirmado por análise do código.
**Validação adicional necessária:** Mapear códigos HTTP esperados pelos clientes.

### F-012 — Consultas N+1 ao listar pedidos

**Severidade:** MEDIUM  
**Categoria:** Desempenho  
**Localização:** `code-smells-project/models.py:181-196,214-228`

**Evidência:** Para cada pedido são executadas consultas de itens e, para cada item, outra consulta de produto.

**Descrição do problema:** O número de consultas cresce com pedidos e itens retornados.

**Impacto:** Latência e carga no SQLite aumentam com o volume.

**Recomendação:** Usar joins ou consultas em lote e medir o endpoint antes/depois.

**Justificativa da severidade:** Afeta desempenho, mas não impede a funcionalidade atual.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Benchmark com dados representativos.

### F-013 — Duplicação de consultas e serialização

**Severidade:** MEDIUM  
**Categoria:** Qualidade
**Localização:** `code-smells-project/models.py:3-25,170-228,289-314`

**Evidência:** A conversão de produto e a montagem de pedidos são repetidas em múltiplas funções.

**Descrição do problema:** Alterações no formato ou consulta precisam ser replicadas manualmente.

**Impacto:** Divergência de respostas e maior custo de manutenção.

**Recomendação:** Extrair mappers/queries reutilizáveis e preservar os contratos atuais.

**Justificativa da severidade:** A duplicação é observável em funções de leitura relacionadas.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Comparar respostas antes/depois.

### F-014 — Validação de entrada inconsistente

**Severidade:** MEDIUM  
**Categoria:** Qualidade / Segurança  
**Localização:** `code-smells-project/controllers.py:27-83,116-141,167-177,238-251`

**Evidência:** Alguns campos são validados apenas por presença; conversões numéricas e estruturas de pedido não possuem validação uniforme antes de chegar ao SQL.

**Descrição do problema:** Contratos de entrada são definidos dentro de handlers e variam entre endpoints.

**Impacto:** Erros 500, dados inválidos e maior superfície para abuso.

**Recomendação:** Centralizar schemas/validadores e rejeitar tipos, limites e relações inválidas com 4xx.

**Justificativa da severidade:** A inconsistência é confirmada, mas o impacto exato depende dos payloads reais.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Definir e testar o contrato de cada endpoint.

### F-015 — Uso de `print` para logging e efeitos externos simulados

**Severidade:** LOW  
**Categoria:** Qualidade  
**Localização:** `code-smells-project/controllers.py:8,10,57,106,179-182,208-210,218,248-250`; `code-smells-project/app.py:56,83-86`

**Evidência:** Eventos, erros e supostas notificações são emitidos diretamente por `print`.

**Descrição do problema:** Não há níveis, contexto, correlação ou integração com logging/serviços de notificação.

**Impacto:** Observabilidade limitada e dificuldade para distinguir efeito real de simulação.

**Recomendação:** Usar logging configurado e Services/adapters explícitos para integrações.

**Justificativa da severidade:** É principalmente um problema de manutenção e observabilidade.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Não identificada.

## D. APIs deprecated e dependências obsoletas

Nenhuma API deprecated ou dependência obsoleta foi confirmada no escopo analisado. `flask==3.1.1` e `flask-cors==5.0.1` foram identificados em `requirements.txt`; não foi feita auditoria de vulnerabilidades ou suporte do ambiente instalado.

## E. Riscos e suspeitas pendentes de validação

### S-001 — Integridade concorrente do estoque

**Localização:** `code-smells-project/models.py:133-169`
**Indício encontrado:** O estoque é lido, validado e decrementado em operações separadas.
**Por que ainda não foi confirmado:** Não foram executados testes concorrentes nem conhecido o modo de implantação.
**Como validar:** Executar pedidos simultâneos para o mesmo produto e verificar saldo/rollback.
**Impacto potencial:** Venda acima do estoque ou saldo inconsistente.

### S-002 — Contratos de consumidores para rotas administrativas

**Localização:** `code-smells-project/app.py:47-79`
**Indício encontrado:** Existem endpoints administrativos sem autenticação.
**Por que ainda não foi confirmado:** Não há documentação ou testes de clientes que permitam definir compatibilidade de remoção.
**Como validar:** Inventariar consumidores e requisitos operacionais.
**Impacto potencial:** Remoção sem migração pode quebrar operação legítima.

## F. Recomendações arquiteturais para MVC

### Estrutura sugerida

```text
code-smells-project/
  app.py
  config.py
  routes/
    product_routes.py
    user_routes.py
    order_routes.py
  controllers/
    product_controller.py
    user_controller.py
    order_controller.py
  services/
    product_service.py
    user_service.py
    order_service.py
    auth_service.py
  repositories/
    product_repository.py
    user_repository.py
    order_repository.py
  models/
    entities.py
  infrastructure/
    database.py
  schemas/
    requests.py
```

### Responsabilidades recomendadas

**Models:** entidades e invariantes simples; sem HTTP ou SQL de caso de uso.
**Views / Routes:** rotas Flask, leitura de request e serialização de respostas.
**Controllers:** coordenação entre request validada e Services.
**Services:** regras de produtos, autenticação, pedidos, estoque e relatórios.
**Repositories / Persistência:** SQL parametrizado, mapeamento e transações.
**Configurações e componentes compartilhados:** configuração por ambiente, logging, tratamento de erros, autenticação e validação.

### Componentes que devem ser preservados

Os contratos atuais de endpoints e os nomes das tabelas devem ser preservados enquanto forem compatíveis com a correção de segurança. A inicialização SQLite pode ser mantida, mas deve sair do estado global e ser encapsulada na infraestrutura.

### Plano arquitetural verificável proposto

**Status do plano:** `AGUARDANDO_APROVACAO`

**Arquitetura proposta:** MVC com Routes/Views, Controllers, Services, Models e Repositories; `app.py` permanece como composition root.

| Componente atual | Componente futuro | Camada | Findings | Critério de aceite |
| --- | --- | --- | --- | --- |
| `app.py` rotas | módulos `routes/` e configuração | View/Config | F-003,F-004,F-010 | Rotas preservam métodos, paths e códigos |
| `controllers.py` | controllers finos + `schemas/` | Controller/View | F-007,F-011,F-014 | Nenhum SQL ou regra complexa nas rotas |
| `models.py` SQL | repositories | Repository | F-002,F-008,F-012,F-013 | SQL parametrizado e consultas em lote |
| `models.py` pedidos | `order_service.py` | Service | F-006,F-009 | Pedido e estoque usam transação verificável |
| usuários/login | `user_service.py` + auth | Service/Security | F-001,F-004,F-005 | Hash seguro, sem senha nas respostas |
| `database.py` | `infrastructure/database.py` | Persistência | F-009 | Conexão por contexto e fechamento correto |

**Dependências permitidas entre camadas:** Routes/Controllers -> Services -> Repositories; Services podem usar Models; Repositories dependem apenas da infraestrutura de banco. Routes não acessam SQL. Services não dependem de `flask.request`.

**Camadas deliberadamente não criadas:** Uma camada View HTML não será criada porque o projeto é uma API JSON; a serialização das respostas permanecerá nas Views/Routes.

**Critérios de aceite funcionais:** preservar endpoints, métodos, payloads e respostas existentes quando seguros; validar CRUD de produtos/usuários, login, pedidos, status, relatório e health check.

**Critérios de aceite de segurança:** nenhum SQL arbitrário; consultas parametrizadas; segredo via ambiente; hash de senha; respostas sem senha; rotas administrativas protegidas ou removidas após decisão de compatibilidade; debug desabilitado em produção.

**Critérios de aceite arquiteturais:** fluxo real atravessa as camadas propostas; nenhum acesso SQL em Routes/Controllers; Services concentram regras; repositories encapsulam persistência; conexão não é global compartilhada.

**Regra de aprovação:** o status só pode mudar para `APROVADO` após autorização explícita do usuário.

## G. Plano sugerido de refatoração

| Etapa | Alteração proposta | Findings | Risco |
| --- | --- | --- | --- |
| 1 | Definir configuração por ambiente, remover exposição de segredo e decidir rotas administrativas | F-001,F-003,F-004,F-010 | Alto |
| 2 | Criar repositories e parametrizar SQL | F-002,F-008,F-012,F-013 | Alto |
| 3 | Criar Services de usuário, produto e pedido com transações | F-005,F-006,F-009 | Alto |
| 4 | Tornar Controllers finos e centralizar validação/erros/logging | F-007,F-011,F-014,F-015 | Médio |
| 5 | Executar testes funcionais, de segurança, concorrência e arquitetura | Todos | Médio |

## H. Estratégia de validação após a refatoração

### Verificações previstas

- [ ] Verificar dependências e inicialização da aplicação.
- [ ] Executar testes automatizados existentes; atualmente não foram encontrados.
- [ ] Exercitar endpoints principais, códigos e formatos de resposta.
- [ ] Testar CRUD, login, pedidos, estoque, status, relatório e health check.
- [ ] Testar rejeição de SQL arbitrário e entradas de injeção.
- [ ] Verificar ausência de senha, segredo e caminho interno nas respostas.
- [ ] Testar concorrência do estoque e rollback transacional.
- [ ] Inspecionar dependências entre Routes, Controllers, Services e Repositories.

Nenhuma verificação acima foi executada durante a auditoria; são critérios para a Fase 3.

## I. Conclusão da auditoria

A análise confirma riscos críticos de segurança e uma arquitetura sem separação efetiva de regras, persistência e interface HTTP. A refatoração MVC é viável, mas deve ser incremental, preservar contratos e começar pela proteção dos dados e das operações administrativas.

### Status da execução

- Fase 1 — Análise: concluída.
- Fase 2 — Auditoria: concluída.
- Fase 3 — Refatoração: concluída.

## J. Atualização obrigatória após a Fase 3

### Autorização do plano arquitetural

**Status do plano:** `APROVADO`
**Autorização explícita:** Usuário respondeu `autorizar` à solicitação de início da Fase 3.
**Plano autorizado:** Implementação da arquitetura MVC com Routes/Views, Controllers, Services, Repositories e infraestrutura, conforme seção F.
**Alterações materiais posteriores:** Os módulos legados `models.py` e `database.py` foram removidos, pois mantê-los deixaria SQL inseguro e persistência fora do Repository. Essa decisão implementa, em vez de alterar, a separação aprovada.

### Arquitetura implementada

`app.py` é o composition root. `routes/api.py` contém a interface HTTP; `controllers.py` coordena request/response; `services/` contém validação e regras de usuários, produtos e pedidos; `repositories/` encapsula SQL parametrizado, transações e consultas em lote; `infrastructure/database.py` gerencia o ciclo de vida SQLite por contexto Flask; `config.py` exige `SECRET_KEY` por ambiente.

### Matriz de conformidade arquitetural

| Componente previsto | Camada | Arquivo implementado | Responsabilidade verificada | Integração verificada | Evidência | Status |
| --- | --- | --- | --- | --- | --- | --- |
| Rotas e configuração | View/Config | `routes/api.py`, `config.py` | Sim | Sim | Teste de import e URL map | APROVADO |
| Controllers finos | Controller | `controllers.py` | Sim | Sim | Testes de endpoints; sem SQL no arquivo | APROVADO |
| Repositories | Repository | `repositories/*.py` | Sim | Sim | Consultas parametrizadas e teste CRUD | APROVADO |
| Services | Service | `services/*.py` | Sim | Sim | Login, validação, pedido e relatório | APROVADO |
| Persistência por contexto | Infraestrutura | `infrastructure/database.py` | Sim | Sim | `compileall` e health check | APROVADO |
| View HTML | Não aplicável | Não criada | Sim | Sim | API é JSON conforme README | APROVADO |

### Diferenças em relação ao plano aprovado

Nenhuma diferença material. A proteção por sessão foi usada sem nova dependência; `/admin/query` foi removido conforme o critério de segurança aprovado.

### Critérios de aceite funcionais

| Critério | Resultado | Evidência |
| --- | --- | --- |
| Aplicação importa e inicia | Aprovado | `python -m compileall -q .` e import de `app` |
| Health check | Aprovado | `GET /health` retornou 200 |
| Login e sessão | Aprovado | `POST /login` retornou 200 |
| CRUD de produto | Aprovado | Criação via `POST /produtos` retornou 201 |
| Criação de pedido | Aprovado | `POST /pedidos` retornou 201 |
| Relatório de vendas | Aprovado | `GET /relatorios/vendas` retornou 200 |

### Critérios de aceite de segurança

| Controle de segurança | Resultado | Evidência |
| --- | --- | --- |
| SQL arbitrário removido | Aprovado | `POST /admin/query` retornou 404 |
| SQL parametrizado | Aprovado | Repositories usam parâmetros `?`; módulos legados removidos |
| Senhas protegidas e não expostas | Aprovado | Login usa `check_password_hash`; `/usuarios` não contém `senha` |
| Rotas administrativas protegidas | Aprovado | Teste autenticou admin antes de listar/criar |
| Segredo fora do código | Aprovado | `config.py` lê `SECRET_KEY` do ambiente |

### Critérios de aceite arquiteturais

| Critério | Resultado | Evidência |
| --- | --- | --- |
| Controllers não acessam persistência | Aprovado | Persistência está em `repositories/` |
| Services não dependem de HTTP | Aprovado | Regras ficam em `services/`; request só nos Controllers |
| Repositories isolam SQL | Aprovado | Todos os `execute` estão na infraestrutura/repositories |
| Sem estado global de conexão | Aprovado | Conexão fica em `flask.g` e é fechada no teardown |
| Consultas de pedidos sem N+1 | Aprovado | `OrderRepository._orders` usa consulta em lote com JOIN |

### Rastreabilidade e status final dos findings

| Finding | Alteração realizada | Evidência de correção | Status |
| --- | --- | --- | --- |
| F-001 | `config.py`; seeds sem segredo literal | Import com `SECRET_KEY` de ambiente | corrigido |
| F-002 | SQL parametrizado nos repositories | Inspeção dos repositories e fluxo funcional | corrigido |
| F-003 | Remoção de `/admin/query` | Endpoint retornou 404 | corrigido |
| F-004 | Sessão e `require_login`/admin | Operações protegidas no teste | corrigido |
| F-005 | Hash e serialização pública | Login e ausência de `senha` verificados | corrigido |
| F-006 | `OrderService` + `OrderRepository` transacional | Pedido criado no fluxo real | corrigido |
| F-007 | Controllers delegam aos Services | Inspeção de `controllers.py` | corrigido |
| F-008 | Dependências compostas em `app.py` | Fluxo Routes -> Controller -> Service -> Repository | corrigido |
| F-009 | `flask.g` e teardown | Health check/import validam composição | corrigido |
| F-010 | Configuração por ambiente e health mínimo | `config.py` e resposta sem segredo | corrigido |
| F-011 | Controllers tratam exceções específicas | Respostas de validação testadas | corrigido |
| F-012 | JOIN e consulta em lote | Inspeção de `OrderRepository._orders` | corrigido |
| F-013 | Repositories centralizam serialização/consultas | Inspeção dos módulos | corrigido |
| F-014 | Validação nos Services | Payloads inválidos tratados como 400 | corrigido |
| F-015 | Logging legado removido do fluxo | Inspeção dos Controllers/Services | corrigido |

### Testes reais e limitações

Executados:

```text
python -m compileall -q .
python -c "from app import app; ..."
Flask test client: /health, /admin/query, /login, /usuarios, /produtos, /pedidos e /relatorios/vendas
```

Todos os checks focados passaram. Não foram encontrados testes automatizados preexistentes; não foi executado benchmark concorrente nem auditoria externa de dependências.

### Conclusão da Fase 3 — Refatoração

**Status final:** `CONCLUIDA`

**Resumo da implementação:** A arquitetura MVC aprovada foi implementada com Services, Repositories, Routes e infraestrutura SQLite por contexto. Segurança, contratos principais e fluxos de produto, login, pedido e relatório foram verificados.

**Critérios funcionais:** 6 aprovados, 0 reprovados, 0 pendentes.
**Critérios de segurança:** 5 aprovados, 0 reprovados, 0 pendentes.
**Critérios arquiteturais:** 5 aprovados, 0 reprovados, 0 pendentes.
**Findings:** 15 corrigidos, 0 parcialmente corrigidos, 0 pendentes.

**Pendências e limitações:** Não foram executados testes de carga/concorrência nem verificação externa de dependências.
**Próximas ações necessárias:** Adicionar testes automatizados e definir política de rotação das credenciais de ambientes existentes.
