# Relatório de Auditoria Arquitetural — code-smells-project

## 1. Objetivo

Este relatório registra a análise arquitetural e a auditoria realizadas nas Fases 1 e 2 da Skill `refactor-arch`. A análise foi baseada nos arquivos do projeto e no catálogo de anti-patterns fornecido pela Skill. Nenhum arquivo-fonte da aplicação foi alterado.

## A. Identificação do projeto

| Campo | Informação |
| --- | --- |
| Projeto | code-smells-project |
| Linguagem | Python; versão não declarada no projeto |
| Framework | Flask 3.1.1 |
| Dependências principais | `flask==3.1.1`, `flask-cors==5.0.1` |
| Banco de dados | SQLite, arquivo `loja.db`, criado em `database.py` |
| Arquitetura atual | Monolítica parcialmente organizada por módulos; não há separação real entre controllers, serviços e persistência |
| Arquitetura desejada | MVC adaptado à API Flask |
| Escopo analisado | `app.py`, `controllers.py`, `models.py`, `database.py`, `requirements.txt`, `README.md` |
| Data da auditoria | 2026-11-05 |

### Resumo da arquitetura atual

`app.py` configura o Flask, registra rotas, expõe rotas administrativas e inicializa o banco. `controllers.py` contém handlers HTTP, validações, regras de negócio, notificações simuladas e tratamento de exceções. `models.py` concentra acesso direto ao SQLite, montagem de SQL, regras de pedidos e relatórios. `database.py` cria o esquema, mantém uma conexão global e insere dados iniciais.

Há uma separação nominal entre `controllers.py` e `models.py`, mas não há uma camada de serviços, repositórios ou autenticação/autorização. A aplicação é um monólito simples com acoplamento direto entre Flask, controllers e SQLite.

### Limitações da análise

- Não há arquivo de versão do Python nem arquivo de lock de dependências.
- Não foram encontrados testes automatizados no escopo analisado.
- Não foi executada a aplicação nem feita varredura externa de vulnerabilidades de dependências nesta fase.
- A confirmação de APIs deprecated ficou limitada à inspeção do código e das versões declaradas; não foi confirmada nenhuma ocorrência.

## B. Resumo executivo

Foram confirmados cinco riscos CRITICAL, incluindo uma chave secreta hardcoded, SQL Injection, execução arbitrária de SQL, ausência de autenticação/autorização em rotas sensíveis e armazenamento/exposição de senhas em texto puro. As ocorrências foram agrupadas para que a mesma evidência não seja contada em mais de um finding; riscos relacionados são mantidos separados somente quando representam causas ou impactos distintos.

Também foram identificados problemas HIGH de configuração e exposição de informações, estado global mutável e concentração de responsabilidades nos controllers. Há problemas MEDIUM de consultas N+1, validação insuficiente e tratamento genérico de exceções, além de problemas LOW de valores mágicos e métodos extensos.

A refatoração deve priorizar a remoção de SQL construído por concatenação, o bloqueio das rotas administrativas, o hash das senhas e a retirada de segredos/debug das respostas e configurações. A reorganização MVC deve preservar os endpoints e contratos existentes, salvo mudanças de segurança explicitamente aprovadas.

### Quantidade de findings

| Severidade | Quantidade |
| --- | ---: |
| CRITICAL | 5 |
| HIGH | 4 |
| MEDIUM | 3 |
| LOW | 2 |
| **Total** | **14** |

### Prioridades recomendadas

1. Eliminar execução arbitrária e SQL Injection, protegendo a integridade do banco.
2. Implementar autenticação/autorização, remover segredos hardcoded e proteger senhas.
3. Separar regras de negócio e persistência, corrigir transações/consultas e centralizar erros.

## C. Findings confirmados

### F-001 — Chave secreta hardcoded

**Severidade:** CRITICAL  
**Categoria:** Segurança  
**Localização:**

- Arquivo: `app.py`
- Linha(s): 7

**Evidência:** O código define a `SECRET_KEY` com valor literal em `app.py:7`.

**Descrição do problema:** A chave secreta da aplicação é armazenada diretamente no código versionado.

**Impacto:** Exposição da chave da aplicação, que pode ser reutilizada por qualquer pessoa com acesso ao código.

**Recomendação:** Usar variáveis de ambiente ou um gerenciador de segredos; remover a chave do código e rotacioná-la caso tenha sido exposta.

**Justificativa da severidade:** A exposição é direta e afeta autenticação/configuração sensível.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Verificar se a chave já foi publicada em repositório ou ambientes compartilhados.

### F-002 — SQL Injection por concatenação de entrada

