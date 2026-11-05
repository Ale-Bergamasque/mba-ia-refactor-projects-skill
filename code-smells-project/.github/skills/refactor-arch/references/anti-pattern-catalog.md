# Catálogo de Anti-patterns e Code Smells

## 1. Objetivo

Este catálogo define os problemas arquiteturais, de segurança, desempenho e qualidade que devem ser investigados durante a Fase 2 — Auditoria.

A Skill deve utilizar este catálogo como referência, adaptando a detecção à linguagem, ao framework, às bibliotecas e à arquitetura identificados no projeto.

Cada anti-pattern possui:

- Nome e descrição.
- Severidade padrão.
- Sinais de detecção.
- Impacto no projeto.
- Recomendação de correção.
- Exemplo ilustrativo, quando aplicável.

A classificação deve considerar o contexto e o impacto real do problema, não apenas a presença de determinado trecho de código.

**Importante:** a existência de um sinal de detecção não comprova automaticamente uma vulnerabilidade. A Skill deve examinar o contexto e apresentar evidências antes de confirmar um finding.

---

## 2. CRITICAL — Problemas críticos

Problemas que representam riscos graves de segurança, exposição de dados ou comprometimento do funcionamento da aplicação.

### 2.1. Hardcoded Credentials (Credenciais no código)

**Severidade padrão:** CRITICAL

**Descrição:**

Ocorre quando credenciais, senhas, tokens, chaves de API ou outros segredos são armazenados diretamente no código-fonte.

**Sinais de detecção:**

- Senhas declaradas diretamente em arquivos de configuração ou aplicação.
- Tokens de autenticação escritos como valores fixos.
- Chaves secretas armazenadas em arquivos versionados.
- Strings com aparência de credenciais utilizadas em conexões ou autenticação.

**Impacto:**

- Exposição de informações confidenciais.
- Possibilidade de acesso não autorizado.
- Comprometimento de bancos de dados e serviços externos.

**Recomendação:**

- Utilizar variáveis de ambiente.
- Implementar gerenciamento seguro de segredos.
- Remover credenciais do código versionado.
- Se houver exposição real, recomendar a rotação das credenciais afetadas.

**Exemplo:**

Antes:

```python
SECRET_KEY = "minha-chave-secreta"
```

Depois:

```python
import os

SECRET_KEY = os.environ.get("SECRET_KEY")
```

**Observação:** a presença de uma variável chamada `password` não comprova que existe uma credencial exposta. Verifique seu conteúdo e contexto sem reproduzir o segredo no relatório.

Não confundir credenciais hardcoded com outras configurações inseguras, como debug habilitado ou exposição de informações internas. Esses casos são tratados separadamente.

### 2.2. SQL Injection

**Severidade padrão:** CRITICAL

**Descrição:**

Ocorre quando dados fornecidos pelo usuário são incorporados diretamente em comandos SQL sem parametrização adequada.

**Sinais de detecção:**

- Concatenação de strings para montar comandos SQL.
- Interpolação de valores recebidos em parâmetros de requisições.
- Construção dinâmica de consultas sem proteção adequada.
- Ausência de consultas parametrizadas em operações sensíveis.

**Impacto:**

- Acesso não autorizado a dados.
- Alteração ou exclusão de registros.
- Possibilidade de comprometimento do banco de dados.

**Recomendação:**

Utilizar consultas parametrizadas, prepared statements ou mecanismos seguros disponibilizados pelo ORM utilizado.

**Exemplo:**

Antes:

```python
query = "SELECT * FROM users WHERE id = " + user_id
cursor.execute(query)
```

Depois:

```python
query = "SELECT * FROM users WHERE id = ?"
cursor.execute(query, (user_id,))
```

**Observação:** o marcador de parâmetros varia conforme o driver e o banco de dados. A correção deve respeitar a tecnologia utilizada.

### 2.3. God Class / God Method

**Severidade padrão:** CRITICAL

**Descrição:**

Ocorre quando uma classe ou método concentra responsabilidades excessivas, acumulando funcionalidades que deveriam estar separadas.

**Sinais de detecção:**

