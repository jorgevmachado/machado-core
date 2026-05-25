## Context

O backend já possui `trainer`, `trainer/battle`, `trainer/my_pokemon`, `trainer/pokedex`, Home agregadora e sessão de batalha stateful para encontros selvagens. A captura agora precisa fechar um caso de uso transversal: validar elegibilidade do treinador, consumir pokebola, resolver sucesso ou falha dentro do ciclo da batalha, materializar o `my-pokemon` capturado e sincronizar a Pokédex do treinador sem duplicar responsabilidades entre domains.

Há também um ponto arquitetural já visível no código: o projeto expõe endpoints sob `/trainer/battle/*`, mas o usuário quer que a orquestração do caso de uso aconteça em `trainer/service.py`, deixando `battle`, `my_pokemon` e `pokedex` como domínios especializados responsáveis pelas suas próprias regras internas. O design precisa manter esse boundary sem criar um fluxo paralelo de captura.

## Goals / Non-Goals

**Goals:**
- Introduzir o fluxo canônico de captura em `POST /trainer/battle/capture`, alinhado ao namespace atual da batalha.
- Fazer `trainer/service.py` orquestrar a captura, reutilizando `BattleSessionService`, `MyPokemonService` e `PokedexService`.
- Validar a elegibilidade de captura com a regra `trainer.capture_rate >= pokemon.capture_rate`.
- Consumir pokebola em toda tentativa enviada ao endpoint de captura, inclusive quando a elegibilidade por `capture_rate` falhar, conforme decisão funcional já tomada.
- Fazer a tentativa elegível consumir o turno da batalha; quando falhar por chance, o Pokémon selvagem deve responder automaticamente no mesmo ciclo.
- Encerrar a sessão com status `CAPTURED` no sucesso e manter a sessão ativa no fracasso.
- Sincronizar a Pokédex apenas em sucesso e apenas para o treinador autenticado.
- Evoluir o `capture_rate` do treinador como parte da recompensa por captura bem-sucedida.
- Definir o destino do `POST /trainer/my-pokemon` para que ele deixe de ser o entrypoint canônico de captura em batalha.

**Non-Goals:**
- Implementar XP, level up ou evolução de Pokémon.
- Adicionar pokebolas especiais, modifiers raros, shiny, breeding ou captura em massa.
- Reescrever o engine de batalha além do necessário para suportar a ação `capture`.
- Recriar `my-pokemon` ou `pokedex` como novos agregados.
- Introduzir nova infraestrutura de UI fora do padrão já usado na tela/modal de batalha.

## Decisions

### 1. Manter o endpoint no namespace de batalha e mover a orquestração para `trainer/service.py`

O contrato público de captura será `POST /trainer/battle/capture`, preservando a consistência com `/trainer/battle/move`, `/switch` e `/flee`. A rota de batalha não será a dona do caso de uso; ela apenas delegará a um fluxo orquestrado no `TrainerService`, que coordenará:

- leitura e validação da battle session ativa;
- consumo de pokebola;
- validação de `capture_rate`;
- resolução da tentativa de captura;
- criação do `my-pokemon` em sucesso;
- descoberta da Pokédex em sucesso;
- atualização do status da battle session e invalidação de cache.

Alternativas consideradas:
- Colocar toda a captura dentro de `BattleSessionService`: rejeitado porque mistura engine de batalha com orquestração cross-domain de criação e descoberta.
- Expor `POST /trainer/capture`: rejeitado porque quebra a coerência do namespace de ações da batalha já existente.

### 2. Tratar `trainer.capture_rate` como gate de elegibilidade antes da chance probabilística

A tentativa de captura terá duas camadas:

1. gate de elegibilidade: `trainer.capture_rate >= pokemon.capture_rate`;
2. resolução probabilística: somente quando elegível, usando HP atual do Pokémon selvagem como fator obrigatório.

Se o treinador falhar no gate de elegibilidade, a captura não pode acontecer. Ainda assim, por decisão funcional do usuário, a pokebola enviada na tentativa será consumida.

Alternativas consideradas:
- Consumir pokebola apenas se o Pokémon fosse elegível: rejeitado por decisão explícita do usuário.
- Ignorar `capture_rate` e usar apenas chance baseada em HP: rejeitado porque perderia a progressão de elegibilidade do treinador.

### 3. Fazer a captura elegível participar do ciclo de turno da battle session

Quando a tentativa passa pela elegibilidade e segue para resolução probabilística, ela conta como ação do turno do treinador. O fluxo será:

1. consumir pokebola;
2. validar elegibilidade;
3. se inelegível, registrar falha por regra e retornar;
4. se elegível, incrementar turno e registrar `CAPTURE_ATTEMPT`;
5. se sucesso, criar `my-pokemon`, descobrir Pokédex, finalizar sessão com `CAPTURED`;
6. se falha, registrar `CAPTURE_FAILED`, manter sessão ativa e processar a resposta automática do Pokémon selvagem no mesmo ciclo.

Alternativas consideradas:
- Não consumir turno em captura falha: rejeitado porque tornaria captura um atalho sem custo tático.
- Fazer a resposta do selvagem em uma segunda requisição: rejeitado porque complica o contrato stateful já consolidado na batalha.

### 4. Evoluir o `capture_rate` do treinador com progressão contínua baseada em capturas bem-sucedidas

A progressão do treinador será parte do mesmo fluxo transacional da captura bem-sucedida. Em vez de marcos fixos como "a cada 5 capturas", o design adota uma progressão contínua por pontos acumulados de captura:

- captura de espécie nova para o treinador: `+2.0` pontos de progresso;
- captura repetida de espécie já descoberta: `+1.0` ponto de progresso.

O `capture_rate` efetivo do treinador será recalculado a partir de um valor base e dos pontos acumulados usando uma fórmula híbrida com crescimento contínuo e retorno desacelerado:

```txt
progress_points = soma acumulada dos pontos de captura bem-sucedida
capture_rate = min(
  255,
  base_capture_rate
  + floor(progress_points / 4)
  + floor(sqrt(progress_points) * 1.5)
)
```

Onde:

- `base_capture_rate` é o valor inicial do treinador no onboarding;
- `progress_points` só aumenta em capturas bem-sucedidas;
- a parcela linear garante progresso perceptível no médio e longo prazo;
- a parcela `sqrt()` acelera o início sem deixar o crescimento explodir cedo demais;
- o teto de `255` preserva o limite já adotado no domínio.

Exemplos esperados:

- 1 espécie nova capturada: `2` pontos, ganho efetivo de `+2`
- 5 espécies novas capturadas: `10` pontos, ganho efetivo de `+6`
- 10 espécies novas capturadas: `20` pontos, ganho efetivo de `+11`
- 20 espécies novas capturadas: `40` pontos, ganho efetivo de `+19`
- 20 espécies novas e 20 repetidas: `60` pontos, ganho efetivo de `+26`

Isso dá sensação de evolução no começo sem saltos exagerados e mantém progresso visível no médio e longo prazo, evitando que o treinador fique travado cedo demais.

Alternativas consideradas:
- Marcos fixos do tipo "a cada 5 capturas ganha +5": rejeitado por gerar progressão em degraus e feeling menos orgânico.
- Aumentar `capture_rate` a cada captura em ritmo linear: rejeitado por escalar rápido demais e dificultar balanceamento.
- Contar apenas espécies inéditas: rejeitado porque pune capturas repetidas mesmo quando ainda têm valor no loop de jogo.

### 5. Reutilizar os serviços existentes como owners das regras de domínio

O `TrainerService` deve chamar:

- `BattleSessionService` para carregar, mutar e finalizar a sessão;
- `MyPokemonService` para criar o Pokémon capturado usando o pipeline já existente de owned Pokémon;
- `PokedexService` para marcar a descoberta do Pokémon do treinador;
- serviços/cache já existentes para invalidar Home, Party e Pokédex.

Isso preserva ownership das regras:

- `battle` continua dona da sessão e dos turnos;
- `my-pokemon` continua dono da materialização do Pokémon do treinador;
- `pokedex` continua dona da descoberta;
- `trainer` coordena o caso de uso agregado.

Alternativas consideradas:
- Duplicar a criação de `my-pokemon` em `trainer/service.py`: rejeitado por violar o boundary do domínio.
- Atualizar a Pokédex diretamente no repositório da batalha: rejeitado por violar ownership e aumentar acoplamento.

### 6. Persistir progresso do treinador sem depender de recontagem pesada a cada captura

O domínio do treinador deve persistir os dados mínimos necessários para recalcular o `capture_rate` sem depender de varrer toda a coleção do treinador a cada nova captura. A abordagem preferida é adicionar ao `trainer` campos próprios para progressão, como:

- `base_capture_rate`
- `capture_progress_points`

O campo público `capture_rate` pode continuar existindo como valor efetivo persistido e atualizado pelo fluxo de progressão, desde que fique claro no modelo qual parte é base e qual parte é progresso acumulado.

Alternativas consideradas:
- Recontar todas as capturas a cada sucesso: rejeitado por custo desnecessário e maior chance de inconsistência.
- Derivar tudo apenas da Pokédex: rejeitado porque espécies inéditas e repetidas têm pesos diferentes, então a Pokédex sozinha não representa todo o progresso.

