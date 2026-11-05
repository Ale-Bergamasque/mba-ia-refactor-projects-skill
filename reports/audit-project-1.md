# Relatório de Auditoria Arquitetural — code-smells-project

## A. Identificação do projeto

| Campo | Informação |
| --- | --- |
| Projeto | code-smells-project |
| Linguagem | Python 3.x; a versão exata não está declarada nos arquivos analisados |
| Framework | Flask 3.1.1 |
| Banco de dados | SQLite, acessado diretamente pelo módulo `database.py` |
| Arquitetura atual | Aplicação monolítica parcialmente organizada por módulos; não há separação efetiva entre Controllers, Services e Repositories |
| Arquitetura desejada | MVC adaptado à tecnologia |
| Escopo analisado | `app.py`, `controllers.py`, `models.py`, `database.py`, `README.md` e `requirements.txt`; arquivos gerados em `__pycache__` foram excluídos |
| Data da auditoria | 2026-11-05 |

### Resumo da arquitetura atual

`app.py` cria a aplicação Flask, registra rotas, configura o ambiente e contém endpoints administrativos. `controllers.py` mistura leitura da requisição, validação, regras de negócio, persistência indireta, notificações e formatação das respostas. `models.py` concentra consultas SQL, persistência e parte significativa das regras de pedidos e relatórios. `database.py` mantém uma conexão global, cria o schema e insere dados iniciais.

Há uma separação nominal entre aplicação, controllers, models e banco, mas os controllers dependem diretamente de `models` e `database`, e os models acumulam responsabilidades de Repository e Service. Não foram encontrados Services, Repositories, autenticação/autorização ou tratamento de erros centralizado.

### Limitações da análise

Não foram encontrados testes automatizados, arquivo de configuração de runtime ou documentação de ambiente além do README. A versão exata do Python não está declarada. A confirmação de APIs deprecated e de vulnerabilidades de dependências exigiria consulta ao ambiente instalado e às fontes oficiais correspondentes; por isso não foram classificados findings confirmados nessa categoria.

## B. Resumo executivo

A aplicação contém riscos críticos de segurança: credenciais e senhas em texto puro, exposição de dados sensíveis, consultas SQL construídas por concatenação, um endpoint que executa SQL arbitrário e rotas administrativas sem autenticação/autorização demonstrada. Esses problemas podem permitir alteração, exclusão ou leitura não autorizada de dados.

Também há forte concentração de responsabilidades em `controllers.py` e `models.py`, estado global de banco, ausência de controle transacional explícito no fluxo de pedidos, consultas N+1 e tratamento genérico de exceções. Recomenda-se priorizar a remoção dos endpoints administrativos inseguros ou sua proteção, parametrizar todas as consultas, migrar senhas para hash seguro e implementar as camadas MVC aprovadas antes de otimizações menores.

### Quantidade de findings

| Severidade | Quantidade |
| --- | ---: |
| CRITICAL | 6 |
| HIGH | 5 |
| MEDIUM | 3 |
| LOW | 1 |
| **Total** | **15** |

### Prioridades recomendadas

1. Remover ou restringir o SQL arbitrário, parametrizar consultas e implementar autenticação/autorização.
2. Retirar credenciais do código, aplicar hash seguro às senhas e remover dados sensíveis das respostas.
3. Separar Controllers, Services e Repositories, além de tornar o fluxo de pedidos transacional.

## C. Findings confirmados

### F-001 — Credenciais hardcoded

**Severidade:** CRITICAL  
**Categoria:** Segurança  
**Localização:**

- Arquivo: `code-smells-project/app.py`
- Linha(s): 7
- Arquivo: `code-smells-project/database.py`
- Linha(s): 75-78

**Evidência:** A chave secreta da aplicação e credenciais de usuários de exemplo estão declaradas diretamente no código-fonte versionado. Os valores não são reproduzidos neste relatório.

**Descrição do problema:** Segredos e credenciais não são obtidos de variáveis de ambiente ou de um gerenciador de segredos.

**Impacto:** Exposição do repositório pode permitir falsificação de sessão ou acesso às contas/secrets associados.

**Recomendação:** Externalizar configurações sensíveis, remover valores versionados e rotacionar credenciais que tenham sido expostas.

