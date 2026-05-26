## 1. Banco e modelos

- [x] 1.1 Criar os models `pokemon_center_healing` e `healing_log` com relacionamentos, timestamps e `deleted_at` seguindo o padrão do projeto
- [x] 1.2 Criar e revisar a migration das novas tabelas, FKs, enums e índices necessários para histórico por treinador e por operação
- [x] 1.3 Ajustar schemas e tipos compartilhados necessários para representar summary de healing, histórico item a item e bloco `last_healing` da Home

## 2. API de Centro Pokémon

- [x] 2.1 Criar o subdomínio `trainer/pokemon_center` com `schema.py`, `repository.py`, `business.py`, `service.py` e `route.py`
- [x] 2.2 Implementar a validação de pré-condições do healing: autenticação, existência de party ativa e bloqueio por battle session ativa
- [x] 2.3 Implementar a restauração atômica da party inteira com revive de Pokémon desmaiados, recuperação total de HP e recuperação total de PP
- [x] 2.4 Persistir o resumo da operação em `pokemon_center_healing` e os registros item a item em `healing_log`
- [x] 2.5 Expor os endpoints autenticados de healing e histórico com payloads normalizados e paginação compatível com o padrão do projeto

## 3. Integrações transversais

- [x] 3.1 Integrar o fluxo de healing ao `TrainerService` e aos services owners de party, `my-pokemon` e battle sem violar boundaries de repository
- [x] 3.2 Atualizar o agregador/contrato da Home para incluir `last_healing` e refletir imediatamente HP e PP restaurados da party
- [x] 3.3 Implementar a invalidação de cache para Home, party, `my-pokemon` e healing-history somente após commit bem-sucedido

## 4. Web e BFF

- [x] 4.1 Adicionar route handlers BFF para executar healing e listar histórico do Centro Pokémon usando o padrão autenticado existente
- [x] 4.2 Criar a tela protegida do Centro Pokémon com estado da party, ação de healing, feedback visual e histórico item a item
- [x] 4.3 Atualizar os tipos e componentes da Home para consumir e exibir o bloco explícito do último evento de cura
- [x] 4.4 Garantir o tratamento de erro normalizado para bloqueio por batalha ativa sem acoplar a UI ao payload completo da battle session

## 5. Testes e validação

- [x] 5.1 Cobrir a API com testes de sucesso, revive, restauração de PP, ausência de party, bloqueio por batalha ativa e histórico item a item
- [x] 5.2 Cobrir BFF/web com testes dos novos contratos, renderização do Centro Pokémon, feedback de healing e destaque de `last_healing` na Home
- [x] 5.3 Executar lint e testes relevantes em `machado-api` e `machado-web`, corrigindo regressões antes da fase de apply
