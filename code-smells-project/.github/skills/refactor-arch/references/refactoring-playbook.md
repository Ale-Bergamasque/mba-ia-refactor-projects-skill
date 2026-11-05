# Playbook de Refatoração Arquitetural

## 1. Objetivo

Este documento orienta a execução da Fase 3 — Refatoração da Skill `refactor-arch`.

Apresenta transformações práticas de código, com exemplos de antes e depois, para corrigir problemas arquiteturais, de segurança, desempenho e qualidade.

A Skill deve consultar este documento juntamente com `mvc-guidelines.md` e com o relatório de auditoria aprovado.

**Regras obrigatórias:**

1. Não executar nenhuma refatoração antes da autorização explícita do usuário.
2. Não aplicar exemplos mecanicamente.
3. Adaptar as transformações à linguagem, ao framework e às bibliotecas existentes.
4. Preservar funcionalidades e contratos HTTP sempre que possível.
5. Não inventar requisitos de negócio, credenciais ou regras de autorização.
6. Executar alterações incrementais.
7. Validar a aplicação e os endpoints após a refatoração.
8. Registrar mudanças, resultados e limitações.

Os exemplos deste documento são ilustrativos e não representam código real dos projetos analisados. Python e JavaScript foram escolhidos apenas para demonstrar transformações práticas. A Skill não deve priorizar essas linguagens, assumir que o projeto utiliza seus frameworks ou reproduzir os exemplos literalmente. Para qualquer outra tecnologia, deve aplicar o princípio arquitetural equivalente utilizando as convenções e ferramentas apropriadas.

---

## 2. Transformação 1 — Fat Controller para Controller + Service

**Problema:** a rota concentra tratamento HTTP e regras de negócio.

**Objetivo:** separar a coordenação HTTP das regras de negócio.

### Antes

```python
@app.route("/orders", methods=["POST"])
def create_order():
    data = request.get_json()
    total = sum(item["price"] * item["quantity"] for item in data["items"])

    if total <= 0:
        return {"error": "Pedido inválido"}, 400

    order = save_order(data, total)
    return jsonify(order), 201
```

### Depois

```python
# routes/order_routes.py
@app.route("/orders", methods=["POST"])
def create_order():
    return order_controller.create(request.get_json())


# controllers/order_controller.py
class OrderController:
    def __init__(self, order_service):
        self.order_service = order_service

    def create(self, data):
        order = self.order_service.create(data)
        return jsonify(order), 201


# services/order_service.py
class OrderService:
    def create(self, data):
        total = sum(
            item["price"] * item["quantity"]
            for item in data["items"]
        )

        if total <= 0:
            raise ValueError("Pedido inválido")

        return save_order(data, total)
```

**Cuidados:**

- Preservar os códigos HTTP e o formato das respostas.
- Garantir que erros de validação continuem retornando respostas adequadas.
- Centralizar o tratamento de exceções quando necessário.
- Não criar um Service para operações triviais sem justificativa.

---

## 3. Transformação 2 — SQL no Controller para Repository

**Problema:** o Controller acessa diretamente o banco de dados.

**Objetivo:** isolar a persistência.

### Antes

```python
@app.route("/products/<int:product_id>")
def get_product(product_id):
    cursor = connection.execute(
        "SELECT * FROM products WHERE id = ?",
        (product_id,)
    )
    product = cursor.fetchone()
    return jsonify(dict(product))
```

### Depois

```python
# repositories/product_repository.py
class ProductRepository:
    def __init__(self, connection):
        self.connection = connection

    def find_by_id(self, product_id):
        cursor = self.connection.execute(
            "SELECT * FROM products WHERE id = ?",
            (product_id,)
        )
        return cursor.fetchone()


# controllers/product_controller.py
class ProductController:
    def __init__(self, repository):
        self.repository = repository

    def get_product(self, product_id):
        product = self.repository.find_by_id(product_id)

        if product is None:
            return {"error": "Produto não encontrado"}, 404

        return jsonify(dict(product))
```