**Justificativa da severidade:** A evidência mostra valores de segredo e credenciais concretos no código.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Rotacionar credenciais e verificar histórico do repositório.

### F-002 — SQL Injection por concatenação de entrada

**Severidade:** CRITICAL  
**Categoria:** Segurança  
**Localização:**

- Arquivo: `code-smells-project/models.py`
- Linha(s): 28, 47-51, 57-62, 68, 92, 109-110, 126-128, 140, 149-164, 174, 188, 192, 220, 224, 280, 289-299

**Evidência:** Valores de IDs, credenciais, campos de produto, status, filtros de busca e itens do pedido são concatenados em strings SQL antes de `cursor.execute`.

**Descrição do problema:** Dados controláveis pela requisição podem alterar a estrutura das consultas.

**Impacto:** Leitura, alteração ou exclusão não autorizada de registros e comprometimento da integridade do banco.

**Recomendação:** Usar parâmetros `?` em todas as consultas, validar tipos e manter a montagem de filtros com uma lista de parâmetros.

**Justificativa da severidade:** Há múltiplos fluxos HTTP que alcançam consultas concatenadas, incluindo login, escrita e busca.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Testes negativos com entradas SQL após a correção.

### F-003 — Execução arbitrária de SQL

**Severidade:** CRITICAL  
**Categoria:** Segurança  
**Localização:**

- Arquivo: `code-smells-project/app.py`
- Linha(s): 59-79

**Evidência:** O endpoint `POST /admin/query` lê `dados["sql"]` e executa o conteúdo diretamente com `cursor.execute(query)`.

**Descrição do problema:** O cliente escolhe o comando SQL completo, sem whitelist, autorização ou restrição de operação.

**Impacto:** Qualquer consumidor da rota pode consultar, modificar ou apagar dados fora das regras da aplicação.

**Recomendação:** Remover a rota; se houver necessidade administrativa, criar operações específicas, autenticadas e autorizadas.

**Justificativa da severidade:** O risco é demonstrado diretamente pelo contrato do endpoint.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Confirmar consumidores legítimos antes de remover o contrato.

### F-004 — Falhas de autenticação e autorização

**Severidade:** CRITICAL  
**Categoria:** Segurança  
**Localização:**

- Arquivo: `code-smells-project/app.py`
- Linha(s): 10-29, 47-79
- Arquivo: `code-smells-project/controllers.py`
- Linha(s): 25-63, 66-110, 148-218, 234-259

**Evidência:** Rotas de criação, atualização, exclusão, alteração de status, relatórios e administração são registradas sem middleware ou verificação de usuário/permissão. O login apenas consulta credenciais e não cria uma sessão/token verificável.

**Descrição do problema:** Não há mecanismo demonstrado para autenticar o chamador ou autorizar operações por papel e propriedade do recurso.

**Impacto:** Usuários não autenticados podem alterar dados, acessar relatórios e executar operações administrativas.

**Recomendação:** Definir autenticação, autorização por papel e verificação de propriedade; proteger rotas administrativas e operações de escrita.

**Justificativa da severidade:** A ausência é observável em todas as rotas protegidas analisadas e inclui operações destrutivas.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Mapear os clientes e o contrato de autenticação esperado.

### F-005 — Armazenamento e exposição insegura de senhas

**Severidade:** CRITICAL  
**Categoria:** Segurança  
**Localização:**

- Arquivo: `code-smells-project/database.py`
- Linha(s): 75-81
- Arquivo: `code-smells-project/models.py`
- Linha(s): 75-100, 105-128
- Arquivo: `code-smells-project/controllers.py`
- Linha(s): 143-183

**Evidência:** Senhas são persistidas e comparadas em texto puro; `get_todos_usuarios` e `get_usuario_por_id` incluem o campo `senha` nas respostas.

**Descrição do problema:** O banco e a API tratam senha como dado recuperável, em vez de armazenar hash seguro e retornar apenas dados não sensíveis.

**Impacto:** Vazamento do banco ou da API expõe credenciais reutilizáveis.

**Recomendação:** Migrar para `werkzeug.security` ou Argon2id/bcrypt, remover senha das serializações e planejar migração das contas existentes.

