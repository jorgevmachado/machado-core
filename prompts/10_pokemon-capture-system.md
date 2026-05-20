# Proposta: pokemon-capture-system

## 0. Contexto Base obrigatório

Antes de gerar a proposta, considerar obrigatoriamente:

* [ARCHITECTURE-API](../docs/architecture-api.md)
* [ARCHITECTURE-WEB](../docs/architecture-web.md)
* [AGENTS API](../machado-api/AGENTS.md)
* [AGENTS WEB](../machado-web/AGENTS.md)

As decisões devem respeitar esses documentos.
Não criar novos padrões se já existir um definido.

---

## Observação importante

Já existem propostas anteriores relacionadas ao fluxo de exploração, batalha, Home, Party, MyPokemon e Pokedex:

* `5_my-pokemon-features`
* `6_pokedex-features`
* `7_trainer-encounter-exploration`
* `8_trainer-home-and-party-refactor`
* `9_wild-pokemon-battle-session`

Esta proposta NÃO deve:

* recriar o sistema de batalha;
* recriar o sistema de exploração;
* recriar o sistema de party;
* recriar o sistema de Home;
* recriar MyPokemon;
* recriar Pokedex;
* duplicar regras de battle-session;
* criar implementação paralela de captura.

Esta proposta deve apenas:

* implementar a captura do Pokémon selvagem;
* integrar com a battle-session já existente;
* reutilizar estruturas já criadas anteriormente;
* transformar Pokémon selvagem em `my-pokemon`;
* sincronizar descoberta da pokedex;
* preparar arquitetura para experiência e progressão futuras.

---

# 1. Objetivo

Implementar o sistema de captura de Pokémon selvagem durante uma batalha, permitindo que o treinador utilize pokebolas para tentar capturar o Pokémon encontrado.

A captura deve:

* consumir pokebolas;
* calcular chance de captura;
* transformar o Pokémon selvagem em `my-pokemon`;
* sincronizar descoberta da pokedex;
* adicionar o Pokémon capturado ao treinador;
* encerrar a battle-session;
* preparar estrutura futura para XP e level up.

---

# 2. Contexto Atual

* Projeto: machado (monorepo API + WEB)
* Domínio afetado:

  * BattleSession
  * Trainer
  * TrainerParty
  * MyPokemon
  * Pokedex
  * Pokemon
* Comportamento atual:

  * existe sistema de exploração;
  * existe battle-session;
  * existe MyPokemon;
  * existe Pokedex;
  * existe gerenciamento de Party;
* Problema identificado:

  * não existe sistema de captura;
  * Pokémon selvagem não pode ser convertido em `my-pokemon`;
  * pokebolas ainda não possuem uso funcional;
  * pokedex não sincroniza automaticamente após captura.

---

# 3. Comportamento Esperado

* O treinador deve conseguir tentar capturar Pokémon selvagem durante a battle-session.
* O sistema deve consumir pokebolas.
* O sistema deve calcular chance de captura.
* Quanto menor o HP do Pokémon selvagem, maior deve ser a chance de captura.
* O sistema deve impedir captura quando a battle-session estiver finalizada.
* O sistema deve impedir captura sem pokebolas.
* O sistema deve criar um `my-pokemon` após captura bem-sucedida.
* O sistema deve copiar atributos relevantes do Pokémon selvagem.
* O sistema deve selecionar 4 movimentos iniciais.
* O sistema deve sincronizar a pokedex do treinador.
* O sistema deve encerrar a battle-session após captura.
* O sistema deve retornar payload normalizado para o frontend.

---

# 4. Escopo

## Dentro do escopo

* Sistema de captura
* Consumo de pokebolas
* Chance de captura
* Conversão para `my-pokemon`
* Integração com battle-session
* Integração com pokedex
* Integração com trainer-party
* Encerramento da batalha
* Logs de captura
* Eventos normalizados
* Estrutura futura para XP e evolução

---

## Fora do escopo

* Sistema de marketplace
* Troca entre treinadores
* Pokebolas especiais
* Sistema competitivo
* Captura lendária avançada
* Sistema de breeding
* Sistema de shiny
* Sistema de habilidade passiva
* Sistema de captura em massa

---

# 5. Regras de Negócio

## Regra 1 — Captura apenas em batalha ativa

**Dado que:** existe uma battle-session
**Quando:** o treinador tentar capturar um Pokémon
**Então:** a captura deve ocorrer apenas se a sessão estiver ativa

---

## Regra 2 — Consumo de pokebola

**Dado que:** o treinador possui pokebolas
**Quando:** tentar capturar um Pokémon
**Então:** uma pokebola deve ser consumida

---

## Regra 3 — Sem pokebolas

**Dado que:** o treinador não possui pokebolas
**Quando:** tentar capturar um Pokémon
**Então:** o sistema deve impedir a captura

---

## Regra 4 — Chance de captura

**Dado que:** o treinador tentou capturar um Pokémon
**Quando:** a captura for processada
**Então:** a chance deve considerar o HP atual do Pokémon selvagem

---

## Regra 5 — Captura bem-sucedida

**Dado que:** a captura foi bem-sucedida
**Quando:** o processamento finalizar
**Então:** o sistema deve criar um `my-pokemon`

---

## Regra 6 — Sincronização da pokedex

**Dado que:** o Pokémon foi capturado
**Quando:** a captura for persistida
**Então:** a pokedex deve marcar o Pokémon como descoberto

---

