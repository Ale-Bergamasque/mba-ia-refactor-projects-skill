# Diretrizes de Arquitetura MVC

## 1. Objetivo

Este documento orienta a Fase 3 — Refatoração da Skill `refactor-arch`.

Seu objetivo é definir responsabilidades arquiteturais e boas práticas para reorganizar aplicações legadas em uma arquitetura baseada em MVC (Model-View-Controller).

A Skill deve adaptar as recomendações à linguagem, ao framework e à estrutura existente.

A refatoração só pode começar após autorização explícita do usuário, concedida depois da apresentação do relatório de auditoria.

---

## 2. Princípios gerais

A refatoração deve seguir os seguintes princípios:

1. Preservar as funcionalidades existentes.
2. Manter os contratos dos endpoints sempre que possível.
3. Separar responsabilidades de apresentação, coordenação, negócio e persistência.
4. Evitar duplicação de regras de negócio.
5. Reduzir o acoplamento entre componentes.
6. Facilitar testes e manutenção.
7. Reaproveitar estruturas existentes que já estejam adequadas.
8. Não introduzir camadas, abstrações ou dependências sem necessidade.
9. Executar mudanças incrementais, verificando seus efeitos.
10. Registrar alterações de comportamento que não puderem ser evitadas.

A ausência de uma pasta chamada `controllers`, `models` ou `views` não significa, por si só, que a arquitetura está incorreta.

O importante é identificar as responsabilidades reais de cada componente.

---

## 3. Responsabilidades das camadas

### 3.1. Model

O Model representa os dados, as entidades e seus relacionamentos.

Responsabilidades possíveis:

- Definir entidades do domínio.
- Representar atributos e relacionamentos.
- Aplicar invariantes simples relacionadas à própria entidade.
- Integrar-se ao ORM, quando a tecnologia utilizar esse padrão.

Evitar:

- Manipular requisições HTTP.
- Construir respostas HTTP.
- Enviar notificações diretamente.
- Concentrar fluxos complexos de negócio.
- Executar operações administrativas não relacionadas à entidade.

Exemplo:

```python
class Product:
    def __init__(self, id, name, price):
        self.id = id
        self.name = name
        self.price = price
```

A implementação real pode utilizar classes, estruturas de dados, schemas ou modelos de ORM.

### 3.2. View / Routes

Em aplicações web tradicionais, a View é responsável pela apresentação.

Em APIs REST, essa responsabilidade geralmente aparece na serialização dos dados e na construção das respostas HTTP. As rotas fazem parte da interface HTTP da aplicação.

Responsabilidades possíveis:

- Definir caminhos e métodos HTTP.
- Receber dados de requisições.
- Encaminhar chamadas aos Controllers.
- Retornar respostas com códigos HTTP apropriados.
- Serializar dados utilizando os mecanismos do framework.

Evitar:

- Executar SQL diretamente.
- Concentrar regras de negócio.
- Realizar cálculos complexos de domínio.
- Controlar transações diretamente nas rotas.

Exemplo em Flask:

```python
@app.route("/products", methods=["GET"])
def list_products():
    return product_controller.list_products()
```

Exemplo em Express:

```javascript
router.get("/products", productController.listProducts);
```

**Observação:** alguns frameworks combinam as responsabilidades de Routes e Controllers. A Skill deve respeitar as convenções existentes e separar essas responsabilidades somente quando houver benefício real.

### 3.3. Controller

O Controller coordena a execução de uma solicitação.

Responsabilidades possíveis:

- Receber dados encaminhados pelas rotas.
- Acionar validações de entrada.
- Chamar Services ou componentes responsáveis.
- Coordenar o resultado da operação.
- Produzir ou encaminhar a resposta apropriada.

Evitar:

- Concentrar regras de negócio complexas.
- Executar consultas SQL diretamente.
- Gerenciar conexões de banco de dados.
- Implementar lógica de pagamento ou estoque dentro do Controller.

Exemplo:

```python
class ProductController:
    def __init__(self, product_service):
        self.product_service = product_service

    def list_products(self):
        products = self.product_service.list_products()
        return {"products": products}
```

A formatação final da resposta deve seguir as convenções do framework utilizado.

### 3.4. Service

A camada Service é complementar ao MVC e deve ser utilizada quando existir lógica de negócio que justifique sua separação.

Responsabilidades possíveis:

