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

Descreva em no máximo 2 ou 3 linhas o que precisa ser feito.

Evite detalhes técnicos aqui.

---

## 2. Contexto Atual

Explique apenas o necessário para entender o problema.

- Projeto:
- Domínio afetado:
- Arquivos/pastas relevantes:
- Comportamento atual:
- Problema identificado:

---

## 3. Comportamento Esperado

Descreva como o sistema deve funcionar após a mudança.

- O sistema deve...
- O usuário deve conseguir...
- A API deve retornar...
- A interface deve exibir...

Se possível, seja específico.

---

## 4. Escopo

### Dentro do escopo

- Item que será implementado
- Item que será alterado
- Item que será criado

### Fora do escopo

- O que NÃO deve ser alterado
- O que NÃO deve ser feito
- Limitações importantes

---

## 5. Regras de Negócio

### Regra 1 — [nome da regra]

**Dado que:** contexto inicial  
**Quando:** ação acontece  
**Então:** resultado esperado  

---

### Regra 2 — [nome da regra]

**Dado que:** contexto inicial  
**Quando:** ação acontece  
**Então:** resultado esperado  

---

## 6. Requisitos Técnicos

### Backend

- Criar/alterar endpoints (ex: `POST /battle/fight`)
- Criar/alterar services
- Criar/alterar repositories
- Criar/alterar models
- Criar/alterar schemas

---

### Frontend

- Criar/alterar páginas
- Criar/alterar hooks
- Criar/alterar componentes
- Tratar estados:
  - loading
  - erro
  - vazio
  - sucesso

---

## 7. Restrições Arquiteturais

- Seguir o padrão existente de domínio (ex: `app/domain/...`)
- Não criar nova arquitetura
- Não alterar autenticação sem necessidade
- Não duplicar lógica
- Utilizar async/await
- Utilizar ORM (SQLAlchemy)
- Reutilizar estruturas existentes sempre que possível

---

## 8. Critérios de Aceite

- [ ] Critério objetivo 1
- [ ] Critério objetivo 2
- [ ] Critério objetivo 3
- [ ] Nenhum fluxo existente foi quebrado
- [ ] Testes foram criados ou ajustados

---

## 9. Plano de Validação

### Validação manual

1. Passo 1  
2. Passo 2  
3. Resultado esperado  

---

### Testes automatizados

- Teste de cenário principal
- Teste de erro
- Teste de edge case

---

## 10. Saída Esperada do OpenSpec

Gerar uma proposta contendo:

1. Resumo da mudança  
2. Problema identificado  
3. Solução proposta  
4. Arquivos impactados  
5. Plano de implementação  
6. Alterações de modelo de dados (se houver)  
7. Alterações de API (se houver)  
8. Alterações de UI (se houver)  
9. Estratégia de testes  
10. Riscos e pontos de atenção