- Uma classe realiza acesso a dados, validações, regras de negócio e roteamento.
- Um método executa diversas operações independentes.
- Grande quantidade de responsabilidades sem separação lógica.
- Forte dependência entre funcionalidades de domínios diferentes.

**Impacto:**

- Alto acoplamento.
- Dificuldade de manutenção e testes.
- Alterações em uma funcionalidade podem afetar outras.
- Violação do princípio da responsabilidade única (SRP).

**Recomendação:**

Dividir as responsabilidades em módulos e componentes menores, utilizando Models, Controllers, Services e outras abstrações adequadas à tecnologia.

**Exemplo:**

Antes:

```python
class Application:
    def create_user(self):
        # Validação
        # SQL
        # Regra de negócio
        # Formatação da resposta
        pass
```

Depois:

```python
class UserController:
    def create(self, data):
        return user_service.create(data)


class UserService:
    def create(self, data):
        user = UserRepository.save(data)
        return user
```

**Observação:** o tamanho do arquivo ou da classe, isoladamente, não comprova esse problema. É necessário analisar a concentração de responsabilidades.

### 2.4. Execução arbitrária de SQL

**Severidade padrão:** CRITICAL

**Descrição:**

Ocorre quando a aplicação permite que o cliente envie instruções SQL completas ou comandos que são executados diretamente no banco de dados, sem restringir as operações às funcionalidades e regras de negócio da aplicação.

**Sinais de detecção:**

- Endpoints que recebem instruções SQL completas.
- Execução de comandos recebidos diretamente da requisição.
- Possibilidade de o cliente escolher livremente quais operações SQL serão executadas.
- Ausência de restrições sobre as operações permitidas.

**Impacto:**

- Possibilidade de consultar, alterar ou excluir dados sem passar pelas regras de negócio.
- Comprometimento da integridade e confidencialidade dos dados.
- Possibilidade de acesso a informações que deveriam ser restritas.

**Recomendação:**

Remover a execução arbitrária de SQL. Criar operações específicas para cada funcionalidade, utilizando consultas parametrizadas e controles de autenticação e autorização.

**Exemplo:**

Antes:

```python
@app.route("/execute", methods=["POST"])
def execute():
    sql = request.json["sql"]
    cursor.execute(sql)
```

Depois:

```python
@app.route("/products", methods=["GET"])
def list_products():
    products = product_service.list_products()
    return jsonify(products)
```

**Observação:** execução arbitrária de SQL e SQL Injection são problemas relacionados, mas distintos.

- SQL Injection ocorre quando entradas manipuladas alteram a estrutura de uma consulta.
- Execução arbitrária ocorre quando a aplicação permite que o cliente determine o comando SQL executado.

Não contabilizar o mesmo problema duas vezes sem justificar a existência de riscos distintos.

### 2.5. Falhas de autenticação e autorização

**Severidade padrão:** CRITICAL

**Descrição:**

Ocorre quando a aplicação não verifica adequadamente a identidade do usuário ou suas permissões antes de permitir o acesso a recursos e operações protegidas.

**Sinais de detecção:**

- Rotas administrativas sem autenticação.
- Endpoints de alteração ou exclusão sem verificar o usuário autenticado.
- Ausência de verificação de propriedade dos recursos.
- Tokens fictícios, previsíveis ou sem validação adequada.
- Identificação de usuário apenas por um dado informado na requisição, sem validar sua identidade.
- Ausência de controle de permissões em operações sensíveis.

**Impacto:**

- Acesso não autorizado a informações.
- Execução de operações administrativas por usuários comuns.
- Alteração ou exclusão de dados de outros usuários.
- Comprometimento de informações protegidas.

**Recomendação:**

Implementar autenticação adequada, proteger endpoints com mecanismos de autorização e verificar permissões e propriedade dos recursos antes de executar operações.

**Exemplo:**

Antes:

```javascript
app.delete("/users/:id", async (req, res) => {
  await userRepository.delete(req.params.id);
  res.sendStatus(204);
});
```

Depois:

```javascript
app.delete("/users/:id", authenticate, authorizeAdmin, userController.remove);
```

**Observação:** a ausência de um middleware em uma rota não comprova, isoladamente, que não existe autenticação. Verifique também os mecanismos globais, configurações e camadas responsáveis pelo controle de acesso.

