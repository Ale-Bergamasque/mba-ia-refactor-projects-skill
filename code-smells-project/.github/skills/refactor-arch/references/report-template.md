# Modelo de Relatório de Auditoria Arquitetural

## 1. Objetivo

Este documento define a estrutura obrigatória do relatório produzido durante a Fase 2 — Auditoria da Skill `refactor-arch`.

A Skill deve preencher o relatório com informações reais obtidas na análise do projeto e com os problemas identificados utilizando `anti-pattern-catalog.md`.

O relatório deve ser escrito em Markdown, de maneira clara e objetiva.

Durante a Fase 2, a Skill pode criar o arquivo de relatório, mas não pode modificar o código-fonte da aplicação.

A refatoração só poderá começar após confirmação explícita do usuário.

---

## 2. Local de armazenamento

Os relatórios devem ser armazenados na pasta `reports/`, na raiz do repositório principal que contém os projetos analisados, conforme definido no `SKILL.md`.

A Skill deve identificar o repositório de destino a partir do workspace e da estrutura real de diretórios. Não deve presumir que o diretório onde a Skill está instalada é o destino do relatório.

Se o repositório de destino não puder ser identificado de maneira inequívoca, solicitar esclarecimento ao usuário antes de criar o relatório.

A execução da Skill a partir de outro diretório não altera o local de armazenamento definido para o relatório.

Para a atividade, utilizar:

- Projeto 1 — `reports/audit-project-1.md`
- Projeto 2 — `reports/audit-project-2.md`
- Projeto 3 — `reports/audit-project-3.md`

O número do projeto deve ser informado pelo usuário ou identificado de maneira inequívoca a partir do contexto da execução.

Se não for possível determinar o número, perguntar ao usuário antes de salvar. Não adivinhar.

A pasta `reports/` pode ser criada durante a Fase 2 exclusivamente para armazenar o relatório.

---

## 3. Estrutura obrigatória do relatório

Utilizar o modelo abaixo, substituindo os campos entre colchetes pelas informações encontradas.

# Relatório de Auditoria Arquitetural — [Nome do projeto]

## A. Identificação do projeto

| Campo                | Informação                             |
| -------------------- | -------------------------------------- |
| Projeto              | [Nome]                                 |
| Linguagem            | [Linguagem e versão, se identificada]  |
| Framework            | [Framework e versão, se identificados] |
| Banco de dados       | [Banco utilizado ou não identificado]  |
| Arquitetura atual    | [Resumo da arquitetura encontrada]     |
| Arquitetura desejada | MVC adaptado à tecnologia              |
| Escopo analisado     | [Diretórios e arquivos relevantes]     |
| Data da auditoria    | [Data real da execução]                |

### Resumo da arquitetura atual

[Descrever brevemente como o projeto está organizado, quais responsabilidades estão concentradas ou distribuídas e quais camadas já existem.]

### Limitações da análise

[Informar arquivos não acessíveis, dependências não instaladas, testes não executados ou outras limitações. Se não houver limitações relevantes, registrar isso.]

---

## B. Resumo executivo

[Escrever um resumo de até três parágrafos com os problemas mais importantes, os principais riscos e a direção recomendada para a refatoração.]

### Quantidade de findings

| Severidade |   Quantidade |
| ---------- | -----------: |
| CRITICAL   |     [Número] |
| HIGH       |     [Número] |
| MEDIUM     |     [Número] |
| LOW        |     [Número] |
| **Total**  | **[Número]** |

Contabilizar somente findings confirmados.

Suspeitas e riscos pendentes de validação devem aparecer em seção separada e não entrar nessa contagem.

### Prioridades recomendadas

1. [Prioridade mais importante]
2. [Segunda prioridade]
3. [Terceira prioridade]

As prioridades devem considerar segurança, integridade dos dados, impacto arquitetural e dependências entre mudanças.

---

## C. Findings confirmados

Organizar os findings por severidade, na seguinte ordem:

1. CRITICAL
2. HIGH
3. MEDIUM
4. LOW

Utilizar um identificador sequencial para cada finding: F-001, F-002, F-003 etc.