**Justificativa da severidade:** O armazenamento em texto puro e a exposição HTTP são confirmados.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Planejar invalidação/rotação das credenciais atuais.

### F-006 — God Method e concentração de responsabilidades em `models.py`

**Severidade:** CRITICAL  
**Categoria:** Arquitetura  
**Localização:**

- Arquivo: `code-smells-project/models.py`
- Linha(s): 133-169

**Evidência:** `criar_pedido` valida estoque, calcula total, cria pedido e itens, atualiza estoque e confirma a transação em uma única função.

**Descrição do problema:** Persistência, regras de negócio e coordenação do caso de uso estão misturadas.

**Impacto:** Mudanças no checkout exigem alterar código de acesso a dados e dificultam testes isolados.

**Recomendação:** Extrair um Service de pedidos e Repositories para produtos/pedidos, mantendo Models como entidades/dados.

**Justificativa da severidade:** O método concentra o fluxo de negócio mais sensível e operações de múltiplas tabelas.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Cobertura de casos de estoque, itens inválidos e rollback.

### F-007 — Global Mutable State

**Severidade:** HIGH  
**Categoria:** Arquitetura / Confiabilidade  
**Localização:**

- Arquivo: `code-smells-project/database.py`
- Linha(s): 4-8

**Evidência:** `db_connection` é uma conexão global, inicializada sob demanda e compartilhada entre requisições com `check_same_thread=False`.

**Descrição do problema:** O ciclo de vida e o isolamento da conexão não são controlados por contexto de aplicação ou requisição.

**Impacto:** Condições de concorrência, conexões stale e comportamento difícil de testar ou reproduzir.

**Recomendação:** Usar conexão por contexto de requisição, fechamento no teardown e transações explícitas.

**Justificativa da severidade:** O estado é mutável e compartilhado em uma API que pode atender requisições concorrentes.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Teste concorrente e verificação do ciclo de vida esperado.

### F-008 — Configuração insegura e exposição de informações

**Severidade:** HIGH  
**Categoria:** Segurança / Configuração  
**Localização:**

- Arquivo: `code-smells-project/app.py`
- Linha(s): 8
- Arquivo: `code-smells-project/controllers.py`
- Linha(s): 284-288, 290-292

**Evidência:** Debug é habilitado, o health check expõe caminho do banco, estado de debug e chave secreta, e handlers retornam `str(e)` diretamente.

**Descrição do problema:** A API divulga informações internas e detalhes de exceções, enquanto a configuração não separa desenvolvimento de produção.

**Impacto:** Facilita reconhecimento do ambiente e pode revelar dados sensíveis em falhas.

**Recomendação:** Configurar por ambiente, remover segredos do health check, usar respostas genéricas e registrar detalhes apenas no servidor.

**Justificativa da severidade:** A exposição de informação interna é observável no endpoint público.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Confirmar o ambiente de implantação para definir a política de debug.

### F-009 — Fat Controller

**Severidade:** HIGH  
**Categoria:** Arquitetura / Qualidade  
**Localização:**

- Arquivo: `code-smells-project/controllers.py`
- Linha(s): 1-292

**Evidência:** O módulo contém handlers de produtos, usuários, login, pedidos, notificações, relatórios e health check, cada um combinando HTTP, validação, regras e logging.

**Descrição do problema:** O controller funciona como camada de transporte, service e parte da apresentação ao mesmo tempo.

**Impacto:** Alto acoplamento, duplicação de validações e dificuldade para testar regras sem Flask.

**Recomendação:** Controllers devem apenas traduzir HTTP; mover casos de uso para Services e serialização/validação para componentes próprios.

**Justificativa da severidade:** O módulo concentra todos os domínios da API e possui quase 300 linhas com responsabilidades distintas.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Mapear contratos atuais antes de extrair cada caso de uso.

### F-010 — Tight Coupling entre Controllers, Models e banco

**Severidade:** HIGH  
**Categoria:** Arquitetura  
**Localização:**

- Arquivo: `code-smells-project/controllers.py`
- Linha(s): 1-3, 266-291
- Arquivo: `code-smells-project/models.py`
- Linha(s): 1-2