**Cuidados:**

- Respeitar a biblioteca de banco existente.
- Preservar a estrutura dos dados retornados.
- Não duplicar uma camada de persistência que já funciona adequadamente.

---

## 4. Transformação 3 — SQL Injection para consulta parametrizada

**Problema:** valores não confiáveis são concatenados em comandos SQL.

**Objetivo:** impedir que entradas alterem a estrutura da consulta.

### Antes

```python
def find_user(email):
    sql = "SELECT * FROM users WHERE email = '" + email + "'"
    return connection.execute(sql).fetchone()
```

### Depois

```python
def find_user(email):
    sql = "SELECT * FROM users WHERE email = ?"
    return connection.execute(sql, (email,)).fetchone()
```

**Cuidados:**

- Adaptar os marcadores de parâmetros ao driver utilizado.
- Parametrizar valores em todas as consultas afetadas.
- Validar separadamente identificadores SQL dinâmicos, como nomes de colunas, utilizando listas permitidas.
- Testar entradas inesperadas sem executar comandos destrutivos.

---

## 5. Transformação 4 — Execução arbitrária de SQL para operações específicas

**Problema:** um endpoint aceita comandos SQL enviados pelo cliente.

**Objetivo:** impedir a execução de instruções arbitrárias.

### Antes

```python
@app.route("/execute", methods=["POST"])
def execute_sql():
    sql = request.json["sql"]
    result = connection.execute(sql)
    return jsonify(result.fetchall())
```

### Depois

```python
@app.route("/products", methods=["GET"])
def list_products():
    products = product_service.list_products()
    return jsonify(products)
```

**Cuidados:**

- Remover a capacidade de executar SQL arbitrário.
- Expor apenas operações necessárias ao domínio.
- Aplicar autenticação e autorização quando necessário.
- Documentar a remoção do endpoint inseguro como mudança intencional de contrato.
- Não preservar comportamentos inseguros apenas por compatibilidade.

---

## 6. Transformação 5 — Credenciais hardcoded para configuração externa

**Problema:** credenciais estão diretamente no código.

**Objetivo:** retirar segredos do código-fonte.

### Antes

```python
SECRET_KEY = "chave-fixa"
SMTP_PASSWORD = "senha-fixa"
```

### Depois

```python
import os

SECRET_KEY = os.environ["SECRET_KEY"]
SMTP_PASSWORD = os.environ["SMTP_PASSWORD"]
```

**Cuidados:**

- Não inventar valores para as credenciais.
- Não registrar segredos em logs ou relatórios.
- Documentar quais variáveis de ambiente são necessárias.
- Não versionar arquivos contendo valores reais.
- Recomendar rotação de credenciais que tenham sido expostas.

---

## 7. Transformação 6 — Senhas inseguras para hash apropriado

**Problema:** senhas são armazenadas em texto puro, Base64 ou hash inadequado.

**Objetivo:** utilizar mecanismos apropriados para armazenamento de senhas.

### Antes

```python
import hashlib

def hash_password(password):
    return hashlib.md5(password.encode()).hexdigest()
```

### Depois

```python
from werkzeug.security import generate_password_hash
from werkzeug.security import check_password_hash

def hash_password(password):
    return generate_password_hash(password)

def verify_password(stored_hash, password):
    return check_password_hash(stored_hash, password)
```

**Cuidados:**

- Nunca retornar senhas ou hashes nas respostas HTTP.
- Não registrar senhas em logs.
- Verificar a compatibilidade com contas já existentes.
- Planejar migração segura dos hashes antigos.
- Não presumir que hashes antigos podem ser convertidos diretamente para o novo formato.
- Solicitar decisão do usuário se a migração exigir redefinição de senhas.

---

## 8. Transformação 7 — Endpoint desprotegido para autenticação e autorização

**Problema:** operações sensíveis podem ser acessadas sem verificar identidade e permissões.