### 2.6. Armazenamento e exposição insegura de senhas

**Severidade padrão:** CRITICAL

**Descrição:**

Ocorre quando a aplicação armazena senhas utilizando texto puro ou mecanismos de codificação e hash inadequados, ou expõe senhas e seus hashes em respostas e registros.

**Sinais de detecção:**

- Senhas armazenadas em texto puro.
- Uso de Base64 como mecanismo de proteção de senhas.
- Uso de MD5 ou outros algoritmos inadequados para armazenamento de senhas.
- Retorno de senhas ou hashes em respostas HTTP.
- Inclusão de senhas em logs.

**Impacto:**

- Exposição das credenciais dos usuários.
- Possibilidade de reutilização indevida de senhas.
- Comprometimento de contas em caso de vazamento de dados.

**Recomendação:**

Utilizar algoritmos próprios para hash de senhas, como Argon2id, bcrypt ou PBKDF2, com parâmetros adequados. Nunca retornar senhas ou hashes nas respostas HTTP.

**Exemplo:**

Antes:

```python
def set_password(self, password):
    self.password = hashlib.md5(
        password.encode()
    ).hexdigest()

def to_dict(self):
    return {
        "id": self.id,
        "password": self.password
    }
```

Depois:

```python
from werkzeug.security import generate_password_hash

def set_password(self, password):
    self.password_hash = generate_password_hash(password)

def to_dict(self):
    return {
        "id": self.id
    }
```

**Observação:** Base64 é codificação, não criptografia nem hash seguro. A análise deve diferenciar essas técnicas.

---

## 3. HIGH — Problemas arquiteturais e de segurança importantes

Problemas que podem comprometer significativamente a manutenção, a segurança, a integridade dos dados ou a evolução do sistema.

### 3.1. Fat Controller

**Severidade padrão:** HIGH

**Descrição:**

Ocorre quando Controllers concentram regras de negócio, consultas ao banco de dados e operações que deveriam estar em outras camadas.

**Sinais de detecção:**

- Controllers executam comandos SQL diretamente.
- Regras de negócio complexas estão dentro de métodos de rotas.
- Validações e cálculos de negócio estão misturados ao tratamento HTTP.
- Métodos de Controller acumulam muitas responsabilidades.

**Impacto:**

- Dificuldade para testar regras de negócio isoladamente.
- Reutilização limitada.
- Forte acoplamento entre regras de negócio e framework web.

**Recomendação:**

Transferir regras de negócio para Services ou componentes de domínio e operações de persistência para Models ou Repositories, conforme a arquitetura adotada.

**Exemplo:**

Antes:

```python
@app.route("/orders", methods=["POST"])
def create_order():
    # Validar pedido
    # Calcular total
    # Executar SQL
    # Atualizar estoque
    # Retornar resposta
    pass
```

Depois:

```python
@app.route("/orders", methods=["POST"])
def create_order():
    result = order_controller.create(request.json)
    return result
```

### 3.2. Tight Coupling / Forte Acoplamento

**Severidade padrão:** HIGH

**Descrição:**

Ocorre quando módulos dependem diretamente de implementações específicas de outros módulos, dificultando substituições, manutenção e testes.

**Sinais de detecção:**

- Controllers instanciam diretamente diversas dependências concretas.
- Regras de negócio dependem diretamente de detalhes de infraestrutura.
- Alterar um módulo exige alterações em vários outros.
- Ausência de abstrações quando existe necessidade real de substituição ou isolamento.

**Impacto:**

- Dificuldade de manutenção.
- Testes isolados mais complexos.
- Baixa flexibilidade arquitetural.
- Violação do princípio da inversão de dependência (DIP), quando aplicável.

**Recomendação:**

Introduzir interfaces, contratos ou abstrações e utilizar injeção de dependência quando isso trouxer benefícios reais.

**Exemplo:**

Antes:

```python
class OrderService:
    def __init__(self):
        self.database = MySQLDatabase()
```

Depois:

```python
class OrderService:
    def __init__(self, repository):
        self.repository = repository
```

### 3.3. Global Mutable State