Repetir o modelo abaixo para cada problema confirmado.

### F-001 — [Nome do anti-pattern]

**Severidade:** [CRITICAL / HIGH / MEDIUM / LOW]

**Categoria:** [Segurança / Arquitetura / Desempenho / Qualidade / Compatibilidade]

**Localização:**

- Arquivo: `[caminho/arquivo.ext]`
- Linha(s): `[número ou intervalo exato]`

**Evidência:**

[Apresentar um pequeno trecho de código ou descrever objetivamente o comportamento encontrado.]

Não reproduzir senhas, tokens ou outros segredos.

**Descrição do problema:**

[Explicar o que está errado e por que isso representa um problema.]

**Impacto:**

[Descrever as possíveis consequências, distinguindo impactos demonstrados de impactos potenciais.]

**Recomendação:**

[Explicar a correção recomendada, considerando a linguagem e a arquitetura do projeto.]

**Justificativa da severidade:**

[Explicar por que essa classificação foi escolhida, especialmente se for diferente da severidade padrão do catálogo.]

**Situação da evidência:** Confirmado por análise do código.

**Validação adicional necessária:** [Informar se testes ou verificações adicionais são recomendados. Caso contrário, escrever "Não identificada".]

---

## D. APIs deprecated e dependências obsoletas

A Skill deve investigar obrigatoriamente o uso de APIs deprecated e dependências obsoletas.

Para cada ocorrência confirmada, registrar:

| Item   | Tipo                                                | Versão identificada | Evidência                          | Alternativa recomendada |
| ------ | --------------------------------------------------- | ------------------- | ---------------------------------- | ----------------------- |
| [Nome] | [API / Dependência direta / Dependência transitiva] | [Versão]            | [Arquivo, linha e/ou documentação] | [Alternativa]           |

Quando houver uma ocorrência confirmada com impacto relevante, ela também deverá ser registrada na seção de findings, sem duplicar sua contagem.

Distinguir:

- API oficialmente deprecated.
- API removida.
- Dependência oficialmente deprecated.
- Dependência sem suporte.
- Dependência com vulnerabilidade conhecida.
- Suspeita de obsolescência ainda não confirmada.

Não classificar um recurso como deprecated apenas por ser antigo.

Se nenhuma ocorrência for confirmada, escrever:

"Nenhuma API deprecated ou dependência obsoleta foi confirmada no escopo analisado."

Se a verificação for incompleta, explicar a limitação.

---

## E. Riscos e suspeitas pendentes de validação

Registrar aqui problemas que apresentam sinais relevantes, mas ainda não possuem evidência suficiente para confirmação.

Para cada suspeita, apresentar:

### S-001 — [Título da suspeita]

**Localização:** [Arquivo e linhas, quando identificados]

**Indício encontrado:** [Descrição objetiva]

**Por que ainda não foi confirmado:** [Limitação ou informação ausente]

**Como validar:** [Teste, inspeção ou consulta necessária]

**Impacto potencial:** [Consequência caso seja confirmado]

Não incluir suspeitas na contagem de findings confirmados.

Não apresentar hipóteses como vulnerabilidades comprovadas.

---

## F. Recomendações arquiteturais para MVC

Descrever como o projeto pode ser organizado em uma arquitetura baseada em MVC, respeitando a tecnologia utilizada.

### Estrutura sugerida

[Apresentar uma proposta de organização de diretórios compatível com o projeto.]

### Responsabilidades recomendadas

**Models:**
[Responsabilidades relacionadas às entidades e aos dados.]

**Views / Routes:**
[Responsabilidades de entrada e saída HTTP, rotas e apresentação de respostas, conforme o framework.]

**Controllers:**
[Responsabilidades de coordenação das requisições.]

**Services:**
[Regras de negócio que justificam uma camada própria.]

**Repositories / Persistência:**
[Acesso ao banco de dados, quando a separação for adequada.]

**Configurações e componentes compartilhados:**
[Configurações, tratamento de erros, autenticação, validação e outros componentes necessários.]

### Componentes que devem ser preservados

