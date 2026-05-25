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

* `my-pokemon-management`
* `pokedex-management`
* `trainer-exploration`
* `wild-pokemon-battle-session`

Esta proposta NÃO deve:

* recriar o sistema de batalha;
* recriar o sistema de exploração;
* recriar o sistema de party;
* recriar o sistema de Home;
* recriar MyPokemon;
* recriar Pokedex;
* duplicar regras de battle-session;
* duplicar regras já pertencentes aos domínios `my-pokemon` e `pokedex`;
* manter fluxos paralelos de captura fora da orquestração principal;
* assumir que o endpoint atual `POST /trainer/my-pokemon` continuará sendo o entrypoint correto para captura.

Esta proposta deve apenas:

* implementar a captura do Pokémon selvagem durante a battle-session;
* integrar com a battle-session já existente;
* orquestrar o fluxo de captura no domínio `trainer`;
* reutilizar `my-pokemon/service.py` para materializar o Pokémon capturado;
* reutilizar `pokedex/service.py` para registrar a descoberta do Pokémon do treinador;
* sincronizar descoberta da pokedex após captura bem-sucedida;
* avaliar a descontinuação, substituição ou absorção do `POST /trainer/my-pokemon` atual;
* preparar arquitetura para experiência e progressão futuras sem implementá-las agora.

---

# 1. Objetivo

Implementar o sistema de captura de Pokémon selvagem durante uma batalha, permitindo que o treinador utilize pokebolas para tentar capturar o Pokémon encontrado.

A captura deve:

* consumir pokebolas;
* calcular chance de captura;
* orquestrar o fluxo completo no domínio `trainer`;
* criar um `my-pokemon` somente em caso de sucesso;
* sincronizar a pokedex do treinador após sucesso;
* encerrar a battle-session com status específico de captura concluída;
* manter a battle-session ativa quando a captura falhar;
* preparar estrutura futura para XP e level up sem ampliar o escopo atual.

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
* Arquivos/pastas relevantes:

  * `machado-api/app/domain/trainer/`
  * `machado-api/app/domain/my_pokemon/`
  * `machado-api/app/domain/pokedex/`
  * `machado-api/app/domain/wild_pokemon_battle_session/`
  * `machado-web/app/`
  * `machado-web/app/ui/features/`
* Comportamento atual:

  * existe sistema de exploração;
  * existe battle-session;
  * existe MyPokemon;
  * existe Pokedex;
  * existe gerenciamento de Party;
  * existe endpoint `POST /trainer/my-pokemon`, mas ele não representa o fluxo completo de captura em batalha;
* Problema identificado:

  * não existe sistema completo de captura integrado à battle-session;
  * pokebolas ainda não possuem uso funcional no fluxo principal;
  * a captura ainda não sincroniza automaticamente a pokedex do treinador;
  * o fluxo atual não está orquestrado no domínio `trainer`;
  * não está definido se o endpoint atual `POST /trainer/my-pokemon` deve continuar existindo.

---

# 3. Comportamento Esperado