**Severidade padrão:** HIGH

**Descrição:**

Ocorre quando diferentes partes da aplicação compartilham e modificam estados globais mutáveis sem controle adequado.

**Sinais de detecção:**

- Variáveis globais utilizadas para armazenar dados de usuários.
- Objetos compartilhados que são modificados por diferentes módulos.
- Estado de requisições armazenado em variáveis globais.
- Dados compartilhados sem mecanismos adequados de isolamento ou sincronização.

**Impacto:**

- Comportamentos imprevisíveis.
- Possibilidade de vazamento de dados entre requisições.
- Problemas de concorrência.
- Dificuldade de reproduzir e testar erros.

**Recomendação:**

Utilizar mecanismos apropriados de gerenciamento de estado, escopos de requisição, injeção de dependência e armazenamento adequado para dados compartilhados.

**Exemplo:**

Antes:

```python
current_user = None

def set_user(user):
    global current_user
    current_user = user
```

Depois:

```python
def process_request(user):
    return user_service.execute(user)
```

**Observação:** estado global imutável e constantes não devem ser classificados automaticamente como problemas.

### 3.4. Configuração insegura e exposição de informações

**Severidade padrão:** HIGH

**Descrição:**

Ocorre quando a aplicação utiliza configurações inadequadas para o ambiente de execução ou expõe informações internas que deveriam permanecer restritas.

**Sinais de detecção:**

- Modo debug habilitado em configuração destinada à produção.
- Servidor de desenvolvimento configurado para uso em produção.
- Endpoints que expõem caminhos internos, configurações ou informações sensíveis.
- Dados de cartões, senhas, tokens ou informações pessoais registrados em logs.
- Mensagens de erro que expõem detalhes internos da aplicação.

**Impacto:**

- Exposição de informações internas.
- Facilitação de ataques.
- Vazamento de dados sensíveis.
- Maior dificuldade para garantir a segurança do ambiente de produção.

**Recomendação:**

- Separar configurações por ambiente.
- Desabilitar debug em produção.
- Utilizar servidores apropriados para produção.
- Restringir informações expostas por endpoints.
- Remover dados sensíveis dos logs.

**Exemplo:**

Antes:

```python
app.run(debug=True, host="0.0.0.0")
```

Depois:

```python
import os

DEBUG = os.environ.get("APP_DEBUG", "false").lower() == "true"

if __name__ == "__main__":
    app.run(debug=DEBUG)
```

**Observação:** não classificar automaticamente qualquer ocorrência de `debug=True` como vulnerabilidade em produção. Verificar o contexto e a configuração de execução. A existência de credenciais hardcoded deve ser registrada no padrão específico de credenciais.

### 3.5. Ausência de controle transacional e concorrência

**Severidade padrão:** HIGH

**Descrição:**

Ocorre quando operações relacionadas são executadas sem garantir atomicidade ou proteção adequada contra acessos simultâneos.

**Sinais de detecção:**

- Fluxos de negócio que realizam múltiplas gravações sem transação.
- Ausência de rollback em caso de falha intermediária.
- Verificação e atualização de estoque sem proteção contra concorrência.
- Operações que podem produzir registros parcialmente atualizados.
- Processos financeiros ou de checkout com etapas dependentes sem controle de consistência.

**Impacto:**

- Dados inconsistentes.
- Pedidos parcialmente registrados.
- Estoque incorreto.
- Pagamentos e registros divergentes.
- Problemas em operações simultâneas.

**Recomendação:**

Utilizar transações para operações relacionadas, implementar rollback em caso de falha e adotar mecanismos de controle de concorrência adequados ao banco de dados.

**Exemplo:**

Antes:

```python
create_order(order)
update_stock(product_id, quantity)
register_payment(order.id)
```

Depois:

```python
try:
    with db.session.begin():
        create_order(order)
        update_stock(product_id, quantity)
        register_payment(order.id)
except Exception:
    db.session.rollback()
    raise
```

**Observação:** a ausência de uma transação explícita no trecho analisado não comprova, por si só, que a operação não possui controle transacional. Verificar se a transação é gerenciada por outra camada ou pelo framework.

---

## 4. MEDIUM — Problemas de qualidade, desempenho e manutenção