**Evidência:** Controllers importam diretamente `models` e `get_db`; o módulo de models acessa a conexão e SQL diretamente.

**Descrição do problema:** Não existe uma fronteira de persistência que permita substituir o banco ou testar regras sem infraestrutura.

**Impacto:** Alterações de banco propagam-se para handlers e regras, aumentando risco de regressões.

**Recomendação:** Introduzir Repositories e injetar dependências; Services não devem depender de `request`, `jsonify` ou conexão global.

**Justificativa da severidade:** O acoplamento é estrutural e atravessa todos os endpoints.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Testes de contrato antes da extração.

### F-011 — Ausência de controle transacional e concorrência no pedido

**Severidade:** HIGH  
**Categoria:** Integridade / Desempenho  
**Localização:**

- Arquivo: `code-smells-project/models.py`
- Linha(s): 133-169

**Evidência:** O estoque é verificado em consultas separadas e decrementado posteriormente; falhas intermediárias não executam rollback explícito.

**Descrição do problema:** A operação de checkout envolve múltiplas leituras e escritas sem proteção explícita contra concorrência ou falha parcial.

**Impacto:** Estoque incorreto, pedidos incompletos ou venda acima do estoque em requisições simultâneas.

**Recomendação:** Usar transação com rollback, atualizar estoque de forma condicional e verificar o número de linhas afetadas.

**Justificativa da severidade:** O fluxo altera estoque e pedido, dados que precisam permanecer consistentes.

**Situação da evidência:** Confirmado por análise do fluxo.  
**Validação adicional necessária:** Teste concorrente e teste de falha entre etapas.

### F-012 — N+1 Query Problem

**Severidade:** MEDIUM  
**Categoria:** Desempenho  
**Localização:**

- Arquivo: `code-smells-project/models.py`
- Linha(s): 171-192, 203-224

**Evidência:** Para cada pedido, são feitas consultas adicionais para itens e, para cada item, outra consulta para o produto.

**Descrição do problema:** A quantidade de consultas cresce com o número de pedidos e itens retornados.

**Impacto:** Latência e carga desnecessárias no SQLite, especialmente nas listagens.

**Recomendação:** Usar JOINs ou consultas em lote e compartilhar uma estratégia de mapeamento de resultados.

**Justificativa da severidade:** O padrão é demonstrado por consultas dentro de loops, mas o impacto depende do volume.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Medir consultas e latência com volume representativo.

### F-013 — Missing Input Validation

**Severidade:** MEDIUM  
**Categoria:** Qualidade / Segurança  
**Localização:**

- Arquivo: `code-smells-project/controllers.py`
- Linha(s): 169-175, 190-202, 239-244

**Evidência:** Algumas rotas acessam `.get` sem garantir que `request.get_json()` retornou um objeto; itens de pedido, tipos numéricos e existência do usuário não são validados completamente.

**Descrição do problema:** Entradas malformadas podem gerar exceções ou chegar às regras/persistência com estrutura inválida.

**Impacto:** Respostas 500 para entradas inválidas, dados inconsistentes e comportamento inesperado.

**Recomendação:** Validar tipo, formato, limites, campos aninhados e referências antes de chamar Services.

**Justificativa da severidade:** Há validações parciais, mas não uma validação consistente em todas as rotas.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Criar matriz de casos inválidos por endpoint.

### F-014 — Tratamento inadequado de erros e exceções

**Severidade:** MEDIUM  
**Categoria:** Qualidade / Segurança  
**Localização:**

- Arquivo: `code-smells-project/controllers.py`
- Linha(s): 8-11, 21-22, 60-61, 95-96, 108-109, 125-126, 133-134, 143-144, 164-165, 185-186, 218-219, 226-227, 234-235, 261-262, 291-292
- Arquivo: `code-smells-project/app.py`
- Linha(s): 77-79

**Evidência:** Handlers capturam `Exception` de forma ampla, imprimem a exceção ou retornam seu texto diretamente ao cliente.

**Descrição do problema:** Erros de infraestrutura, validação e domínio não são diferenciados nem tratados por um mecanismo central.

**Impacto:** Diagnóstico inconsistente, vazamento de detalhes internos e contratos de erro instáveis.

