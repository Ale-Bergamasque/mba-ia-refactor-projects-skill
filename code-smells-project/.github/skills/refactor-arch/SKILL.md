---
name: refactor-arch
description: Analisa, audita e refatora projetos de software para uma arquitetura baseada em MVC, identificando problemas arquiteturais, de segurança, performance e qualidade de código. Use quando solicitado a auditar ou refatorar a arquitetura de um projeto.
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

Consulte:

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

Consulte:

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

Ao finalizar, pergunte ao usuário:

**"Auditoria concluída. Deseja prosseguir com a Fase 3 — Refatoração?"**

Aguarde a autorização explícita antes de continuar.

Se o usuário não autorizar, encerre a execução sem modificar o projeto.

## Fase 3 — Refatoração e Validação

### Pré-condição

Esta fase somente poderá ser executada após a autorização explícita do usuário.

### Objetivo

Corrigir os problemas identificados e organizar o projeto em uma arquitetura baseada em MVC, preservando seu comportamento original.

### Procedimentos

1. Consulte `references/mvc-guidelines.md`.
2. Consulte `references/refactoring-playbook.md`.
3. Planeje as alterações necessárias antes de executá-las.
4. Identifique quais responsabilidades pertencem a Models, Views/Routes e Controllers.
5. Extraia configurações e credenciais do código-fonte.
6. Organize as responsabilidades em módulos e camadas adequados à tecnologia.
7. Centralize o tratamento de erros quando aplicável.
8. Preserve os endpoints, contratos, regras de negócio e funcionalidades existentes.
9. Atualize imports, dependências e configurações afetadas.
10. Evite alterações desnecessárias em partes que já estejam corretamente organizadas.

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
7. Informe problemas que não puderam ser corrigidos.

Registre os comandos utilizados e os resultados reais das verificações. Para cada endpoint testado, informe método, rota e resultado obtido. Caso não seja possível executar a aplicação ou testar algum endpoint, informe o motivo e marque a validação como pendente, sem declarar sucesso.

Nunca declare que a aplicação está funcionando sem executar as verificações correspondentes.

### Saída esperada

Apresente:

- Nova estrutura de diretórios.
- Principais arquivos criados, alterados ou removidos.
- Problemas corrigidos.
- Problemas pendentes.
- Resultado dos testes.
- Resultado da inicialização da aplicação.
- Resultado da validação dos endpoints.

## Regras finais

- Trabalhe sempre em etapas sequenciais.
- Priorize segurança e preservação de comportamento.
- Não aplique transformações sem justificativa técnica.
- Não considere que a simples criação de pastas MVC significa que a arquitetura está correta.
- Seja transparente sobre limitações e validações não realizadas.
