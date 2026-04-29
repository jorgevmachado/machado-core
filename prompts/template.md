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

## 6. Escopo Técnicos

### 6.1 API

Toda regra de negócio deve estar na API.
O frontend deve ser apenas consumidor.

Funcionalidade
- endpoints necessários
- filtros, paginação, parâmetros
Fluxos

Descrever cenários:

1. inicial (ex: base vazia)
2. intermediário
3. ideal (com cache/dados completos)

#### 6.1.1 Integrações externas (se houver)

Endpoints de referência:

{{external-endpoints}}

Regras:

- Usar endpoints para enriquecer o domínio
- Priorizar dados completos e normalizados
- Evitar chamadas redundantes

#### 6.1.2 Enriquecimento de dados

Durante o processamento, a API pode buscar e persistir:

{{data-fields}}

Regras:

- Buscar apenas quando necessário
- Persistir para evitar novas chamadas
- Não falhar se algum campo não existir

#### 6.1.3 Cache (se aplicável)

Descrever:

- objetivo
- chaves
- TTL

Exemplo:

{{cache-keys}}

#### 6.1.4 Consistência e performance
- Garantir idempotência
- Evitar duplicação
- Utilizar constraints únicas
- Controlar paralelismo
- Limitar chamadas externas
- Aplicar fallback

#### 6.1.5 Contrato da API

A API deve:

- retornar dados normalizados
- evitar transformação no frontend
- manter consistência de naming

### 6.2 WEB
#### 6.2.1 Páginas
- páginas envolvidas
- fluxo de navegação

#### 6.2.2 Componentes
- componentes principais
- comportamento esperado
- estados:
- - loading
- - erro
- - vazio
- - sucesso

#### 6.2.3 Renderização

Descrever como os dados devem aparecer:

{{ui-snippet}}

#### 6.2.4 Interações
- ações do usuário
- paginação
- carregamento incremental
- botões e eventos

---

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

- [ ] funcionalidade implementada
- [ ] integração funcionando
- [ ] dados persistidos corretamente
- [ ] cache aplicado corretamente
- [ ] sem duplicação de registros
- [ ] frontend consumindo corretamente
- [ ] UI consistente
- [ ] sem regressões
- [ ] Novas tabelas possuem suporte a soft delete
- [ ] Consultas padrão ignoram registros deletados logicamente
- [ ] Nenhuma exclusão física foi implementada sem justificativa

## 9. Plano de Validação

Validação manual
1. Passo
2. Passo
3. Resultado esperado

Testes automatizados
- cenário principal
- erro
- edge cases

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