Problemas que prejudicam a qualidade, a eficiência, a confiabilidade e a evolução do projeto.

### 4.1. Duplicated Business Logic

**Severidade padrão:** MEDIUM

**Descrição:**

Ocorre quando a mesma regra de negócio é implementada repetidamente em diferentes partes da aplicação.

**Sinais de detecção:**

- Métodos diferentes executam os mesmos cálculos.
- Regras de validação repetidas em Controllers distintos.
- Trechos semelhantes de código com pequenas variações.
- Alterações de uma regra exigem modificações em vários arquivos.

**Impacto:**

- Inconsistência de comportamento.
- Maior custo de manutenção.
- Aumento da possibilidade de erros.

**Recomendação:**

Centralizar regras compartilhadas em Services, funções ou componentes de domínio reutilizáveis.

**Exemplo:**

Antes:

```python
def calculate_order_a(price, tax):
    return price + (price * tax)

def calculate_order_b(price, tax):
    return price + (price * tax)
```

Depois:

```python
def calculate_total(price, tax):
    return price + (price * tax)
```

### 4.2. N+1 Query Problem

**Severidade padrão:** MEDIUM

**Descrição:**

Ocorre quando uma consulta inicial ao banco de dados é seguida por várias consultas adicionais individuais para recuperar dados relacionados.

**Sinais de detecção:**

- Consultas SQL executadas dentro de loops.
- Acesso repetido a relacionamentos de entidades.
- Consultas individuais para cada registro de uma coleção.
- Ausência de estratégias adequadas de carregamento de relacionamentos.

**Impacto:**

- Aumento desnecessário de consultas.
- Lentidão em operações com muitos registros.
- Maior consumo de recursos do banco de dados.

**Recomendação:**

Utilizar carregamento em lote, JOINs ou mecanismos de eager loading disponibilizados pelo ORM, quando apropriado.

**Exemplo:**

Antes:

```python
users = get_users()

for user in users:
    orders = get_orders_by_user(user.id)
```

Depois:

```python
users = get_users_with_orders()
```

**Observação:** a segunda implementação representa uma estratégia possível. A consulta real deve ser adaptada ao banco e à biblioteca utilizados.

### 4.3. Missing Input Validation

**Severidade padrão:** MEDIUM

**Descrição:**

Ocorre quando dados recebidos pela aplicação não são adequadamente validados antes de serem processados.

**Sinais de detecção:**

- Rotas recebem dados sem verificar campos obrigatórios.
- Ausência de validação de tipos e formatos.
- Falta de limites para valores numéricos.
- Dados inválidos chegam às regras de negócio ou persistência.

**Impacto:**

- Erros durante a execução.
- Dados inconsistentes.
- Possibilidade de comportamentos inesperados.
- Aumento da superfície de ataque quando existem entradas não confiáveis.

**Recomendação:**

Implementar validações de entrada utilizando mecanismos apropriados à tecnologia, mantendo validações de negócio nas camadas responsáveis.

**Exemplo:**

Antes:

```python
@app.route("/users", methods=["POST"])
def create_user():
    data = request.json
    save_user(data)
```

Depois:

```python
@app.route("/users", methods=["POST"])
def create_user():
    data = request.get_json()

    if not data or not data.get("email"):
        return {"error": "Email obrigatório"}, 400

    save_user(data)
```

**Observação:** se a ausência de validação permitir exploração de segurança, a severidade poderá ser elevada para HIGH ou CRITICAL conforme o impacto demonstrado.

### 4.4. Tratamento inadequado de erros e exceções

**Severidade padrão:** MEDIUM

**Descrição:**

Ocorre quando a aplicação captura, ignora ou trata erros de maneira genérica, podendo ocultar falhas e retornar respostas incorretas aos clientes.

**Sinais de detecção:**

- Blocos `except:` sem especificação do tipo de exceção.
- Blocos `catch` que ignoram erros.
- Retorno de sucesso mesmo quando uma operação falha.
- Mensagens genéricas que dificultam o diagnóstico.
- Ausência de tratamento centralizado de erros HTTP.
- Exposição de detalhes internos em respostas de erro.