## Regra 7 — Encerramento da batalha

**Dado que:** o Pokémon foi capturado
**Quando:** a captura finalizar
**Então:** a battle-session deve ser encerrada

---

# 6. Escopo Técnicos

## 6.1 API

Toda regra de negócio deve estar na API.
O frontend deve ser apenas consumidor.

### Funcionalidades

* endpoint para captura
* endpoint para consultar resultado da captura
* integração com battle-session
* integração com my-pokemon
* integração com pokedex

### Fluxos

1. treinador entra em battle-session
2. treinador utiliza pokebola
3. sistema calcula chance
4. sistema processa captura
5. sistema cria `my-pokemon`
6. sistema sincroniza pokedex
7. sistema encerra battle-session

---

## 6.1.1 Integrações externas

Não consumir PokéAPI diretamente nesta proposta.

Utilizar apenas dados persistidos localmente.

---

## 6.1.2 Enriquecimento de dados

### pokemon_capture

Persistir:

* trainer_id
* battle_session_id
* pokemon_id
* my_pokemon_id
* success
* pokeball_used
* created_at

---

### trainer

Atualizar:

* pokeball_quantity

---

### battle_log

Registrar eventos:

* CAPTURE_ATTEMPT
* CAPTURE_SUCCESS
* CAPTURE_FAILED

---

## 6.1.3 Cache

Utilizar o padrão existente do projeto.

Sugestão de chaves:

```txt
trainer:captures:{trainer_id}
trainer:home:{trainer_id}
trainer:pokedex:{trainer_id}
trainer:party:{trainer_id}
```

Regras:

* invalidar cache após captura
* invalidar cache após atualização da pokedex
* invalidar cache após alteração da party

---

## 6.1.4 Consistência e performance

* evitar duplicação de capturas
* evitar múltiplas capturas da mesma sessão
* manter integridade referencial
* utilizar soft delete
* evitar N+1 queries
* reutilizar estruturas existentes
* manter separação clara entre battle e capture

---

## 6.1.5 Contrato da API

A API deve:

* retornar dados normalizados;
* evitar transformação no frontend;
* manter consistência de naming.

Exemplo conceitual:

```json
{
  "capture_result": {
    "success": true,
    "my_pokemon": {},
    "pokedex_updated": true
  }
}
```

---

# 6.2 WEB

## 6.2.1 Páginas

* tela de batalha
* modal de captura
* detalhe do Pokémon capturado

---

## 6.2.2 Componentes

* botão de captura
* animação de captura
* feedback de captura
* contador de pokebolas
* card do Pokémon capturado

Estados:

* loading
* erro
* vazio
* sucesso

---

## 6.2.3 Renderização

A tela de batalha deve exibir:

* quantidade de pokebolas
* botão de captura
* feedback visual da captura
* resultado da captura
* informações do Pokémon capturado

---

## 6.2.4 Interações

* utilizar pokebola
* tentar capturar Pokémon
* visualizar resultado
* acessar Pokémon capturado
* atualizar Home após captura

---

# 7. Restrições Arquiteturais

* Não criar nova arquitetura
* Não duplicar lógica entre API e WEB
* Toda regra de negócio deve estar na API
* Reutilizar padrões existentes
* Minimizar impacto no código atual
* Não reimplementar battle-session
* Não duplicar regras de battle
* Não criar services gigantes
* Toda nova tabela criada no banco de dados deve suportar soft delete
* Não realizar exclusão física de registros, salvo se já existir padrão explícito no projeto
* Usar o padrão existente do projeto para soft delete, preferencialmente com `deleted_at`
* Consultas devem ignorar registros com soft delete ativo, salvo quando explicitamente necessário

---

# 8. Critérios de Aceite

* [ ] captura funcionando
* [ ] consumo de pokebolas funcionando
* [ ] cálculo de chance funcionando
* [ ] criação de `my-pokemon` funcionando
* [ ] sincronização da pokedex funcionando
* [ ] encerramento da battle-session funcionando
* [ ] logs funcionando
* [ ] payload normalizado
* [ ] integração com battle-session funcionando
* [ ] integração com pokedex funcionando
* [ ] integração com party funcionando
* [ ] sem duplicação de lógica
* [ ] sem regressões
* [ ] cache funcionando
* [ ] services organizados corretamente
* [ ] sem services excessivamente grandes
* [ ] novas tabelas possuem suporte a soft delete
* [ ] consultas padrão ignoram registros deletados logicamente
* [ ] nenhuma exclusão física foi implementada sem justificativa

---

# 9. Plano de Validação

## Validação manual

1. Criar treinador
2. Entrar em batalha
3. Reduzir HP do Pokémon selvagem
4. Utilizar pokebola
5. Validar consumo de pokebola
6. Validar captura
7. Validar criação do `my-pokemon`
8. Validar atualização da pokedex
9. Validar encerramento da battle-session
10. Validar atualização da Home

---

## Testes automatizados

* captura de Pokémon
* consumo de pokebola
* cálculo de chance
* sincronização da pokedex
* criação de `my-pokemon`
* encerramento da battle-session
* integração com battle-session
* integração com pokedex
* cenários de erro
* edge cases

---

# 10. Saída Esperada do OpenSpec

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
10. Estratégia de cache
11. Estratégia de rollback
12. Riscos da implementação
13. Dependências entre domínios
14. Estratégia de separação de responsabilidades
15. Estratégia futura para XP, evolução e progressão