**Objetivo:** proteger os recursos conforme as regras do sistema.

### Antes

```javascript
router.delete("/users/:id", async (req, res) => {
  await userRepository.remove(req.params.id);
  res.sendStatus(204);
});
```

### Depois

```javascript
router.delete(
  "/users/:id",
  authenticate,
  authorizeAdmin,
  userController.remove,
);
```

**Cuidados:**

- Os middlewares do exemplo são ilustrativos e precisam existir ou ser implementados.
- Verificar mecanismos de autenticação já disponíveis.
- Não inventar políticas de acesso.
- Diferenciar autenticação de autorização.
- Validar permissões sobre recursos pertencentes a outros usuários.
- Testar acessos autorizados e não autorizados.

---

## 9. Transformação 8 — Operações isoladas para transação

**Problema:** várias operações dependentes podem ser executadas parcialmente.

**Objetivo:** garantir atomicidade e consistência.

### Antes

```python
create_order(order)
update_stock(product_id, quantity)
register_payment(order.id)
```

### Depois

```python
with db.session.begin():
    create_order(order)
    update_stock(product_id, quantity)
    register_payment(order.id)
```

**Cuidados:**

- Adaptar o gerenciamento transacional ao ORM ou driver.
- Confirmar que todas as operações relevantes participam da mesma transação.
- Verificar o comportamento de rollback.
- Considerar concorrência em atualizações de estoque.
- Não presumir que operações em serviços externos podem participar da transação do banco.
- Para pagamentos externos, avaliar mecanismos apropriados de consistência e compensação.

---

## 10. Transformação 9 — N+1 Query para carregamento eficiente

**Problema:** a aplicação executa consultas adicionais para cada registro recuperado.

**Objetivo:** reduzir consultas repetitivas.

### Antes

```python
users = User.query.all()

for user in users:
    print(user.orders)
```

### Depois

```python
from sqlalchemy.orm import selectinload

users = (
    db.session.query(User)
    .options(selectinload(User.orders))
    .all()
)

for user in users:
    print(user.orders)
```

**Cuidados:**

- Verificar a versão do ORM.
- Avaliar JOIN, eager loading ou consultas em lote.
- Confirmar que a alteração reduz consultas.
- Não carregar dados desnecessários.
- Medir desempenho quando possível.

---

## 11. Transformação 10 — Tratamento genérico para erros específicos

**Problema:** exceções são ignoradas ou tratadas de maneira genérica.

**Objetivo:** melhorar diagnóstico e consistência das respostas.

### Antes

```python
try:
    save_user(data)
except:
    return {"error": "Erro"}, 200
```

### Depois

```python
try:
    save_user(data)
except DatabaseError:
    logger.exception("Falha ao salvar usuário")
    return {"error": "Erro interno"}, 500
```

**Cuidados:**

- Importar ou definir os tipos reais de exceção.
- Não retornar sucesso em caso de falha.
- Evitar expor detalhes internos.
- Centralizar o tratamento quando apropriado.
- Preservar respostas existentes que estejam corretas.

---

## 12. Transformação 11 — Lógica duplicada para função reutilizável

**Problema:** a mesma regra aparece em vários lugares.

**Objetivo:** centralizar o comportamento compartilhado.

### Antes

```javascript
function calculateCartTotal(items) {
  return items.reduce((sum, item) => {
    return sum + item.price * item.quantity;
  }, 0);
}

function calculateOrderTotal(items) {
  return items.reduce((sum, item) => {
    return sum + item.price * item.quantity;
  }, 0);
}
```

### Depois

```javascript
function calculateTotal(items) {
  return items.reduce((sum, item) => {
    return sum + item.price * item.quantity;
  }, 0);
}
```

**Cuidados:**

- Confirmar que as regras são realmente equivalentes.
- Não unificar comportamentos que possuem diferenças de negócio.
- Atualizar os pontos de chamada.
- Executar testes de regressão.

---