**Severidade:** CRITICAL  
**Categoria:** Segurança  
**Localização:**

- Arquivo: `models.py`
- Linha(s): 28, 47-50, 57-60, 68, 92, 110, 127-129, 140, 148-163, 174, 188, 192, 220, 224, 279, 299-304

**Evidência:** Consultas são montadas com concatenação de parâmetros como `email`, `senha`, `nome`, `termo`, `categoria` e identificadores antes de `cursor.execute`.

**Descrição do problema:** Dados controláveis por requisições são incorporados diretamente ao SQL, sem placeholders.

**Impacto:** Um atacante pode alterar a semântica das consultas, consultar dados indevidos, modificar registros ou excluir dados.

**Recomendação:** Usar consultas parametrizadas (`?` no SQLite) em todas as operações e validar tipos/formas de entrada nas camadas apropriadas.

**Justificativa da severidade:** Há entradas HTTP que alcançam diretamente consultas SQL construídas por concatenação.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Criar testes de regressão para entradas com aspas, curingas e tentativas de alteração de consulta.

### F-003 — Execução arbitrária de SQL

**Severidade:** CRITICAL  
**Categoria:** Segurança  
**Localização:**

- Arquivo: `app.py`
- Linha(s): 59-79

**Evidência:** A rota `POST /admin/query` lê `dados["sql"]` e executa o conteúdo diretamente com `cursor.execute(query)`.

**Descrição do problema:** O cliente escolhe o comando SQL completo que será executado.

**Impacto:** Leitura, alteração ou exclusão arbitrária de dados e possível comprometimento total do banco.

**Recomendação:** Remover a rota; expor somente operações de negócio específicas, com autenticação/autorização e consultas parametrizadas.

**Justificativa da severidade:** O endpoint oferece uma capacidade destrutiva e irrestrita por design.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Verificar consumidores legítimos antes de remover o endpoint.

### F-004 — Ausência de autenticação e autorização

**Severidade:** CRITICAL  
**Categoria:** Segurança  
**Localização:**

- Arquivo: `app.py`
- Linha(s): 11-29, 47-56

**Evidência:** As rotas de criação, atualização, exclusão, alteração de status e `/admin/reset-db` são registradas sem middleware ou verificação de usuário/permissão. A execução arbitrária de SQL em `/admin/query` é tratada separadamente em F-003.

**Descrição do problema:** O endpoint `/login` existe, mas não há mecanismo que proteja as demais operações ou valide propriedade/administrador.

**Impacto:** Usuários anônimos podem operar sobre produtos, usuários, pedidos e banco; dados de outros usuários podem ser consultados ou alterados.

**Recomendação:** Implementar autenticação, autorização por papel e verificação de propriedade, protegendo especialmente rotas administrativas e mutações.

**Justificativa da severidade:** Operações destrutivas e administrativas estão acessíveis sem controle de acesso observável.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Confirmar se existe algum controle externo no ambiente de implantação.

### F-005 — Senhas armazenadas e expostas em texto puro

**Severidade:** CRITICAL  
**Categoria:** Segurança  
**Localização:**

- Arquivo: `database.py`
- Linha(s): 31-34, 76-81
- Arquivo: `models.py`
- Linha(s): 83-101, 105-120

**Evidência:** A coluna `usuarios.senha` armazena a senha diretamente; `get_todos_usuarios` e `get_usuario_por_id` incluem `senha` nas respostas.

**Descrição do problema:** Senhas não são transformadas em hash e podem ser retornadas pela API.

**Impacto:** Um vazamento do banco ou uma chamada aos endpoints de usuários expõe credenciais reutilizáveis.

**Recomendação:** Usar Argon2id, bcrypt ou PBKDF2 via biblioteca apropriada, migrar dados existentes e nunca retornar senha ou hash em respostas.

**Justificativa da severidade:** O problema combina armazenamento inseguro com exposição por API.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Verificar se o banco de desenvolvimento foi compartilhado e planejar rotação das credenciais.

### F-006 — Configuração de debug e informações internas expostas

**Severidade:** HIGH  
**Categoria:** Segurança / Configuração  
**Localização:**

- Arquivo: `app.py`
- Linha(s): 8, 102
- Arquivo: `controllers.py`
- Linha(s): 276-289

**Evidência:** `DEBUG` está fixado como `True`; a resposta de `/health` inclui `db_path` e `debug`. A chave hardcoded é tratada separadamente em F-001.

**Descrição do problema:** Configuração de desenvolvimento e detalhes internos são expostos como comportamento de produção.

