## Why

O projeto já possui exploração, sessão de batalha ativa, `my-pokemon` e `pokedex`, mas ainda não fecha o ciclo principal de captura durante o encounter com Pokémon selvagem. Sem esse fluxo, pokebolas não têm papel funcional, a batalha não produz aquisição real de criaturas, e a Pokédex do treinador não é sincronizada automaticamente após uma captura bem-sucedida.

Além disso, o `capture_rate` do treinador hoje é estático. Isso reduz o senso de progressão do loop de captura e impede que o treinador evolua gradualmente sua capacidade de capturar Pokémons mais exigentes ao longo do tempo.

## What Changes

- Criar a capability de captura de Pokémon selvagem durante batalha, com endpoint autenticado em `POST /trainer/battle/capture`.
- Orquestrar o caso de uso de captura em `trainer/service.py`, reutilizando `my-pokemon/service.py` para materializar o Pokémon capturado e `pokedex/service.py` para marcar a descoberta do treinador.
- Introduzir validação obrigatória de elegibilidade por `capture_rate`, exigindo `trainer.capture_rate >= pokemon.capture_rate` para a tentativa de captura prosseguir.
- Fazer a tentativa de captura consumir pokebola mesmo quando a elegibilidade por `capture_rate` falhar, conforme regra de negócio definida para esta mudança.
- Fazer a captura elegível consumir o turno da batalha; quando a tentativa falhar por chance, a sessão deve continuar ativa e o Pokémon selvagem deve responder automaticamente no mesmo ciclo.
- Encerrar a battle session com status público `CAPTURED` quando a captura for bem-sucedida.
- Atualizar a Pokédex apenas em sucesso e apenas para o treinador atual.
- Evoluir o `capture_rate` do treinador após capturas bem-sucedidas usando uma fórmula contínua de progressão, com peso maior para espécie nova do que para captura repetida.
- Avaliar e executar a substituição, absorção ou descontinuação do endpoint `POST /trainer/my-pokemon` como entrypoint canônico de captura para evitar sobreposição de responsabilidades.
- Manter fora de escopo XP de Pokémon, evolução de Pokémon, level up de Pokémon, pokebolas especiais e mecânicas avançadas de captura.

## Capabilities

### New Capabilities
- `pokemon-capture-system`: Captura de Pokémon selvagem durante battle session, incluindo elegibilidade por `capture_rate`, consumo de pokebola, resolução de sucesso/falha, progressão do `capture_rate` do treinador e orquestração com `my-pokemon` e `pokedex`.

### Modified Capabilities
- `wild-pokemon-battle-session`: A battle session passa a expor ação autenticada de captura, novo status terminal `CAPTURED` e semântica de turno para tentativa de captura e resposta automática do Pokémon selvagem.
- `my-pokemon-management`: A criação de `my-pokemon` passa a incluir o fluxo orquestrado de captura em batalha como origem suportada, sem manter um entrypoint paralelo conflitante para esse caso de uso.
- `pokedex-management`: A descoberta da Pokédex do treinador passa a poder acontecer automaticamente após captura bem-sucedida, sem UI manual adicional.

## Impact

- API: mudanças em `machado-api/app/domain/trainer/`, `trainer/battle`, `trainer/my_pokemon`, `trainer/pokedex` e serviços orquestradores relacionados.
- Web: ajustes no fluxo de batalha para expor ação de captura, modal/animação/feedback, estado de falha e atualização de Home/Pokédex após sucesso.
- Banco e persistência: possíveis ajustes em logs, status de battle session, campos de progresso do treinador e estruturas associadas; a proposta deve evitar criar agregados redundantes de captura sem necessidade.
- Contratos: novo endpoint `POST /trainer/battle/capture`, novo status público `CAPTURED`, respostas normalizadas para sucesso, inelegibilidade e falha de captura.
- Progressão: atualização automática do `capture_rate` do treinador após capturas bem-sucedidas, com regra diferenciando espécie nova e repetida.
- Compatibilidade: revisão explícita do papel do `POST /trainer/my-pokemon` atual para evitar duplicidade de responsabilidade com o novo fluxo canônico.
