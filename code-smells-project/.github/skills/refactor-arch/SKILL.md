---
name: refactor-arch
description: Analisa, audita e refatora projetos de software para uma arquitetura baseada em MVC, identificando problemas arquiteturais, de segurança, performance e qualidade de código. Use quando solicitado a auditar ou refatorar a arquitetura de um projeto.
user-invocable: true
---

# Skill: Refatoração Arquitetural

## Objetivo

Analisar uma codebase, identificar problemas arquiteturais e de qualidade, gerar um relatório de auditoria e, mediante autorização do usuário, refatorar o projeto para uma arquitetura baseada em MVC.

A Skill deve funcionar com diferentes linguagens, frameworks e bancos de dados, sem assumir uma tecnologia específica.

## Regras gerais

1. Analise o projeto antes de propor alterações.
2. Não presuma a linguagem, framework ou arquitetura: identifique-os examinando os arquivos existentes.
3. Consulte os arquivos de referência indicados em cada fase.
4. Preserve as funcionalidades e os contratos existentes da aplicação.
5. Não invente problemas, arquivos, linhas ou resultados de testes.
6. Informe sempre os arquivos e as linhas relacionados aos problemas encontrados.
7. Adapte as recomendações à tecnologia identificada.
8. Na Fase 1, não modifique arquivos. Na Fase 2, permita apenas a criação do relatório de auditoria; não altere o código-fonte nem outros arquivos da aplicação.
9. Nunca prossiga para a Fase 3 sem autorização explícita do usuário.
10. A arquitetura aprovada deve ser implementada de forma verificável; correções isoladas de segurança, performance ou organização não substituem a implementação das camadas aprovadas.
11. Mantenha rastreabilidade entre cada finding, a alteração correspondente, a evidência de validação e o status final.

## Fase 1 — Análise do Projeto

### Objetivo

Identificar a tecnologia, o domínio e a arquitetura atual do projeto.

### Procedimentos

1. Examine a estrutura de diretórios e os arquivos relevantes.
2. Identifique a linguagem de programação.
3. Identifique o framework e suas versões, quando disponíveis.
4. Identifique as dependências e os bancos de dados utilizados.
5. Identifique o domínio e as principais funcionalidades da aplicação.
6. Mapeie a organização atual das responsabilidades e camadas.
7. Identifique os pontos de entrada da aplicação.
8. Determine a quantidade de arquivos analisados.

Consulte obrigatoriamente:

`references/project-analysis.md`

### Saída esperada

Apresente um resumo contendo:

- Linguagem e versão.
- Framework e versão.
- Principais dependências.
- Banco de dados.
- Domínio da aplicação.
- Arquitetura atual.
- Estrutura de diretórios relevante.
- Quantidade de arquivos analisados.

Ao terminar, avance para a Fase 2.

## Fase 2 — Auditoria Arquitetural

### Objetivo

Identificar e classificar os anti-patterns e code smells encontrados no projeto.

### Procedimentos

1. Consulte `references/anti-pattern-catalog.md`.
2. Analise os arquivos relevantes da aplicação.
3. Compare os problemas encontrados com os critérios do catálogo.
4. Identifique violações arquiteturais, problemas de segurança, performance e qualidade.
5. Verifique o uso de APIs deprecated, considerando a tecnologia e a versão identificadas.
6. Classifique cada problema em CRITICAL, HIGH, MEDIUM ou LOW.
7. Informe o arquivo e as linhas exatas de cada ocorrência.
8. Descreva o problema, seu impacto e a recomendação de correção.
9. Organize os achados por ordem de severidade.
10. Consulte `references/mvc-guidelines.md` e elabore uma proposta arquitetural verificável, adaptada à tecnologia e às necessidades reais do projeto, contendo:
    - a arquitetura proposta e as camadas que deverão ser implementadas;
    - as responsabilidades de cada camada;
    - o mapeamento entre componentes existentes e componentes propostos;
    - as dependências permitidas entre as camadas;
    - os findings relacionados a cada mudança arquitetural;
    - os critérios de aceite funcionais, de segurança e arquiteturais;
    - a justificativa para qualquer camada não aplicável ou que não será implementada.
11. Inclua a proposta arquitetural no relatório de auditoria, identificando-a como `AGUARDANDO_APROVACAO`.
12. Não implemente a proposta nesta fase. A implementação somente poderá ocorrer após autorização explícita do usuário.

Consulte obrigatoriamente:

- `references/anti-pattern-catalog.md`
- `references/report-template.md`