**Impacto:**

- Falhas podem passar despercebidas.
- Diagnóstico e manutenção ficam mais difíceis.
- Respostas inconsistentes para os clientes.
- Informações internas podem ser expostas.

**Recomendação:**

Capturar exceções específicas, registrar erros sem expor informações sensíveis e centralizar o tratamento de erros com respostas HTTP adequadas.

**Exemplo:**

Antes:

```python
try:
    user = repository.find(user_id)
except:
    return {"message": "Erro"}
```

Depois:

```python
try:
    user = repository.find(user_id)
except DatabaseError:
    logger.exception("Erro ao consultar usuário")
    raise InternalServerError()
```

**Observação:** a exceção utilizada no exemplo deve ser adaptada à biblioteca real. Não substituir indiscriminadamente todos os tratamentos de erro por uma única exceção genérica.

---

## 5. LOW — Problemas de legibilidade e padronização

Problemas que afetam principalmente a compreensão, a organização e a padronização do código.

### 5.1. Magic Numbers / Magic Strings

**Severidade padrão:** LOW

**Descrição:**

Ocorre quando valores constantes aparecem diretamente no código sem identificação clara de seu significado.

**Sinais de detecção:**

- Números sem explicação em cálculos ou condições.
- Strings repetidas representando estados ou configurações.
- Valores fixos espalhados por diferentes módulos.

**Impacto:**

- Dificuldade de compreensão.
- Maior risco de inconsistências ao alterar valores.
- Manutenção mais difícil.

**Recomendação:**

Extrair valores para constantes nomeadas ou configurações apropriadas.

**Exemplo:**

Antes:

```python
if user.age >= 18:
    allow_access()
```

Depois:

```python
MINIMUM_AGE = 18

if user.age >= MINIMUM_AGE:
    allow_access()
```

### 5.2. Poor Naming Conventions

**Severidade padrão:** LOW

**Descrição:**

Ocorre quando variáveis, métodos, funções ou classes possuem nomes pouco claros ou inconsistentes com as convenções da linguagem.

**Sinais de detecção:**

- Nomes genéricos como `x`, `data1` ou `temp` em contextos importantes.
- Abreviações sem significado evidente.
- Métodos com nomes que não representam sua responsabilidade.
- Inconsistência na nomenclatura dos módulos.

**Impacto:**

- Dificuldade de leitura.
- Maior tempo de compreensão do código.
- Dificuldade de manutenção e colaboração.

**Recomendação:**

Utilizar nomes descritivos e convenções reconhecidas pela linguagem e pelo projeto.

**Exemplo:**

Antes:

```python
def calc(p, t):
    return p + (p * t)
```

Depois:

```python
def calculate_total(price, tax):
    return price + (price * tax)
```

### 5.3. Long Methods / Excessive Method Complexity

**Severidade padrão:** LOW

**Descrição:**

Ocorre quando métodos ou funções acumulam muitas instruções, condições ou etapas, dificultando a compreensão de sua finalidade.

**Sinais de detecção:**

- Funções extensas com várias etapas independentes.
- Muitos níveis de condicionais aninhadas.
- Mistura de validações, cálculos e formatação em um único método.
- Dificuldade de identificar a responsabilidade principal da função.

**Impacto:**

- Leitura e manutenção difíceis.
- Maior dificuldade de testes.
- Aumento da possibilidade de erros.

**Recomendação:**

Dividir métodos em funções menores, cada uma com responsabilidade clara, preservando a legibilidade e evitando fragmentação excessiva.

**Exemplo:**

Antes:

```python
def process_order(order):
    # Validar dados
    # Calcular valores
    # Verificar estoque
    # Salvar pedido
    # Enviar notificação
    pass
```

Depois:

```python
def process_order(order):
    validate_order(order)
    calculate_order_total(order)
    check_stock(order)
    save_order(order)
    notify_customer(order)
```

### 5.4. Dependências desnecessárias ou obsoletas

**Severidade padrão:** LOW

**Descrição:**

Ocorre quando o projeto declara dependências aparentemente não utilizadas ou mantém pacotes em versões obsoletas, sem suporte ou incompatíveis com a evolução da aplicação.