- Implementar regras de negócio.
- Coordenar operações de domínio.
- Aplicar validações de negócio.
- Orquestrar operações relacionadas.
- Coordenar transações, quando apropriado.
- Integrar diferentes componentes de domínio.

Evitar:

- Depender diretamente de objetos de requisição HTTP.
- Construir respostas específicas do framework web.
- Repetir regras já implementadas em outros Services.

Exemplo:

```python
class OrderService:
    def __init__(self, order_repository):
        self.order_repository = order_repository

    def create_order(self, order_data):
        self.validate_order(order_data)
        return self.order_repository.save(order_data)

    def validate_order(self, order_data):
        if not order_data.get("items"):
            raise ValueError("Pedido sem itens")
```

Não criar um Service que apenas repasse chamadas sem acrescentar organização ou responsabilidade útil.

### 3.5. Repository / Persistência

A camada Repository também é complementar ao MVC.

Seu objetivo é isolar operações de acesso aos dados quando essa separação fizer sentido.

Responsabilidades possíveis:

- Executar consultas ao banco de dados.
- Recuperar e persistir entidades.
- Encapsular detalhes de SQL ou ORM.
- Organizar consultas reutilizáveis.

Evitar:

- Construir respostas HTTP.
- Implementar regras de autenticação.
- Concentrar cálculos e regras de negócio.
- Enviar notificações diretamente.

Exemplo:

```python
class ProductRepository:
    def __init__(self, connection):
        self.connection = connection

    def find_by_id(self, product_id):
        cursor = self.connection.execute(
            "SELECT * FROM products WHERE id = ?",
            (product_id,)
        )
        return cursor.fetchone()
```

**Observação:** em projetos pequenos que utilizam um ORM, uma camada Repository adicional pode não ser necessária. A Skill deve justificar sua criação.

---

## 4. Fluxo recomendado de uma requisição

Em uma API com camadas separadas, o fluxo pode seguir:

```text
Cliente HTTP
    |
    v
Route
    |
    v
Controller
    |
    v
Service
    |
    v
Repository
    |
    v
Banco de dados
```

O resultado retorna pelas camadas responsáveis até ser convertido em resposta HTTP.

Esse fluxo é uma referência, não uma estrutura obrigatória para todos os projetos.

Por exemplo, uma operação simples pode não precisar de Service ou Repository específicos.

---

## 5. Estrutura de diretórios

A Skill deve primeiro identificar a estrutura existente e preservar os componentes que já possuem responsabilidades adequadas.

### 5.1. Exemplo para Python / Flask

```text
project/
├── app.py
├── config.py
├── models/
├── routes/
├── controllers/
├── services/
├── repositories/
├── schemas/
├── tests/
└── requirements.txt
```

Possíveis responsabilidades:

- `app.py`: inicialização e configuração da aplicação.
- `config.py`: configurações por ambiente.
- `models/`: entidades e modelos do banco.
- `routes/`: definição dos endpoints.
- `controllers/`: coordenação das requisições.
- `services/`: regras de negócio.
- `repositories/`: persistência.
- `schemas/`: validação e serialização.
- `tests/`: testes automatizados.

Não criar todas essas pastas automaticamente.

### 5.2. Exemplo para Node.js / Express

```text
project/
├── src/
│   ├── app.js
│   ├── routes/
│   ├── controllers/
│   ├── models/
│   ├── services/
│   ├── repositories/
│   ├── middlewares/
│   └── config/
├── tests/
└── package.json
```

Possíveis responsabilidades:

- `app.js`: configuração e inicialização.
- `routes/`: declaração das rotas.
- `controllers/`: tratamento e coordenação das requisições.
- `models/`: entidades e modelos de dados.
- `services/`: regras de negócio.
- `repositories/`: operações de persistência.
- `middlewares/`: autenticação, autorização e tratamento de erros.
- `config/`: configurações da aplicação.

A estrutura deve respeitar as convenções do projeto e das bibliotecas utilizadas.

---

## 6. Regras para separação de responsabilidades

### 6.0. Plano arquitetural verificável

Antes de modificar o código, a Skill deve registrar um plano específico para o projeto. O plano deve conter a arquitetura escolhida, as camadas realmente implementadas, suas responsabilidades, o mapeamento de componentes atuais para futuros componentes, as dependências permitidas, os findings relacionados e critérios de aceite funcionais, de segurança e arquiteturais.

O plano não deve impor nomes de pastas ou tecnologias. Uma camada opcional, como Service ou Repository, pode ser omitida somente quando houver justificativa explícita. Se a camada for escolhida, sua implementação é obrigatória; não é suficiente manter a responsabilidade no componente legado com outro nome.