* O treinador deve conseguir tentar capturar Pokémon selvagem durante a battle-session.
* O sistema deve consumir pokebolas no fluxo orquestrado de captura.
* O sistema deve validar elegibilidade de captura comparando `trainer.capture_rate` com `pokemon.capture_rate`.
* O treinador só pode tentar capturar o Pokémon quando `trainer.capture_rate >= pokemon.capture_rate`.
* O sistema deve calcular chance de captura.
* Quanto menor o HP do Pokémon selvagem, maior deve ser a chance de captura.
* A fórmula pode ser definida no proposal, mas deve seguir diretrizes claras e ser consistente.
* O sistema deve impedir captura quando a battle-session estiver finalizada.
* O sistema deve impedir captura sem pokebolas.
* O sistema deve impedir captura quando `trainer.capture_rate < pokemon.capture_rate`.
* O sistema deve impedir múltiplas capturas bem-sucedidas na mesma battle-session.
* O sistema deve rejeitar tentativas repetidas após captura já concluída.
* O fluxo de captura deve ser orquestrado em `trainer/service.py`.
* O serviço de `trainer` deve acionar `my-pokemon/service.py` para criar o Pokémon capturado.
* O serviço de `trainer` deve acionar `pokedex/service.py` para registrar a descoberta do Pokémon para o treinador atual.
* O sistema deve criar um `my-pokemon` apenas após captura bem-sucedida.
* O sistema não deve criar `my-pokemon` em captura falha.
* O sistema não deve atualizar a pokedex em captura falha.
* O sistema deve copiar atributos relevantes do Pokémon selvagem.
* O sistema deve selecionar 4 movimentos iniciais segundo as regras já existentes do domínio.
* O sistema deve sincronizar a pokedex apenas para o treinador atual.
* O sistema deve encerrar a battle-session após captura bem-sucedida com status `CAPTURED`.
* O sistema deve manter a battle-session ativa após captura falha e retornar o estado atualizado da batalha.
* O sistema deve retornar payload normalizado para o frontend.

---

# 4. Escopo

## Dentro do escopo

* Sistema de captura em batalha
* Consumo de pokebolas
* Validação de elegibilidade por `capture_rate`
* Chance de captura
* Orquestração do caso de uso no domínio `trainer`
* Reutilização do serviço de `my-pokemon`
* Reutilização do serviço de `pokedex`
* Conversão do Pokémon selvagem em `my-pokemon`
* Integração com battle-session
* Integração com pokedex
* Integração com trainer-party
* Encerramento da batalha por captura
* Logs de captura
* Eventos normalizados
* Avaliação do futuro do endpoint `POST /trainer/my-pokemon`
* Estrutura futura para XP e evolução, sem implementação funcional nesta proposta

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
* Implementação de XP
* Implementação de level up
* Implementação de evolução

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
**Então:** uma pokebola deve ser consumida pelo fluxo orquestrado de captura

---

## Regra 3 — Sem pokebolas

**Dado que:** o treinador não possui pokebolas  
**Quando:** tentar capturar um Pokémon  
**Então:** o sistema deve impedir a captura imediatamente

---

## Regra 4 — Chance de captura

**Dado que:** o treinador tentou capturar um Pokémon  
**Quando:** a captura for processada  
**Então:** a chance deve considerar o HP atual do Pokémon selvagem

---

## Regra 5 — Elegibilidade por capture_rate

**Dado que:** o treinador tentou capturar um Pokémon  
**Quando:** `trainer.capture_rate` for menor que `pokemon.capture_rate`  
**Então:** o sistema deve impedir a captura

---

## Regra 6 — Captura bem-sucedida

**Dado que:** a captura foi bem-sucedida  
**Quando:** o processamento finalizar  
**Então:** o sistema deve criar um `my-pokemon`

---

## Regra 7 — Sincronização da pokedex

**Dado que:** o Pokémon foi capturado  
**Quando:** a captura for persistida  
**Então:** a pokedex do treinador deve marcar o Pokémon como descoberto

---

## Regra 8 — Captura falha

**Dado que:** a captura falhou  
**Quando:** o processamento finalizar  
**Então:** o sistema não deve criar `my-pokemon`, não deve atualizar a pokedex e deve manter a battle-session ativa

---

## Regra 9 — Encerramento da batalha por captura

**Dado que:** o Pokémon foi capturado  
**Quando:** a captura finalizar  
**Então:** a battle-session deve ser encerrada com status `CAPTURED`

---

## Regra 10 — Apenas um sucesso por battle-session

**Dado que:** a battle-session já teve captura concluída  
**Quando:** uma nova tentativa for enviada  
**Então:** o sistema deve rejeitar a operação

---

## Regra 11 — Orquestração no domínio trainer

**Dado que:** a captura é um caso de uso composto  
**Quando:** a operação for executada  
**Então:** `trainer/service.py` deve orquestrar o fluxo sem duplicar regras internas de `my-pokemon` e `pokedex`