[Identificar partes da organização atual que já estão adequadas e não precisam ser reestruturadas.]

Não criar camadas desnecessárias apenas para reproduzir uma estrutura fixa.

### Plano arquitetural verificável proposto

**Status do plano:** `AGUARDANDO_APROVACAO`

**Arquitetura proposta:** [Descrever a arquitetura escolhida e as camadas previstas, conforme a tecnologia e as necessidades do projeto.]

| Componente atual | Responsabilidade atual | Componente futuro    | Camada   | Finding relacionado | Critério de aceite   |
| ---------------- | ---------------------- | -------------------- | -------- | ------------------- | -------------------- |
| [Arquivo/função] | [Responsabilidade]     | [Arquivo/componente] | [Camada] | [F-xxx ou N/A]      | [Evidência esperada] |

**Dependências permitidas entre camadas:** [Descrever a direção proposta e as exceções justificadas.]

**Camadas deliberadamente não criadas:** [Listar e justificar. Se nenhuma, escrever "Nenhuma".]

**Critérios de aceite funcionais:** [Descrever os comportamentos, contratos e funcionalidades que deverão ser preservados ou verificados.]

**Critérios de aceite de segurança:** [Descrever as verificações de segurança relacionadas aos findings e às alterações propostas.]

**Critérios de aceite arquiteturais:** [Listar verificações de responsabilidades, integrações e dependências.]

**Regra de aprovação:** O status somente poderá ser alterado para `APROVADO` após autorização explícita do usuário para implementar este plano. A conclusão da auditoria não constitui aprovação automática.

Se o plano precisar de alterações materiais após a aprovação, registrar as diferenças e solicitar nova autorização antes de executar as mudanças afetadas.

---

## G. Plano sugerido de refatoração

Apresentar uma sequência de alterações recomendadas, sem executá-las durante a Fase 2.

| Etapa | Alteração proposta | Findings relacionados | Risco da mudança       |
| ----- | ------------------ | --------------------- | ---------------------- |
| 1     | [Alteração]        | [IDs]                 | [Baixo / Médio / Alto] |
| 2     | [Alteração]        | [IDs]                 | [Baixo / Médio / Alto] |
| 3     | [Alteração]        | [IDs]                 | [Baixo / Médio / Alto] |

Priorizar correções de segurança e integridade dos dados, respeitando as dependências entre mudanças.

O plano deve preservar, sempre que possível:

- Contratos dos endpoints.
- Formatos das respostas.
- Regras de negócio existentes.
- Compatibilidade com o banco de dados.
- Funcionalidades já implementadas.

Se alguma alteração exigir mudança de comportamento ou contrato, informar isso explicitamente antes de executá-la.

---

## H. Estratégia de validação após a refatoração

Descrever como a Fase 3 deverá verificar se a aplicação continua funcionando.

### Verificações previstas

- [ ] Instalar ou verificar dependências necessárias.
- [ ] Verificar se a aplicação inicia sem erros.
- [ ] Executar os testes automatizados existentes.
- [ ] Verificar os endpoints principais.
- [ ] Verificar códigos de status e formatos de resposta.
- [ ] Verificar operações relevantes de leitura e escrita.
- [ ] Verificar possíveis regressões.
- [ ] Registrar testes que não puderem ser executados.
- [ ] Verificar a existência e a conformidade das camadas aprovadas no plano arquitetural.
- [ ] Verificar as dependências entre camadas e a ausência de acessos proibidos.
- [ ] Separar resultados funcionais, de segurança e arquiteturais.

Adaptar os comandos de execução e testes à linguagem e ao framework encontrados.

Não afirmar que testes foram executados se eles ainda não foram realizados.

---

## I. Conclusão da auditoria

[Resumir os principais problemas encontrados, a viabilidade da refatoração e os cuidados necessários.]

### Status da execução

- Fase 1 — Análise: concluída.
- Fase 2 — Auditoria: concluída.
- Fase 3 — Refatoração: aguardando autorização.

### Confirmação obrigatória

Ao finalizar o relatório, a Skill deve apresentar ao usuário a seguinte pergunta:

**"A auditoria foi concluída e o relatório está disponível. Você autoriza iniciar a Fase 3 — Refatoração MVC, conforme o plano apresentado?"**

A Skill deve aguardar uma resposta explícita.

Não interpretar silêncio, mudança de assunto ou resposta ambígua como autorização.

---

## J. Atualização obrigatória após a Fase 3

Esta seção deve ser preenchida durante a Fase 3 e finalizada após a refatoração, no mesmo relatório.

### Autorização do plano arquitetural

**Status do plano:** [AGUARDANDO_APROVACAO / APROVADO]

**Autorização explícita:** [Registrar a confirmação efetivamente fornecida pelo usuário, sem inventar aprovação.]

**Plano autorizado:** [Identificar a proposta arquitetural aprovada na seção F e eventuais revisões autorizadas.]

**Alterações materiais posteriores:** [Registrar mudanças propostas, diferenças em relação ao plano original e respectivas autorizações. Se não houver, escrever "Nenhuma".]

O status `APROVADO` somente pode ser utilizado quando existir autorização explícita do usuário. A aprovação do plano não significa que sua implementação já foi concluída ou validada.

### Arquitetura implementada

[Descrever as camadas realmente criadas ou preservadas, suas responsabilidades, dependências e integração com o fluxo da aplicação.]

### Matriz de conformidade arquitetural

| Componente previsto            | Camada   | Arquivo implementado               | Responsabilidade verificada | Integração verificada    | Evidência                              | Status                        |
| ------------------------------ | -------- | ---------------------------------- | --------------------------- | ------------------------ | -------------------------------------- | ----------------------------- |
| [Componente do plano aprovado] | [Camada] | [Caminho real ou não implementado] | [Sim/Não/Não verificado]    | [Sim/Não/Não verificado] | [Arquivo, teste, comando ou limitação] | [APROVADO/REPROVADO/PENDENTE] |

Regras de preenchimento:

1. Incluir uma linha individual para cada arquivo ou componente explicitamente previsto no plano arquitetural aprovado, identificando a camada correspondente. Não agrupar componentes distintos em entradas genéricas como `Services`, `Repositories`, `Controllers` ou `Routes`. Quando um componente previsto não existir, registrar `Não implementado` e classificar sua situação conforme as regras de conformidade. Se outro componente assumir sua responsabilidade, documentar a substituição e verificar se houve autorização para eventual alteração material do plano.
2. Verificar se cada componente desempenha sua responsabilidade e participa do fluxo real da aplicação.
3. Registrar evidências verificáveis, sem presumir que a existência de arquivos ou diretórios comprova conformidade.
4. Classificar como `REPROVADO` componentes que comprovadamente não atendam ao plano aprovado.
5. Classificar como `PENDENTE` componentes cuja implementação ou integração não tenha sido possível verificar, indicando o motivo.
6. Não declarar a refatoração totalmente concluída enquanto existirem critérios arquiteturais `REPROVADO` ou `PENDENTE`.
7. Quando a implementação estiver incompleta, registrar as limitações e aplicar as regras de conclusão parcial definidas no `SKILL.md`.

### Diferenças em relação ao plano aprovado

[Listar cada diferença. Se houver mudança relevante não autorizada, a Fase 3 não deve ser considerada concluída.]

### Critérios de aceite funcionais

| Critério                      | Resultado                         | Evidência               |
| ----------------------------- | --------------------------------- | ----------------------- |
| [Endpoint, contrato ou regra] | [Aprovado / Reprovado / Pendente] | [Comando/teste/arquivo] |

### Critérios de aceite de segurança

| Critério                | Resultado                         | Evidência               |
| ----------------------- | --------------------------------- | ----------------------- |
| [Controle de segurança] | [Aprovado / Reprovado / Pendente] | [Comando/teste/arquivo] |

### Critérios de aceite arquiteturais

| Critério                                  | Resultado                         | Evidência                |
| ----------------------------------------- | --------------------------------- | ------------------------ |
| [Camada, responsabilidade ou dependência] | [Aprovado / Reprovado / Pendente] | [Inspeção/comando/teste] |