### Regras obrigatórias

- Não invente findings para atingir uma quantidade mínima.
- Diferencie problemas confirmados de suspeitas que precisam de investigação.
- Não modifique o código-fonte nesta fase; permita somente a criação do relatório de auditoria.
- Gere o relatório completo de auditoria.
- Salve o relatório em `reports/`, na raiz do repositório principal (ao lado das pastas dos projetos), usando o nome `audit-project-1.md`, `audit-project-2.md` ou `audit-project-3.md`, conforme o projeto analisado.
- Crie a pasta `reports/` na raiz do repositório principal se necessário. A criação do relatório é permitida na Fase 2; não modifique o código-fonte da aplicação.
- Apresente ao usuário o caminho do relatório gerado antes de solicitar autorização para a Fase 3.
- A arquitetura descrita no relatório deve ser identificada como `PROPOSTA`, nunca como aprovada antes da autorização explícita do usuário.
- A conclusão da auditoria não representa aprovação automática da arquitetura nem autorização para modificar o código.
- Registre no relatório o status do plano arquitetural como `AGUARDANDO_APROVACAO` até que o usuário autorize explicitamente sua implementação.

Ao finalizar, pergunte ao usuário:

**"Auditoria concluída. Deseja prosseguir com a Fase 3 — Refatoração?"**

Aguarde a autorização explícita antes de continuar.

Se o usuário não autorizar, encerre a execução sem modificar o projeto.

## Fase 3 — Refatoração e Validação

### Pré-condição

Esta fase somente poderá ser executada após a autorização explícita do usuário.

Antes de qualquer alteração, a Skill deve também consultar obrigatoriamente:

- `references/mvc-guidelines.md`, para responsabilidades e dependências entre camadas;
- `references/refactoring-playbook.md`, para transformações e ordem de execução;
- `references/report-template.md`, para critérios de aceite, rastreabilidade e atualização final do relatório.

### Objetivo

Corrigir os problemas identificados e organizar o projeto em uma arquitetura baseada em MVC, preservando seu comportamento original.

### Procedimentos

1. Consulte `references/mvc-guidelines.md`.
2. Consulte `references/refactoring-playbook.md`.
3. Antes de editar, verifique o plano arquitetural apresentado no relatório de auditoria e a autorização explícita do usuário. O plano deve conter:
   - a arquitetura escolhida e as camadas que serão realmente implementadas;
   - as responsabilidades de cada camada;
   - o mapeamento entre componentes atuais e futuros componentes;
   - as dependências permitidas entre camadas;
   - os findings relacionados a cada mudança;
   - critérios de aceite funcionais, de segurança e arquiteturais;
   - a justificativa explícita para qualquer camada prevista que não será criada.
4. Se o relatório já apresentar um plano arquitetural completo e o usuário autorizar explicitamente sua implementação, registre o status como `APROVADO` e prossiga com a execução, sem solicitar uma segunda autorização para o mesmo plano.
5. Se o plano estiver incompleto ou exigir mudanças materiais em relação à proposta apresentada na auditoria, complete ou revise o plano, apresente as diferenças ao usuário e solicite autorização explícita antes de modificar o código. Enquanto essa autorização não for concedida, mantenha o status `AGUARDANDO_APROVACAO`.
6. Implemente todas as camadas, responsabilidades e dependências previstas no plano aprovado. Não substitua uma camada aprovada por reorganização superficial, renomeação de funções ou mera correção de segurança.
7. Se uma mudança relevante exigir alterar camadas, responsabilidades, dependências ou contratos previstos no plano, interrompa a execução, descreva a diferença e solicite nova autorização. Não continue com o plano alterado sem essa autorização.
8. Identifique quais responsabilidades pertencem a Models, Views/Routes, Controllers, Services, Repositories e componentes compartilhados, conforme as camadas escolhidas.
9. Extraia configurações e credenciais do código-fonte.
10. Organize as responsabilidades em módulos e camadas adequados à tecnologia, implementando efetivamente as camadas aprovadas.
11. Centralize o tratamento de erros quando aplicável.
12. Preserve os endpoints, contratos, regras de negócio e funcionalidades existentes, salvo mudanças de segurança aprovadas e documentadas.
13. Atualize imports, dependências e configurações afetadas.
14. Evite alterações desnecessárias em partes que já estejam corretamente organizadas.
15. Após cada mudança relevante, verifique as dependências entre camadas e confirme que regras de negócio e persistência permanecem nas camadas definidas pelo plano.

### Adaptação arquitetural