**Sinais de detecção:**

- Dependências declaradas sem uso identificado no código.
- Pacotes com versões antigas.
- Dependências incompatíveis com a versão atual do framework.
- Dependências transitivas obsoletas identificadas nos arquivos de lock.
- Pacotes sem manutenção ou suporte identificável.

**Impacto:**

- Aumento da complexidade de manutenção.
- Dificuldade para atualizar o projeto.
- Possibilidade de incompatibilidades.
- Riscos de segurança quando existem vulnerabilidades conhecidas.

**Recomendação:**

- Verificar a utilização real das dependências.
- Consultar documentação e informações oficiais do pacote.
- Avaliar atualizações e compatibilidade.
- Remover dependências somente após confirmar que não são utilizadas.

**Observação:**

- Diferenciar dependências diretas de transitivas.
- Não afirmar que uma dependência é vulnerável sem evidência.
- Não considerar uma dependência inutilizada apenas porque não aparece nos arquivos inicialmente analisados.
- Não atualizar ou remover pacotes automaticamente durante a Fase 2.

---

## 6. APIs Deprecated — Detecção obrigatória

A Skill deve verificar o uso de APIs, métodos, bibliotecas ou funcionalidades oficialmente depreciadas ou obsoletas.

**Severidade padrão:** MEDIUM

**Descrição:**

Ocorre quando o projeto utiliza recursos que foram marcados oficialmente como deprecated ou que possuem substitutos recomendados nas versões aplicáveis da tecnologia.

**Sinais de detecção:**

- Métodos identificados como deprecated na documentação oficial.
- Avisos de depreciação durante execução ou testes.
- Uso de APIs antigas com substitutos oficialmente recomendados.
- Dependências que utilizam APIs obsoletas.
- Funcionalidades removidas ou sem suporte na versão utilizada.

**Impacto:**

- Risco de incompatibilidade em atualizações.
- Possibilidade de falhas em versões futuras.
- Dificuldade de manutenção.
- Riscos de segurança quando o recurso não recebe mais atualizações.

**Recomendação:**

1. Identificar a versão da tecnologia utilizada.
2. Confirmar a depreciação na documentação oficial.
3. Identificar o substituto recomendado.
4. Avaliar impactos e compatibilidade.
5. Recomendar ou aplicar a migração durante a Fase 3, após autorização.

**Exemplo ilustrativo:**

Antes:

```javascript
// Exemplo hipotético de API antiga
legacyMethod();
```

Depois:

```javascript
// Exemplo hipotético de API substituta
modernMethod();
```

**Importante:**

Os nomes acima são ilustrativos. A Skill deve identificar APIs reais conforme a linguagem, framework e versão encontrados.

Não deve classificar uma API como deprecated apenas por ser antiga ou pouco utilizada.

Diferenciar:

- API deprecated: recurso oficialmente marcado como obsoleto.
- API removida: recurso que já não está disponível na versão analisada.
- Dependência deprecated: pacote ou biblioteca com status de depreciação.
- Dependência vulnerável: pacote com vulnerabilidade de segurança identificada.
- Código legado: código antigo que não necessariamente está deprecated.

Se não houver documentação disponível para confirmar a depreciação, registrar a ocorrência como suspeita, sem apresentá-la como finding confirmado.

A severidade poderá ser elevada quando a API representar risco concreto de segurança, incompatibilidade ou perda de suporte.

---

## 7. Regras gerais de classificação

A severidade padrão de cada item serve como referência inicial.

A classificação final deve considerar:

- Gravidade do impacto.
- Possibilidade de exploração.
- Abrangência do problema.
- Quantidade de funcionalidades afetadas.
- Risco para segurança e integridade dos dados.
- Consequências para manutenção e evolução.
- Ambiente em que a aplicação será executada.

### Regras adicionais