**Recomendação:** Centralizar handlers, capturar exceções específicas, registrar stack trace no servidor e mapear erros de domínio para respostas seguras.

**Justificativa da severidade:** O padrão se repete em quase todas as rotas e expõe mensagens internas.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Definir catálogo de erros e códigos HTTP esperados.

### F-015 — Long Methods / Excessive Method Complexity

**Severidade:** LOW  
**Categoria:** Qualidade  
**Localização:**

- Arquivo: `code-smells-project/controllers.py`
- Linha(s): 25-63, 190-220
- Arquivo: `code-smells-project/models.py`
- Linha(s): 133-169

**Evidência:** Funções de criação de produto e pedido combinam validações, cálculos, persistência, notificações e formatação.

**Descrição do problema:** Funções com etapas independentes dificultam leitura e testes unitários.

**Impacto:** Maior custo de manutenção e risco de regressão.

**Recomendação:** Extrair validadores, casos de uso e adaptadores de notificação conforme o plano MVC.

**Justificativa da severidade:** O problema é de manutenção; os riscos de segurança e integridade já estão registrados nos findings específicos.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Não identificada.

## D. APIs deprecated e dependências obsoletas

Nenhuma API deprecated ou dependência obsoleta foi confirmada no escopo analisado. `requirements.txt` declara Flask 3.1.1 e Flask-CORS 5.0.1, mas não há lockfile nem saída de auditoria do ambiente instalado. A confirmação completa requer consulta às fontes oficiais e às dependências resolvidas.

## E. Riscos e suspeitas pendentes de validação

### S-001 — Contexto de produção do debug

**Localização:** `code-smells-project/app.py:8, 88`  
**Indício encontrado:** Debug está ativado e o servidor embutido é iniciado em `0.0.0.0`.

**Por que ainda não foi confirmado:** O README descreve execução local, portanto não é possível afirmar apenas pelo código que essa configuração é usada em produção.

**Como validar:** Inspecionar o ambiente de implantação e a configuração de execução.

**Impacto potencial:** Exposição de debugger e informações internas caso seja publicado em produção.

## F. Recomendações arquiteturais para MVC

### Estrutura sugerida

```text
code-smells-project/
  app.py
  config.py
  routes/
    products.py
    users.py
    orders.py
    admin.py
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
    product.py
    user.py
    order.py
  infrastructure/
    database.py
  shared/
    errors.py
    validation.py
```

### Responsabilidades recomendadas

**Models:** Entidades e regras invariantes simples de Produto, Usuário, Pedido e ItemPedido; sem acesso direto a HTTP ou conexão.

**Views / Routes:** Registro das rotas e tradução de entrada/saída HTTP; sem SQL ou regra de negócio.

**Controllers:** Coordenação de um caso de uso, recebendo dados já extraídos da requisição e delegando ao Service.

**Services:** Regras de negócio de produtos, autenticação, pedidos e relatórios; independentes de Flask e SQLite.

**Repositories / Persistência:** Consultas parametrizadas, mapeamento de linhas, transações e ciclo de vida da conexão.

**Configurações e componentes compartilhados:** Configuração por ambiente, autenticação/autorização, validação, logging e handlers centralizados de erro.

### Componentes que devem ser preservados

O contrato das rotas, os formatos JSON documentados pelo comportamento atual, o schema SQLite e as regras de cálculo do relatório devem ser preservados durante a refatoração, exceto mudanças de segurança aprovadas e documentadas.

### Plano arquitetural verificável aprovado

| Componente atual | Responsabilidade atual | Componente futuro | Camada | Finding relacionado | Critério de aceite |
| --- | --- | --- | --- | --- | --- |
| `app.py` | Criação da app e rotas | `app.py`, `config.py`, `routes/` | Views/Routes | F-004, F-008 | Rotas originais preservadas e configuração sem segredos |
| `controllers.py` | HTTP, validação e regras | Controllers por domínio | Controllers | F-009, F-013, F-014, F-015 | Controllers sem SQL e sem regras de persistência |
| Funções de negócio em `controllers.py`/`models.py` | Casos de uso | `services/` | Services | F-006, F-009, F-011 | Services independentes de Flask/request |
| SQL em `models.py` | Persistência e mapeamento | `repositories/` | Repositories | F-002, F-010, F-012 | Consultas parametrizadas e sem acesso direto pelos controllers |
| `database.py` | Conexão e schema | `infrastructure/database.py` | Persistência | F-007, F-011 | Conexão por contexto e transações verificáveis |
| Dados de domínio | Estrutura de registros | `models/` | Models | F-005, F-006 | Senhas nunca serializadas; entidades sem SQL |

