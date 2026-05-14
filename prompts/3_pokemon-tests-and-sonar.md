# Proposta: [nome-curto-da-mudança]

## 0. Contexto Base obrigatório

Antes de gerar a proposta, considerar obrigatoriamente:

- [ARCHITECTURE-API](../docs/architecture-api.md)
- [ARCHITECTURE-WEB](../docs/architecture-web.md)
- [AGENTS API](../machado-api/AGENTS.md)
- [AGENTS WEB](../machado-web/AGENTS.md)

As decisões devem respeitar esses documentos.
Não criar novos padrões se já exister um definido.

---

## 1. Objetivo

Aumentar a cobertura de testes unitários e de integração, além de melhorar a qualidade do código utilizando o SonarQube.

---

## 2. Contexto Atual

Explique apenas o necessário para entender o problema.

- Projeto: machado (monorepo API e WEB)
- Domínio afetado: Todos os arquivos.
- Comportamento atual: baixa cobertura de testes e qualidade de código abaixo do ideal.
- Problema identificado: 
- - Baixa cobertura de testes unitários no projeto machado-web 
- - Baixa cobertura de testes e2e no projeto machado-web
- - Ajuste na cobertura do projeto machado-api que está em 99% reclamando do arquivo `machado-api/app/domain/pokemon/schema.py`
- - Sonar instalado na IDE pyCharn reclamando de 11765 issues em 253 arquivos.
---

## 3. Comportamento Esperado

Deve ser aumentar a cobertura dos testes unitários e de integração do projeto machado-web e os testes unitários do projeto machado-api.
Assim como configurar via ambiente local o uso do SonarQube para melhorar a qualidade do código e reduzir as issues apontadas.

---

## 4. Escopo

### Dentro do escopo

- Aumento de cobertura de testes unitários do projeto machado-web
- Aumento de cobertura de testes de integração do projeto machado-web
- Aumento de cobertura de testes unitários do projeto machado-api
- Uso do SonarQube para melhoria de código.
- Executar SonarQube localmente.
- Corrigir todos os issues apontados pelo SonarQube.

### Fora do escopo

- Não implementar novas features.

---

## 5. Regras de Negócio

### Regra 1 — SonarQube deve ser utilizado para identificar e corrigir problemas de qualidade de código.

**Dado que:** SonarQube esteja configurado 
**Quando:** Ao ser executado para analisar o código do projeto
**Então:** Corrigir todas as issues apontadas, incluindo bugs, vulnerabilidades e code smells, para melhorar a qualidade do código, sem que quebre a aplicação.

---

### Regra 2 — Limitar onde o SonarQube pode ser executado.

**Dado que:** SonarQube esteja configurado para ser executado apenas em ambiente local  
**Quando:** Ao ser executado para analisar o código do projeto  
**Então:** Não deve analisar arquivos que sejam criados automaticamente, como coverage e migrations  

---

## 6. Escopo Técnicos

### 6.1 API

- Aumentar a cobertura do arquivo `machado-api/app/domain/pokemon/schema.py`
- Executar o SonarQube localmente
- Corrigir todos os issues apontados pelo SonarQube
- Ignorar arquivos que sejam criados automaticamente, como coverage e migrations, para análise do SonarQube.
- Ajustar lint/ruff do projeto `machado-api`


### 6.2 WEB
 - Aumentar a cobertura dos testes unitários do projeto `machado-web`.
 - Aumentar a cobertura dos testes de integração do projeto `machado-api`.
 - Executar o SonarQube localmente
 - Corrigir todos os issues apontados pelo SonarQube
 - Ignorar arquivos que sejam criados automaticamente, como coverage e migrations, para análise do SonarQube.
 - Ajustar lint do projeto `machado-web`

## 7. Restrições Arquiteturais
- Não criar nova arquitetura
- Não duplicar lógica entre API e WEB
- Toda regra de negócio deve estar na API
- Reutilizar padrões existentes
- Minimizar impacto no código atual
- Toda nova tabela criada no banco de dados deve suportar soft delete.
- Não realizar exclusão física de registros, salvo se já existir padrão explícito no projeto.
- Usar o padrão existente do projeto para soft delete, preferencialmente com `deleted_at`.
- Consultas devem ignorar registros com soft delete ativo, salvo quando explicitamente necessário.

---

## 8. Critérios de Aceite

- [ ] 100% de cobertura de testes unitários no projeto `machado-web`
- [ ] 100% de cobertura de testes de integração no projeto `machado-web`
- [ ] 100% de cobertura de testes unitários no projeto `machado-api`
- [ ] Execução do SonarQube localmente
- [ ] Correção de todos os issues apontados pelo SonarQube
- [ ] Ignorar arquivos que sejam criados automaticamente, como coverage e migrations, para análise do SonarQube.
- [ ] Ajuste da lint/ruff do projeto `machado-api`
- [ ] Ajuste da lint do projeto `machado-web`

## 9. Plano de Validação

Validação manual
1. Executar o comando `yarn test --coverage` no projeto `machado-web` e confirmar que esteja em 100% de cobertura.
2. Executar o comando `yarn test:e2e` no projeto `machado-web` e confirmar que esteja em 100% de cobertura.
3. Executar o comando `make test` no projeto `machado-api` e confirmar que esteja em 100% de cobertura.
4. Executar o SonarQube localmente e confirmar que esteja em 100% de cobertura.

---

## 10. Saída Esperada do OpenSpec

Gerar uma proposta contendo:

1. Resumo da mudança  
2. Problema identificado  
3. Solução proposta  
4. Arquivos impactados  
5. Plano de implementação  
6. Alterações de modelo  
7. Alterações de API  
8. Alterações de UI  
9. Estratégia de testes  
10. Riscos e pontos de atenção