### Rastreabilidade e status final dos findings

| Finding | Alteração realizada    | Evidência de correção | Status                                          |
| ------- | ---------------------- | --------------------- | ----------------------------------------------- |
| [F-001] | [Alteração e arquivos] | [Teste/inspeção]      | [Corrigido / Parcialmente corrigido / Pendente] |

Nenhum finding pode ser marcado como `corrigido` sem evidência funcional, de segurança ou arquitetural compatível com sua categoria.

### Testes reais e limitações

[Registrar comandos, resultados reais, endpoints testados, testes não executados e limitações do ambiente.]

### Conclusão da Fase 3 — Refatoração

**Status final:** [CONCLUIDA / PARCIALMENTE_CONCLUIDA / INCOMPLETA]

**Resumo da implementação:** [Descrever objetivamente o que foi implementado e validado.]

**Critérios funcionais:** [Quantidade aprovada, reprovada e pendente.]

**Critérios de segurança:** [Quantidade aprovada, reprovada e pendente.]

**Critérios arquiteturais:** [Quantidade aprovada, reprovada e pendente.]

**Findings:** [Quantidade corrigida, parcialmente corrigida e pendente.]

**Pendências e limitações:** [Listar os itens não resolvidos ou não verificados. Se não houver, escrever "Nenhuma".]

**Próximas ações necessárias:** [Descrever correções, verificações ou decisões pendentes. Se não houver, escrever "Nenhuma".]

#### Regras para determinar o status final

- `CONCLUIDA`: todos os critérios funcionais, de segurança e arquiteturais previstos no plano aprovado foram atendidos e possuem evidências reais. Não existem critérios reprovados ou pendentes.
- `PARCIALMENTE_CONCLUIDA`: parte das alterações foi implementada, mas ainda existem critérios reprovados ou pendentes. Esse status somente pode ser utilizado quando o usuário autorizar explicitamente a conclusão parcial, conforme as regras do `SKILL.md`.
- `INCOMPLETA`: existem alterações ou verificações pendentes e não houve autorização para considerar a execução parcialmente concluída, ou a implementação não atende às condições de conclusão.

A existência de arquivos, classes, diretórios ou testes executados não comprova, isoladamente, que a arquitetura aprovada foi implementada.

Não declarar a Fase 3 como `CONCLUIDA` quando houver divergências não autorizadas, critérios reprovados, critérios pendentes ou ausência de evidências necessárias.

O status final deve ser fundamentado nas matrizes de conformidade, nos critérios de aceite e nos resultados reais das verificações.

---

## 4. Regras de preenchimento do relatório

1. Utilizar apenas informações encontradas no projeto ou confirmadas em fontes confiáveis.
2. Não inventar arquivos, linhas, versões, dependências ou resultados de testes.
3. Verificar os números de linha no código real antes de registrar findings.
4. Não incluir exemplos hipotéticos como se fossem evidências do projeto.
5. Não reproduzir credenciais ou informações sensíveis.
6. Evitar contabilizar o mesmo problema mais de uma vez.
7. Diferenciar findings confirmados de suspeitas.
8. Explicar limitações da análise.
9. Adaptar recomendações à tecnologia encontrada.
10. Preservar a estrutura do relatório mesmo quando alguma seção não possuir ocorrências.
11. Utilizar linguagem objetiva e compreensível.
12. Não modificar o código-fonte durante a auditoria.
13. Solicitar autorização explícita antes de iniciar a refatoração.

---

## 5. Resultado esperado

Ao concluir a Fase 2, a Skill deverá:

1. Salvar o relatório Markdown na pasta `reports/`.
2. Apresentar um resumo dos findings por severidade.
3. Destacar os problemas de maior impacto.
4. Informar os riscos que ainda precisam de validação.
5. Apresentar a proposta de refatoração MVC.
6. Informar quais testes deverão ser realizados na Fase 3.
7. Solicitar confirmação explícita do usuário.

**A criação do relatório não autoriza alterações no código da aplicação.**