**Dependências permitidas entre camadas:** Routes/Views -> Controllers -> Services -> Repositories/Models; infraestrutura fornece conexão aos Repositories. Models não dependem de Flask; Services não dependem de request/response; Controllers não executam SQL.

**Camadas deliberadamente não criadas:** Nenhuma das camadas previstas acima; cada uma é necessária para corrigir os achados e tornar a arquitetura verificável.

**Critérios de aceite arquiteturais:** Existência efetiva das camadas; ausência de SQL em Controllers/Services; consultas parametrizadas nos Repositories; Services sem Flask; conexão não global; autenticação/autorização fora das regras de persistência; Models sem senha em respostas.

## G. Plano sugerido de refatoração

| Etapa | Alteração proposta | Findings relacionados | Risco |
| --- | --- | --- | --- |
| 1 | Remover/restringir SQL arbitrário, externalizar configuração e definir autenticação/autorização | F-001, F-003, F-004, F-008 | Alto |
| 2 | Criar infraestrutura de banco, Repositories e parametrizar consultas | F-002, F-007, F-010, F-011, F-012 | Alto |
| 3 | Criar Models e Services por domínio, incluindo hash de senha e checkout transacional | F-005, F-006, F-011 | Alto |
| 4 | Separar Routes/Controllers, validação e tratamento centralizado de erros | F-009, F-013, F-014, F-015 | Médio |
| 5 | Executar testes de contrato, segurança, concorrência e desempenho | Todos | Médio |

O plano preserva endpoints, formatos de resposta, regras de relatório e compatibilidade com o schema sempre que possível. A remoção da rota de SQL arbitrário e a mudança do fluxo de autenticação são mudanças de segurança que deverão ser explicitamente aprovadas antes da execução.

## H. Estratégia de validação após a refatoração

### Verificações previstas

- [ ] Verificar dependências instaladas com `pip install -r requirements.txt`.
- [ ] Iniciar a aplicação sem erros.
- [ ] Criar testes automatizados para rotas existentes.
- [ ] Verificar `GET /`, `GET /health`, rotas de produtos, usuários, pedidos, login e relatório.
- [ ] Verificar códigos HTTP e formatos JSON existentes.
- [ ] Testar entradas SQL maliciosas e confirmar que não alteram a consulta.
- [ ] Confirmar que senhas são armazenadas com hash e nunca aparecem em respostas.
- [ ] Confirmar proteção das rotas administrativas e de escrita.
- [ ] Testar rollback e concorrência no checkout.
- [ ] Medir consultas das listagens de pedidos após eliminar N+1.
- [ ] Confirmar imports e dependências permitidas entre camadas.

Nenhum endpoint foi executado nesta Fase 2; a validação de runtime fica para a Fase 3, após autorização.

## I. Conclusão da auditoria

A Fase 1 identificou uma API Flask/SQLite monolítica, com quatro arquivos-fonte de aplicação analisados. A Fase 2 confirmou 15 findings, incluindo seis críticos, e propôs uma arquitetura MVC com Services e Repositories efetivamente implementados. A refatoração é viável, mas deve começar pelos riscos de segurança e manter rastreabilidade entre cada finding, mudança e teste.

### Status da execução

- Fase 1 — Análise: concluída.
- Fase 2 — Auditoria: concluída.
- Fase 3 — Refatoração: aguardando autorização.

### Confirmação obrigatória

**A auditoria foi concluída e o relatório está disponível. Você autoriza iniciar a Fase 3 — Refatoração MVC, conforme o plano apresentado?**

## J. Atualização obrigatória após a Fase 3

Esta seção deverá ser preenchida ao final da refatoração, com a arquitetura realmente implementada, diferenças em relação ao plano, critérios funcionais/de segurança/arquiteturais, rastreabilidade dos findings, testes reais e limitações.