### 7. Não criar automaticamente uma nova entidade de captura, a menos que a implementação mostre necessidade real

O ponto central da mudança é o fluxo composto, não um novo agregado. O design prioriza reaproveitar:

- `battle_session` e seus status;
- `battle_log` para auditoria de tentativa, sucesso e falha;
- `my_pokemon` como resultado materializado da captura;
- `pokedex` como resultado de descoberta.

Se durante a implementação aparecer uma necessidade operacional clara para uma tabela específica de captura, isso deve ser tratado como decisão justificada, não como premissa.

Alternativas consideradas:
- Criar `pokemon_capture` desde o início: rejeitado por ampliar schema e complexidade sem evidência suficiente.

### 8. Tornar `CAPTURED` um status terminal público da battle session

O contrato público da batalha precisa diferenciar vitória por derrota do selvagem, fuga e captura concluída. A sessão bem-sucedida deve encerrar com `CAPTURED`, e esse status deve aparecer de forma consistente na API, BFF e frontend.

Alternativas consideradas:
- Reaproveitar `FINISHED`: rejeitado por ser ambíguo para UI, logs, analytics e extensões futuras de progressão.

### 9. Tratar o `POST /trainer/my-pokemon` atual como fluxo não-canônico para captura

O design precisa explicitar uma estratégia de compatibilidade:

- o fluxo canônico de captura em batalha passa a ser `POST /trainer/battle/capture`;
- o proposal deve avaliar se `POST /trainer/my-pokemon` será removido, limitado a usos administrativos/manuais ou absorvido por outro fluxo;
- a implementação não deve manter dois entrypoints equivalentes para captura em batalha.

Alternativas consideradas:
- Manter ambos como caminhos oficiais: rejeitado por duplicar responsabilidades e abrir divergência de regras.

## Risks / Trade-offs

- [Orquestração em `TrainerService` virar ponto de acoplamento excessivo] → Mitigar mantendo nele apenas coordenação e delegando regras puras aos serviços owners.
- [Consumo de pokebola em tentativa inelegível gerar estranhamento de produto] → Mitigar documentando explicitamente a regra no spec, nos testes e no contrato de erro/feedback.
- [Falha parcial entre criação de `my-pokemon` e descoberta da Pokédex] → Mitigar com transação única e commit centralizado no fluxo de captura.
- [Progressão do `capture_rate` ficar desbalanceada cedo ou travar no mid/late game] → Mitigar com fórmula híbrida (`linear + sqrt`), pesos simples e teto máximo de `255`.
- [Semântica de `pokemon.capture_rate` conflitar com o valor bruto vindo da PokéAPI] → Mitigar documentando claramente se o campo representa dificuldade de captura, requisito mínimo de treinador ou outro valor derivado antes de acoplar a progressão a ele.
- [Battle session perder consistência entre logs, turno e status] → Mitigar centralizando mutações de sessão e logs no boundary de batalha, mesmo quando a decisão parte do `TrainerService`.
- [Mudança no papel de `POST /trainer/my-pokemon` quebrar clientes existentes] → Mitigar com estratégia explícita de compatibilidade, documentação e eventual fase de depreciação.
- [Diferença entre falha por inelegibilidade e falha probabilística confundir o frontend] → Mitigar com payload normalizado contendo motivo da falha e estado final da sessão.

## Migration Plan

1. Adicionar o novo contrato `POST /trainer/battle/capture` e o status público `CAPTURED`.
2. Ajustar o engine/serviço de batalha para suportar logs e transições associados à tentativa de captura.
3. Adicionar os campos de progresso necessários no `trainer` e definir a atualização do `capture_rate` efetivo após captura bem-sucedida.
4. Implementar a orquestração de captura em `TrainerService` com transação única para `my-pokemon`, `pokedex` e progressão do treinador.
5. Atualizar BFF/web para consumir o novo endpoint, exibir feedback de sucesso, inelegibilidade e falha probabilística.
6. Revisar o `POST /trainer/my-pokemon` atual e aplicar a estratégia definida: restringir, descontinuar ou remover o papel de captura em batalha.

## Open Questions

- A fórmula exata da chance de captura após a elegibilidade continua propositalmente em aberto, mas deverá ser explicitada durante a implementação com base nas diretrizes do prompt.
- A resposta de API para falha por inelegibilidade deve deixar claro se a battle session permanece no mesmo turno ou se o consumo de pokebola sem avanço de turno é a regra final; o prompt hoje fixa apenas o consumo da pokebola, não a semântica completa do turno nesse ramo.