---

# 6. Escopo Técnico

## 6.1 API

Toda regra de negócio deve estar na API.
O frontend deve ser apenas consumidor.

### Funcionalidades

* endpoint de captura no domínio `trainer`
* integração com battle-session
* integração com `my-pokemon/service.py`
* integração com `pokedex/service.py`
* avaliação da substituição ou descontinuação do `POST /trainer/my-pokemon`

### Fluxos

1. treinador entra em battle-session
2. treinador utiliza pokebola
3. endpoint de captura no domínio `trainer` valida a sessão, o inventário e a elegibilidade por `capture_rate`
4. `trainer/service.py` rejeita a operação quando `trainer.capture_rate < pokemon.capture_rate`
5. `trainer/service.py` calcula a tentativa de captura quando o Pokémon for elegível
6. em caso de sucesso, `trainer/service.py` aciona `my-pokemon/service.py`
7. em caso de sucesso, `trainer/service.py` aciona `pokedex/service.py`
8. o sistema encerra a battle-session com status `CAPTURED`
9. o sistema retorna payload normalizado

### Observações arquiteturais obrigatórias

* `trainer/service.py` deve ser o orquestrador do caso de uso
* `my-pokemon` continua responsável pelas regras de criação do Pokémon do treinador
* `pokedex` continua responsável pelas regras de descoberta
* a proposta deve avaliar se o `POST /trainer/my-pokemon` atual deve ser removido, substituído ou absorvido para evitar sobreposição de responsabilidades
* não criar fluxo paralelo de captura fora do domínio `trainer`

---

## 6.1.1 Integrações externas

Não consumir PokéAPI diretamente nesta proposta.

Utilizar apenas dados persistidos localmente.

---

## 6.1.2 Enriquecimento de dados

Não assumir automaticamente a criação de uma nova entidade `pokemon_capture`.

A proposta deve avaliar primeiro o reaproveitamento das estruturas já existentes, especialmente:

* `battle_log`
* `battle_session`
* `my_pokemon`
* `pokedex`

### trainer

Atualizar:

* `pokeball_quantity`
* `capture_rate`, apenas como dado de validação do fluxo, sem duplicação de regra fora do domínio dono

---

### my_pokemon

Criar apenas em captura bem-sucedida, reaproveitando o domínio existente.

Persistir ou derivar:

* `trainer_id`
* `pokemon_id`
* atributos necessários do Pokémon capturado
* movimentos iniciais segundo regra existente do domínio

---

### pokedex

Atualizar apenas em captura bem-sucedida:

* descoberta do Pokémon para o treinador atual

---

### battle_log

Registrar eventos:

* `CAPTURE_ATTEMPT`
* `CAPTURE_SUCCESS`
* `CAPTURE_FAILED`

---

## 6.1.3 Cache

Utilizar o padrão existente do projeto.

Sugestão de chaves:

```txt
trainer:captures:{trainer_id}
trainer:home:{trainer_id}
trainer:pokedex:{trainer_id}
trainer:party:{trainer_id}
trainer:battle-session:{trainer_id}
```

Regras:

* invalidar cache após captura bem-sucedida
* invalidar cache após atualização da pokedex
* invalidar cache após alteração da party ou do estado agregado do treinador
* invalidar cache da battle-session quando houver mudança de status

---

## 6.1.4 Consistência e performance

* evitar duplicação de capturas
* evitar múltiplas capturas bem-sucedidas da mesma sessão
* manter integridade referencial
* utilizar soft delete quando houver novas tabelas
* evitar N+1 queries
* reutilizar estruturas existentes
* manter separação clara entre battle, trainer, capture, my-pokemon e pokedex
* impedir estado parcial entre criação de `my-pokemon` e atualização da pokedex

---

## 6.1.5 Contrato da API

A API deve:

* retornar dados normalizados;
* evitar transformação no frontend;
* manter consistência de naming;
* diferenciar claramente captura bem-sucedida de captura falha;
* diferenciar falha por inelegibilidade de `capture_rate` e falha por chance de captura;
* retornar o estado atualizado da battle-session após falha;
* retornar o resultado agregado da captura após sucesso.

Exemplo conceitual:

```json
{
  "capture_result": {
    "success": true,
    "battle_status": "CAPTURED",
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
* estado atualizado da batalha quando a captura falhar

---

## 6.2.4 Interações

* utilizar pokebola
* tentar capturar Pokémon
* visualizar resultado
* visualizar feedback de falha sem sair da batalha
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
* Não duplicar regras internas de `my-pokemon` e `pokedex`
* O caso de uso de captura deve ser orquestrado em `trainer/service.py`
* Não criar services gigantes
* Toda nova tabela criada no banco de dados deve suportar soft delete
* Não realizar exclusão física de registros, salvo se já existir padrão explícito no projeto
* Usar o padrão existente do projeto para soft delete, preferencialmente com `deleted_at`
* Consultas devem ignorar registros com soft delete ativo, salvo quando explicitamente necessário

---

# 8. Critérios de Aceite

* [ ] captura funcionando
* [ ] consumo de pokebolas funcionando
* [ ] validação de elegibilidade por `trainer.capture_rate` e `pokemon.capture_rate` funcionando
* [ ] cálculo de chance funcionando
* [ ] endpoint de captura implementado no domínio `trainer`
* [ ] `trainer/service.py` orquestra o caso de uso completo
* [ ] criação de `my-pokemon` funcionando apenas em sucesso
* [ ] sincronização da pokedex funcionando apenas em sucesso
* [ ] captura falha mantém a battle-session ativa
* [ ] captura é rejeitada quando `trainer.capture_rate < pokemon.capture_rate`
* [ ] captura bem-sucedida encerra a battle-session com status `CAPTURED`
* [ ] múltiplas capturas bem-sucedidas na mesma sessão são impedidas
* [ ] tentativas repetidas após sucesso são rejeitadas
* [ ] logs funcionando
* [ ] payload normalizado
* [ ] integração com battle-session funcionando
* [ ] integração com pokedex funcionando
* [ ] integração com party funcionando
* [ ] proposta avalia explicitamente o destino do `POST /trainer/my-pokemon`
* [ ] sem duplicação de lógica
* [ ] sem regressões
* [ ] cache funcionando
* [ ] services organizados corretamente
* [ ] sem services excessivamente grandes
* [ ] novas tabelas, se existirem, possuem suporte a soft delete
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
6. Validar cenário elegível por `capture_rate`
7. Validar captura bem-sucedida
8. Validar criação do `my-pokemon`
9. Validar atualização da pokedex do treinador
10. Validar encerramento da battle-session com status `CAPTURED`
11. Executar cenário com `trainer.capture_rate < pokemon.capture_rate`
12. Validar rejeição da captura por inelegibilidade
13. Executar cenário de falha de captura após elegibilidade
14. Validar ausência de criação de `my-pokemon` na falha
15. Validar ausência de atualização da pokedex na falha
16. Validar manutenção da battle-session ativa após falha
17. Validar rejeição de nova captura após sucesso

---

## Testes automatizados

* captura de Pokémon
* consumo de pokebola
* validação de elegibilidade por `capture_rate`
* cálculo de chance
* sincronização da pokedex
* criação de `my-pokemon`
* encerramento da battle-session
* rejeição de captura quando `trainer.capture_rate < pokemon.capture_rate`
* retorno do estado atualizado da batalha após falha
* rejeição de captura sem pokebola
* rejeição de captura com sessão finalizada
* rejeição de múltiplas capturas bem-sucedidas na mesma sessão
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
15. Avaliação explícita do destino do endpoint `POST /trainer/my-pokemon`
16. Estratégia futura para XP, evolução e progressão sem implementação nesta proposta