**Impacto:** Pode haver exposição de detalhes de execução úteis para exploração; debug pode revelar informações adicionais em erros.

**Recomendação:** Configurar por ambiente, desabilitar debug em produção e restringir o health check a informações operacionais não sensíveis.

**Justificativa da severidade:** A exposição é acessível por endpoint público e inclui material sensível.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Confirmar o ambiente de implantação.

### F-007 — Estado global mutável para conexão SQLite

**Severidade:** HIGH  
**Categoria:** Arquitetura / Concorrência  
**Localização:**

- Arquivo: `database.py`
- Linha(s): 4, 8-10

**Evidência:** `db_connection` é global, inicializado uma vez e compartilhado com `check_same_thread=False`.

**Descrição do problema:** A conexão é compartilhada entre requisições e threads sem gerenciamento explícito de ciclo de vida ou isolamento.

**Impacto:** Risco de condições de corrida, interferência entre requisições e falhas difíceis de reproduzir.

**Recomendação:** Usar conexão por contexto/requisição, fechamento controlado e transações explícitas; avaliar um mecanismo adequado ao modelo de implantação.

**Justificativa da severidade:** O estado compartilhado afeta todas as operações de persistência concorrentes.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Teste de concorrência em ambiente representativo.

### F-008 — Fat Controller

**Severidade:** HIGH  
**Categoria:** Arquitetura / Qualidade  
**Localização:**

- Arquivo: `controllers.py`
- Linha(s): 24-108, 146-220, 237-291

**Evidência:** Handlers fazem parsing e validação, aplicam regras de negócio, chamam persistência, imprimem notificações e formatam respostas HTTP.

**Descrição do problema:** Controllers concentram responsabilidades de apresentação, negócio, integração e tratamento de erro.

**Impacto:** Testes isolados ficam difíceis e mudanças no domínio ficam acopladas ao Flask.

**Recomendação:** Manter controllers como coordenadores; extrair validações/regras para services, persistência para repositories/models e notificações para componentes próprios.

**Justificativa da severidade:** A concentração é transversal aos principais fluxos de produtos, usuários e pedidos.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Mapear contratos atuais antes de extrair componentes.

### F-009 — Falta de controle transacional no fluxo de pedido

**Severidade:** HIGH  
**Categoria:** Integridade / Arquitetura  
**Localização:**

- Arquivo: `models.py`
- Linha(s): 133-168

**Evidência:** `criar_pedido` faz várias leituras e escritas e só faz `commit` ao final; não há rollback explícito no fluxo de erro nem proteção contra concorrência na atualização do estoque.

**Descrição do problema:** O fluxo possui um `commit` único, mas não explicita o tratamento de falhas da unidade de trabalho nem protege a atualização concorrente do estoque. O finding não afirma ausência completa de transação.

**Impacto:** Falhas ou concorrência podem produzir inconsistências, pedidos parciais ou estoque incorreto.

**Recomendação:** Tornar explícitos rollback e tratamento de falhas, validar todos os itens antes das escritas e atualizar estoque de forma atômica com verificação de quantidade e controle de concorrência.

**Justificativa da severidade:** O fluxo altera múltiplas tabelas e afeta diretamente a integridade financeira e de estoque; o risco decorre do tratamento incompleto de falhas e da concorrência, não da ausência completa de transação.

**Situação da evidência:** Confirmado por análise estrutural do código.  
**Validação adicional necessária:** Testes de falha no meio da operação e concorrência.

### F-010 — N+1 queries ao carregar pedidos

**Severidade:** MEDIUM  
**Categoria:** Desempenho  
**Localização:**

- Arquivo: `models.py`
- Linha(s): 171-229

**Evidência:** Para cada pedido é criada uma consulta de itens e, para cada item, outra consulta de produto (`cursor2` e `cursor3`).

**Descrição do problema:** A quantidade de consultas cresce com o número de pedidos e itens retornados.

**Impacto:** Latência e carga no SQLite aumentam desnecessariamente em listagens maiores.

**Recomendação:** Usar JOINs ou consultas em lote e medir o ganho com dados representativos.

**Justificativa da severidade:** O impacto é de desempenho e tende a crescer com o volume de dados, sem evidência de indisponibilidade atual.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Medir número de queries e latência com massa de dados.

### F-011 — Validação de entrada incompleta

**Severidade:** MEDIUM  
**Categoria:** Qualidade / Segurança  
**Localização:**

- Arquivo: `controllers.py`
- Linha(s): 148-181, 190-204, 239-244

**Evidência:** Campos são lidos sem validação consistente de tipo/estrutura; login usa `dados.get` sem validar se `request.get_json()` retornou objeto; itens de pedido e status não têm validação de estrutura/existência completa.