Aplique os princípios do MVC respeitando as convenções da tecnologia utilizada.

Não force uma estrutura idêntica entre linguagens e frameworks diferentes.

Se o projeto já possuir camadas bem organizadas, preserve-as e faça apenas as melhorias necessárias.

Os exemplos em Python e JavaScript presentes nos arquivos de referência são ilustrativos. Identifique primeiro a tecnologia real do projeto e aplique os princípios arquiteturais equivalentes, sem copiar os exemplos literalmente ou introduzir dependências desnecessárias.

### Validação obrigatória

Após a refatoração:

1. Execute os testes existentes.
2. Verifique se a aplicação inicia sem erros.
3. Verifique os endpoints originais.
4. Execute testes adicionais quando necessário.
5. Verifique imports, dependências e configurações.
6. Compare o comportamento anterior e posterior.
7. Verifique a arquitetura implementada:
   - confirme que as camadas aprovadas existem ou documente a divergência autorizada;
   - confirme que cada componente está na camada planejada;
   - confirme que Controllers não executam persistência diretamente;
   - confirme que Services não dependem de requisições ou respostas do framework;
   - confirme que Repositories encapsulam a persistência quando previstos;
   - confirme que Models não acumulam responsabilidades de Service e Repository;
   - confirme que as dependências entre camadas seguem o plano.
8. Separe os critérios de aceite em:
   - **Funcionais:** endpoints, contratos, regras de negócio e regressões;
   - **Segurança:** autenticação, autorização, segredos, validação, SQL e dados sensíveis;
   - **Arquiteturais:** camadas, responsabilidades, dependências e ausência de acoplamentos proibidos.

9. Para cada finding auditado, registre a alteração, os arquivos envolvidos, a evidência de verificação e classifique-o como `corrigido`, `parcialmente corrigido` ou `pendente`.
10. Preencha no relatório de auditoria a matriz de conformidade arquitetural definida em `references/report-template.md`, comparando todos os componentes previstos no plano aprovado com os componentes realmente implementados. Verifique responsabilidades, integração no fluxo da aplicação e dependências entre camadas, registrando evidências e classificando cada item como `APROVADO`, `REPROVADO` ou `PENDENTE`.
11. Informe problemas que não puderam ser corrigidos.

### Critério de conclusão da Fase 3

A Fase 3 somente poderá ser marcada como concluída quando todos os critérios arquiteturais aprovados no plano tiverem sido atendidos e evidenciados, além dos critérios funcionais e de segurança. Para cada critério não atendido, o relatório deve registrar a evidência, a limitação e o status `pendente`.

Se houver qualquer critério arquitetural aprovado pendente, a Skill deve interromper a conclusão normal, apresentar a lista de pendências e solicitar autorização explícita do usuário para concluir parcialmente. Sem essa autorização, a Fase 3 permanece incompleta.

Registre os comandos utilizados e os resultados reais das verificações. Para cada endpoint testado, informe método, rota e resultado obtido. Caso não seja possível executar a aplicação ou testar algum endpoint, informe o motivo e marque a validação como pendente, sem declarar sucesso.

Nunca declare que a aplicação está funcionando sem executar as verificações correspondentes.

### Saída esperada

Apresente:

- Plano arquitetural executado e comparação com o plano aprovado.
- Nova estrutura de diretórios.
- Principais arquivos criados, alterados ou removidos.
- Findings com rastreabilidade entre problema, alteração, evidência e classificação final.
- Critérios funcionais, de segurança e arquiteturais, cada um com resultado real.
- Problemas corrigidos, parcialmente corrigidos e pendentes.
- Resultado dos testes.
- Resultado da inicialização da aplicação.
- Resultado da validação dos endpoints.
- Limitações e diferenças em relação ao plano aprovado.
- Relatório de auditoria atualizado ao final da Fase 3.
- Evidências reais dos testes, inspeções e comandos executados para cada critério.
- Status final de cada finding: `corrigido`, `parcialmente corrigido` ou `pendente`.
- Registro explícito de qualquer critério arquitetural pendente e da autorização do usuário para eventual conclusão parcial.

## Regras finais

- Trabalhe sempre em etapas sequenciais.
- Priorize segurança e preservação de comportamento.
- Não aplique transformações sem justificativa técnica.
- Não considere que a simples criação de pastas MVC significa que a arquitetura está correta.
- Não considere a refatoração concluída se a arquitetura aprovada não tiver sido implementada ou se a divergência não tiver nova autorização.
- Seja transparente sobre limitações e validações não realizadas.