## 13. Transformação 12 — API deprecated para alternativa suportada

**Problema:** a aplicação utiliza uma API oficialmente marcada como deprecated.

**Objetivo:** substituir a API por uma alternativa compatível.

### Antes

Exemplo para SQLAlchemy 2.x:

```python
user = User.query.get(user_id)
```

### Depois

```python
user = db.session.get(User, user_id)
```

**Cuidados:**

- Confirmar as versões de Flask-SQLAlchemy e SQLAlchemy.
- Consultar a documentação oficial da versão utilizada.
- Verificar diferenças de comportamento.
- Atualizar os testes relacionados.
- Não substituir APIs apenas porque parecem antigas.
- Não modificar dependências sem avaliar compatibilidade.

---

## 14. Como escolher as transformações

A Skill não deve aplicar todas as transformações indiscriminadamente.

Para cada finding aprovado:

1. Identificar o problema e sua localização.
2. Consultar a transformação correspondente.
3. Verificar se o exemplo é compatível com a tecnologia.
4. Identificar dependências entre alterações.
5. Planejar uma mudança pequena e verificável.
6. Implementar a alteração.
7. Executar verificações relevantes.
8. Registrar o resultado.

Se um problema não possuir exemplo específico neste documento, a Skill poderá propor uma solução fundamentada nas diretrizes MVC e nas convenções da tecnologia.

Não inventar soluções sem analisar o código real.

---

## 15. Ordem sugerida de execução

A ordem deve ser definida conforme os findings e o contexto do projeto.

Como referência:

1. Registrar o comportamento atual e preparar verificações.
2. Corrigir vulnerabilidades críticas aprovadas.
3. Organizar configurações e credenciais.
4. Separar responsabilidades arquiteturais.
5. Isolar regras de negócio e persistência quando necessário.
6. Corrigir problemas de transações e concorrência.
7. Melhorar validações e tratamento de erros.
8. Corrigir problemas de desempenho.
9. Atualizar APIs deprecated confirmadas.
10. Melhorar legibilidade e remover duplicações.
11. Executar validação final.

Se uma alteração crítica exigir uma decisão de negócio desconhecida, solicitar esclarecimento antes de executá-la.

---

## 16. Validação obrigatória após a refatoração

A Skill deve verificar o funcionamento da aplicação após as mudanças.

### Inicialização

- Verificar dependências.
- Executar o comando de inicialização apropriado.
- Identificar erros de importação ou configuração.
- Confirmar se a aplicação inicia corretamente.

### Testes

- Executar os testes automatizados existentes.
- Registrar testes aprovados e falhas.
- Não inventar resultados.
- Informar quando testes não estiverem disponíveis.

### Endpoints

Verificar os endpoints relevantes, considerando:

- Método HTTP.
- Caminho da rota.
- Código de status.
- Estrutura da resposta.
- Validação de entrada.
- Tratamento de erros.
- Autenticação e autorização.
- Persistência dos dados, quando aplicável.

### Registro

Ao final, apresentar:

- Arquivos criados.
- Arquivos modificados.
- Findings corrigidos.
- Alterações de comportamento.
- Comandos executados.
- Resultados dos testes.
- Endpoints validados.
- Problemas pendentes.
- Limitações da validação.

**Nunca declarar sucesso sem evidências das verificações realizadas.**

---

## 17. Resultado esperado

Após executar as transformações aprovadas, a Skill deverá entregar:

1. Código organizado em uma arquitetura baseada em MVC.
2. Separação adequada de responsabilidades.
3. Correções dos findings aprovados.
4. Preservação das funcionalidades sempre que possível.
5. Registro das mudanças de contrato inevitáveis.
6. Aplicação inicializando corretamente, quando o ambiente permitir.
7. Endpoints relevantes verificados.
8. Relatório final de alterações e validações.

A Skill não deve considerar a refatoração totalmente validada se não conseguir executar as verificações necessárias. Nesse caso, deve informar claramente o que ficou pendente.