**Descrição do problema:** Entradas inválidas podem alcançar regras de negócio e persistência ou gerar exceções.

**Impacto:** Erros 500, dados inconsistentes e comportamento inesperado em operações de pedido e autenticação.

**Recomendação:** Validar schema, tipos, limites e relações antes de chamar a camada de negócio, retornando erros 4xx consistentes.

**Justificativa da severidade:** Há controles parciais, mas não cobertura suficiente para os contratos recebidos.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Definir schemas e testar entradas ausentes, nulas e de tipo incorreto.

### F-012 — Tratamento genérico de exceções e exposição de detalhes

**Severidade:** MEDIUM  
**Categoria:** Qualidade / Segurança  
**Localização:**

- Arquivo: `app.py`
- Linha(s): 77-79
- Arquivo: `controllers.py`
- Linha(s): 10-11, 21-22, 60-61, 95-96, 108-109, 125-126, 143-144, 164-165, 185-186, 218-219, 226-227, 234-235, 254-255, 261-262, 291

**Evidência:** Exceções são capturadas como `Exception` e `str(e)` é devolvido diretamente ao cliente; vários fluxos também usam `print` para erros.

**Descrição do problema:** O tratamento não diferencia erros esperados de falhas internas e pode expor mensagens de infraestrutura.

**Impacto:** Respostas inconsistentes, diagnóstico frágil e possível divulgação de nomes de tabelas, SQL ou detalhes internos.

**Recomendação:** Centralizar handlers, capturar exceções específicas, registrar detalhes em logger interno e retornar mensagens públicas estáveis.

**Justificativa da severidade:** O padrão está disseminado e combina observabilidade inadequada com exposição de erro.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Catalogar os erros esperados por endpoint.

### F-013 — Valores mágicos e strings de domínio espalhados

**Severidade:** LOW  
**Categoria:** Qualidade  
**Localização:**

- Arquivo: `controllers.py`
- Linha(s): 44, 241
- Arquivo: `models.py`
- Linha(s): 146-147, 247-253, 294-297

**Evidência:** Categorias, status de pedido, limites de desconto e mensagens SQL são definidos diretamente em handlers e consultas.

**Descrição do problema:** Regras e estados de domínio ficam espalhados e sem constantes ou estruturas nomeadas.

**Impacto:** Maior risco de inconsistência e manutenção mais difícil.

**Recomendação:** Centralizar enums/constantes e regras de domínio em módulo apropriado.

**Justificativa da severidade:** O impacto principal é de manutenção, sem risco independente demonstrado.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Não identificada.

### F-014 — Métodos extensos com responsabilidades múltiplas

**Severidade:** LOW  
**Categoria:** Qualidade  
**Localização:**

- Arquivo: `models.py`
- Linha(s): 133-168, 171-200, 203-229
- Arquivo: `database.py`
- Linha(s): 7-86

**Evidência:** `criar_pedido`, os carregadores de pedidos e `get_db` combinam múltiplas etapas de validação, consulta, transformação, persistência e inicialização.

**Descrição do problema:** Métodos longos dificultam entendimento, testes unitários e evolução.

**Impacto:** Maior chance de regressões e baixo isolamento de regras.

**Recomendação:** Extrair funções coesas após definir os limites de services/repositories e preservar os contratos atuais.

**Justificativa da severidade:** É um problema de manutenibilidade, sem impacto operacional independente confirmado.

**Situação da evidência:** Confirmado por análise do código.  
**Validação adicional necessária:** Não identificada.

## D. APIs deprecated e dependências obsoletas

Nenhuma API deprecated ou dependência obsoleta foi confirmada no escopo analisado.

`requirements.txt` declara Flask 3.1.1 e Flask-CORS 5.0.1, mas não há arquivo de lock para verificar dependências transitivas. A análise não classificou APIs como deprecated apenas por serem antigas ou por não haver informação de versão do Python.

## E. Riscos e suspeitas pendentes de validação

### S-001 — Exposição histórica de credenciais

**Localização:** `app.py:7` e `database.py:76-79`  
**Indício encontrado:** Há segredos e senhas literais no código.

**Por que ainda não foi confirmado:** Não foi verificado o histórico do repositório nem os ambientes em que os valores foram usados.

**Como validar:** Auditar histórico, artefatos de CI/CD e ambientes; rotacionar valores caso tenham sido publicados ou utilizados fora de desenvolvimento.

**Impacto potencial:** Comprometimento de sessões, contas e dados.

### S-002 — Vulnerabilidades em dependências transitivas