1. Um mesmo problema não deve ser contabilizado várias vezes sem justificativa.
2. Problemas relacionados podem ser apresentados separadamente quando representam causas ou impactos distintos.
3. Um finding deve possuir evidências concretas no código ou em fontes confiáveis.
4. A ausência de uma camada ou diretório não comprova, isoladamente, um anti-pattern.
5. Problemas não confirmados devem ser identificados como suspeitas ou itens pendentes de validação.
6. A severidade pode ser ajustada quando o contexto justificar, explicando o motivo.
7. Os findings devem ser apresentados na ordem: CRITICAL, HIGH, MEDIUM e LOW.
8. Não elevar a severidade apenas porque um problema parece importante. Justificar a classificação pelo impacto demonstrado.
9. Não considerar um risco hipotético como vulnerabilidade confirmada.
10. Não inventar problemas para atingir quantidades mínimas.
11. Diferenciar problemas encontrados diretamente no código daqueles identificados por testes ou documentação.
12. Quando a análise não permitir confirmar um problema, informar a limitação.

### Achados principais e complementares

Quando houver muitos problemas identificados, a Skill poderá separar os findings em:

- **Achados principais:** problemas de maior impacto ou relevância para a auditoria.
- **Achados complementares:** problemas adicionais que merecem registro, mas não possuem a mesma prioridade.

Essa separação não altera a severidade técnica. Um achado complementar pode ser CRITICAL, HIGH, MEDIUM ou LOW.

A Skill não deve omitir problemas relevantes apenas para manter o relatório curto.

---

## 8. Evidências obrigatórias

Para cada finding confirmado, apresentar:

- Nome do anti-pattern.
- Severidade.
- Arquivo afetado.
- Linha ou intervalo exato.
- Trecho ou descrição objetiva da evidência.
- Descrição do problema.
- Impacto.
- Recomendação.

Quando um problema envolver vários arquivos, identificar os principais pontos relacionados.

A Skill deve examinar o código para localizar as linhas exatas, evitando utilizar estimativas ou referências genéricas.

Para APIs deprecated e dependências obsoletas, apresentar também:

- Nome da API ou dependência.
- Versão identificada, quando disponível.
- Evidência da depreciação.
- Fonte oficial consultada, quando disponível.
- Alternativa recomendada.

Para riscos que dependem de execução, configuração ou ambiente, indicar o que ainda precisa ser verificado.

**Nunca inventar evidências, versões, linhas, documentação ou resultados de testes para preencher o relatório.**

Não reproduzir credenciais, tokens, senhas ou outros segredos no relatório. Descrever a exposição sem revelar o valor encontrado.

---

## 9. Agnosticismo tecnológico

Os anti-patterns deste catálogo representam problemas de arquitetura e qualidade que podem ocorrer em diferentes linguagens e frameworks.

A Skill deve adaptar os sinais de detecção às convenções da tecnologia identificada.

Exemplos:

- SQL Injection pode ocorrer em Python, JavaScript, Java, PHP e outras linguagens.
- God Class pode ocorrer em qualquer linguagem orientada a objetos.
- Fat Controller pode ocorrer em diferentes frameworks MVC.
- N+1 pode ocorrer com SQL, ORMs e outros mecanismos de persistência.
- APIs deprecated podem existir em qualquer ecossistema tecnológico.
- Falhas de autenticação podem ocorrer em APIs REST, aplicações web e outros sistemas.
- Problemas de concorrência podem ocorrer em diferentes bancos de dados e arquiteturas.

Os exemplos de código são ilustrativos. A detecção deve considerar a implementação real, não depender da sintaxe de uma linguagem específica.

A Skill não deve impor uma estrutura de diretórios ou uma tecnologia específica quando o projeto utilizar uma organização diferente e adequada.

---

## 10. Resultado esperado

Ao final da auditoria, a Skill deverá:

- Identificar os problemas encontrados.
- Classificar cada finding por severidade.
- Apresentar evidências com arquivo e linhas.
- Explicar impactos e recomendações.
- Investigar APIs deprecated e dependências obsoletas.
- Identificar riscos de segurança, arquitetura, desempenho e qualidade.
- Organizar os findings do mais grave ao menos grave.
- Diferenciar achados confirmados de suspeitas.
- Diferenciar achados principais de complementares, quando necessário.
- Informar limitações da análise.
- Não inventar problemas para atingir quantidades mínimas.

A Fase 2 deve gerar o relatório de auditoria e solicitar confirmação explícita do usuário antes de qualquer alteração nos arquivos do projeto.