Qualquer mudança relevante no plano, incluindo remover uma camada aprovada, manter persistência em uma camada que deveria ser apenas de domínio ou alterar a direção das dependências, exige nova autorização explícita do usuário.

### 6.0.1. Critérios arquiteturais mínimos

Ao final, verificar por evidências:

- cada responsabilidade está na camada definida no plano;
- Controllers coordenam HTTP e não executam persistência diretamente;
- Services concentram regras de negócio complexas e não dependem de objetos HTTP;
- Repositories isolam SQL/ORM quando previstos;
- Models não acumulam responsabilidades de Service e Repository;
- Views/Routes não concentram regras de negócio ou transações;
- as dependências entre camadas seguem a direção aprovada;
- não existe apenas uma renomeação ou redistribuição superficial para simular a arquitetura.

### 6.1. Regras de negócio

Regras de negócio não devem ficar misturadas ao tratamento HTTP.

Exemplos:

- Cálculo de total de pedidos.
- Verificação de disponibilidade de estoque.
- Regras de matrícula ou inscrição.
- Regras de pagamento.
- Critérios de alteração de status.

Quando essas regras forem complexas ou reutilizadas, devem ser extraídas para Services ou componentes de domínio.

### 6.2. Persistência

O acesso ao banco de dados deve estar organizado em componentes apropriados.

A Skill deve:

- Evitar SQL diretamente em rotas.
- Utilizar consultas parametrizadas.
- Respeitar o ORM existente, quando houver.
- Preservar o esquema do banco sempre que possível.
- Evitar mudanças desnecessárias em tabelas e relacionamentos.
- Se o plano aprovar uma camada Repository, toda consulta, escrita, transação e detalhe específico do mecanismo de persistência deve ficar nessa camada ou em componentes internos subordinados a ela.
- Se o projeto preservar Models como camada de persistência, registrar essa decisão no plano e não declarar uma arquitetura Models/Services/Repositories como implementada.

### 6.3. Validação

Distinguir dois tipos de validação:

**Validação de entrada:**

- Campos obrigatórios.
- Tipos de dados.
- Formatos.
- Limites de tamanho.
- Estrutura da requisição.

**Validação de negócio:**

- Estoque suficiente.
- Permissão para realizar uma operação.
- Regras de status.
- Restrições de domínio.

A validação de entrada pode ocorrer em schemas, middlewares ou mecanismos do framework.

A validação de negócio deve permanecer nos componentes responsáveis pelo domínio.

### 6.4. Tratamento de erros

A Skill deve buscar um tratamento consistente de erros.

Recomendações:

- Utilizar exceções específicas.
- Evitar capturar erros silenciosamente.
- Retornar códigos HTTP apropriados.
- Registrar falhas sem expor segredos.
- Centralizar o tratamento de erros quando o framework permitir.

### 6.5. Configurações e credenciais

Configurações sensíveis não devem ficar fixas no código.

Recomendações:

- Utilizar variáveis de ambiente.
- Separar configurações de desenvolvimento e produção.
- Evitar expor credenciais em logs ou endpoints.
- Não versionar arquivos com segredos.
- Documentar variáveis necessárias sem divulgar seus valores.

---

## 7. Segurança durante a refatoração

A Skill deve considerar os findings de segurança identificados na auditoria.

Prioridades possíveis:

1. Remover execução arbitrária de SQL.
2. Corrigir SQL Injection.
3. Proteger autenticação e autorização.
4. Corrigir armazenamento e exposição de senhas.
5. Remover credenciais hardcoded.
6. Corrigir exposição de dados sensíveis.
7. Revisar configurações inseguras.

Essas prioridades devem ser ajustadas conforme os riscos e as dependências do projeto.

**Importante:** correções de segurança podem alterar o comportamento de endpoints anteriormente inseguros.

Quando isso acontecer, a Skill deve explicar a mudança, seus impactos e os testes necessários.

Não inventar credenciais, tokens, regras de autorização ou políticas de acesso.

Se uma decisão de segurança depender de requisitos desconhecidos, solicitar esclarecimento ao usuário.

---

## 8. Transações e integridade dos dados

Operações de negócio que envolvem múltiplas alterações relacionadas devem preservar a consistência dos dados.

Exemplos:

- Criar pedido e atualizar estoque.
- Registrar matrícula e atualizar disponibilidade.
- Registrar pagamento e alterar status.
- Excluir registros com relacionamentos dependentes.

A Skill deve verificar:

- Necessidade de transações.
- Comportamento em caso de erro.
- Possibilidade de operações parciais.
- Riscos de concorrência.
- Recursos disponíveis no banco utilizado.

Não aplicar soluções genéricas sem considerar a tecnologia e o mecanismo transacional existente.

---

## 9. Estratégia de refatoração incremental

A Skill deve evitar reescrever toda a aplicação de uma vez.

Sequência recomendada:

1. Revisar o relatório de auditoria aprovado.
2. Identificar os arquivos e responsabilidades afetados.
3. Definir e apresentar o plano arquitetural verificável.
4. Obter autorização para o plano.
5. Registrar o comportamento atual dos endpoints.
6. Criar ou ajustar componentes de maneira incremental.
7. Mover regras de negócio para componentes apropriados.
8. Isolar persistência conforme as camadas aprovadas.
9. Corrigir problemas de segurança e qualidade priorizados.
10. Executar testes funcionais, de segurança e arquiteturais após mudanças relevantes.
11. Documentar alterações, rastreabilidade dos findings, diferenças do plano e limitações.

Sempre que possível, preservar nomes públicos, caminhos de endpoints, formatos de resposta e contratos existentes.

Se uma alteração exigir mudança de contrato ou comportamento existente, documente a necessidade, o impacto e a justificativa, e solicite autorização explícita do usuário antes de executá-la. Mudanças de segurança também devem seguir essa regra. Não considere a aprovação genérica da Fase 3 como autorização para alterações de contrato que não estejam previstas no plano aprovado.

---

## 10. Validação obrigatória

Após a refatoração, a Skill deve verificar se a aplicação continua funcionando.

### 10.1. Inicialização

Verificar:

- Instalação e resolução das dependências.
- Ausência de erros de importação.
- Inicialização da aplicação.
- Configuração do banco de dados.
- Disponibilidade dos endpoints, quando possível.

### 10.2. Testes

Executar os testes existentes, quando disponíveis.

Quando não houver testes automatizados, realizar verificações compatíveis com o projeto, sem inventar resultados.

### 10.3. Endpoints

Verificar, conforme o escopo:

- Métodos e caminhos HTTP.
- Códigos de status.
- Formatos de resposta.
- Validações de entrada.
- Operações de leitura e escrita.
- Tratamento de erros.
- Autenticação e autorização, quando aplicáveis.

### 10.4. Registro dos resultados

Informar:

- Comandos executados.
- Testes aprovados.
- Testes que falharam.
- Endpoints verificados.
- Endpoints não verificados.
- Limitações do ambiente.
- Alterações que podem exigir revisão manual.

**Nunca afirmar que a aplicação iniciou ou que um endpoint funcionou sem realizar a verificação correspondente.**

---

## 11. Cuidados com os três projetos da atividade

A Skill será aplicada a projetos com características diferentes.

### Projeto Python / Flask com estrutura monolítica

A Skill deve identificar responsabilidades misturadas e propor separações incrementais.

Não assumir que todos os módulos precisam ser reescritos.

### Projeto Node.js / Express com classe centralizadora

A Skill deve avaliar a concentração de responsabilidades e distribuir funcionalidades entre componentes adequados.

Preservar as rotas e os contratos existentes sempre que possível.

### Projeto Python / Flask parcialmente organizado

A Skill deve reaproveitar as pastas e camadas já existentes.

Não recriar estruturas que já estejam adequadas apenas para padronizar os três projetos.

Essas orientações são exemplos relacionados à atividade. A Skill deve continuar funcionando em outros projetos e tecnologias.

---

## 12. Resultado esperado

Ao concluir a Fase 3, a Skill deverá:

- Organizar o código em uma arquitetura baseada em MVC.
- Separar responsabilidades inadequadamente misturadas.
- Preservar funcionalidades e contratos sempre que possível.
- Corrigir os problemas aprovados no plano de refatoração.
- Evitar abstrações e dependências desnecessárias.
- Verificar a inicialização da aplicação.
- Executar testes disponíveis.
- Validar endpoints relevantes.
- Registrar alterações, resultados e limitações.

A refatoração só estará considerada validada quando houver evidências das verificações realizadas.

Se a aplicação não puder ser inicializada ou os endpoints não puderem ser testados, a Skill deve informar explicitamente que a validação ficou incompleta.