**Localização:** `requirements.txt`  
**Indício encontrado:** Não existe lock file nem relatório de auditoria de dependências.

**Por que ainda não foi confirmado:** Não foi executada consulta de vulnerabilidades ou resolução do ambiente.

**Como validar:** Gerar lock em ambiente controlado e executar ferramenta de auditoria compatível com Python.

**Impacto potencial:** Vulnerabilidades herdadas de dependências transitivas.

## F. Recomendações arquiteturais para MVC

### Estrutura sugerida

```text
app/
  __init__.py
  config.py
  routes/
    products.py
    users.py
    orders.py
    reports.py
  controllers/
    products.py
    users.py
    orders.py
  services/
    product_service.py
    user_service.py
    order_service.py
  models/
    product.py
    user.py
    order.py
  repositories/
    product_repository.py
    user_repository.py
    order_repository.py
  database.py
  errors.py
run.py
```

### Responsabilidades recomendadas

**Models:** Representar entidades e regras simples de domínio, sem conhecer Flask.

**Views / Routes:** Registrar rotas, receber requisições e delegar aos controllers; não executar SQL.

**Controllers:** Coordenar entrada, chamar services e construir respostas HTTP.

**Services:** Implementar validações de negócio, autenticação/autorização, pedidos, estoque e notificações.

**Repositories / Persistência:** Encapsular SQLite, usar queries parametrizadas e controlar transações.

**Configurações e componentes compartilhados:** Variáveis de ambiente, logging, tratamento centralizado de erros, schemas de entrada e middleware de autenticação.

### Componentes que devem ser preservados

Os endpoints e formatos de resposta atuais devem ser preservados durante a migração, sempre que compatível com a correção dos riscos. A inicialização do SQLite e os dados de exemplo podem permanecer como mecanismo de desenvolvimento, desde que credenciais e configuração sejam externalizadas.

## G. Plano sugerido de refatoração

| Etapa | Alteração proposta | Findings relacionados | Risco da mudança |
| --- | --- | --- | --- |
| 1 | Remover `/admin/query`, proteger `/admin/reset-db` e substituir SQL concatenado por queries parametrizadas | F-002, F-003, F-004 | Alto |
| 2 | Externalizar segredo, desabilitar debug por padrão, retirar dados sensíveis do health check e implementar logging seguro | F-001, F-006, F-012 | Médio |
| 3 | Migrar senhas para hash, remover senha das respostas e adicionar autenticação/autorização | F-004, F-005 | Alto |
| 4 | Separar controllers, services e repositories; controlar conexão e transações | F-007, F-008, F-009, F-014 | Alto |
| 5 | Corrigir carregamento de pedidos, schemas de entrada e constantes de domínio | F-010, F-011, F-013 | Médio |

O plano deve preservar endpoints, formatos de resposta e regras de negócio documentadas. A remoção de `/admin/query` e a alteração do tratamento de senhas são mudanças intencionais de segurança e exigem confirmação antes da execução.

## H. Estratégia de validação após a refatoração

### Verificações previstas

- [ ] Verificar e fixar a versão de Python e as dependências.
- [ ] Executar testes automatizados, caso sejam adicionados ou encontrados.
- [ ] Iniciar a aplicação sem debug inseguro e verificar `/` e `/health`.
- [ ] Verificar endpoints de produtos, usuários, login, pedidos e relatório.
- [ ] Confirmar códigos HTTP e formatos JSON existentes.
- [ ] Testar autenticação e autorização em rotas públicas, protegidas e administrativas.
- [ ] Testar SQL Injection, payloads inválidos e erros sem exposição de detalhes internos.
- [ ] Testar criação de pedidos, rollback e concorrência de estoque.
- [ ] Medir consultas de listagem de pedidos após a correção do N+1.

Nenhum teste foi executado nesta auditoria; os itens acima são validações previstas para a Fase 3.

## I. Conclusão da auditoria

A auditoria confirmou riscos críticos que devem ser tratados antes de qualquer exposição da API em ambiente não controlado, especialmente SQL arbitrário, SQL Injection, ausência de autorização e senhas em texto puro. A estrutura atual permite uma migração incremental para MVC, mas a ordem deve priorizar segurança e integridade transacional antes da reorganização de pastas.

### Status da execução

- Fase 1 — Análise: concluída.
- Fase 2 — Auditoria: concluída.
- Fase 3 — Refatoração: aguardando autorização.

### Confirmação obrigatória

**A auditoria foi concluída e o relatório está disponível. Você autoriza iniciar a Fase 3 — Refatoração MVC, conforme o plano apresentado?**
