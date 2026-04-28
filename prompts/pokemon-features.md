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

Implementar Listagem e Detalhamento de pokémon.

---

## 2. Contexto Atual

Não existe listagem e nem detalhamento de pokémon.

---

## 3. Comportamento Esperado

Na página `pokemon` autenticado, o sistema deve apresentar uma `listagem de pokémon` Com filtros e paginação. Com ao pção de clicar em um dos cards da listagem e assim redirecionar para a página de `Detalhes de pokémon`.

### 3.1. `Listagem de pokémon`

Ela deve ter um mecanismo de cache com o redis, com um tempo maior do que 2 horas, dado que esses dados são usados com pouquissima frequencia. Deve haver um cache com a chave `pokemon:meta` onde vai guardar o total de pokémons na base de dados e o total de pokemons de uma api externa (Toda vez que ao buscar e não tiver no cache, deve buscar na api externa o total de itens e na base de dados o total de itens e salvar no cache). E o outro cache vai guardar os dados da listagem podendo ser paginada ou sómente uma lista.

#### 3.1.1. Base de dados vazia

Na primeira vez que ele for acionado, não haverá dados na base de dados sendo assim, será acionada uma api externa
`https://pokeapi.co/api/v2/pokemon?offset=0&limit=1350` onde ela vai ter um retorno similar a esse:

```json
{
  "count": 1350,
 "next": null,
 "previous": null,
  "results: [
 {
   "name": "bulbasaur",
   "url": "https://pokeapi.co/api/v2/pokemon/1/"
  },
  {
   "name": "ivysaur",
   "url": "https://pokeapi.co/api/v2/pokemon/2/"
  },
  {
   "name": "venusaur",
   "url": "https://pokeapi.co/api/v2/pokemon/3/"
  },
  ]
}
```

A partir dos dados acima, o sistema vai:

- Salvar o atributo `name`.
- Salvar o atributo `url`.
- Chamar a função `ensure_order_number` de `from app.shared.utils.number import ensure_order_number`  passando a `url` e assim com o retorno salvar o atributo `order`.
- Chamar a função `ensure_external_image` de `from app.shared.utils.image import ensure_external_image` passando o atributo `order` e assim com o retorno salvar o `external_image`.
- Salvar o atributo status como `INCOMPLETE`.
- Salvar o atributo `hp` com o valor 0.
- Salvar o atributo `image` com o valor `None`.
- Salvar o atributo `speed` com o valor 0.
- Salvar o atributo `height` com o valor 0.
- Salvar o atributo `weight` com o valor 0.
- Salvar o atributo `attack` com o valor 0.
- Salvar o atributo `defense` com o valor 0.
- Salvar o atributo `habitat` com o valor 0.
- Salvar o atributo `is_baby` com o valor `false`.
- Salvar o atributo `shape_url` com o valor `None`.
- Salvar o atributo `shape_name` com o valor `None`.
- Salvar o atributo `is_mythical` com o valor `false`.
- Salvar o atributo `gender_rate` com o valor 0.
- Salvar o atributo `is_legendary` com o valor `false`.
- Salvar o atributo `capture_rate` com o valor 0.
- Salvar o atributo `hatch_counter` com o valor 0.
- Salvar o atributo `base_happiness` com o valor 0.
- Salvar o atributo `special_attack` com o valor 0.
- Salvar o atributo `base_experience` com o valor 0.
- Salvar o atributo `special_defense` com o valor 0.
- Salvar o atributo `evolution_chain` com o valor `None`.
- Salvar o atributo `evolves_from_species` com o valor `None`.
- Salvar o atributo `has_gender_differences` com o valor `false`.
- Salvar o atributo `growth_rate_id` com o valor `None`.
- Salvar o atributo `created_at` com a data atual.
- Salvar o atributo `updated_at` com o valor `None`.
- Salvar o atributo `deleted_at` com o valor `None`.
- Com todos os relascionamentos vazios.

Ao finalizar irá salvar na tabela `pokemon` item por item.

#### 3.1.2 Base de dados incompleta

Caso ao buscar os dados perceber que existem dados na base de dados, mas o total está diferente do total mínimo do cache de `pokemon:meta`, então irá acionar a api externa novamente e só vai adicionar na base de dados os dados que não tiverem na base de dados atualizará o cache de `pokemon` e `pokemon:meta` e retornara a lista ou lista paginada.  

#### 3.1.3 Base de dados Completa

Caso ao buscar os dados e perceber que o total bate com o cache `pokemon:meta` então validará com o cache e retornara a lista ou lista paginada.

#### 3.1.4 Filtros

A página deve ser capaz de filtrar por:

- Nome
- Tipo: Será um autocomplete, na qual vai puxar da lista de `PokemonType` tabela de relascionamento todos os tipos, caso ela esteja vazia, não deve apresentar o filtro.
- Ordem
- Status

### 3.2 `Detalhes de pokémon`

Ao detalhar o pokémon pela primeira vez ou ele tiver com o status `INCOMPLETE`, a aplicação deve acionar um fluxo para completar o pokémon.

A paǵina deve contér:

- Header
- - Imagem do Pokémon Grande
- - Número de ordem do pokémon
- - Pequena descrição sobre o pokémon
- Informações do card
- - Altura do pokémon
- - Peso do pokémon
- - Total de pontos necessários para a captura do pokémon
- Habilidades
- - Deve apresentar no formato <Badge/> uma lista de habilidades.
- Movimentos
- - Deve apresentar no formato <Badge/> uma lista de Movimentos, na qual vai apresentar os dois primeiros com um botão ver mais, e toda vez que clicar irá apresentando mais 2.
- Tipos
- - Os tipos vêm como objetos:

```ts
{
  name: string
  text_color: string
  background_color: string
}
```

Render using:

```tsx
<Badge
  style={{
    color: type.text_color,
    backgroundColor: type.background_color
  }}
>
  {type.name}
</Badge>
```

- Fraco contra os tipos.
- - Mesma estrutura dos tipos
- Forte contra os tipos.
- - Mesma estrutura dos tipos
- Estastisticas
- - HP
- - Ataque
- - Defesa
- - Velocidade
- - Ataque Especial
- - Defesa Especial
- - Construir com Barras de progresso horizontais e indicadores visuais claros.
- Evoluções (Timeline)
- - Criar um timeline horizontal com os Highlight das evoluções do Pokemon (examplo: `Bulbasaur → Ivysaur → Venusaur`)

#### 3.2.1 Completar Pokémon

Quando o pokémon estiver com o status `INCOMPLETE, a aplicação deve acionar as seguintes apis externas:

`https://pokeapi.co/api/v2/pokemon/bulbasaur` que vai ter um retorno similar a esse:

```json
{
 "abilities": [
  {
   "ability": {
    "name": "overgrow",
    "url": "https://pokeapi.co/api/v2/ability/65/"
   },
   "is_hidden": false,
   "slot": 1
  },
  {
   "ability": {
    "name": "chlorophyll",
    "url": "https://pokeapi.co/api/v2/ability/34/"
   },
   "is_hidden": true,
   "slot": 3
  }
 ],
 "base_experience": 64,
 "cries": {
  "latest": "https://raw.githubusercontent.com/PokeAPI/cries/main/cries/pokemon/latest/1.ogg",
  "legacy": "https://raw.githubusercontent.com/PokeAPI/cries/main/cries/pokemon/legacy/1.ogg"
 },
 "forms": [
  {
   "name": "bulbasaur",
   "url": "https://pokeapi.co/api/v2/pokemon-form/1/"
  }
 ],
 "game_indices": [
  {
   "game_index": 153,
   "version": {
    "name": "red",
    "url": "https://pokeapi.co/api/v2/version/1/"
   }
  },
  {
   "game_index": 153,
   "version": {
    "name": "blue",
    "url": "https://pokeapi.co/api/v2/version/2/"
   }
  },
  {
   "game_index": 153,
   "version": {
    "name": "yellow",
    "url": "https://pokeapi.co/api/v2/version/3/"
   }
  },
  {
   "game_index": 1,
   "version": {
    "name": "gold",
    "url": "https://pokeapi.co/api/v2/version/4/"
   }
  },
  {
   "game_index": 1,
   "version": {
    "name": "silver",
    "url": "https://pokeapi.co/api/v2/version/5/"
   }
  },
  {
   "game_index": 1,
   "version": {
    "name": "crystal",
    "url": "https://pokeapi.co/api/v2/version/6/"
   }
  },
  {
   "game_index": 1,
   "version": {
    "name": "ruby",
    "url": "https://pokeapi.co/api/v2/version/7/"
   }
  },
  {
   "game_index": 1,
   "version": {
    "name": "sapphire",
    "url": "https://pokeapi.co/api/v2/version/8/"
   }
  },
  {
   "game_index": 1,
   "version": {
    "name": "emerald",
    "url": "https://pokeapi.co/api/v2/version/9/"
   }
  },
  {
   "game_index": 1,
   "version": {
    "name": "firered",
    "url": "https://pokeapi.co/api/v2/version/10/"
   }
  },
  {
   "game_index": 1,
   "version": {
    "name": "leafgreen",
    "url": "https://pokeapi.co/api/v2/version/11/"
   }
  },
  {
   "game_index": 1,
   "version": {
    "name": "diamond",
    "url": "https://pokeapi.co/api/v2/version/12/"
   }
  },
  {
   "game_index": 1,
   "version": {
    "name": "pearl",
    "url": "https://pokeapi.co/api/v2/version/13/"
   }
  },
  {
   "game_index": 1,
   "version": {
    "name": "platinum",
    "url": "https://pokeapi.co/api/v2/version/14/"
   }
  },
  {
   "game_index": 1,
   "version": {
    "name": "heartgold",
    "url": "https://pokeapi.co/api/v2/version/15/"
   }
  },
  {
   "game_index": 1,
   "version": {
    "name": "soulsilver",
    "url": "https://pokeapi.co/api/v2/version/16/"
   }
  },
  {
   "game_index": 1,
   "version": {
    "name": "black",
    "url": "https://pokeapi.co/api/v2/version/17/"
   }
  },
  {
   "game_index": 1,
   "version": {
    "name": "white",
    "url": "https://pokeapi.co/api/v2/version/18/"
   }
  },
  {
   "game_index": 1,
   "version": {
    "name": "black-2",
    "url": "https://pokeapi.co/api/v2/version/21/"
   }
  },
  {
   "game_index": 1,
   "version": {
    "name": "white-2",
    "url": "https://pokeapi.co/api/v2/version/22/"
   }
  }
 ],
 "height": 7,
 "held_items": [],
 "id": 1,
 "is_default": true,
 "location_area_encounters": "https://pokeapi.co/api/v2/pokemon/1/encounters",
 "moves": [
  {
   "move": {
    "name": "razor-wind",
    "url": "https://pokeapi.co/api/v2/move/13/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "swords-dance",
    "url": "https://pokeapi.co/api/v2/move/14/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "cut",
    "url": "https://pokeapi.co/api/v2/move/15/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "bind",
    "url": "https://pokeapi.co/api/v2/move/20/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "vine-whip",
    "url": "https://pokeapi.co/api/v2/move/22/"
   },
   "version_group_details": [
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 10,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 10,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 10,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 10,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 10,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 9,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 9,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 9,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 9,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 10,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 10,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 9,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 9,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 9,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 9,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 9,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 5,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 3,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 3,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 3,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "headbutt",
    "url": "https://pokeapi.co/api/v2/move/29/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "tackle",
    "url": "https://pokeapi.co/api/v2/move/33/"
   },
   "version_group_details": [
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "body-slam",
    "url": "https://pokeapi.co/api/v2/move/34/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "take-down",
    "url": "https://pokeapi.co/api/v2/move/36/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 18,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 21,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 21,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 21,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "double-edge",
    "url": "https://pokeapi.co/api/v2/move/38/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 27,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 27,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 27,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 27,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 27,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 27,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 27,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 27,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 27,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 32,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 33,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 33,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "growl",
    "url": "https://pokeapi.co/api/v2/move/45/"
   },
   "version_group_details": [
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 4,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 4,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 4,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 4,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 4,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 3,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 3,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 3,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 3,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 4,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 4,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 3,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 3,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 3,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 3,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 3,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 1,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "strength",
    "url": "https://pokeapi.co/api/v2/move/70/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "mega-drain",
    "url": "https://pokeapi.co/api/v2/move/72/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "leech-seed",
    "url": "https://pokeapi.co/api/v2/move/73/"
   },
   "version_group_details": [
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 9,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 9,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 9,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 9,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 7,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "growth",
    "url": "https://pokeapi.co/api/v2/move/74/"
   },
   "version_group_details": [
    {
     "level_learned_at": 34,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 34,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 32,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 32,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 32,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 32,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 32,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 25,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 25,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 25,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 25,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 32,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 32,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 25,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 25,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 25,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 25,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 25,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 27,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 6,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 6,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 6,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 34,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 34,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "razor-leaf",
    "url": "https://pokeapi.co/api/v2/move/75/"
   },
   "version_group_details": [
    {
     "level_learned_at": 27,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 27,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 20,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 20,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 20,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 20,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 20,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 19,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 19,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 19,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 19,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 20,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 20,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 19,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 19,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 19,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 19,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 19,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 23,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 12,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 12,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 12,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 27,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 27,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "solar-beam",
    "url": "https://pokeapi.co/api/v2/move/76/"
   },
   "version_group_details": [
    {
     "level_learned_at": 48,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 48,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 46,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 46,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 46,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 46,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 46,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 46,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 46,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 36,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 36,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 36,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 48,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 48,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "poison-powder",
    "url": "https://pokeapi.co/api/v2/move/77/"
   },
   "version_group_details": [
    {
     "level_learned_at": 20,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 20,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 14,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 1,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 20,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 20,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "sleep-powder",
    "url": "https://pokeapi.co/api/v2/move/79/"
   },
   "version_group_details": [
    {
     "level_learned_at": 41,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 41,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 13,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 14,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 15,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": 2,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 41,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 41,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "petal-dance",
    "url": "https://pokeapi.co/api/v2/move/80/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "string-shot",
    "url": "https://pokeapi.co/api/v2/move/81/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "toxic",
    "url": "https://pokeapi.co/api/v2/move/92/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "rage",
    "url": "https://pokeapi.co/api/v2/move/99/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "mimic",
    "url": "https://pokeapi.co/api/v2/move/102/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "double-team",
    "url": "https://pokeapi.co/api/v2/move/104/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "defense-curl",
    "url": "https://pokeapi.co/api/v2/move/111/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "light-screen",
    "url": "https://pokeapi.co/api/v2/move/113/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "reflect",
    "url": "https://pokeapi.co/api/v2/move/115/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "bide",
    "url": "https://pokeapi.co/api/v2/move/117/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "sludge",
    "url": "https://pokeapi.co/api/v2/move/124/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "skull-bash",
    "url": "https://pokeapi.co/api/v2/move/130/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "amnesia",
    "url": "https://pokeapi.co/api/v2/move/133/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "flash",
    "url": "https://pokeapi.co/api/v2/move/148/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "rest",
    "url": "https://pokeapi.co/api/v2/move/156/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "substitute",
    "url": "https://pokeapi.co/api/v2/move/164/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-blue",
      "url": "https://pokeapi.co/api/v2/version-group/1/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "yellow",
      "url": "https://pokeapi.co/api/v2/version-group/2/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "red-green-japan",
      "url": "https://pokeapi.co/api/v2/version-group/28/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "blue-japan",
      "url": "https://pokeapi.co/api/v2/version-group/29/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "snore",
    "url": "https://pokeapi.co/api/v2/move/173/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "curse",
    "url": "https://pokeapi.co/api/v2/move/174/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "protect",
    "url": "https://pokeapi.co/api/v2/move/182/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "sludge-bomb",
    "url": "https://pokeapi.co/api/v2/move/188/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "mud-slap",
    "url": "https://pokeapi.co/api/v2/move/189/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "outrage",
    "url": "https://pokeapi.co/api/v2/move/200/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "giga-drain",
    "url": "https://pokeapi.co/api/v2/move/202/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "endure",
    "url": "https://pokeapi.co/api/v2/move/203/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "charm",
    "url": "https://pokeapi.co/api/v2/move/204/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "false-swipe",
    "url": "https://pokeapi.co/api/v2/move/206/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "swagger",
    "url": "https://pokeapi.co/api/v2/move/207/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "fury-cutter",
    "url": "https://pokeapi.co/api/v2/move/210/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "attract",
    "url": "https://pokeapi.co/api/v2/move/213/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "sleep-talk",
    "url": "https://pokeapi.co/api/v2/move/214/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "return",
    "url": "https://pokeapi.co/api/v2/move/216/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "frustration",
    "url": "https://pokeapi.co/api/v2/move/218/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "safeguard",
    "url": "https://pokeapi.co/api/v2/move/219/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "sweet-scent",
    "url": "https://pokeapi.co/api/v2/move/230/"
   },
   "version_group_details": [
    {
     "level_learned_at": 25,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 25,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 25,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 25,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 25,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 21,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 21,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 21,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 21,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 25,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 25,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 21,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 21,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 21,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 21,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 21,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 24,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 24,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 24,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "synthesis",
    "url": "https://pokeapi.co/api/v2/move/235/"
   },
   "version_group_details": [
    {
     "level_learned_at": 39,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 39,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 39,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 39,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 39,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 33,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 33,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 33,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 33,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 39,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 39,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 33,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 33,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 33,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 33,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 33,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 27,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 27,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 27,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "hidden-power",
    "url": "https://pokeapi.co/api/v2/move/237/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "sunny-day",
    "url": "https://pokeapi.co/api/v2/move/241/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "gold-silver",
      "url": "https://pokeapi.co/api/v2/version-group/3/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "crystal",
      "url": "https://pokeapi.co/api/v2/version-group/4/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "rock-smash",
    "url": "https://pokeapi.co/api/v2/move/249/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "facade",
    "url": "https://pokeapi.co/api/v2/move/263/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "lets-go-pikachu-lets-go-eevee",
      "url": "https://pokeapi.co/api/v2/version-group/19/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "nature-power",
    "url": "https://pokeapi.co/api/v2/move/267/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "helping-hand",
    "url": "https://pokeapi.co/api/v2/move/270/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "ingrain",
    "url": "https://pokeapi.co/api/v2/move/275/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "knock-off",
    "url": "https://pokeapi.co/api/v2/move/282/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "secret-power",
    "url": "https://pokeapi.co/api/v2/move/290/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "weather-ball",
    "url": "https://pokeapi.co/api/v2/move/311/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "grass-whistle",
    "url": "https://pokeapi.co/api/v2/move/320/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "bullet-seed",
    "url": "https://pokeapi.co/api/v2/move/331/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "colosseum",
      "url": "https://pokeapi.co/api/v2/version-group/12/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "xd",
      "url": "https://pokeapi.co/api/v2/version-group/13/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "magical-leaf",
    "url": "https://pokeapi.co/api/v2/move/345/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ruby-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/5/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "emerald",
      "url": "https://pokeapi.co/api/v2/version-group/6/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "firered-leafgreen",
      "url": "https://pokeapi.co/api/v2/version-group/7/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "natural-gift",
    "url": "https://pokeapi.co/api/v2/move/363/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "worry-seed",
    "url": "https://pokeapi.co/api/v2/move/388/"
   },
   "version_group_details": [
    {
     "level_learned_at": 31,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 31,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 31,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 31,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 31,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 31,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 31,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 31,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 31,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 30,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 30,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 30,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "seed-bomb",
    "url": "https://pokeapi.co/api/v2/move/402/"
   },
   "version_group_details": [
    {
     "level_learned_at": 37,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 37,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 37,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 37,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 37,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 37,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 37,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 37,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 37,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 18,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 18,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 18,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "energy-ball",
    "url": "https://pokeapi.co/api/v2/move/412/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "leaf-storm",
    "url": "https://pokeapi.co/api/v2/move/437/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "power-whip",
    "url": "https://pokeapi.co/api/v2/move/438/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 33,
     "move_learn_method": {
      "name": "level-up",
      "url": "https://pokeapi.co/api/v2/move-learn-method/1/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "captivate",
    "url": "https://pokeapi.co/api/v2/move/445/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "grass-knot",
    "url": "https://pokeapi.co/api/v2/move/447/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "diamond-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/8/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "platinum",
      "url": "https://pokeapi.co/api/v2/version-group/9/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "heartgold-soulsilver",
      "url": "https://pokeapi.co/api/v2/version-group/10/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "venoshock",
    "url": "https://pokeapi.co/api/v2/move/474/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "acid-spray",
    "url": "https://pokeapi.co/api/v2/move/491/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "round",
    "url": "https://pokeapi.co/api/v2/move/496/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "echoed-voice",
    "url": "https://pokeapi.co/api/v2/move/497/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "grass-pledge",
    "url": "https://pokeapi.co/api/v2/move/520/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "black-white",
      "url": "https://pokeapi.co/api/v2/version-group/11/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "black-2-white-2",
      "url": "https://pokeapi.co/api/v2/version-group/14/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "work-up",
    "url": "https://pokeapi.co/api/v2/move/526/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "grassy-terrain",
    "url": "https://pokeapi.co/api/v2/move/580/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "egg",
      "url": "https://pokeapi.co/api/v2/move-learn-method/2/"
     },
     "order": null,
     "version_group": {
      "name": "brilliant-diamond-shining-pearl",
      "url": "https://pokeapi.co/api/v2/version-group/23/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "confide",
    "url": "https://pokeapi.co/api/v2/move/590/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "x-y",
      "url": "https://pokeapi.co/api/v2/version-group/15/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "omega-ruby-alpha-sapphire",
      "url": "https://pokeapi.co/api/v2/version-group/16/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "sun-moon",
      "url": "https://pokeapi.co/api/v2/version-group/17/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "ultra-sun-ultra-moon",
      "url": "https://pokeapi.co/api/v2/version-group/18/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "grassy-glide",
    "url": "https://pokeapi.co/api/v2/move/803/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "tutor",
      "url": "https://pokeapi.co/api/v2/move-learn-method/3/"
     },
     "order": null,
     "version_group": {
      "name": "sword-shield",
      "url": "https://pokeapi.co/api/v2/version-group/20/"
     }
    },
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "tera-blast",
    "url": "https://pokeapi.co/api/v2/move/851/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  },
  {
   "move": {
    "name": "trailblaze",
    "url": "https://pokeapi.co/api/v2/move/885/"
   },
   "version_group_details": [
    {
     "level_learned_at": 0,
     "move_learn_method": {
      "name": "machine",
      "url": "https://pokeapi.co/api/v2/move-learn-method/4/"
     },
     "order": null,
     "version_group": {
      "name": "scarlet-violet",
      "url": "https://pokeapi.co/api/v2/version-group/25/"
     }
    }
   ]
  }
 ],
 "name": "bulbasaur",
 "order": 1,
 "past_abilities": [
  {
   "abilities": [
    {
     "ability": null,
     "is_hidden": true,
     "slot": 3
    }
   ],
   "generation": {
    "name": "generation-iv",
    "url": "https://pokeapi.co/api/v2/generation/4/"
   }
  }
 ],
 "past_stats": [
  {
   "generation": {
    "name": "generation-i",
    "url": "https://pokeapi.co/api/v2/generation/1/"
   },
   "stats": [
    {
     "base_stat": 65,
     "effort": 0,
     "stat": {
      "name": "special",
      "url": "https://pokeapi.co/api/v2/stat/9/"
     }
    }
   ]
  }
 ],
 "past_types": [],
 "species": {
  "name": "bulbasaur",
  "url": "https://pokeapi.co/api/v2/pokemon-species/1/"
 },
 "sprites": {
  "back_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/back/1.png",
  "back_female": null,
  "back_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/back/shiny/1.png",
  "back_shiny_female": null,
  "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/1.png",
  "front_female": null,
  "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/shiny/1.png",
  "front_shiny_female": null,
  "other": {
   "dream_world": {
    "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/dream-world/1.svg",
    "front_female": null
   },
   "home": {
    "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/home/1.png",
    "front_female": null,
    "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/home/shiny/1.png",
    "front_shiny_female": null
   },
   "official-artwork": {
    "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/1.png",
    "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/shiny/1.png"
   },
   "showdown": {
    "back_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/showdown/back/1.gif",
    "back_female": null,
    "back_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/showdown/back/shiny/1.gif",
    "back_shiny_female": null,
    "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/showdown/1.gif",
    "front_female": null,
    "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/showdown/shiny/1.gif",
    "front_shiny_female": null
   }
  },
  "versions": {
   "generation-i": {
    "red-blue": {
     "back_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-i/red-blue/back/1.png",
     "back_gray": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-i/red-blue/back/gray/1.png",
     "back_transparent": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-i/red-blue/transparent/back/1.png",
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-i/red-blue/1.png",
     "front_gray": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-i/red-blue/gray/1.png",
     "front_transparent": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-i/red-blue/transparent/1.png"
    },
    "yellow": {
     "back_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-i/yellow/back/1.png",
     "back_gray": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-i/yellow/back/gray/1.png",
     "back_transparent": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-i/yellow/transparent/back/1.png",
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-i/yellow/1.png",
     "front_gray": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-i/yellow/gray/1.png",
     "front_transparent": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-i/yellow/transparent/1.png"
    }
   },
   "generation-ii": {
    "crystal": {
     "back_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/crystal/back/1.png",
     "back_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/crystal/back/shiny/1.png",
     "back_shiny_transparent": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/crystal/transparent/back/shiny/1.png",
     "back_transparent": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/crystal/transparent/back/1.png",
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/crystal/1.png",
     "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/crystal/shiny/1.png",
     "front_shiny_transparent": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/crystal/transparent/shiny/1.png",
     "front_transparent": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/crystal/transparent/1.png"
    },
    "gold": {
     "back_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/gold/back/1.png",
     "back_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/gold/back/shiny/1.png",
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/gold/1.png",
     "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/gold/shiny/1.png",
     "front_transparent": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/gold/transparent/1.png"
    },
    "silver": {
     "back_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/silver/back/1.png",
     "back_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/silver/back/shiny/1.png",
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/silver/1.png",
     "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/silver/shiny/1.png",
     "front_transparent": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ii/silver/transparent/1.png"
    }
   },
   "generation-iii": {
    "emerald": {
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/emerald/1.png",
     "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/emerald/shiny/1.png"
    },
    "firered-leafgreen": {
     "back_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/firered-leafgreen/back/1.png",
     "back_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/firered-leafgreen/back/shiny/1.png",
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/firered-leafgreen/1.png",
     "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/firered-leafgreen/shiny/1.png"
    },
    "ruby-sapphire": {
     "back_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/ruby-sapphire/back/1.png",
     "back_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/ruby-sapphire/back/shiny/1.png",
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/ruby-sapphire/1.png",
     "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/ruby-sapphire/shiny/1.png"
    }
   },
   "generation-iv": {
    "diamond-pearl": {
     "back_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iv/diamond-pearl/back/1.png",
     "back_female": null,
     "back_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iv/diamond-pearl/back/shiny/1.png",
     "back_shiny_female": null,
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iv/diamond-pearl/1.png",
     "front_female": null,
     "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iv/diamond-pearl/shiny/1.png",
     "front_shiny_female": null
    },
    "heartgold-soulsilver": {
     "back_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iv/heartgold-soulsilver/back/1.png",
     "back_female": null,
     "back_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iv/heartgold-soulsilver/back/shiny/1.png",
     "back_shiny_female": null,
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iv/heartgold-soulsilver/1.png",
     "front_female": null,
     "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iv/heartgold-soulsilver/shiny/1.png",
     "front_shiny_female": null
    },
    "platinum": {
     "back_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iv/platinum/back/1.png",
     "back_female": null,
     "back_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iv/platinum/back/shiny/1.png",
     "back_shiny_female": null,
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iv/platinum/1.png",
     "front_female": null,
     "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iv/platinum/shiny/1.png",
     "front_shiny_female": null
    }
   },
   "generation-ix": {
    "scarlet-violet": {
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-ix/scarlet-violet/1.png",
     "front_female": null
    }
   },
   "generation-v": {
    "black-white": {
     "animated": {
      "back_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-v/black-white/animated/back/1.gif",
      "back_female": null,
      "back_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-v/black-white/animated/back/shiny/1.gif",
      "back_shiny_female": null,
      "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-v/black-white/animated/1.gif",
      "front_female": null,
      "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-v/black-white/animated/shiny/1.gif",
      "front_shiny_female": null
     },
     "back_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-v/black-white/back/1.png",
     "back_female": null,
     "back_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-v/black-white/back/shiny/1.png",
     "back_shiny_female": null,
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-v/black-white/1.png",
     "front_female": null,
     "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-v/black-white/shiny/1.png",
     "front_shiny_female": null
    }
   },
   "generation-vi": {
    "omegaruby-alphasapphire": {
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-vi/omegaruby-alphasapphire/1.png",
     "front_female": null,
     "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-vi/omegaruby-alphasapphire/shiny/1.png",
     "front_shiny_female": null
    },
    "x-y": {
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-vi/x-y/1.png",
     "front_female": null,
     "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-vi/x-y/shiny/1.png",
     "front_shiny_female": null
    }
   },
   "generation-vii": {
    "icons": {
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-vii/icons/1.png",
     "front_female": null
    },
    "ultra-sun-ultra-moon": {
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-vii/ultra-sun-ultra-moon/1.png",
     "front_female": null,
     "front_shiny": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-vii/ultra-sun-ultra-moon/shiny/1.png",
     "front_shiny_female": null
    }
   },
   "generation-viii": {
    "brilliant-diamond-shining-pearl": {
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-viii/brilliant-diamond-shining-pearl/1.png",
     "front_female": null
    },
    "icons": {
     "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-viii/icons/1.png",
     "front_female": null
    }
   }
  }
 },
 "stats": [
  {
   "base_stat": 45,
   "effort": 0,
   "stat": {
    "name": "hp",
    "url": "https://pokeapi.co/api/v2/stat/1/"
   }
  },
  {
   "base_stat": 49,
   "effort": 0,
   "stat": {
    "name": "attack",
    "url": "https://pokeapi.co/api/v2/stat/2/"
   }
  },
  {
   "base_stat": 49,
   "effort": 0,
   "stat": {
    "name": "defense",
    "url": "https://pokeapi.co/api/v2/stat/3/"
   }
  },
  {
   "base_stat": 65,
   "effort": 1,
   "stat": {
    "name": "special-attack",
    "url": "https://pokeapi.co/api/v2/stat/4/"
   }
  },
  {
   "base_stat": 65,
   "effort": 0,
   "stat": {
    "name": "special-defense",
    "url": "https://pokeapi.co/api/v2/stat/5/"
   }
  },
  {
   "base_stat": 45,
   "effort": 0,
   "stat": {
    "name": "speed",
    "url": "https://pokeapi.co/api/v2/stat/6/"
   }
  }
 ],
 "types": [
  {
   "slot": 1,
   "type": {
    "name": "grass",
    "url": "https://pokeapi.co/api/v2/type/12/"
   }
  },
  {
   "slot": 2,
   "type": {
    "name": "poison",
    "url": "https://pokeapi.co/api/v2/type/4/"
   }
  }
 ],
 "weight": 69
}
```

`https://pokeapi.co/api/v2/pokemon-species/bulbasaur` que vai ter um retorno similar a esses:

```json
{
 "base_happiness": 70,
 "capture_rate": 45,
 "color": {
  "name": "green",
  "url": "https://pokeapi.co/api/v2/pokemon-color/5/"
 },
 "egg_groups": [
  {
   "name": "monster",
   "url": "https://pokeapi.co/api/v2/egg-group/1/"
  },
  {
   "name": "plant",
   "url": "https://pokeapi.co/api/v2/egg-group/7/"
  }
 ],
 "evolution_chain": {
  "url": "https://pokeapi.co/api/v2/evolution-chain/1/"
 },
 "evolves_from_species": null,
 "flavor_text_entries": [
  {
   "flavor_text": "A strange seed was\nplanted on its\nback at birth.\fThe plant sprouts\nand grows with\nthis POKéMON.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "red",
    "url": "https://pokeapi.co/api/v2/version/1/"
   }
  },
  {
   "flavor_text": "A strange seed was\nplanted on its\nback at birth.\fThe plant sprouts\nand grows with\nthis POKéMON.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "blue",
    "url": "https://pokeapi.co/api/v2/version/2/"
   }
  },
  {
   "flavor_text": "It can go for days\nwithout eating a\nsingle morsel.\fIn the bulb on\nits back, it\nstores energy.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "yellow",
    "url": "https://pokeapi.co/api/v2/version/3/"
   }
  },
  {
   "flavor_text": "The seed on its\nback is filled\nwith nutrients.\fThe seed grows\nsteadily larger as\nits body grows.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "gold",
    "url": "https://pokeapi.co/api/v2/version/4/"
   }
  },
  {
   "flavor_text": "It carries a seed\non its back right\nfrom birth. As it\fgrows older, the\nseed also grows\nlarger.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "silver",
    "url": "https://pokeapi.co/api/v2/version/5/"
   }
  },
  {
   "flavor_text": "While it is young,\nit uses the\nnutrients that are\fstored in the\nseeds on its back\nin order to grow.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "crystal",
    "url": "https://pokeapi.co/api/v2/version/6/"
   }
  },
  {
   "flavor_text": "BULBASAUR can be seen napping in\nbright sunlight.\nThere is a seed on its back.\fBy soaking up the sun’s rays, the seed\ngrows progressively larger.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "ruby",
    "url": "https://pokeapi.co/api/v2/version/7/"
   }
  },
  {
   "flavor_text": "BULBASAUR can be seen napping in\nbright sunlight.\nThere is a seed on its back.\fBy soaking up the sun’s rays, the seed\ngrows progressively larger.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "sapphire",
    "url": "https://pokeapi.co/api/v2/version/8/"
   }
  },
  {
   "flavor_text": "BULBASAUR can be seen napping in bright\nsunlight. There is a seed on its back.\nBy soaking up the sun’s rays, the seed\ngrows progressively larger.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "emerald",
    "url": "https://pokeapi.co/api/v2/version/9/"
   }
  },
  {
   "flavor_text": "There is a plant seed on its back right\nfrom the day this POKéMON is born.\nThe seed slowly grows larger.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "firered",
    "url": "https://pokeapi.co/api/v2/version/10/"
   }
  },
  {
   "flavor_text": "A strange seed was planted on its back at\nbirth. The plant sprouts and grows with\nthis POKéMON.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "leafgreen",
    "url": "https://pokeapi.co/api/v2/version/11/"
   }
  },
  {
   "flavor_text": "For some time after its birth, it\ngrows by gaining nourishment from\nthe seed on its back.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "diamond",
    "url": "https://pokeapi.co/api/v2/version/12/"
   }
  },
  {
   "flavor_text": "For some time after its birth, it\ngrows by gaining nourishment from\nthe seed on its back.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "pearl",
    "url": "https://pokeapi.co/api/v2/version/13/"
   }
  },
  {
   "flavor_text": "For some time after its birth, it\ngrows by gaining nourishment from\nthe seed on its back.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "platinum",
    "url": "https://pokeapi.co/api/v2/version/14/"
   }
  },
  {
   "flavor_text": "The seed on its back is filled\nwith nutrients. The seed grows\nsteadily larger as its body grows.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "heartgold",
    "url": "https://pokeapi.co/api/v2/version/15/"
   }
  },
  {
   "flavor_text": "It carries a seed on its back right\nfrom birth. As it grows older, the\nseed also grows larger.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "soulsilver",
    "url": "https://pokeapi.co/api/v2/version/16/"
   }
  },
  {
   "flavor_text": "Au matin de sa vie, la graine sur\nson dos lui fournit les éléments\ndont il a besoin pour grandir.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version": {
    "name": "black",
    "url": "https://pokeapi.co/api/v2/version/17/"
   }
  },
  {
   "flavor_text": "For some time after its birth, it\ngrows by gaining nourishment from\nthe seed on its back.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "black",
    "url": "https://pokeapi.co/api/v2/version/17/"
   }
  },
  {
   "flavor_text": "Au matin de sa vie, la graine sur\nson dos lui fournit les éléments\ndont il a besoin pour grandir.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version": {
    "name": "white",
    "url": "https://pokeapi.co/api/v2/version/18/"
   }
  },
  {
   "flavor_text": "For some time after its birth, it\ngrows by gaining nourishment from\nthe seed on its back.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "white",
    "url": "https://pokeapi.co/api/v2/version/18/"
   }
  },
  {
   "flavor_text": "For some time after its birth, it\ngrows by gaining nourishment from\nthe seed on its back.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "black-2",
    "url": "https://pokeapi.co/api/v2/version/21/"
   }
  },
  {
   "flavor_text": "For some time after its birth, it\ngrows by gaining nourishment from\nthe seed on its back.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "white-2",
    "url": "https://pokeapi.co/api/v2/version/22/"
   }
  },
  {
   "flavor_text": "うまれたときから　せなかに\nふしぎな　タネが　うえてあって\nからだと　ともに　そだつという。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version": {
    "name": "x",
    "url": "https://pokeapi.co/api/v2/version/23/"
   }
  },
  {
   "flavor_text": "태어났을 때부터 등에\n이상한 씨앗이 심어져 있으며\n몸과 함께 자란다고 한다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version": {
    "name": "x",
    "url": "https://pokeapi.co/api/v2/version/23/"
   }
  },
  {
   "flavor_text": "Il a une étrange graine plantée sur son dos.\nElle grandit avec lui depuis sa naissance.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version": {
    "name": "x",
    "url": "https://pokeapi.co/api/v2/version/23/"
   }
  },
  {
   "flavor_text": "Dieses Pokémon trägt von Geburt an einen Samen\nauf dem Rücken, der mit ihm keimt und wächst.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version": {
    "name": "x",
    "url": "https://pokeapi.co/api/v2/version/23/"
   }
  },
  {
   "flavor_text": "Una rara semilla le fue plantada en el lomo al nacer.\nLa planta brota y crece con este Pokémon.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version": {
    "name": "x",
    "url": "https://pokeapi.co/api/v2/version/23/"
   }
  },
  {
   "flavor_text": "Alla nascita gli è stato piantato sulla schiena un seme\nraro. La pianta sboccia e cresce con lui.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version": {
    "name": "x",
    "url": "https://pokeapi.co/api/v2/version/23/"
   }
  },
  {
   "flavor_text": "A strange seed was planted on its back at birth.\nThe plant sprouts and grows with this Pokémon.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "x",
    "url": "https://pokeapi.co/api/v2/version/23/"
   }
  },
  {
   "flavor_text": "生まれたときから　背中に\n不思議な　タネが　植えてあって\n体と　ともに　育つという。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version": {
    "name": "x",
    "url": "https://pokeapi.co/api/v2/version/23/"
   }
  },
  {
   "flavor_text": "うまれてから　しばらくの　あいだは\nせなかの　タネから　えいようを\nもらって　おおきく　そだつ。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version": {
    "name": "y",
    "url": "https://pokeapi.co/api/v2/version/24/"
   }
  },
  {
   "flavor_text": "태어나서부터 얼마 동안은\n등의 씨앗으로부터 영양을\n공급받아 크게 성장한다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version": {
    "name": "y",
    "url": "https://pokeapi.co/api/v2/version/24/"
   }
  },
  {
   "flavor_text": "Au matin de sa vie, la graine sur son dos lui fournit\nles éléments dont il a besoin pour grandir.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version": {
    "name": "y",
    "url": "https://pokeapi.co/api/v2/version/24/"
   }
  },
  {
   "flavor_text": "Nach der Geburt nimmt es für eine Weile Nährstoffe\nüber den Samen auf seinem Rücken auf.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version": {
    "name": "y",
    "url": "https://pokeapi.co/api/v2/version/24/"
   }
  },
  {
   "flavor_text": "Después de nacer, crece alimentándose de las\nsemillas de su lomo.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version": {
    "name": "y",
    "url": "https://pokeapi.co/api/v2/version/24/"
   }
  },
  {
   "flavor_text": "Dopo la nascita, cresce traendo nutrimento dal seme\npiantato sul suo dorso.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version": {
    "name": "y",
    "url": "https://pokeapi.co/api/v2/version/24/"
   }
  },
  {
   "flavor_text": "For some time after its birth, it grows by gaining\nnourishment from the seed on its back.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "y",
    "url": "https://pokeapi.co/api/v2/version/24/"
   }
  },
  {
   "flavor_text": "生まれてから　しばらくの　あいだは\n背中の　タネから　栄養を　もらって\n大きく　育つ。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version": {
    "name": "y",
    "url": "https://pokeapi.co/api/v2/version/24/"
   }
  },
  {
   "flavor_text": "ひなたで　ひるねを　する　すがたを　みかける。\nたいようの　ひかりを　いっぱい　あびることで\nせなかの　タネが　おおきく　そだつのだ。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version": {
    "name": "omega-ruby",
    "url": "https://pokeapi.co/api/v2/version/25/"
   }
  },
  {
   "flavor_text": "양지에서 낮잠 자는 모습을 볼 수 있다.\n태양의 빛을 많이 받으면\n등의 씨앗이 크게 자란다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version": {
    "name": "omega-ruby",
    "url": "https://pokeapi.co/api/v2/version/25/"
   }
  },
  {
   "flavor_text": "Bulbizarre passe son temps à faire la sieste sous le soleil.\nIl y a une graine sur son dos. Il absorbe les rayons du soleil\npour faire doucement pousser la graine.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version": {
    "name": "omega-ruby",
    "url": "https://pokeapi.co/api/v2/version/25/"
   }
  },
  {
   "flavor_text": "Bisasam macht gern einmal ein Nickerchen im\nSonnenschein. Auf seinem Rücken trägt es einen\nSamen. Indem es Sonnenstrahlen aufsaugt,\nwird der Samen zunehmend größer.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version": {
    "name": "omega-ruby",
    "url": "https://pokeapi.co/api/v2/version/25/"
   }
  },
  {
   "flavor_text": "A Bulbasaur es fácil verle echándose una siesta al sol.\nLa semilla que tiene en el lomo va creciendo cada vez más\na medida que absorbe los rayos del sol.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version": {
    "name": "omega-ruby",
    "url": "https://pokeapi.co/api/v2/version/25/"
   }
  },
  {
   "flavor_text": "È possibile vedere Bulbasaur mentre schiaccia un pisolino\nsotto il sole. Ha un seme piantato sulla schiena. Grazie ai\nraggi solari il seme cresce ingrandendosi progressivamente.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version": {
    "name": "omega-ruby",
    "url": "https://pokeapi.co/api/v2/version/25/"
   }
  },
  {
   "flavor_text": "Bulbasaur can be seen napping in bright sunlight.\nThere is a seed on its back. By soaking up the sun’s rays,\nthe seed grows progressively larger.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "omega-ruby",
    "url": "https://pokeapi.co/api/v2/version/25/"
   }
  },
  {
   "flavor_text": "日なたで　昼寝を　する　姿を　見かける。\n太陽の　光を　いっぱい　浴びることで\n背中の　タネが　大きく　育つのだ。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version": {
    "name": "omega-ruby",
    "url": "https://pokeapi.co/api/v2/version/25/"
   }
  },
  {
   "flavor_text": "ひなたで　ひるねを　する　すがたを　みかける。\nたいようの　ひかりを　いっぱい　あびることで\nせなかの　タネが　おおきく　そだつのだ。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version": {
    "name": "alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version/26/"
   }
  },
  {
   "flavor_text": "양지에서 낮잠 자는 모습을 볼 수 있다.\n태양의 빛을 많이 받으면\n등의 씨앗이 크게 자란다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version": {
    "name": "alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version/26/"
   }
  },
  {
   "flavor_text": "Bulbizarre passe son temps à faire la sieste sous le soleil.\nIl y a une graine sur son dos. Il absorbe les rayons du soleil\npour faire doucement pousser la graine.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version": {
    "name": "alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version/26/"
   }
  },
  {
   "flavor_text": "Bisasam macht gern einmal ein Nickerchen im\nSonnenschein. Auf seinem Rücken trägt es einen\nSamen. Indem es Sonnenstrahlen aufsaugt,\nwird er zunehmend größer.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version": {
    "name": "alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version/26/"
   }
  },
  {
   "flavor_text": "A Bulbasaur es fácil verle echándose una siesta al sol.\nLa semilla que tiene en el lomo va creciendo cada vez más\na medida que absorbe los rayos del sol.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version": {
    "name": "alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version/26/"
   }
  },
  {
   "flavor_text": "È possibile vedere Bulbasaur mentre schiaccia un pisolino\nsotto il sole. Ha un seme piantato sulla schiena. Grazie ai\nraggi solari il seme cresce ingrandendosi progressivamente.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version": {
    "name": "alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version/26/"
   }
  },
  {
   "flavor_text": "Bulbasaur can be seen napping in bright sunlight.\nThere is a seed on its back. By soaking up the sun’s rays,\nthe seed grows progressively larger.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version/26/"
   }
  },
  {
   "flavor_text": "日なたで　昼寝を　する　姿を　見かける。\n太陽の　光を　いっぱい　浴びることで\n背中の　タネが　大きく　育つのだ。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version": {
    "name": "alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version/26/"
   }
  },
  {
   "flavor_text": "なんにちだって　なにも　たべなくても\nげんき！　せなかのタネに　たくさん\nえいようが　あるから　へいきだ！",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version": {
    "name": "lets-go-pikachu",
    "url": "https://pokeapi.co/api/v2/version/31/"
   }
  },
  {
   "flavor_text": "며칠 동안 아무것도 먹지 않아도\n건강하다! 등에 있는 씨앗에는\n많은 영양분이 있어서 문제없다!",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version": {
    "name": "lets-go-pikachu",
    "url": "https://pokeapi.co/api/v2/version/31/"
   }
  },
  {
   "flavor_text": "背上的種子裡存著很多營養，\n所以就算好幾天不吃東西\n也能活得好好的！",
   "language": {
    "name": "zh-hant",
    "url": "https://pokeapi.co/api/v2/language/4/"
   },
   "version": {
    "name": "lets-go-pikachu",
    "url": "https://pokeapi.co/api/v2/version/31/"
   }
  },
  {
   "flavor_text": "Il peut survivre plusieurs jours sans manger\ngrâce aux nutriments contenus dans le bulbe\nsur son dos.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version": {
    "name": "lets-go-pikachu",
    "url": "https://pokeapi.co/api/v2/version/31/"
   }
  },
  {
   "flavor_text": "Es kommt tagelang ohne Nahrung aus, da es\nin den Samen auf seinem Rücken Nährstoffe\nspeichert.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version": {
    "name": "lets-go-pikachu",
    "url": "https://pokeapi.co/api/v2/version/31/"
   }
  },
  {
   "flavor_text": "Puede sobrevivir largo tiempo sin probar\nbocado gracias a los nutrientes que guarda\nen el bulbo del lomo.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version": {
    "name": "lets-go-pikachu",
    "url": "https://pokeapi.co/api/v2/version/31/"
   }
  },
  {
   "flavor_text": "Questo Pokémon può stare a lungo senza\nmangiare. Accumula energia nel bulbo che\nha sulla schiena.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version": {
    "name": "lets-go-pikachu",
    "url": "https://pokeapi.co/api/v2/version/31/"
   }
  },
  {
   "flavor_text": "It can go for days without eating a single morsel.\nIn the bulb on its back, it stores energy.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "lets-go-pikachu",
    "url": "https://pokeapi.co/api/v2/version/31/"
   }
  },
  {
   "flavor_text": "何日だって　なにも　食べなくても\n元気！　背中のタネに　たくさん\n栄養が　あるから　平気だ！",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version": {
    "name": "lets-go-pikachu",
    "url": "https://pokeapi.co/api/v2/version/31/"
   }
  },
  {
   "flavor_text": "背上的种子里储存着营养，\n所以即使好几天不吃东西\n也可以活得好好的！",
   "language": {
    "name": "zh-hans",
    "url": "https://pokeapi.co/api/v2/language/12/"
   },
   "version": {
    "name": "lets-go-pikachu",
    "url": "https://pokeapi.co/api/v2/version/31/"
   }
  },
  {
   "flavor_text": "なんにちだって　なにも　たべなくても\nげんき！　せなかのタネに　たくさん\nえいようが　あるから　へいきだ！",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version": {
    "name": "lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version/32/"
   }
  },
  {
   "flavor_text": "며칠 동안 아무것도 먹지 않아도\n건강하다! 등에 있는 씨앗에는\n많은 영양분이 있어서 문제없다!",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version": {
    "name": "lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version/32/"
   }
  },
  {
   "flavor_text": "背上的種子裡存著很多營養，\n所以就算好幾天不吃東西\n也能活得好好的！",
   "language": {
    "name": "zh-hant",
    "url": "https://pokeapi.co/api/v2/language/4/"
   },
   "version": {
    "name": "lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version/32/"
   }
  },
  {
   "flavor_text": "Il peut survivre plusieurs jours sans manger\ngrâce aux nutriments contenus dans le bulbe\nsur son dos.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version": {
    "name": "lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version/32/"
   }
  },
  {
   "flavor_text": "Es kommt tagelang ohne Nahrung aus, da es\nin den Samen auf seinem Rücken Nährstoffe\nspeichert.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version": {
    "name": "lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version/32/"
   }
  },
  {
   "flavor_text": "Puede sobrevivir largo tiempo sin probar\nbocado gracias a los nutrientes que guarda\nen el bulbo del lomo.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version": {
    "name": "lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version/32/"
   }
  },
  {
   "flavor_text": "Questo Pokémon può stare a lungo senza\nmangiare. Accumula energia nel bulbo che\nha sulla schiena.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version": {
    "name": "lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version/32/"
   }
  },
  {
   "flavor_text": "It can go for days without eating a single morsel.\nIn the bulb on its back, it stores energy.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version/32/"
   }
  },
  {
   "flavor_text": "何日だって　なにも　食べなくても\n元気！　背中のタネに　たくさん\n栄養が　あるから　平気だ！",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version": {
    "name": "lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version/32/"
   }
  },
  {
   "flavor_text": "背上的种子里储存着营养，\n所以即使好几天不吃东西\n也可以活得好好的！",
   "language": {
    "name": "zh-hans",
    "url": "https://pokeapi.co/api/v2/language/12/"
   },
   "version": {
    "name": "lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version/32/"
   }
  },
  {
   "flavor_text": "うまれたときから　せなかに\nしょくぶつの　タネが　あって\nすこしずつ　おおきく　そだつ。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version": {
    "name": "sword",
    "url": "https://pokeapi.co/api/v2/version/33/"
   }
  },
  {
   "flavor_text": "태어났을 때부터 등에\n식물의 씨앗이 있으며\n조금씩 크게 자란다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version": {
    "name": "sword",
    "url": "https://pokeapi.co/api/v2/version/33/"
   }
  },
  {
   "flavor_text": "從出生的時候開始\n背上就有一顆植物種子。\n這顆種子會漸漸地長大。",
   "language": {
    "name": "zh-hant",
    "url": "https://pokeapi.co/api/v2/language/4/"
   },
   "version": {
    "name": "sword",
    "url": "https://pokeapi.co/api/v2/version/33/"
   }
  },
  {
   "flavor_text": "Il y a une graine sur son dos depuis sa naissance.\nElle grossit un peu chaque jour.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version": {
    "name": "sword",
    "url": "https://pokeapi.co/api/v2/version/33/"
   }
  },
  {
   "flavor_text": "Dieses Pokémon trägt von Geburt an einen\nSamen auf dem Rücken, der im Laufe der Zeit\nkeimt und wächst.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version": {
    "name": "sword",
    "url": "https://pokeapi.co/api/v2/version/33/"
   }
  },
  {
   "flavor_text": "Este Pokémon nace con una semilla en el lomo,\nque brota con el paso del tiempo.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version": {
    "name": "sword",
    "url": "https://pokeapi.co/api/v2/version/33/"
   }
  },
  {
   "flavor_text": "Fin dalla nascita questo Pokémon ha sulla schiena\nun seme che cresce lentamente.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version": {
    "name": "sword",
    "url": "https://pokeapi.co/api/v2/version/33/"
   }
  },
  {
   "flavor_text": "There is a plant seed on its back right from the\nday this Pokémon is born. The seed slowly\ngrows larger.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "sword",
    "url": "https://pokeapi.co/api/v2/version/33/"
   }
  },
  {
   "flavor_text": "生まれたときから　背中に\n植物の　タネが　あって\n少しずつ　大きく　育つ。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version": {
    "name": "sword",
    "url": "https://pokeapi.co/api/v2/version/33/"
   }
  },
  {
   "flavor_text": "从出生的时候开始，\n背上就有一颗植物种子。\n这颗种子会渐渐地长大。",
   "language": {
    "name": "zh-hans",
    "url": "https://pokeapi.co/api/v2/language/12/"
   },
   "version": {
    "name": "sword",
    "url": "https://pokeapi.co/api/v2/version/33/"
   }
  },
  {
   "flavor_text": "うまれて　しばらくの　あいだ\nせなかの　タネに　つまった\nえいようを　とって　そだつ。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version": {
    "name": "shield",
    "url": "https://pokeapi.co/api/v2/version/34/"
   }
  },
  {
   "flavor_text": "태어나서 얼마 동안\n등의 씨앗에 담긴\n영양을 섭취하며 자란다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version": {
    "name": "shield",
    "url": "https://pokeapi.co/api/v2/version/34/"
   }
  },
  {
   "flavor_text": "在出生後的一段時間內，\n牠會吸收背上種子裡\n儲存著的營養成長。",
   "language": {
    "name": "zh-hant",
    "url": "https://pokeapi.co/api/v2/language/4/"
   },
   "version": {
    "name": "shield",
    "url": "https://pokeapi.co/api/v2/version/34/"
   }
  },
  {
   "flavor_text": "Quand il est jeune, il absorbe les nutriments\nconservés dans son dos pour grandir\net se développer.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version": {
    "name": "shield",
    "url": "https://pokeapi.co/api/v2/version/34/"
   }
  },
  {
   "flavor_text": "Nach der Geburt nimmt es für eine Weile\nNährstoffe über den Samen auf seinem\nRücken auf.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version": {
    "name": "shield",
    "url": "https://pokeapi.co/api/v2/version/34/"
   }
  },
  {
   "flavor_text": "Desde que nace, crece alimentándose de los\nnutrientes que contiene la semilla de su lomo.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version": {
    "name": "shield",
    "url": "https://pokeapi.co/api/v2/version/34/"
   }
  },
  {
   "flavor_text": "Appena nato, trae nutrimento dalle sostanze\ncontenute nel seme sul dorso.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version": {
    "name": "shield",
    "url": "https://pokeapi.co/api/v2/version/34/"
   }
  },
  {
   "flavor_text": "While it is young, it uses the nutrients that are\nstored in the seed on its back in order to grow.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version": {
    "name": "shield",
    "url": "https://pokeapi.co/api/v2/version/34/"
   }
  },
  {
   "flavor_text": "生まれて　しばらくの　あいだ\n背中の　タネに　つまった\n栄養を　とって　育つ。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version": {
    "name": "shield",
    "url": "https://pokeapi.co/api/v2/version/34/"
   }
  },
  {
   "flavor_text": "在出生后的一段时间内，\n它会吸收背上种子里\n储存着的营养成长。",
   "language": {
    "name": "zh-hans",
    "url": "https://pokeapi.co/api/v2/language/12/"
   },
   "version": {
    "name": "shield",
    "url": "https://pokeapi.co/api/v2/version/34/"
   }
  }
 ],
 "form_descriptions": [],
 "forms_switchable": false,
 "gender_rate": 1,
 "genera": [
  {
   "genus": "たねポケモン",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   }
  },
  {
   "genus": "씨앗포켓몬",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   }
  },
  {
   "genus": "種子寶可夢",
   "language": {
    "name": "zh-hant",
    "url": "https://pokeapi.co/api/v2/language/4/"
   }
  },
  {
   "genus": "Pokémon Graine",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   }
  },
  {
   "genus": "Samen-Pokémon",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   }
  },
  {
   "genus": "Pokémon Semilla",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   }
  },
  {
   "genus": "Pokémon Seme",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   }
  },
  {
   "genus": "Seed Pokémon",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   }
  },
  {
   "genus": "たねポケモン",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   }
  },
  {
   "genus": "种子宝可梦",
   "language": {
    "name": "zh-hans",
    "url": "https://pokeapi.co/api/v2/language/12/"
   }
  }
 ],
 "generation": {
  "name": "generation-i",
  "url": "https://pokeapi.co/api/v2/generation/1/"
 },
 "growth_rate": {
  "name": "medium-slow",
  "url": "https://pokeapi.co/api/v2/growth-rate/4/"
 },
 "habitat": {
  "name": "grassland",
  "url": "https://pokeapi.co/api/v2/pokemon-habitat/3/"
 },
 "has_gender_differences": false,
 "hatch_counter": 20,
 "id": 1,
 "is_baby": false,
 "is_legendary": false,
 "is_mythical": false,
 "name": "bulbasaur",
 "names": [
  {
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "name": "フシギダネ"
  },
  {
   "language": {
    "name": "ja-roma",
    "url": "https://pokeapi.co/api/v2/language/2/"
   },
   "name": "Fushigidane"
  },
  {
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "name": "이상해씨"
  },
  {
   "language": {
    "name": "zh-hant",
    "url": "https://pokeapi.co/api/v2/language/4/"
   },
   "name": "妙蛙種子"
  },
  {
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "name": "Bulbizarre"
  },
  {
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "name": "Bisasam"
  },
  {
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "name": "Bulbasaur"
  },
  {
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "name": "Bulbasaur"
  },
  {
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "name": "Bulbasaur"
  },
  {
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "name": "フシギダネ"
  },
  {
   "language": {
    "name": "zh-hans",
    "url": "https://pokeapi.co/api/v2/language/12/"
   },
   "name": "妙蛙种子"
  }
 ],
 "order": 1,
 "pal_park_encounters": [
  {
   "area": {
    "name": "field",
    "url": "https://pokeapi.co/api/v2/pal-park-area/2/"
   },
   "base_score": 50,
   "rate": 30
  }
 ],
 "pokedex_numbers": [
  {
   "entry_number": 1,
   "pokedex": {
    "name": "national",
    "url": "https://pokeapi.co/api/v2/pokedex/1/"
   }
  },
  {
   "entry_number": 1,
   "pokedex": {
    "name": "kanto",
    "url": "https://pokeapi.co/api/v2/pokedex/2/"
   }
  },
  {
   "entry_number": 226,
   "pokedex": {
    "name": "original-johto",
    "url": "https://pokeapi.co/api/v2/pokedex/3/"
   }
  },
  {
   "entry_number": 231,
   "pokedex": {
    "name": "updated-johto",
    "url": "https://pokeapi.co/api/v2/pokedex/7/"
   }
  },
  {
   "entry_number": 80,
   "pokedex": {
    "name": "kalos-central",
    "url": "https://pokeapi.co/api/v2/pokedex/12/"
   }
  },
  {
   "entry_number": 1,
   "pokedex": {
    "name": "letsgo-kanto",
    "url": "https://pokeapi.co/api/v2/pokedex/26/"
   }
  },
  {
   "entry_number": 68,
   "pokedex": {
    "name": "isle-of-armor",
    "url": "https://pokeapi.co/api/v2/pokedex/28/"
   }
  },
  {
   "entry_number": 164,
   "pokedex": {
    "name": "blueberry",
    "url": "https://pokeapi.co/api/v2/pokedex/33/"
   }
  },
  {
   "entry_number": 148,
   "pokedex": {
    "name": "lumiose-city",
    "url": "https://pokeapi.co/api/v2/pokedex/34/"
   }
  }
 ],
 "shape": {
  "name": "quadruped",
  "url": "https://pokeapi.co/api/v2/pokemon-shape/8/"
 },
 "varieties": [
  {
   "is_default": true,
   "pokemon": {
    "name": "bulbasaur",
    "url": "https://pokeapi.co/api/v2/pokemon/1/"
   }
  }
 ]
}
```

`https://pokeapi.co/api/v2/move/14` que vai ter um retorno similar a esses:

```json
{
 "accuracy": null,
 "contest_combos": {
  "normal": {
   "use_after": null,
   "use_before": [
    {
     "name": "cut",
     "url": "https://pokeapi.co/api/v2/move/15/"
    },
    {
     "name": "crabhammer",
     "url": "https://pokeapi.co/api/v2/move/152/"
    },
    {
     "name": "slash",
     "url": "https://pokeapi.co/api/v2/move/163/"
    },
    {
     "name": "false-swipe",
     "url": "https://pokeapi.co/api/v2/move/206/"
    },
    {
     "name": "fury-cutter",
     "url": "https://pokeapi.co/api/v2/move/210/"
    },
    {
     "name": "crush-claw",
     "url": "https://pokeapi.co/api/v2/move/306/"
    }
   ]
  },
  "super": {
   "use_after": null,
   "use_before": null
  }
 },
 "contest_effect": {
  "url": "https://pokeapi.co/api/v2/contest-effect/32/"
 },
 "contest_type": {
  "name": "beauty",
  "url": "https://pokeapi.co/api/v2/contest-type/2/"
 },
 "damage_class": {
  "name": "status",
  "url": "https://pokeapi.co/api/v2/move-damage-class/1/"
 },
 "effect_chance": null,
 "effect_changes": [],
 "effect_entries": [
  {
   "effect": "Augmente l'Attaque du lanceur de deux niveaux.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "short_effect": "Augmente l'Attaque du lanceur de deux niveaux."
  },
  {
   "effect": "Raises the user’s Attack by two stages.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "short_effect": "Raises the user’s Attack by two stages."
  }
 ],
 "flavor_text_entries": [
  {
   "flavor_text": "A dance that in­\ncreases ATTACK.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "gold-silver",
    "url": "https://pokeapi.co/api/v2/version-group/3/"
   }
  },
  {
   "flavor_text": "A dance that in­\ncreases ATTACK.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "crystal",
    "url": "https://pokeapi.co/api/v2/version-group/4/"
   }
  },
  {
   "flavor_text": "A fighting dance that\nsharply raises ATTACK.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "ruby-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/5/"
   }
  },
  {
   "flavor_text": "A fighting dance that\nsharply raises ATTACK.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "emerald",
    "url": "https://pokeapi.co/api/v2/version-group/6/"
   }
  },
  {
   "flavor_text": "A frenetic dance of\nfighting. It sharply\nraises the ATTACK\nstat.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "firered-leafgreen",
    "url": "https://pokeapi.co/api/v2/version-group/7/"
   }
  },
  {
   "flavor_text": "A frenetic dance to\nuplift the fighting\nspirit. It sharply\nraises the user’s\nAttack stat.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "diamond-pearl",
    "url": "https://pokeapi.co/api/v2/version-group/8/"
   }
  },
  {
   "flavor_text": "A frenetic dance to\nuplift the fighting\nspirit. It sharply\nraises the user’s\nAttack stat.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "platinum",
    "url": "https://pokeapi.co/api/v2/version-group/9/"
   }
  },
  {
   "flavor_text": "A frenetic dance to\nuplift the fighting\nspirit. It sharply\nraises the user’s\nAttack stat.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "heartgold-soulsilver",
    "url": "https://pokeapi.co/api/v2/version-group/10/"
   }
  },
  {
   "flavor_text": "Danse frénétique qui exalte l’esprit\ncombatif. Augmente beaucoup\nl’Attaque du lanceur.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "black-white",
    "url": "https://pokeapi.co/api/v2/version-group/11/"
   }
  },
  {
   "flavor_text": "A frenetic dance to uplift the fighting\nspirit. It sharply raises the user’s\nAttack stat.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "black-white",
    "url": "https://pokeapi.co/api/v2/version-group/11/"
   }
  },
  {
   "flavor_text": "A frenetic dance to uplift the fighting\nspirit. It sharply raises the user’s\nAttack stat.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "black-2-white-2",
    "url": "https://pokeapi.co/api/v2/version-group/14/"
   }
  },
  {
   "flavor_text": "たたかいのまいを　はげしく　おどって\nきあいを　たかめる。\nじぶんの　こうげきを　ぐーんと　あげる。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  },
  {
   "flavor_text": "싸움의 춤을 격렬하게 추며\n기세를 높인다.\n자신의 공격을 크게 올린다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  },
  {
   "flavor_text": "Danse frénétique qui exalte l’esprit combatif.\nAugmente beaucoup l’Attaque du lanceur.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  },
  {
   "flavor_text": "Ein wilder Kampftanz, der den eigenen\nAngriffs-Wert stark erhöht.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  },
  {
   "flavor_text": "Baile frenético que aumenta mucho el Ataque.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  },
  {
   "flavor_text": "Danza frenetica che incrementa lo spirito\ncombattivo. Chi la usa aumenta di molto\nil suo Attacco.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  },
  {
   "flavor_text": "A frenetic dance to uplift the fighting\nspirit. This sharply raises the user’s\nAttack stat.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  },
  {
   "flavor_text": "戦いの舞を　激しく　おどって\n気合を　高める。\n自分の　攻撃を　ぐーんと　あげる。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  },
  {
   "flavor_text": "たたかいのまいを　はげしく　おどって\nきあいを　たかめる。\nじぶんの　こうげきを　ぐーんと　あげる。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version_group": {
    "name": "omega-ruby-alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/16/"
   }
  },
  {
   "flavor_text": "싸움의 춤을 격렬하게 추며\n기세를 높인다.\n자신의 공격을 크게 올린다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version_group": {
    "name": "omega-ruby-alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/16/"
   }
  },
  {
   "flavor_text": "Danse frénétique qui exalte l’esprit combatif.\nAugmente beaucoup l’Attaque du lanceur.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "omega-ruby-alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/16/"
   }
  },
  {
   "flavor_text": "Ein wilder Kampftanz, der den eigenen\nAngriffs-Wert stark erhöht.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version_group": {
    "name": "omega-ruby-alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/16/"
   }
  },
  {
   "flavor_text": "Baile frenético que aumenta mucho el Ataque.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version_group": {
    "name": "omega-ruby-alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/16/"
   }
  },
  {
   "flavor_text": "Danza frenetica che incrementa lo spirito\ncombattivo. Chi la usa aumenta di molto\nil suo Attacco.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version_group": {
    "name": "omega-ruby-alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/16/"
   }
  },
  {
   "flavor_text": "A frenetic dance to uplift the fighting\nspirit. This sharply raises the user’s\nAttack stat.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "omega-ruby-alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/16/"
   }
  },
  {
   "flavor_text": "戦いの舞を　激しく　おどって\n気合を　高める。\n自分の　攻撃を　ぐーんと　あげる。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version_group": {
    "name": "omega-ruby-alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/16/"
   }
  },
  {
   "flavor_text": "たたかいのまいを　はげしく　おどって\nきあいを　たかめる。\nじぶんの　こうげきを　ぐーんと　あげる。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "싸움의 춤을 격렬하게 춰서\n기세를 높인다.\n자신의 공격을 크게 올린다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "激烈地跳起戰舞提升氣勢。\n大幅提高自己的攻擊。",
   "language": {
    "name": "zh-hant",
    "url": "https://pokeapi.co/api/v2/language/4/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "Danse frénétique qui exalte l’esprit combatif.\nAugmente beaucoup l’Attaque du lanceur.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "Ein wilder Kampftanz, der den eigenen Angriffs-Wert\nstark erhöht.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "Baile frenético que aumenta mucho el Ataque.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "Danza frenetica che incrementa lo spirito\ncombattivo. Chi la usa aumenta di molto\nil suo Attacco.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "A frenetic dance to uplift the fighting spirit.\nThis sharply raises the user’s Attack stat.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "戦いの舞を　激しく　おどって\n気合を　高める。\n自分の　攻撃を　ぐーんと　あげる。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "激烈地跳起战舞提高气势。\n大幅提高自己的攻击。",
   "language": {
    "name": "zh-hans",
    "url": "https://pokeapi.co/api/v2/language/12/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "たたかいのまいを　はげしく　おどって\nきあいを　たかめる。\nじぶんの　こうげきを　ぐーんと　あげる。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "싸움의 춤을 격렬하게 춰서\n기세를 높인다.\n자신의 공격을 크게 올린다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "激烈地跳起戰舞提升氣勢。\n大幅提高自己的攻擊。",
   "language": {
    "name": "zh-hant",
    "url": "https://pokeapi.co/api/v2/language/4/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "Danse frénétique qui exalte l’esprit combatif.\nAugmente beaucoup l’Attaque du lanceur.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "Ein wilder Kampftanz, der den eigenen Angriffs-Wert\nstark erhöht.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "Baile frenético que aumenta mucho el Ataque.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "Danza frenetica che incrementa lo spirito\ncombattivo. Chi la usa aumenta di molto\nil suo Attacco.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "A frenetic dance to uplift the fighting spirit.\nThis sharply raises the user’s Attack stat.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "戦いの舞を　激しく　おどって\n気合を　高める。\n自分の　攻撃を　ぐーんと　あげる。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "激烈地跳起战舞提高气势。\n大幅提高自己的攻击。",
   "language": {
    "name": "zh-hans",
    "url": "https://pokeapi.co/api/v2/language/12/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "たたかいのまいを　はげしく　おどって\nきあいを　たかめる。\nじぶんの　こうげきを　ぐーんと　あげる。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "싸움의 춤을 격렬하게 춰서\n기세를 높인다.\n자신의 공격을 크게 올린다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "激烈地跳起戰舞提升氣勢。\n大幅提高自己的攻擊。",
   "language": {
    "name": "zh-hant",
    "url": "https://pokeapi.co/api/v2/language/4/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "Une danse frénétique qui exalte l’esprit combatif.\nAugmente beaucoup l’Attaque du lanceur.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "Ein wilder Tanz, der den Kampfgeist wecken soll.\nDer Angriffs-Wert des Anwenders wird stark\nerhöht.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "Baile frenético que aumenta mucho el Ataque.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "Danza frenetica che incrementa lo spirito\ncombattivo. Chi la usa aumenta di molto\nil suo Attacco.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "A frenetic dance to uplift the fighting spirit.\nThis sharply raises the user’s Attack stat.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "戦いの舞を　激しく　おどって\n気合を　高める。\n自分の　攻撃を　ぐーんと　あげる。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "激烈地跳起战舞提高气势。\n大幅提高自己的攻击。",
   "language": {
    "name": "zh-hans",
    "url": "https://pokeapi.co/api/v2/language/12/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "たたかいのまいを　はげしく　おどって\nきあいを　たかめる。\nじぶんの　こうげきを　ぐーんと　あげる。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "싸움의 춤을 격렬하게 춰서\n기세를 높인다.\n자신의 공격을 크게 올린다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "激烈地跳起戰舞提升氣勢，\n大幅提高自己的攻擊。",
   "language": {
    "name": "zh-hant",
    "url": "https://pokeapi.co/api/v2/language/4/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "Une danse frénétique qui exalte l’esprit combatif.\nAugmente beaucoup l’Attaque du lanceur.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "Ein wilder Tanz, der den Kampfgeist wecken soll.\nDer Angriffs-Wert des Anwenders wird stark erhöht.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "Baile frenético que aumenta mucho el Ataque.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "Danza frenetica che incrementa lo spirito\ncombattivo. Chi la usa aumenta di molto\nil suo Attacco.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "A frenetic dance to uplift the fighting spirit.\nThis sharply raises the user’s Attack stat.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "戦いの舞を　激しく　おどって\n気合を　高める。\n自分の　攻撃を　ぐーんと　あげる。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "激烈地跳起战舞提高气势。\n大幅提高自己的攻击。",
   "language": {
    "name": "zh-hans",
    "url": "https://pokeapi.co/api/v2/language/12/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "A frenetic dance to uplift the fighting spirit. This raises the user’s offensive stats.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "legends-arceus",
    "url": "https://pokeapi.co/api/v2/version-group/24/"
   }
  },
  {
   "flavor_text": "A frenetic dance to uplift the fighting spirit. This sharply boosts the user's Attack stat.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "scarlet-violet",
    "url": "https://pokeapi.co/api/v2/version-group/25/"
   }
  }
 ],
 "generation": {
  "name": "generation-i",
  "url": "https://pokeapi.co/api/v2/generation/1/"
 },
 "id": 14,
 "learned_by_pokemon": [
  {
   "name": "bulbasaur",
   "url": "https://pokeapi.co/api/v2/pokemon/1/"
  },
  {
   "name": "ivysaur",
   "url": "https://pokeapi.co/api/v2/pokemon/2/"
  },
  {
   "name": "venusaur",
   "url": "https://pokeapi.co/api/v2/pokemon/3/"
  },
  {
   "name": "charmander",
   "url": "https://pokeapi.co/api/v2/pokemon/4/"
  },
  {
   "name": "charmeleon",
   "url": "https://pokeapi.co/api/v2/pokemon/5/"
  },
  {
   "name": "charizard",
   "url": "https://pokeapi.co/api/v2/pokemon/6/"
  },
  {
   "name": "beedrill",
   "url": "https://pokeapi.co/api/v2/pokemon/15/"
  },
  {
   "name": "raticate",
   "url": "https://pokeapi.co/api/v2/pokemon/20/"
  },
  {
   "name": "sandshrew",
   "url": "https://pokeapi.co/api/v2/pokemon/27/"
  },
  {
   "name": "sandslash",
   "url": "https://pokeapi.co/api/v2/pokemon/28/"
  },
  {
   "name": "oddish",
   "url": "https://pokeapi.co/api/v2/pokemon/43/"
  },
  {
   "name": "gloom",
   "url": "https://pokeapi.co/api/v2/pokemon/44/"
  },
  {
   "name": "vileplume",
   "url": "https://pokeapi.co/api/v2/pokemon/45/"
  },
  {
   "name": "paras",
   "url": "https://pokeapi.co/api/v2/pokemon/46/"
  },
  {
   "name": "parasect",
   "url": "https://pokeapi.co/api/v2/pokemon/47/"
  },
  {
   "name": "diglett",
   "url": "https://pokeapi.co/api/v2/pokemon/50/"
  },
  {
   "name": "dugtrio",
   "url": "https://pokeapi.co/api/v2/pokemon/51/"
  },
  {
   "name": "bellsprout",
   "url": "https://pokeapi.co/api/v2/pokemon/69/"
  },
  {
   "name": "weepinbell",
   "url": "https://pokeapi.co/api/v2/pokemon/70/"
  },
  {
   "name": "victreebel",
   "url": "https://pokeapi.co/api/v2/pokemon/71/"
  },
  {
   "name": "tentacool",
   "url": "https://pokeapi.co/api/v2/pokemon/72/"
  },
  {
   "name": "tentacruel",
   "url": "https://pokeapi.co/api/v2/pokemon/73/"
  },
  {
   "name": "rapidash",
   "url": "https://pokeapi.co/api/v2/pokemon/78/"
  },
  {
   "name": "farfetchd",
   "url": "https://pokeapi.co/api/v2/pokemon/83/"
  },
  {
   "name": "doduo",
   "url": "https://pokeapi.co/api/v2/pokemon/84/"
  },
  {
   "name": "dodrio",
   "url": "https://pokeapi.co/api/v2/pokemon/85/"
  },
  {
   "name": "krabby",
   "url": "https://pokeapi.co/api/v2/pokemon/98/"
  },
  {
   "name": "kingler",
   "url": "https://pokeapi.co/api/v2/pokemon/99/"
  },
  {
   "name": "exeggcute",
   "url": "https://pokeapi.co/api/v2/pokemon/102/"
  },
  {
   "name": "exeggutor",
   "url": "https://pokeapi.co/api/v2/pokemon/103/"
  },
  {
   "name": "cubone",
   "url": "https://pokeapi.co/api/v2/pokemon/104/"
  },
  {
   "name": "marowak",
   "url": "https://pokeapi.co/api/v2/pokemon/105/"
  },
  {
   "name": "hitmonlee",
   "url": "https://pokeapi.co/api/v2/pokemon/106/"
  },
  {
   "name": "hitmonchan",
   "url": "https://pokeapi.co/api/v2/pokemon/107/"
  },
  {
   "name": "lickitung",
   "url": "https://pokeapi.co/api/v2/pokemon/108/"
  },
  {
   "name": "rhyhorn",
   "url": "https://pokeapi.co/api/v2/pokemon/111/"
  },
  {
   "name": "rhydon",
   "url": "https://pokeapi.co/api/v2/pokemon/112/"
  },
  {
   "name": "tangela",
   "url": "https://pokeapi.co/api/v2/pokemon/114/"
  },
  {
   "name": "goldeen",
   "url": "https://pokeapi.co/api/v2/pokemon/118/"
  },
  {
   "name": "seaking",
   "url": "https://pokeapi.co/api/v2/pokemon/119/"
  },
  {
   "name": "scyther",
   "url": "https://pokeapi.co/api/v2/pokemon/123/"
  },
  {
   "name": "pinsir",
   "url": "https://pokeapi.co/api/v2/pokemon/127/"
  },
  {
   "name": "kabutops",
   "url": "https://pokeapi.co/api/v2/pokemon/141/"
  },
  {
   "name": "mew",
   "url": "https://pokeapi.co/api/v2/pokemon/151/"
  },
  {
   "name": "chikorita",
   "url": "https://pokeapi.co/api/v2/pokemon/152/"
  },
  {
   "name": "bayleef",
   "url": "https://pokeapi.co/api/v2/pokemon/153/"
  },
  {
   "name": "meganium",
   "url": "https://pokeapi.co/api/v2/pokemon/154/"
  },
  {
   "name": "totodile",
   "url": "https://pokeapi.co/api/v2/pokemon/158/"
  },
  {
   "name": "croconaw",
   "url": "https://pokeapi.co/api/v2/pokemon/159/"
  },
  {
   "name": "feraligatr",
   "url": "https://pokeapi.co/api/v2/pokemon/160/"
  },
  {
   "name": "ledyba",
   "url": "https://pokeapi.co/api/v2/pokemon/165/"
  },
  {
   "name": "ledian",
   "url": "https://pokeapi.co/api/v2/pokemon/166/"
  },
  {
   "name": "ariados",
   "url": "https://pokeapi.co/api/v2/pokemon/168/"
  },
  {
   "name": "bellossom",
   "url": "https://pokeapi.co/api/v2/pokemon/182/"
  },
  {
   "name": "hoppip",
   "url": "https://pokeapi.co/api/v2/pokemon/187/"
  },
  {
   "name": "skiploom",
   "url": "https://pokeapi.co/api/v2/pokemon/188/"
  },
  {
   "name": "jumpluff",
   "url": "https://pokeapi.co/api/v2/pokemon/189/"
  },
  {
   "name": "sunkern",
   "url": "https://pokeapi.co/api/v2/pokemon/191/"
  },
  {
   "name": "sunflora",
   "url": "https://pokeapi.co/api/v2/pokemon/192/"
  },
  {
   "name": "yanma",
   "url": "https://pokeapi.co/api/v2/pokemon/193/"
  },
  {
   "name": "gligar",
   "url": "https://pokeapi.co/api/v2/pokemon/207/"
  },
  {
   "name": "qwilfish",
   "url": "https://pokeapi.co/api/v2/pokemon/211/"
  },
  {
   "name": "scizor",
   "url": "https://pokeapi.co/api/v2/pokemon/212/"
  },
  {
   "name": "heracross",
   "url": "https://pokeapi.co/api/v2/pokemon/214/"
  },
  {
   "name": "sneasel",
   "url": "https://pokeapi.co/api/v2/pokemon/215/"
  },
  {
   "name": "teddiursa",
   "url": "https://pokeapi.co/api/v2/pokemon/216/"
  },
  {
   "name": "ursaring",
   "url": "https://pokeapi.co/api/v2/pokemon/217/"
  },
  {
   "name": "skarmory",
   "url": "https://pokeapi.co/api/v2/pokemon/227/"
  },
  {
   "name": "celebi",
   "url": "https://pokeapi.co/api/v2/pokemon/251/"
  },
  {
   "name": "treecko",
   "url": "https://pokeapi.co/api/v2/pokemon/252/"
  },
  {
   "name": "grovyle",
   "url": "https://pokeapi.co/api/v2/pokemon/253/"
  },
  {
   "name": "sceptile",
   "url": "https://pokeapi.co/api/v2/pokemon/254/"
  },
  {
   "name": "torchic",
   "url": "https://pokeapi.co/api/v2/pokemon/255/"
  },
  {
   "name": "combusken",
   "url": "https://pokeapi.co/api/v2/pokemon/256/"
  },
  {
   "name": "blaziken",
   "url": "https://pokeapi.co/api/v2/pokemon/257/"
  },
  {
   "name": "lotad",
   "url": "https://pokeapi.co/api/v2/pokemon/270/"
  },
  {
   "name": "lombre",
   "url": "https://pokeapi.co/api/v2/pokemon/271/"
  },
  {
   "name": "ludicolo",
   "url": "https://pokeapi.co/api/v2/pokemon/272/"
  },
  {
   "name": "seedot",
   "url": "https://pokeapi.co/api/v2/pokemon/273/"
  },
  {
   "name": "nuzleaf",
   "url": "https://pokeapi.co/api/v2/pokemon/274/"
  },
  {
   "name": "shiftry",
   "url": "https://pokeapi.co/api/v2/pokemon/275/"
  },
  {
   "name": "shroomish",
   "url": "https://pokeapi.co/api/v2/pokemon/285/"
  },
  {
   "name": "breloom",
   "url": "https://pokeapi.co/api/v2/pokemon/286/"
  },
  {
   "name": "ninjask",
   "url": "https://pokeapi.co/api/v2/pokemon/291/"
  },
  {
   "name": "mawile",
   "url": "https://pokeapi.co/api/v2/pokemon/303/"
  },
  {
   "name": "roselia",
   "url": "https://pokeapi.co/api/v2/pokemon/315/"
  },
  {
   "name": "gulpin",
   "url": "https://pokeapi.co/api/v2/pokemon/316/"
  },
  {
   "name": "swalot",
   "url": "https://pokeapi.co/api/v2/pokemon/317/"
  },
  {
   "name": "cacnea",
   "url": "https://pokeapi.co/api/v2/pokemon/331/"
  },
  {
   "name": "cacturne",
   "url": "https://pokeapi.co/api/v2/pokemon/332/"
  },
  {
   "name": "zangoose",
   "url": "https://pokeapi.co/api/v2/pokemon/335/"
  },
  {
   "name": "seviper",
   "url": "https://pokeapi.co/api/v2/pokemon/336/"
  },
  {
   "name": "solrock",
   "url": "https://pokeapi.co/api/v2/pokemon/338/"
  },
  {
   "name": "corphish",
   "url": "https://pokeapi.co/api/v2/pokemon/341/"
  },
  {
   "name": "crawdaunt",
   "url": "https://pokeapi.co/api/v2/pokemon/342/"
  },
  {
   "name": "lileep",
   "url": "https://pokeapi.co/api/v2/pokemon/345/"
  },
  {
   "name": "cradily",
   "url": "https://pokeapi.co/api/v2/pokemon/346/"
  },
  {
   "name": "anorith",
   "url": "https://pokeapi.co/api/v2/pokemon/347/"
  },
  {
   "name": "armaldo",
   "url": "https://pokeapi.co/api/v2/pokemon/348/"
  },
  {
   "name": "banette",
   "url": "https://pokeapi.co/api/v2/pokemon/354/"
  },
  {
   "name": "tropius",
   "url": "https://pokeapi.co/api/v2/pokemon/357/"
  },
  {
   "name": "absol",
   "url": "https://pokeapi.co/api/v2/pokemon/359/"
  },
  {
   "name": "walrein",
   "url": "https://pokeapi.co/api/v2/pokemon/365/"
  },
  {
   "name": "groudon",
   "url": "https://pokeapi.co/api/v2/pokemon/383/"
  },
  {
   "name": "rayquaza",
   "url": "https://pokeapi.co/api/v2/pokemon/384/"
  },
  {
   "name": "turtwig",
   "url": "https://pokeapi.co/api/v2/pokemon/387/"
  },
  {
   "name": "grotle",
   "url": "https://pokeapi.co/api/v2/pokemon/388/"
  },
  {
   "name": "torterra",
   "url": "https://pokeapi.co/api/v2/pokemon/389/"
  },
  {
   "name": "chimchar",
   "url": "https://pokeapi.co/api/v2/pokemon/390/"
  },
  {
   "name": "monferno",
   "url": "https://pokeapi.co/api/v2/pokemon/391/"
  },
  {
   "name": "infernape",
   "url": "https://pokeapi.co/api/v2/pokemon/392/"
  },
  {
   "name": "empoleon",
   "url": "https://pokeapi.co/api/v2/pokemon/395/"
  },
  {
   "name": "bidoof",
   "url": "https://pokeapi.co/api/v2/pokemon/399/"
  },
  {
   "name": "bibarel",
   "url": "https://pokeapi.co/api/v2/pokemon/400/"
  },
  {
   "name": "kricketune",
   "url": "https://pokeapi.co/api/v2/pokemon/402/"
  },
  {
   "name": "budew",
   "url": "https://pokeapi.co/api/v2/pokemon/406/"
  },
  {
   "name": "roserade",
   "url": "https://pokeapi.co/api/v2/pokemon/407/"
  },
  {
   "name": "cranidos",
   "url": "https://pokeapi.co/api/v2/pokemon/408/"
  },
  {
   "name": "rampardos",
   "url": "https://pokeapi.co/api/v2/pokemon/409/"
  },
  {
   "name": "cherubi",
   "url": "https://pokeapi.co/api/v2/pokemon/420/"
  },
  {
   "name": "cherrim",
   "url": "https://pokeapi.co/api/v2/pokemon/421/"
  },
  {
   "name": "gible",
   "url": "https://pokeapi.co/api/v2/pokemon/443/"
  },
  {
   "name": "gabite",
   "url": "https://pokeapi.co/api/v2/pokemon/444/"
  },
  {
   "name": "garchomp",
   "url": "https://pokeapi.co/api/v2/pokemon/445/"
  },
  {
   "name": "riolu",
   "url": "https://pokeapi.co/api/v2/pokemon/447/"
  },
  {
   "name": "lucario",
   "url": "https://pokeapi.co/api/v2/pokemon/448/"
  },
  {
   "name": "skorupi",
   "url": "https://pokeapi.co/api/v2/pokemon/451/"
  },
  {
   "name": "drapion",
   "url": "https://pokeapi.co/api/v2/pokemon/452/"
  },
  {
   "name": "toxicroak",
   "url": "https://pokeapi.co/api/v2/pokemon/454/"
  },
  {
   "name": "carnivine",
   "url": "https://pokeapi.co/api/v2/pokemon/455/"
  },
  {
   "name": "snover",
   "url": "https://pokeapi.co/api/v2/pokemon/459/"
  },
  {
   "name": "abomasnow",
   "url": "https://pokeapi.co/api/v2/pokemon/460/"
  },
  {
   "name": "weavile",
   "url": "https://pokeapi.co/api/v2/pokemon/461/"
  },
  {
   "name": "lickilicky",
   "url": "https://pokeapi.co/api/v2/pokemon/463/"
  },
  {
   "name": "rhyperior",
   "url": "https://pokeapi.co/api/v2/pokemon/464/"
  },
  {
   "name": "tangrowth",
   "url": "https://pokeapi.co/api/v2/pokemon/465/"
  },
  {
   "name": "yanmega",
   "url": "https://pokeapi.co/api/v2/pokemon/469/"
  },
  {
   "name": "leafeon",
   "url": "https://pokeapi.co/api/v2/pokemon/470/"
  },
  {
   "name": "gliscor",
   "url": "https://pokeapi.co/api/v2/pokemon/472/"
  },
  {
   "name": "gallade",
   "url": "https://pokeapi.co/api/v2/pokemon/475/"
  },
  {
   "name": "darkrai",
   "url": "https://pokeapi.co/api/v2/pokemon/491/"
  },
  {
   "name": "shaymin-land",
   "url": "https://pokeapi.co/api/v2/pokemon/492/"
  },
  {
   "name": "arceus",
   "url": "https://pokeapi.co/api/v2/pokemon/493/"
  },
  {
   "name": "snivy",
   "url": "https://pokeapi.co/api/v2/pokemon/495/"
  },
  {
   "name": "servine",
   "url": "https://pokeapi.co/api/v2/pokemon/496/"
  },
  {
   "name": "serperior",
   "url": "https://pokeapi.co/api/v2/pokemon/497/"
  },
  {
   "name": "oshawott",
   "url": "https://pokeapi.co/api/v2/pokemon/501/"
  },
  {
   "name": "dewott",
   "url": "https://pokeapi.co/api/v2/pokemon/502/"
  },
  {
   "name": "samurott",
   "url": "https://pokeapi.co/api/v2/pokemon/503/"
  },
  {
   "name": "patrat",
   "url": "https://pokeapi.co/api/v2/pokemon/504/"
  },
  {
   "name": "watchog",
   "url": "https://pokeapi.co/api/v2/pokemon/505/"
  },
  {
   "name": "drilbur",
   "url": "https://pokeapi.co/api/v2/pokemon/529/"
  },
  {
   "name": "excadrill",
   "url": "https://pokeapi.co/api/v2/pokemon/530/"
  },
  {
   "name": "leavanny",
   "url": "https://pokeapi.co/api/v2/pokemon/542/"
  },
  {
   "name": "scolipede",
   "url": "https://pokeapi.co/api/v2/pokemon/545/"
  },
  {
   "name": "lilligant",
   "url": "https://pokeapi.co/api/v2/pokemon/549/"
  },
  {
   "name": "dwebble",
   "url": "https://pokeapi.co/api/v2/pokemon/557/"
  },
  {
   "name": "crustle",
   "url": "https://pokeapi.co/api/v2/pokemon/558/"
  },
  {
   "name": "scrafty",
   "url": "https://pokeapi.co/api/v2/pokemon/560/"
  },
  {
   "name": "zorua",
   "url": "https://pokeapi.co/api/v2/pokemon/570/"
  },
  {
   "name": "zoroark",
   "url": "https://pokeapi.co/api/v2/pokemon/571/"
  },
  {
   "name": "sawsbuck",
   "url": "https://pokeapi.co/api/v2/pokemon/586/"
  },
  {
   "name": "karrablast",
   "url": "https://pokeapi.co/api/v2/pokemon/588/"
  },
  {
   "name": "escavalier",
   "url": "https://pokeapi.co/api/v2/pokemon/589/"
  },
  {
   "name": "ferrothorn",
   "url": "https://pokeapi.co/api/v2/pokemon/598/"
  },
  {
   "name": "axew",
   "url": "https://pokeapi.co/api/v2/pokemon/610/"
  },
  {
   "name": "fraxure",
   "url": "https://pokeapi.co/api/v2/pokemon/611/"
  },
  {
   "name": "haxorus",
   "url": "https://pokeapi.co/api/v2/pokemon/612/"
  },
  {
   "name": "beartic",
   "url": "https://pokeapi.co/api/v2/pokemon/614/"
  },
  {
   "name": "mienfoo",
   "url": "https://pokeapi.co/api/v2/pokemon/619/"
  },
  {
   "name": "mienshao",
   "url": "https://pokeapi.co/api/v2/pokemon/620/"
  },
  {
   "name": "pawniard",
   "url": "https://pokeapi.co/api/v2/pokemon/624/"
  },
  {
   "name": "bisharp",
   "url": "https://pokeapi.co/api/v2/pokemon/625/"
  },
  {
   "name": "bouffalant",
   "url": "https://pokeapi.co/api/v2/pokemon/626/"
  },
  {
   "name": "cobalion",
   "url": "https://pokeapi.co/api/v2/pokemon/638/"
  },
  {
   "name": "terrakion",
   "url": "https://pokeapi.co/api/v2/pokemon/639/"
  },
  {
   "name": "virizion",
   "url": "https://pokeapi.co/api/v2/pokemon/640/"
  },
  {
   "name": "landorus-incarnate",
   "url": "https://pokeapi.co/api/v2/pokemon/645/"
  },
  {
   "name": "keldeo-ordinary",
   "url": "https://pokeapi.co/api/v2/pokemon/647/"
  },
  {
   "name": "meloetta-aria",
   "url": "https://pokeapi.co/api/v2/pokemon/648/"
  },
  {
   "name": "chespin",
   "url": "https://pokeapi.co/api/v2/pokemon/650/"
  },
  {
   "name": "quilladin",
   "url": "https://pokeapi.co/api/v2/pokemon/651/"
  },
  {
   "name": "chesnaught",
   "url": "https://pokeapi.co/api/v2/pokemon/652/"
  },
  {
   "name": "frogadier",
   "url": "https://pokeapi.co/api/v2/pokemon/657/"
  },
  {
   "name": "greninja",
   "url": "https://pokeapi.co/api/v2/pokemon/658/"
  },
  {
   "name": "bunnelby",
   "url": "https://pokeapi.co/api/v2/pokemon/659/"
  },
  {
   "name": "diggersby",
   "url": "https://pokeapi.co/api/v2/pokemon/660/"
  },
  {
   "name": "fletchling",
   "url": "https://pokeapi.co/api/v2/pokemon/661/"
  },
  {
   "name": "fletchinder",
   "url": "https://pokeapi.co/api/v2/pokemon/662/"
  },
  {
   "name": "talonflame",
   "url": "https://pokeapi.co/api/v2/pokemon/663/"
  },
  {
   "name": "pancham",
   "url": "https://pokeapi.co/api/v2/pokemon/674/"
  },
  {
   "name": "pangoro",
   "url": "https://pokeapi.co/api/v2/pokemon/675/"
  },
  {
   "name": "honedge",
   "url": "https://pokeapi.co/api/v2/pokemon/679/"
  },
  {
   "name": "doublade",
   "url": "https://pokeapi.co/api/v2/pokemon/680/"
  },
  {
   "name": "aegislash-shield",
   "url": "https://pokeapi.co/api/v2/pokemon/681/"
  },
  {
   "name": "binacle",
   "url": "https://pokeapi.co/api/v2/pokemon/688/"
  },
  {
   "name": "barbaracle",
   "url": "https://pokeapi.co/api/v2/pokemon/689/"
  },
  {
   "name": "clauncher",
   "url": "https://pokeapi.co/api/v2/pokemon/692/"
  },
  {
   "name": "clawitzer",
   "url": "https://pokeapi.co/api/v2/pokemon/693/"
  },
  {
   "name": "hawlucha",
   "url": "https://pokeapi.co/api/v2/pokemon/701/"
  },
  {
   "name": "rowlet",
   "url": "https://pokeapi.co/api/v2/pokemon/722/"
  },
  {
   "name": "dartrix",
   "url": "https://pokeapi.co/api/v2/pokemon/723/"
  },
  {
   "name": "decidueye",
   "url": "https://pokeapi.co/api/v2/pokemon/724/"
  },
  {
   "name": "litten",
   "url": "https://pokeapi.co/api/v2/pokemon/725/"
  },
  {
   "name": "torracat",
   "url": "https://pokeapi.co/api/v2/pokemon/726/"
  },
  {
   "name": "incineroar",
   "url": "https://pokeapi.co/api/v2/pokemon/727/"
  },
  {
   "name": "pikipek",
   "url": "https://pokeapi.co/api/v2/pokemon/731/"
  },
  {
   "name": "trumbeak",
   "url": "https://pokeapi.co/api/v2/pokemon/732/"
  },
  {
   "name": "toucannon",
   "url": "https://pokeapi.co/api/v2/pokemon/733/"
  },
  {
   "name": "oricorio-baile",
   "url": "https://pokeapi.co/api/v2/pokemon/741/"
  },
  {
   "name": "rockruff",
   "url": "https://pokeapi.co/api/v2/pokemon/744/"
  },
  {
   "name": "lycanroc-midday",
   "url": "https://pokeapi.co/api/v2/pokemon/745/"
  },
  {
   "name": "fomantis",
   "url": "https://pokeapi.co/api/v2/pokemon/753/"
  },
  {
   "name": "lurantis",
   "url": "https://pokeapi.co/api/v2/pokemon/754/"
  },
  {
   "name": "stufful",
   "url": "https://pokeapi.co/api/v2/pokemon/759/"
  },
  {
   "name": "bewear",
   "url": "https://pokeapi.co/api/v2/pokemon/760/"
  },
  {
   "name": "golisopod",
   "url": "https://pokeapi.co/api/v2/pokemon/768/"
  },
  {
   "name": "type-null",
   "url": "https://pokeapi.co/api/v2/pokemon/772/"
  },
  {
   "name": "silvally",
   "url": "https://pokeapi.co/api/v2/pokemon/773/"
  },
  {
   "name": "komala",
   "url": "https://pokeapi.co/api/v2/pokemon/775/"
  },
  {
   "name": "mimikyu-disguised",
   "url": "https://pokeapi.co/api/v2/pokemon/778/"
  },
  {
   "name": "bruxish",
   "url": "https://pokeapi.co/api/v2/pokemon/779/"
  },
  {
   "name": "dhelmise",
   "url": "https://pokeapi.co/api/v2/pokemon/781/"
  },
  {
   "name": "jangmo-o",
   "url": "https://pokeapi.co/api/v2/pokemon/782/"
  },
  {
   "name": "hakamo-o",
   "url": "https://pokeapi.co/api/v2/pokemon/783/"
  },
  {
   "name": "kommo-o",
   "url": "https://pokeapi.co/api/v2/pokemon/784/"
  },
  {
   "name": "tapu-bulu",
   "url": "https://pokeapi.co/api/v2/pokemon/787/"
  },
  {
   "name": "kartana",
   "url": "https://pokeapi.co/api/v2/pokemon/798/"
  },
  {
   "name": "necrozma",
   "url": "https://pokeapi.co/api/v2/pokemon/800/"
  },
  {
   "name": "grookey",
   "url": "https://pokeapi.co/api/v2/pokemon/810/"
  },
  {
   "name": "thwackey",
   "url": "https://pokeapi.co/api/v2/pokemon/811/"
  },
  {
   "name": "rillaboom",
   "url": "https://pokeapi.co/api/v2/pokemon/812/"
  },
  {
   "name": "raboot",
   "url": "https://pokeapi.co/api/v2/pokemon/814/"
  },
  {
   "name": "cinderace",
   "url": "https://pokeapi.co/api/v2/pokemon/815/"
  },
  {
   "name": "inteleon",
   "url": "https://pokeapi.co/api/v2/pokemon/818/"
  },
  {
   "name": "greedent",
   "url": "https://pokeapi.co/api/v2/pokemon/820/"
  },
  {
   "name": "dubwool",
   "url": "https://pokeapi.co/api/v2/pokemon/832/"
  },
  {
   "name": "drednaw",
   "url": "https://pokeapi.co/api/v2/pokemon/834/"
  },
  {
   "name": "hatterene",
   "url": "https://pokeapi.co/api/v2/pokemon/858/"
  },
  {
   "name": "perrserker",
   "url": "https://pokeapi.co/api/v2/pokemon/863/"
  },
  {
   "name": "sirfetchd",
   "url": "https://pokeapi.co/api/v2/pokemon/865/"
  },
  {
   "name": "falinks",
   "url": "https://pokeapi.co/api/v2/pokemon/870/"
  },
  {
   "name": "duraludon",
   "url": "https://pokeapi.co/api/v2/pokemon/884/"
  },
  {
   "name": "zacian",
   "url": "https://pokeapi.co/api/v2/pokemon/888/"
  },
  {
   "name": "kubfu",
   "url": "https://pokeapi.co/api/v2/pokemon/891/"
  },
  {
   "name": "urshifu-single-strike",
   "url": "https://pokeapi.co/api/v2/pokemon/892/"
  },
  {
   "name": "zarude",
   "url": "https://pokeapi.co/api/v2/pokemon/893/"
  },
  {
   "name": "glastrier",
   "url": "https://pokeapi.co/api/v2/pokemon/896/"
  },
  {
   "name": "kleavor",
   "url": "https://pokeapi.co/api/v2/pokemon/900/"
  },
  {
   "name": "ursaluna",
   "url": "https://pokeapi.co/api/v2/pokemon/901/"
  },
  {
   "name": "sneasler",
   "url": "https://pokeapi.co/api/v2/pokemon/903/"
  },
  {
   "name": "overqwil",
   "url": "https://pokeapi.co/api/v2/pokemon/904/"
  },
  {
   "name": "quaquaval",
   "url": "https://pokeapi.co/api/v2/pokemon/914/"
  },
  {
   "name": "lokix",
   "url": "https://pokeapi.co/api/v2/pokemon/920/"
  },
  {
   "name": "ceruledge",
   "url": "https://pokeapi.co/api/v2/pokemon/937/"
  },
  {
   "name": "shroodle",
   "url": "https://pokeapi.co/api/v2/pokemon/944/"
  },
  {
   "name": "grafaiai",
   "url": "https://pokeapi.co/api/v2/pokemon/945/"
  },
  {
   "name": "klawf",
   "url": "https://pokeapi.co/api/v2/pokemon/950/"
  },
  {
   "name": "tinkatink",
   "url": "https://pokeapi.co/api/v2/pokemon/957/"
  },
  {
   "name": "tinkatuff",
   "url": "https://pokeapi.co/api/v2/pokemon/958/"
  },
  {
   "name": "tinkaton",
   "url": "https://pokeapi.co/api/v2/pokemon/959/"
  },
  {
   "name": "flamigo",
   "url": "https://pokeapi.co/api/v2/pokemon/973/"
  },
  {
   "name": "kingambit",
   "url": "https://pokeapi.co/api/v2/pokemon/983/"
  },
  {
   "name": "iron-hands",
   "url": "https://pokeapi.co/api/v2/pokemon/992/"
  },
  {
   "name": "iron-thorns",
   "url": "https://pokeapi.co/api/v2/pokemon/995/"
  },
  {
   "name": "frigibax",
   "url": "https://pokeapi.co/api/v2/pokemon/996/"
  },
  {
   "name": "arctibax",
   "url": "https://pokeapi.co/api/v2/pokemon/997/"
  },
  {
   "name": "baxcalibur",
   "url": "https://pokeapi.co/api/v2/pokemon/998/"
  },
  {
   "name": "chien-pao",
   "url": "https://pokeapi.co/api/v2/pokemon/1002/"
  },
  {
   "name": "iron-valiant",
   "url": "https://pokeapi.co/api/v2/pokemon/1006/"
  },
  {
   "name": "koraidon",
   "url": "https://pokeapi.co/api/v2/pokemon/1007/"
  },
  {
   "name": "miraidon",
   "url": "https://pokeapi.co/api/v2/pokemon/1008/"
  },
  {
   "name": "iron-leaves",
   "url": "https://pokeapi.co/api/v2/pokemon/1010/"
  },
  {
   "name": "fezandipiti",
   "url": "https://pokeapi.co/api/v2/pokemon/1016/"
  },
  {
   "name": "ogerpon",
   "url": "https://pokeapi.co/api/v2/pokemon/1017/"
  },
  {
   "name": "archaludon",
   "url": "https://pokeapi.co/api/v2/pokemon/1018/"
  },
  {
   "name": "iron-boulder",
   "url": "https://pokeapi.co/api/v2/pokemon/1022/"
  },
  {
   "name": "iron-crown",
   "url": "https://pokeapi.co/api/v2/pokemon/1023/"
  },
  {
   "name": "shaymin-sky",
   "url": "https://pokeapi.co/api/v2/pokemon/10006/"
  },
  {
   "name": "meloetta-pirouette",
   "url": "https://pokeapi.co/api/v2/pokemon/10018/"
  },
  {
   "name": "landorus-therian",
   "url": "https://pokeapi.co/api/v2/pokemon/10021/"
  },
  {
   "name": "keldeo-resolute",
   "url": "https://pokeapi.co/api/v2/pokemon/10024/"
  },
  {
   "name": "aegislash-blade",
   "url": "https://pokeapi.co/api/v2/pokemon/10026/"
  },
  {
   "name": "venusaur-mega",
   "url": "https://pokeapi.co/api/v2/pokemon/10033/"
  },
  {
   "name": "charizard-mega-x",
   "url": "https://pokeapi.co/api/v2/pokemon/10034/"
  },
  {
   "name": "charizard-mega-y",
   "url": "https://pokeapi.co/api/v2/pokemon/10035/"
  },
  {
   "name": "pinsir-mega",
   "url": "https://pokeapi.co/api/v2/pokemon/10040/"
  },
  {
   "name": "scizor-mega",
   "url": "https://pokeapi.co/api/v2/pokemon/10046/"
  },
  {
   "name": "heracross-mega",
   "url": "https://pokeapi.co/api/v2/pokemon/10047/"
  },
  {
   "name": "blaziken-mega",
   "url": "https://pokeapi.co/api/v2/pokemon/10050/"
  },
  {
   "name": "mawile-mega",
   "url": "https://pokeapi.co/api/v2/pokemon/10052/"
  },
  {
   "name": "absol-mega",
   "url": "https://pokeapi.co/api/v2/pokemon/10057/"
  },
  {
   "name": "garchomp-mega",
   "url": "https://pokeapi.co/api/v2/pokemon/10058/"
  },
  {
   "name": "lucario-mega",
   "url": "https://pokeapi.co/api/v2/pokemon/10059/"
  },
  {
   "name": "abomasnow-mega",
   "url": "https://pokeapi.co/api/v2/pokemon/10060/"
  },
  {
   "name": "sceptile-mega",
   "url": "https://pokeapi.co/api/v2/pokemon/10065/"
  },
  {
   "name": "gallade-mega",
   "url": "https://pokeapi.co/api/v2/pokemon/10068/"
  },
  {
   "name": "groudon-primal",
   "url": "https://pokeapi.co/api/v2/pokemon/10078/"
  },
  {
   "name": "rayquaza-mega",
   "url": "https://pokeapi.co/api/v2/pokemon/10079/"
  },
  {
   "name": "beedrill-mega",
   "url": "https://pokeapi.co/api/v2/pokemon/10090/"
  },
  {
   "name": "raticate-alola",
   "url": "https://pokeapi.co/api/v2/pokemon/10092/"
  },
  {
   "name": "raticate-totem-alola",
   "url": "https://pokeapi.co/api/v2/pokemon/10093/"
  },
  {
   "name": "sandshrew-alola",
   "url": "https://pokeapi.co/api/v2/pokemon/10101/"
  },
  {
   "name": "sandslash-alola",
   "url": "https://pokeapi.co/api/v2/pokemon/10102/"
  },
  {
   "name": "diglett-alola",
   "url": "https://pokeapi.co/api/v2/pokemon/10105/"
  },
  {
   "name": "dugtrio-alola",
   "url": "https://pokeapi.co/api/v2/pokemon/10106/"
  },
  {
   "name": "exeggutor-alola",
   "url": "https://pokeapi.co/api/v2/pokemon/10114/"
  },
  {
   "name": "marowak-alola",
   "url": "https://pokeapi.co/api/v2/pokemon/10115/"
  },
  {
   "name": "greninja-ash",
   "url": "https://pokeapi.co/api/v2/pokemon/10117/"
  },
  {
   "name": "oricorio-pom-pom",
   "url": "https://pokeapi.co/api/v2/pokemon/10123/"
  },
  {
   "name": "oricorio-pau",
   "url": "https://pokeapi.co/api/v2/pokemon/10124/"
  },
  {
   "name": "oricorio-sensu",
   "url": "https://pokeapi.co/api/v2/pokemon/10125/"
  },
  {
   "name": "lycanroc-midnight",
   "url": "https://pokeapi.co/api/v2/pokemon/10126/"
  },
  {
   "name": "lurantis-totem",
   "url": "https://pokeapi.co/api/v2/pokemon/10128/"
  },
  {
   "name": "mimikyu-busted",
   "url": "https://pokeapi.co/api/v2/pokemon/10143/"
  },
  {
   "name": "mimikyu-totem-disguised",
   "url": "https://pokeapi.co/api/v2/pokemon/10144/"
  },
  {
   "name": "mimikyu-totem-busted",
   "url": "https://pokeapi.co/api/v2/pokemon/10145/"
  },
  {
   "name": "kommo-o-totem",
   "url": "https://pokeapi.co/api/v2/pokemon/10146/"
  },
  {
   "name": "marowak-totem",
   "url": "https://pokeapi.co/api/v2/pokemon/10149/"
  },
  {
   "name": "rockruff-own-tempo",
   "url": "https://pokeapi.co/api/v2/pokemon/10151/"
  },
  {
   "name": "lycanroc-dusk",
   "url": "https://pokeapi.co/api/v2/pokemon/10152/"
  },
  {
   "name": "necrozma-dusk",
   "url": "https://pokeapi.co/api/v2/pokemon/10155/"
  },
  {
   "name": "necrozma-dawn",
   "url": "https://pokeapi.co/api/v2/pokemon/10156/"
  },
  {
   "name": "necrozma-ultra",
   "url": "https://pokeapi.co/api/v2/pokemon/10157/"
  },
  {
   "name": "meowth-galar",
   "url": "https://pokeapi.co/api/v2/pokemon/10161/"
  },
  {
   "name": "rapidash-galar",
   "url": "https://pokeapi.co/api/v2/pokemon/10163/"
  },
  {
   "name": "farfetchd-galar",
   "url": "https://pokeapi.co/api/v2/pokemon/10166/"
  },
  {
   "name": "zacian-crowned",
   "url": "https://pokeapi.co/api/v2/pokemon/10188/"
  },
  {
   "name": "urshifu-rapid-strike",
   "url": "https://pokeapi.co/api/v2/pokemon/10191/"
  },
  {
   "name": "zarude-dada",
   "url": "https://pokeapi.co/api/v2/pokemon/10192/"
  },
  {
   "name": "calyrex-ice",
   "url": "https://pokeapi.co/api/v2/pokemon/10193/"
  },
  {
   "name": "qwilfish-hisui",
   "url": "https://pokeapi.co/api/v2/pokemon/10234/"
  },
  {
   "name": "sneasel-hisui",
   "url": "https://pokeapi.co/api/v2/pokemon/10235/"
  },
  {
   "name": "samurott-hisui",
   "url": "https://pokeapi.co/api/v2/pokemon/10236/"
  },
  {
   "name": "lilligant-hisui",
   "url": "https://pokeapi.co/api/v2/pokemon/10237/"
  },
  {
   "name": "zoroark-hisui",
   "url": "https://pokeapi.co/api/v2/pokemon/10239/"
  },
  {
   "name": "decidueye-hisui",
   "url": "https://pokeapi.co/api/v2/pokemon/10244/"
  },
  {
   "name": "koraidon-limited-build",
   "url": "https://pokeapi.co/api/v2/pokemon/10264/"
  },
  {
   "name": "koraidon-sprinting-build",
   "url": "https://pokeapi.co/api/v2/pokemon/10265/"
  },
  {
   "name": "koraidon-swimming-build",
   "url": "https://pokeapi.co/api/v2/pokemon/10266/"
  },
  {
   "name": "koraidon-gliding-build",
   "url": "https://pokeapi.co/api/v2/pokemon/10267/"
  },
  {
   "name": "miraidon-low-power-mode",
   "url": "https://pokeapi.co/api/v2/pokemon/10268/"
  },
  {
   "name": "miraidon-drive-mode",
   "url": "https://pokeapi.co/api/v2/pokemon/10269/"
  },
  {
   "name": "miraidon-aquatic-mode",
   "url": "https://pokeapi.co/api/v2/pokemon/10270/"
  },
  {
   "name": "miraidon-glide-mode",
   "url": "https://pokeapi.co/api/v2/pokemon/10271/"
  },
  {
   "name": "ursaluna-bloodmoon",
   "url": "https://pokeapi.co/api/v2/pokemon/10272/"
  },
  {
   "name": "ogerpon-wellspring-mask",
   "url": "https://pokeapi.co/api/v2/pokemon/10273/"
  },
  {
   "name": "ogerpon-hearthflame-mask",
   "url": "https://pokeapi.co/api/v2/pokemon/10274/"
  },
  {
   "name": "ogerpon-cornerstone-mask",
   "url": "https://pokeapi.co/api/v2/pokemon/10275/"
  }
 ],
 "machines": [
  {
   "machine": {
    "url": "https://pokeapi.co/api/v2/machine/46/"
   },
   "version_group": {
    "name": "red-blue",
    "url": "https://pokeapi.co/api/v2/version-group/1/"
   }
  },
  {
   "machine": {
    "url": "https://pokeapi.co/api/v2/machine/47/"
   },
   "version_group": {
    "name": "yellow",
    "url": "https://pokeapi.co/api/v2/version-group/2/"
   }
  },
  {
   "machine": {
    "url": "https://pokeapi.co/api/v2/machine/66/"
   },
   "version_group": {
    "name": "red-green-japan",
    "url": "https://pokeapi.co/api/v2/version-group/28/"
   }
  },
  {
   "machine": {
    "url": "https://pokeapi.co/api/v2/machine/67/"
   },
   "version_group": {
    "name": "blue-japan",
    "url": "https://pokeapi.co/api/v2/version-group/29/"
   }
  },
  {
   "machine": {
    "url": "https://pokeapi.co/api/v2/machine/1352/"
   },
   "version_group": {
    "name": "diamond-pearl",
    "url": "https://pokeapi.co/api/v2/version-group/8/"
   }
  },
  {
   "machine": {
    "url": "https://pokeapi.co/api/v2/machine/1353/"
   },
   "version_group": {
    "name": "platinum",
    "url": "https://pokeapi.co/api/v2/version-group/9/"
   }
  },
  {
   "machine": {
    "url": "https://pokeapi.co/api/v2/machine/1354/"
   },
   "version_group": {
    "name": "heartgold-soulsilver",
    "url": "https://pokeapi.co/api/v2/version-group/10/"
   }
  },
  {
   "machine": {
    "url": "https://pokeapi.co/api/v2/machine/1355/"
   },
   "version_group": {
    "name": "black-white",
    "url": "https://pokeapi.co/api/v2/version-group/11/"
   }
  },
  {
   "machine": {
    "url": "https://pokeapi.co/api/v2/machine/1356/"
   },
   "version_group": {
    "name": "black-2-white-2",
    "url": "https://pokeapi.co/api/v2/version-group/14/"
   }
  },
  {
   "machine": {
    "url": "https://pokeapi.co/api/v2/machine/1357/"
   },
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  },
  {
   "machine": {
    "url": "https://pokeapi.co/api/v2/machine/1358/"
   },
   "version_group": {
    "name": "omega-ruby-alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/16/"
   }
  },
  {
   "machine": {
    "url": "https://pokeapi.co/api/v2/machine/1359/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "machine": {
    "url": "https://pokeapi.co/api/v2/machine/1360/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "machine": {
    "url": "https://pokeapi.co/api/v2/machine/1577/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "machine": {
    "url": "https://pokeapi.co/api/v2/machine/1903/"
   },
   "version_group": {
    "name": "scarlet-violet",
    "url": "https://pokeapi.co/api/v2/version-group/25/"
   }
  },
  {
   "machine": {
    "url": "https://pokeapi.co/api/v2/machine/2111/"
   },
   "version_group": {
    "name": "the-indigo-disk",
    "url": "https://pokeapi.co/api/v2/version-group/27/"
   }
  }
 ],
 "meta": {
  "ailment": {
   "name": "none",
   "url": "https://pokeapi.co/api/v2/move-ailment/0/"
  },
  "ailment_chance": 0,
  "category": {
   "name": "net-good-stats",
   "url": "https://pokeapi.co/api/v2/move-category/2/"
  },
  "crit_rate": 0,
  "drain": 0,
  "flinch_chance": 0,
  "healing": 0,
  "max_hits": null,
  "max_turns": null,
  "min_hits": null,
  "min_turns": null,
  "stat_chance": 0
 },
 "name": "swords-dance",
 "names": [
  {
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "name": "つるぎのまい"
  },
  {
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "name": "칼춤"
  },
  {
   "language": {
    "name": "zh-hant",
    "url": "https://pokeapi.co/api/v2/language/4/"
   },
   "name": "劍舞"
  },
  {
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "name": "Danse Lames"
  },
  {
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "name": "Schwerttanz"
  },
  {
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "name": "Danza Espada"
  },
  {
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "name": "Danzaspada"
  },
  {
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "name": "Swords Dance"
  },
  {
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "name": "つるぎのまい"
  },
  {
   "language": {
    "name": "zh-hans",
    "url": "https://pokeapi.co/api/v2/language/12/"
   },
   "name": "剑舞"
  }
 ],
 "past_values": [
  {
   "accuracy": null,
   "effect_chance": null,
   "effect_entries": [],
   "power": null,
   "pp": 30,
   "type": null,
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  }
 ],
 "power": null,
 "pp": 20,
 "priority": 0,
 "stat_changes": [
  {
   "change": 2,
   "stat": {
    "name": "attack",
    "url": "https://pokeapi.co/api/v2/stat/2/"
   }
  }
 ],
 "super_contest_effect": {
  "url": "https://pokeapi.co/api/v2/super-contest-effect/11/"
 },
 "target": {
  "name": "user",
  "url": "https://pokeapi.co/api/v2/move-target/7/"
 },
 "type": {
  "name": "normal",
  "url": "https://pokeapi.co/api/v2/type/1/"
 }
}

```

`https://pokeapi.co/api/v2/type/7` que vai ter um retorno similar a esses:

```json
{
 "damage_relations": {
  "double_damage_from": [
   {
    "name": "flying",
    "url": "https://pokeapi.co/api/v2/type/3/"
   },
   {
    "name": "rock",
    "url": "https://pokeapi.co/api/v2/type/6/"
   },
   {
    "name": "fire",
    "url": "https://pokeapi.co/api/v2/type/10/"
   }
  ],
  "double_damage_to": [
   {
    "name": "grass",
    "url": "https://pokeapi.co/api/v2/type/12/"
   },
   {
    "name": "psychic",
    "url": "https://pokeapi.co/api/v2/type/14/"
   },
   {
    "name": "dark",
    "url": "https://pokeapi.co/api/v2/type/17/"
   }
  ],
  "half_damage_from": [
   {
    "name": "fighting",
    "url": "https://pokeapi.co/api/v2/type/2/"
   },
   {
    "name": "ground",
    "url": "https://pokeapi.co/api/v2/type/5/"
   },
   {
    "name": "grass",
    "url": "https://pokeapi.co/api/v2/type/12/"
   }
  ],
  "half_damage_to": [
   {
    "name": "fighting",
    "url": "https://pokeapi.co/api/v2/type/2/"
   },
   {
    "name": "flying",
    "url": "https://pokeapi.co/api/v2/type/3/"
   },
   {
    "name": "poison",
    "url": "https://pokeapi.co/api/v2/type/4/"
   },
   {
    "name": "ghost",
    "url": "https://pokeapi.co/api/v2/type/8/"
   },
   {
    "name": "steel",
    "url": "https://pokeapi.co/api/v2/type/9/"
   },
   {
    "name": "fire",
    "url": "https://pokeapi.co/api/v2/type/10/"
   },
   {
    "name": "fairy",
    "url": "https://pokeapi.co/api/v2/type/18/"
   }
  ],
  "no_damage_from": [],
  "no_damage_to": []
 },
 "game_indices": [
  {
   "game_index": 7,
   "generation": {
    "name": "generation-i",
    "url": "https://pokeapi.co/api/v2/generation/1/"
   }
  },
  {
   "game_index": 7,
   "generation": {
    "name": "generation-ii",
    "url": "https://pokeapi.co/api/v2/generation/2/"
   }
  },
  {
   "game_index": 6,
   "generation": {
    "name": "generation-iii",
    "url": "https://pokeapi.co/api/v2/generation/3/"
   }
  },
  {
   "game_index": 6,
   "generation": {
    "name": "generation-iv",
    "url": "https://pokeapi.co/api/v2/generation/4/"
   }
  },
  {
   "game_index": 6,
   "generation": {
    "name": "generation-v",
    "url": "https://pokeapi.co/api/v2/generation/5/"
   }
  },
  {
   "game_index": 6,
   "generation": {
    "name": "generation-vi",
    "url": "https://pokeapi.co/api/v2/generation/6/"
   }
  }
 ],
 "generation": {
  "name": "generation-i",
  "url": "https://pokeapi.co/api/v2/generation/1/"
 },
 "id": 7,
 "move_damage_class": {
  "name": "physical",
  "url": "https://pokeapi.co/api/v2/move-damage-class/2/"
 },
 "moves": [
  {
   "name": "twineedle",
   "url": "https://pokeapi.co/api/v2/move/41/"
  },
  {
   "name": "pin-missile",
   "url": "https://pokeapi.co/api/v2/move/42/"
  },
  {
   "name": "string-shot",
   "url": "https://pokeapi.co/api/v2/move/81/"
  },
  {
   "name": "leech-life",
   "url": "https://pokeapi.co/api/v2/move/141/"
  },
  {
   "name": "spider-web",
   "url": "https://pokeapi.co/api/v2/move/169/"
  },
  {
   "name": "fury-cutter",
   "url": "https://pokeapi.co/api/v2/move/210/"
  },
  {
   "name": "megahorn",
   "url": "https://pokeapi.co/api/v2/move/224/"
  },
  {
   "name": "tail-glow",
   "url": "https://pokeapi.co/api/v2/move/294/"
  },
  {
   "name": "silver-wind",
   "url": "https://pokeapi.co/api/v2/move/318/"
  },
  {
   "name": "signal-beam",
   "url": "https://pokeapi.co/api/v2/move/324/"
  },
  {
   "name": "u-turn",
   "url": "https://pokeapi.co/api/v2/move/369/"
  },
  {
   "name": "x-scissor",
   "url": "https://pokeapi.co/api/v2/move/404/"
  },
  {
   "name": "bug-buzz",
   "url": "https://pokeapi.co/api/v2/move/405/"
  },
  {
   "name": "bug-bite",
   "url": "https://pokeapi.co/api/v2/move/450/"
  },
  {
   "name": "attack-order",
   "url": "https://pokeapi.co/api/v2/move/454/"
  },
  {
   "name": "defend-order",
   "url": "https://pokeapi.co/api/v2/move/455/"
  },
  {
   "name": "heal-order",
   "url": "https://pokeapi.co/api/v2/move/456/"
  },
  {
   "name": "rage-powder",
   "url": "https://pokeapi.co/api/v2/move/476/"
  },
  {
   "name": "quiver-dance",
   "url": "https://pokeapi.co/api/v2/move/483/"
  },
  {
   "name": "struggle-bug",
   "url": "https://pokeapi.co/api/v2/move/522/"
  },
  {
   "name": "steamroller",
   "url": "https://pokeapi.co/api/v2/move/537/"
  },
  {
   "name": "sticky-web",
   "url": "https://pokeapi.co/api/v2/move/564/"
  },
  {
   "name": "fell-stinger",
   "url": "https://pokeapi.co/api/v2/move/565/"
  },
  {
   "name": "powder",
   "url": "https://pokeapi.co/api/v2/move/600/"
  },
  {
   "name": "infestation",
   "url": "https://pokeapi.co/api/v2/move/611/"
  },
  {
   "name": "savage-spin-out--physical",
   "url": "https://pokeapi.co/api/v2/move/634/"
  },
  {
   "name": "savage-spin-out--special",
   "url": "https://pokeapi.co/api/v2/move/635/"
  },
  {
   "name": "first-impression",
   "url": "https://pokeapi.co/api/v2/move/660/"
  },
  {
   "name": "pollen-puff",
   "url": "https://pokeapi.co/api/v2/move/676/"
  },
  {
   "name": "lunge",
   "url": "https://pokeapi.co/api/v2/move/679/"
  },
  {
   "name": "max-flutterby",
   "url": "https://pokeapi.co/api/v2/move/758/"
  },
  {
   "name": "skitter-smack",
   "url": "https://pokeapi.co/api/v2/move/806/"
  },
  {
   "name": "silk-trap",
   "url": "https://pokeapi.co/api/v2/move/852/"
  },
  {
   "name": "pounce",
   "url": "https://pokeapi.co/api/v2/move/884/"
  }
 ],
 "name": "bug",
 "names": [
  {
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "name": "むし"
  },
  {
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "name": "벌레"
  },
  {
   "language": {
    "name": "zh-hant",
    "url": "https://pokeapi.co/api/v2/language/4/"
   },
   "name": "蟲"
  },
  {
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "name": "Insecte"
  },
  {
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "name": "Käfer"
  },
  {
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "name": "Bicho"
  },
  {
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "name": "Coleottero"
  },
  {
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "name": "Bug"
  },
  {
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "name": "むし"
  },
  {
   "language": {
    "name": "zh-hans",
    "url": "https://pokeapi.co/api/v2/language/12/"
   },
   "name": "虫"
  }
 ],
 "past_damage_relations": [
  {
   "damage_relations": {
    "double_damage_from": [
     {
      "name": "flying",
      "url": "https://pokeapi.co/api/v2/type/3/"
     },
     {
      "name": "rock",
      "url": "https://pokeapi.co/api/v2/type/6/"
     },
     {
      "name": "fire",
      "url": "https://pokeapi.co/api/v2/type/10/"
     },
     {
      "name": "poison",
      "url": "https://pokeapi.co/api/v2/type/4/"
     }
    ],
    "double_damage_to": [
     {
      "name": "grass",
      "url": "https://pokeapi.co/api/v2/type/12/"
     },
     {
      "name": "psychic",
      "url": "https://pokeapi.co/api/v2/type/14/"
     },
     {
      "name": "poison",
      "url": "https://pokeapi.co/api/v2/type/4/"
     }
    ],
    "half_damage_from": [
     {
      "name": "fighting",
      "url": "https://pokeapi.co/api/v2/type/2/"
     },
     {
      "name": "ground",
      "url": "https://pokeapi.co/api/v2/type/5/"
     },
     {
      "name": "grass",
      "url": "https://pokeapi.co/api/v2/type/12/"
     }
    ],
    "half_damage_to": [
     {
      "name": "fighting",
      "url": "https://pokeapi.co/api/v2/type/2/"
     },
     {
      "name": "flying",
      "url": "https://pokeapi.co/api/v2/type/3/"
     },
     {
      "name": "ghost",
      "url": "https://pokeapi.co/api/v2/type/8/"
     },
     {
      "name": "fire",
      "url": "https://pokeapi.co/api/v2/type/10/"
     }
    ],
    "no_damage_from": [],
    "no_damage_to": []
   },
   "generation": {
    "name": "generation-i",
    "url": "https://pokeapi.co/api/v2/generation/1/"
   }
  }
 ],
 "pokemon": [
  {
   "pokemon": {
    "name": "caterpie",
    "url": "https://pokeapi.co/api/v2/pokemon/10/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "metapod",
    "url": "https://pokeapi.co/api/v2/pokemon/11/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "butterfree",
    "url": "https://pokeapi.co/api/v2/pokemon/12/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "weedle",
    "url": "https://pokeapi.co/api/v2/pokemon/13/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "kakuna",
    "url": "https://pokeapi.co/api/v2/pokemon/14/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "beedrill",
    "url": "https://pokeapi.co/api/v2/pokemon/15/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "paras",
    "url": "https://pokeapi.co/api/v2/pokemon/46/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "parasect",
    "url": "https://pokeapi.co/api/v2/pokemon/47/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "venonat",
    "url": "https://pokeapi.co/api/v2/pokemon/48/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "venomoth",
    "url": "https://pokeapi.co/api/v2/pokemon/49/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "scyther",
    "url": "https://pokeapi.co/api/v2/pokemon/123/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "pinsir",
    "url": "https://pokeapi.co/api/v2/pokemon/127/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "ledyba",
    "url": "https://pokeapi.co/api/v2/pokemon/165/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "ledian",
    "url": "https://pokeapi.co/api/v2/pokemon/166/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "spinarak",
    "url": "https://pokeapi.co/api/v2/pokemon/167/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "ariados",
    "url": "https://pokeapi.co/api/v2/pokemon/168/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "yanma",
    "url": "https://pokeapi.co/api/v2/pokemon/193/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "pineco",
    "url": "https://pokeapi.co/api/v2/pokemon/204/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "forretress",
    "url": "https://pokeapi.co/api/v2/pokemon/205/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "scizor",
    "url": "https://pokeapi.co/api/v2/pokemon/212/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "shuckle",
    "url": "https://pokeapi.co/api/v2/pokemon/213/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "heracross",
    "url": "https://pokeapi.co/api/v2/pokemon/214/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "wurmple",
    "url": "https://pokeapi.co/api/v2/pokemon/265/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "silcoon",
    "url": "https://pokeapi.co/api/v2/pokemon/266/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "beautifly",
    "url": "https://pokeapi.co/api/v2/pokemon/267/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "cascoon",
    "url": "https://pokeapi.co/api/v2/pokemon/268/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "dustox",
    "url": "https://pokeapi.co/api/v2/pokemon/269/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "surskit",
    "url": "https://pokeapi.co/api/v2/pokemon/283/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "masquerain",
    "url": "https://pokeapi.co/api/v2/pokemon/284/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "nincada",
    "url": "https://pokeapi.co/api/v2/pokemon/290/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "ninjask",
    "url": "https://pokeapi.co/api/v2/pokemon/291/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "shedinja",
    "url": "https://pokeapi.co/api/v2/pokemon/292/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "volbeat",
    "url": "https://pokeapi.co/api/v2/pokemon/313/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "illumise",
    "url": "https://pokeapi.co/api/v2/pokemon/314/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "anorith",
    "url": "https://pokeapi.co/api/v2/pokemon/347/"
   },
   "slot": 2
  },
  {
   "pokemon": {
    "name": "armaldo",
    "url": "https://pokeapi.co/api/v2/pokemon/348/"
   },
   "slot": 2
  },
  {
   "pokemon": {
    "name": "kricketot",
    "url": "https://pokeapi.co/api/v2/pokemon/401/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "kricketune",
    "url": "https://pokeapi.co/api/v2/pokemon/402/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "burmy",
    "url": "https://pokeapi.co/api/v2/pokemon/412/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "wormadam-plant",
    "url": "https://pokeapi.co/api/v2/pokemon/413/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "mothim",
    "url": "https://pokeapi.co/api/v2/pokemon/414/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "combee",
    "url": "https://pokeapi.co/api/v2/pokemon/415/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "vespiquen",
    "url": "https://pokeapi.co/api/v2/pokemon/416/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "skorupi",
    "url": "https://pokeapi.co/api/v2/pokemon/451/"
   },
   "slot": 2
  },
  {
   "pokemon": {
    "name": "yanmega",
    "url": "https://pokeapi.co/api/v2/pokemon/469/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "sewaddle",
    "url": "https://pokeapi.co/api/v2/pokemon/540/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "swadloon",
    "url": "https://pokeapi.co/api/v2/pokemon/541/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "leavanny",
    "url": "https://pokeapi.co/api/v2/pokemon/542/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "venipede",
    "url": "https://pokeapi.co/api/v2/pokemon/543/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "whirlipede",
    "url": "https://pokeapi.co/api/v2/pokemon/544/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "scolipede",
    "url": "https://pokeapi.co/api/v2/pokemon/545/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "dwebble",
    "url": "https://pokeapi.co/api/v2/pokemon/557/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "crustle",
    "url": "https://pokeapi.co/api/v2/pokemon/558/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "karrablast",
    "url": "https://pokeapi.co/api/v2/pokemon/588/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "escavalier",
    "url": "https://pokeapi.co/api/v2/pokemon/589/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "joltik",
    "url": "https://pokeapi.co/api/v2/pokemon/595/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "galvantula",
    "url": "https://pokeapi.co/api/v2/pokemon/596/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "shelmet",
    "url": "https://pokeapi.co/api/v2/pokemon/616/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "accelgor",
    "url": "https://pokeapi.co/api/v2/pokemon/617/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "durant",
    "url": "https://pokeapi.co/api/v2/pokemon/632/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "larvesta",
    "url": "https://pokeapi.co/api/v2/pokemon/636/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "volcarona",
    "url": "https://pokeapi.co/api/v2/pokemon/637/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "genesect",
    "url": "https://pokeapi.co/api/v2/pokemon/649/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "scatterbug",
    "url": "https://pokeapi.co/api/v2/pokemon/664/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "spewpa",
    "url": "https://pokeapi.co/api/v2/pokemon/665/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "vivillon",
    "url": "https://pokeapi.co/api/v2/pokemon/666/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "grubbin",
    "url": "https://pokeapi.co/api/v2/pokemon/736/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "charjabug",
    "url": "https://pokeapi.co/api/v2/pokemon/737/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "vikavolt",
    "url": "https://pokeapi.co/api/v2/pokemon/738/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "cutiefly",
    "url": "https://pokeapi.co/api/v2/pokemon/742/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "ribombee",
    "url": "https://pokeapi.co/api/v2/pokemon/743/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "dewpider",
    "url": "https://pokeapi.co/api/v2/pokemon/751/"
   },
   "slot": 2
  },
  {
   "pokemon": {
    "name": "araquanid",
    "url": "https://pokeapi.co/api/v2/pokemon/752/"
   },
   "slot": 2
  },
  {
   "pokemon": {
    "name": "wimpod",
    "url": "https://pokeapi.co/api/v2/pokemon/767/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "golisopod",
    "url": "https://pokeapi.co/api/v2/pokemon/768/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "buzzwole",
    "url": "https://pokeapi.co/api/v2/pokemon/794/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "pheromosa",
    "url": "https://pokeapi.co/api/v2/pokemon/795/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "blipbug",
    "url": "https://pokeapi.co/api/v2/pokemon/824/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "dottler",
    "url": "https://pokeapi.co/api/v2/pokemon/825/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "orbeetle",
    "url": "https://pokeapi.co/api/v2/pokemon/826/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "sizzlipede",
    "url": "https://pokeapi.co/api/v2/pokemon/850/"
   },
   "slot": 2
  },
  {
   "pokemon": {
    "name": "centiskorch",
    "url": "https://pokeapi.co/api/v2/pokemon/851/"
   },
   "slot": 2
  },
  {
   "pokemon": {
    "name": "snom",
    "url": "https://pokeapi.co/api/v2/pokemon/872/"
   },
   "slot": 2
  },
  {
   "pokemon": {
    "name": "frosmoth",
    "url": "https://pokeapi.co/api/v2/pokemon/873/"
   },
   "slot": 2
  },
  {
   "pokemon": {
    "name": "kleavor",
    "url": "https://pokeapi.co/api/v2/pokemon/900/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "tarountula",
    "url": "https://pokeapi.co/api/v2/pokemon/917/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "spidops",
    "url": "https://pokeapi.co/api/v2/pokemon/918/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "nymble",
    "url": "https://pokeapi.co/api/v2/pokemon/919/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "lokix",
    "url": "https://pokeapi.co/api/v2/pokemon/920/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "rellor",
    "url": "https://pokeapi.co/api/v2/pokemon/953/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "rabsca",
    "url": "https://pokeapi.co/api/v2/pokemon/954/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "slither-wing",
    "url": "https://pokeapi.co/api/v2/pokemon/988/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "wormadam-sandy",
    "url": "https://pokeapi.co/api/v2/pokemon/10004/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "wormadam-trash",
    "url": "https://pokeapi.co/api/v2/pokemon/10005/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "pinsir-mega",
    "url": "https://pokeapi.co/api/v2/pokemon/10040/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "scizor-mega",
    "url": "https://pokeapi.co/api/v2/pokemon/10046/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "heracross-mega",
    "url": "https://pokeapi.co/api/v2/pokemon/10047/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "beedrill-mega",
    "url": "https://pokeapi.co/api/v2/pokemon/10090/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "vikavolt-totem",
    "url": "https://pokeapi.co/api/v2/pokemon/10122/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "ribombee-totem",
    "url": "https://pokeapi.co/api/v2/pokemon/10150/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "araquanid-totem",
    "url": "https://pokeapi.co/api/v2/pokemon/10153/"
   },
   "slot": 2
  },
  {
   "pokemon": {
    "name": "butterfree-gmax",
    "url": "https://pokeapi.co/api/v2/pokemon/10198/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "orbeetle-gmax",
    "url": "https://pokeapi.co/api/v2/pokemon/10213/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "centiskorch-gmax",
    "url": "https://pokeapi.co/api/v2/pokemon/10220/"
   },
   "slot": 2
  },
  {
   "pokemon": {
    "name": "scolipede-mega",
    "url": "https://pokeapi.co/api/v2/pokemon/10288/"
   },
   "slot": 1
  },
  {
   "pokemon": {
    "name": "golisopod-mega",
    "url": "https://pokeapi.co/api/v2/pokemon/10316/"
   },
   "slot": 1
  }
 ],
 "sprites": {
  "generation-iii": {
   "colosseum": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-iii/colosseum/7.png",
    "symbol_icon": null
   },
   "emerald": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-iii/emerald/7.png",
    "symbol_icon": null
   },
   "firered-leafgreen": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-iii/firered-leafgreen/7.png",
    "symbol_icon": null
   },
   "ruby-sapphire": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-iii/ruby-sapphire/7.png",
    "symbol_icon": null
   },
   "xd": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-iii/xd/7.png",
    "symbol_icon": null
   }
  },
  "generation-iv": {
   "diamond-pearl": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-iv/diamond-pearl/7.png",
    "symbol_icon": null
   },
   "heartgold-soulsilver": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-iv/heartgold-soulsilver/7.png",
    "symbol_icon": null
   },
   "platinum": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-iv/platinum/7.png",
    "symbol_icon": null
   }
  },
  "generation-ix": {
   "scarlet-violet": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-ix/scarlet-violet/7.png",
    "symbol_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-ix/scarlet-violet/small/7.png"
   }
  },
  "generation-v": {
   "black-2-white-2": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-v/black-2-white-2/7.png",
    "symbol_icon": null
   },
   "black-white": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-v/black-white/7.png",
    "symbol_icon": null
   }
  },
  "generation-vi": {
   "omega-ruby-alpha-sapphire": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-vi/omega-ruby-alpha-sapphire/7.png",
    "symbol_icon": null
   },
   "x-y": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-vi/x-y/7.png",
    "symbol_icon": null
   }
  },
  "generation-vii": {
   "lets-go-pikachu-lets-go-eevee": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-vii/lets-go-pikachu-lets-go-eevee/7.png",
    "symbol_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-vii/lets-go-pikachu-lets-go-eevee/small/7.png"
   },
   "sun-moon": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-vii/sun-moon/7.png",
    "symbol_icon": null
   },
   "ultra-sun-ultra-moon": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-vii/ultra-sun-ultra-moon/7.png",
    "symbol_icon": null
   }
  },
  "generation-viii": {
   "brilliant-diamond-shining-pearl": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-viii/brilliant-diamond-shining-pearl/7.png",
    "symbol_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-viii/brilliant-diamond-shining-pearl/small/7.png"
   },
   "legends-arceus": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-viii/legends-arceus/7.png",
    "symbol_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-viii/legends-arceus/small/7.png"
   },
   "sword-shield": {
    "name_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-viii/sword-shield/7.png",
    "symbol_icon": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/types/generation-viii/sword-shield/small/7.png"
   }
  }
 }
}
```

`https://pokeapi.co/api/v2/ability/9` que vai ter um retorno similar a esses:

```json
{
 "effect_changes": [
  {
   "effect_entries": [
    {
     "effect": "N'a pas d'effet en dehors du combat.",
     "language": {
      "name": "fr",
      "url": "https://pokeapi.co/api/v2/language/5/"
     }
    },
    {
     "effect": "Hat außerhalb vom Kampf keinen Effekt.",
     "language": {
      "name": "de",
      "url": "https://pokeapi.co/api/v2/language/6/"
     }
    },
    {
     "effect": "Has no overworld effect.",
     "language": {
      "name": "en",
      "url": "https://pokeapi.co/api/v2/language/9/"
     }
    }
   ],
   "version_group": {
    "name": "emerald",
    "url": "https://pokeapi.co/api/v2/version-group/6/"
   }
  }
 ],
 "effect_entries": [
  {
   "effect": "Quand une capacité crée un contact avec ce Pokémon, le lanceur de cette capacité a 30% de chances d'être paralysé.\n\nLes Pokémon immunisés aux capacités de type Électrik peuvent toujours être paralysés par ce talent.\n\nMonde extérieur: Si le Pokémon en tête d'équipe à ce talent, il y a 50% de chances que les rencontres soient avec un Pokémon de type Électrik, si possible.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "short_effect": "A 30% de chances de paralyser le Pokémon attaquant en cas de contact."
  },
  {
   "effect": "Wenn ein Pokémon mit dieser Fähigkeit von einer Attacke mit Kontakt getroffen wird, besteht eine 30% Chance, dass der Angreifer paralysiert wird.\n\nPokémon die immun gegen Elektro Attacken sind können trotzdem von dieser Fähigkeit paralysiert werden.\n\nAußerhalb vom Kampf: Wenn ein Pokémon mit dieser Fähigkeit an erster Stelle im Team steht, besteht eine 50% Chance einem Elektro Pokémon zu begegnen, falls es welche gibt.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "short_effect": "30% Chance das angreifende Pokémon bei Kontakt zu paralysieren."
  },
  {
   "effect": "Whenever a move makes contact with this Pokémon, the move's user has a 30% chance of being paralyzed.\n\nPokémon that are immune to Electric-type moves can still be paralyzed by this ability.\n\nOverworld: If the lead Pokémon has this ability, there is a 50% chance that encounters will be with an Electric Pokémon, if applicable.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "short_effect": "Has a 30% chance of paralyzing attacking Pokémon on contact."
  }
 ],
 "flavor_text_entries": [
  {
   "flavor_text": "Paralyse au toucher.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "ruby-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/5/"
   }
  },
  {
   "flavor_text": "Paralyzes on contact.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "ruby-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/5/"
   }
  },
  {
   "flavor_text": "Paralyse au toucher.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "emerald",
    "url": "https://pokeapi.co/api/v2/version-group/6/"
   }
  },
  {
   "flavor_text": "Paralyzes on contact.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "emerald",
    "url": "https://pokeapi.co/api/v2/version-group/6/"
   }
  },
  {
   "flavor_text": "Paralyse au toucher.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "firered-leafgreen",
    "url": "https://pokeapi.co/api/v2/version-group/7/"
   }
  },
  {
   "flavor_text": "Paralyzes on contact.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "firered-leafgreen",
    "url": "https://pokeapi.co/api/v2/version-group/7/"
   }
  },
  {
   "flavor_text": "Un contact avec le Pokémon peut paralyser.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "diamond-pearl",
    "url": "https://pokeapi.co/api/v2/version-group/8/"
   }
  },
  {
   "flavor_text": "Contact with the Pokémon\nmay cause paralysis.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "diamond-pearl",
    "url": "https://pokeapi.co/api/v2/version-group/8/"
   }
  },
  {
   "flavor_text": "Un contact avec le Pokémon peut paralyser.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "platinum",
    "url": "https://pokeapi.co/api/v2/version-group/9/"
   }
  },
  {
   "flavor_text": "Contact with the Pokémon\nmay cause paralysis.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "platinum",
    "url": "https://pokeapi.co/api/v2/version-group/9/"
   }
  },
  {
   "flavor_text": "Un contact avec le Pokémon peut paralyser.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "heartgold-soulsilver",
    "url": "https://pokeapi.co/api/v2/version-group/10/"
   }
  },
  {
   "flavor_text": "Contact with the Pokémon\nmay cause paralysis.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "heartgold-soulsilver",
    "url": "https://pokeapi.co/api/v2/version-group/10/"
   }
  },
  {
   "flavor_text": "Un contact avec le Pokémon peut paralyser.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "black-white",
    "url": "https://pokeapi.co/api/v2/version-group/11/"
   }
  },
  {
   "flavor_text": "Contact with the Pokémon\nmay cause paralysis.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "black-white",
    "url": "https://pokeapi.co/api/v2/version-group/11/"
   }
  },
  {
   "flavor_text": "Un contact avec le Pokémon peut paralyser.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "xd",
    "url": "https://pokeapi.co/api/v2/version-group/13/"
   }
  },
  {
   "flavor_text": "Un contact avec le Pokémon peut paralyser.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "black-2-white-2",
    "url": "https://pokeapi.co/api/v2/version-group/14/"
   }
  },
  {
   "flavor_text": "Contact with the Pokémon\nmay cause paralysis.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "black-2-white-2",
    "url": "https://pokeapi.co/api/v2/version-group/14/"
   }
  },
  {
   "flavor_text": "さわった　あいてを\nまひさせる　ことがある。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  },
  {
   "flavor_text": "접촉한 상대를\n마비시킬 때가 있다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  },
  {
   "flavor_text": "Un contact avec le Pokémon\npeut paralyser.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  },
  {
   "flavor_text": "Kann bei Berührung paralysieren.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  },
  {
   "flavor_text": "Puede paralizar al mínimo\ncontacto.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  },
  {
   "flavor_text": "Il contatto fisico con il Pokémon\npuò causare paralisi.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  },
  {
   "flavor_text": "Contact with the Pokémon\nmay cause paralysis.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  },
  {
   "flavor_text": "触った　相手を\nまひさせる　ことがある。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version_group": {
    "name": "x-y",
    "url": "https://pokeapi.co/api/v2/version-group/15/"
   }
  },
  {
   "flavor_text": "さわった　あいてを\nまひさせる　ことがある。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version_group": {
    "name": "omega-ruby-alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/16/"
   }
  },
  {
   "flavor_text": "접촉한 상대를\n마비시킬 때가 있다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version_group": {
    "name": "omega-ruby-alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/16/"
   }
  },
  {
   "flavor_text": "Un contact avec le Pokémon peut paralyser.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "omega-ruby-alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/16/"
   }
  },
  {
   "flavor_text": "Kann bei Berührung paralysieren.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version_group": {
    "name": "omega-ruby-alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/16/"
   }
  },
  {
   "flavor_text": "Puede paralizar al mínimo\ncontacto.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version_group": {
    "name": "omega-ruby-alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/16/"
   }
  },
  {
   "flavor_text": "Il contatto fisico con il Pokémon\npuò causare paralisi.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version_group": {
    "name": "omega-ruby-alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/16/"
   }
  },
  {
   "flavor_text": "Contact with the Pokémon\nmay cause paralysis.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "omega-ruby-alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/16/"
   }
  },
  {
   "flavor_text": "触った　相手を\nまひさせる　ことがある。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version_group": {
    "name": "omega-ruby-alpha-sapphire",
    "url": "https://pokeapi.co/api/v2/version-group/16/"
   }
  },
  {
   "flavor_text": "せいでんきを　からだに　まとい\nさわった　あいてを\nまひさせる　ことがある。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "정전기를 몸에 둘러\n접촉한 상대를\n마비시킬 때가 있다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "身上帶有靜電，\n有時會令接觸到的對手麻痺。",
   "language": {
    "name": "zh-hant",
    "url": "https://pokeapi.co/api/v2/language/4/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "Le Pokémon charge son corps en électricité statique, et tout contact avec lui peut paralyser.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "Kann bei Berührung durch statisch aufgeladenen\nKörper paralysieren.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "La electricidad estática que lo envuelve puede\nparalizar al mínimo contacto.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "Il contatto fisico con il corpo percorso\ndall’elettricità statica del Pokémon può\ncausare paralisi.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "The Pokémon is charged with static electricity, so\ncontact with it may cause paralysis.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "静電気を　体に　まとい\n触った　相手を\nまひさせる　ことがある。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "身上带有静电，\n有时会让接触到的对手麻痹。",
   "language": {
    "name": "zh-hans",
    "url": "https://pokeapi.co/api/v2/language/12/"
   },
   "version_group": {
    "name": "sun-moon",
    "url": "https://pokeapi.co/api/v2/version-group/17/"
   }
  },
  {
   "flavor_text": "せいでんきを　からだに　まとい\nさわった　あいてを\nまひさせる　ことがある。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "정전기를 몸에 둘러\n접촉한 상대를\n마비시킬 때가 있다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "身上帶有靜電，\n有時會令接觸到的對手麻痺。",
   "language": {
    "name": "zh-hant",
    "url": "https://pokeapi.co/api/v2/language/4/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "Le Pokémon charge son corps en électricité statique, et tout contact avec lui peut paralyser.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "Kann bei Berührung durch statisch aufgeladenen\nKörper paralysieren.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "La electricidad estática que lo envuelve puede\nparalizar al mínimo contacto.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "Il contatto fisico con il corpo percorso\ndall’elettricità statica del Pokémon può\ncausare paralisi.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "The Pokémon is charged with static electricity, so\ncontact with it may cause paralysis.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "静電気を　体に　まとい\n触った　相手を\nまひさせる　ことがある。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "身上带有静电，\n有时会让接触到的对手麻痹。",
   "language": {
    "name": "zh-hans",
    "url": "https://pokeapi.co/api/v2/language/12/"
   },
   "version_group": {
    "name": "ultra-sun-ultra-moon",
    "url": "https://pokeapi.co/api/v2/version-group/18/"
   }
  },
  {
   "flavor_text": "せいでんきを　からだに　まとい\nさわった　あいてを\nまひさせる　ことがある。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "정전기를 몸에 둘러\n접촉한 상대를\n마비시킬 때가 있다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "身上帶有靜電，\n有時會令接觸到的對手麻痺。",
   "language": {
    "name": "zh-hant",
    "url": "https://pokeapi.co/api/v2/language/4/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "Le Pokémon charge son corps en électricité statique,\net tout contact avec lui peut paralyser.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "Kann bei Berührung durch statisch aufgeladenen\nKörper paralysieren.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "La electricidad estática que lo envuelve puede\nparalizar al mínimo contacto.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "Il contatto fisico con il corpo percorso\ndall’elettricità statica del Pokémon può\ncausare paralisi.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "The Pokémon is charged with static electricity, so\ncontact with it may cause paralysis.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "静電気を　体に　まとい\n触った　相手を\nまひさせる　ことがある。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "身上带有静电，\n有时会让接触到的对手麻痹。",
   "language": {
    "name": "zh-hans",
    "url": "https://pokeapi.co/api/v2/language/12/"
   },
   "version_group": {
    "name": "lets-go-pikachu-lets-go-eevee",
    "url": "https://pokeapi.co/api/v2/version-group/19/"
   }
  },
  {
   "flavor_text": "せいでんきを　からだに　まとい\nさわった　あいてを\nまひさせる　ことがある。",
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "정전기를 몸에 둘러\n접촉한 상대를\n마비시킬 때가 있다.",
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "身上帶有靜電，\n有時會令接觸到的對手麻痺。",
   "language": {
    "name": "zh-hant",
    "url": "https://pokeapi.co/api/v2/language/4/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "Le Pokémon charge son corps en électricité statique, et tout contact avec lui peut paralyser.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "Kann bei Berührung durch statisch aufgeladenen Körper\nparalysieren.",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "La electricidad estática que lo envuelve puede paralizar\nal mínimo contacto.",
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "Il contatto fisico con il corpo percorso\ndall’elettricità statica del Pokémon può\ncausare paralisi.",
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "The Pokémon is charged with static electricity, so\ncontact with it may cause paralysis.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "静電気を　体に　まとい\n触った　相手を\nまひさせる　ことがある。",
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "身上带有静电，\n有时会让接触到的对手麻痹。",
   "language": {
    "name": "zh-hans",
    "url": "https://pokeapi.co/api/v2/language/12/"
   },
   "version_group": {
    "name": "sword-shield",
    "url": "https://pokeapi.co/api/v2/version-group/20/"
   }
  },
  {
   "flavor_text": "Le Pokémon charge son corps en électricité statique, et tout contact avec lui peut paralyser.",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "version_group": {
    "name": "scarlet-violet",
    "url": "https://pokeapi.co/api/v2/version-group/25/"
   }
  },
  {
   "flavor_text": "The Pokémon is charged with static electricity and may paralyze attackers that make direct contact with it.",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "version_group": {
    "name": "scarlet-violet",
    "url": "https://pokeapi.co/api/v2/version-group/25/"
   }
  }
 ],
 "generation": {
  "name": "generation-iii",
  "url": "https://pokeapi.co/api/v2/generation/3/"
 },
 "id": 9,
 "is_main_series": true,
 "name": "static",
 "names": [
  {
   "language": {
    "name": "ja-hrkt",
    "url": "https://pokeapi.co/api/v2/language/1/"
   },
   "name": "せいでんき"
  },
  {
   "language": {
    "name": "ko",
    "url": "https://pokeapi.co/api/v2/language/3/"
   },
   "name": "정전기"
  },
  {
   "language": {
    "name": "zh-hant",
    "url": "https://pokeapi.co/api/v2/language/4/"
   },
   "name": "靜電"
  },
  {
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   },
   "name": "Statik"
  },
  {
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   },
   "name": "Statik"
  },
  {
   "language": {
    "name": "es",
    "url": "https://pokeapi.co/api/v2/language/7/"
   },
   "name": "Elec. Estática"
  },
  {
   "language": {
    "name": "it",
    "url": "https://pokeapi.co/api/v2/language/8/"
   },
   "name": "Statico"
  },
  {
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   },
   "name": "Static"
  },
  {
   "language": {
    "name": "ja",
    "url": "https://pokeapi.co/api/v2/language/11/"
   },
   "name": "せいでんき"
  },
  {
   "language": {
    "name": "zh-hans",
    "url": "https://pokeapi.co/api/v2/language/12/"
   },
   "name": "静电"
  }
 ],
 "pokemon": [
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pikachu",
    "url": "https://pokeapi.co/api/v2/pokemon/25/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "raichu",
    "url": "https://pokeapi.co/api/v2/pokemon/26/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "voltorb",
    "url": "https://pokeapi.co/api/v2/pokemon/100/"
   },
   "slot": 2
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "electrode",
    "url": "https://pokeapi.co/api/v2/pokemon/101/"
   },
   "slot": 2
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "electabuzz",
    "url": "https://pokeapi.co/api/v2/pokemon/125/"
   },
   "slot": 1
  },
  {
   "is_hidden": true,
   "pokemon": {
    "name": "zapdos",
    "url": "https://pokeapi.co/api/v2/pokemon/145/"
   },
   "slot": 3
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pichu",
    "url": "https://pokeapi.co/api/v2/pokemon/172/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "mareep",
    "url": "https://pokeapi.co/api/v2/pokemon/179/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "flaaffy",
    "url": "https://pokeapi.co/api/v2/pokemon/180/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "ampharos",
    "url": "https://pokeapi.co/api/v2/pokemon/181/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "elekid",
    "url": "https://pokeapi.co/api/v2/pokemon/239/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "electrike",
    "url": "https://pokeapi.co/api/v2/pokemon/309/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "manectric",
    "url": "https://pokeapi.co/api/v2/pokemon/310/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "emolga",
    "url": "https://pokeapi.co/api/v2/pokemon/587/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "stunfisk",
    "url": "https://pokeapi.co/api/v2/pokemon/618/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "toxel",
    "url": "https://pokeapi.co/api/v2/pokemon/848/"
   },
   "slot": 2
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "arctozolt",
    "url": "https://pokeapi.co/api/v2/pokemon/881/"
   },
   "slot": 2
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pawmi",
    "url": "https://pokeapi.co/api/v2/pokemon/921/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "tadbulb",
    "url": "https://pokeapi.co/api/v2/pokemon/938/"
   },
   "slot": 2
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "bellibolt",
    "url": "https://pokeapi.co/api/v2/pokemon/939/"
   },
   "slot": 2
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pikachu-rock-star",
    "url": "https://pokeapi.co/api/v2/pokemon/10080/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pikachu-belle",
    "url": "https://pokeapi.co/api/v2/pokemon/10081/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pikachu-pop-star",
    "url": "https://pokeapi.co/api/v2/pokemon/10082/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pikachu-phd",
    "url": "https://pokeapi.co/api/v2/pokemon/10083/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pikachu-libre",
    "url": "https://pokeapi.co/api/v2/pokemon/10084/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pikachu-cosplay",
    "url": "https://pokeapi.co/api/v2/pokemon/10085/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pikachu-original-cap",
    "url": "https://pokeapi.co/api/v2/pokemon/10094/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pikachu-hoenn-cap",
    "url": "https://pokeapi.co/api/v2/pokemon/10095/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pikachu-sinnoh-cap",
    "url": "https://pokeapi.co/api/v2/pokemon/10096/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pikachu-unova-cap",
    "url": "https://pokeapi.co/api/v2/pokemon/10097/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pikachu-kalos-cap",
    "url": "https://pokeapi.co/api/v2/pokemon/10098/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pikachu-alola-cap",
    "url": "https://pokeapi.co/api/v2/pokemon/10099/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pikachu-partner-cap",
    "url": "https://pokeapi.co/api/v2/pokemon/10148/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pikachu-starter",
    "url": "https://pokeapi.co/api/v2/pokemon/10158/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pikachu-world-cap",
    "url": "https://pokeapi.co/api/v2/pokemon/10160/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "pikachu-gmax",
    "url": "https://pokeapi.co/api/v2/pokemon/10199/"
   },
   "slot": 1
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "voltorb-hisui",
    "url": "https://pokeapi.co/api/v2/pokemon/10231/"
   },
   "slot": 2
  },
  {
   "is_hidden": false,
   "pokemon": {
    "name": "electrode-hisui",
    "url": "https://pokeapi.co/api/v2/pokemon/10232/"
   },
   "slot": 2
  }
 ]
}
```

`https://pokeapi.co/api/v2/growth-rate/medium-slow` que vai ter um retorno similar a esses:

```json
{
 "descriptions": [
  {
   "description": "parabolique",
   "language": {
    "name": "fr",
    "url": "https://pokeapi.co/api/v2/language/5/"
   }
  },
  {
   "description": "mittel langsam",
   "language": {
    "name": "de",
    "url": "https://pokeapi.co/api/v2/language/6/"
   }
  },
  {
   "description": "medium slow",
   "language": {
    "name": "en",
    "url": "https://pokeapi.co/api/v2/language/9/"
   }
  }
 ],
 "formula": "\\frac{6x^3}{5} - 15x^2 + 100x - 140",
 "id": 4,
 "levels": [
  {
   "experience": 0,
   "level": 1
  },
  {
   "experience": 9,
   "level": 2
  },
  {
   "experience": 57,
   "level": 3
  },
  {
   "experience": 96,
   "level": 4
  },
  {
   "experience": 135,
   "level": 5
  },
  {
   "experience": 179,
   "level": 6
  },
  {
   "experience": 236,
   "level": 7
  },
  {
   "experience": 314,
   "level": 8
  },
  {
   "experience": 419,
   "level": 9
  },
  {
   "experience": 560,
   "level": 10
  },
  {
   "experience": 742,
   "level": 11
  },
  {
   "experience": 973,
   "level": 12
  },
  {
   "experience": 1261,
   "level": 13
  },
  {
   "experience": 1612,
   "level": 14
  },
  {
   "experience": 2035,
   "level": 15
  },
  {
   "experience": 2535,
   "level": 16
  },
  {
   "experience": 3120,
   "level": 17
  },
  {
   "experience": 3798,
   "level": 18
  },
  {
   "experience": 4575,
   "level": 19
  },
  {
   "experience": 5460,
   "level": 20
  },
  {
   "experience": 6458,
   "level": 21
  },
  {
   "experience": 7577,
   "level": 22
  },
  {
   "experience": 8825,
   "level": 23
  },
  {
   "experience": 10208,
   "level": 24
  },
  {
   "experience": 11735,
   "level": 25
  },
  {
   "experience": 13411,
   "level": 26
  },
  {
   "experience": 15244,
   "level": 27
  },
  {
   "experience": 17242,
   "level": 28
  },
  {
   "experience": 19411,
   "level": 29
  },
  {
   "experience": 21760,
   "level": 30
  },
  {
   "experience": 24294,
   "level": 31
  },
  {
   "experience": 27021,
   "level": 32
  },
  {
   "experience": 29949,
   "level": 33
  },
  {
   "experience": 33084,
   "level": 34
  },
  {
   "experience": 36435,
   "level": 35
  },
  {
   "experience": 40007,
   "level": 36
  },
  {
   "experience": 43808,
   "level": 37
  },
  {
   "experience": 47846,
   "level": 38
  },
  {
   "experience": 52127,
   "level": 39
  },
  {
   "experience": 56660,
   "level": 40
  },
  {
   "experience": 61450,
   "level": 41
  },
  {
   "experience": 66505,
   "level": 42
  },
  {
   "experience": 71833,
   "level": 43
  },
  {
   "experience": 77440,
   "level": 44
  },
  {
   "experience": 83335,
   "level": 45
  },
  {
   "experience": 89523,
   "level": 46
  },
  {
   "experience": 96012,
   "level": 47
  },
  {
   "experience": 102810,
   "level": 48
  },
  {
   "experience": 109923,
   "level": 49
  },
  {
   "experience": 117360,
   "level": 50
  },
  {
   "experience": 125126,
   "level": 51
  },
  {
   "experience": 133229,
   "level": 52
  },
  {
   "experience": 141677,
   "level": 53
  },
  {
   "experience": 150476,
   "level": 54
  },
  {
   "experience": 159635,
   "level": 55
  },
  {
   "experience": 169159,
   "level": 56
  },
  {
   "experience": 179056,
   "level": 57
  },
  {
   "experience": 189334,
   "level": 58
  },
  {
   "experience": 199999,
   "level": 59
  },
  {
   "experience": 211060,
   "level": 60
  },
  {
   "experience": 222522,
   "level": 61
  },
  {
   "experience": 234393,
   "level": 62
  },
  {
   "experience": 246681,
   "level": 63
  },
  {
   "experience": 259392,
   "level": 64
  },
  {
   "experience": 272535,
   "level": 65
  },
  {
   "experience": 286115,
   "level": 66
  },
  {
   "experience": 300140,
   "level": 67
  },
  {
   "experience": 314618,
   "level": 68
  },
  {
   "experience": 329555,
   "level": 69
  },
  {
   "experience": 344960,
   "level": 70
  },
  {
   "experience": 360838,
   "level": 71
  },
  {
   "experience": 377197,
   "level": 72
  },
  {
   "experience": 394045,
   "level": 73
  },
  {
   "experience": 411388,
   "level": 74
  },
  {
   "experience": 429235,
   "level": 75
  },
  {
   "experience": 447591,
   "level": 76
  },
  {
   "experience": 466464,
   "level": 77
  },
  {
   "experience": 485862,
   "level": 78
  },
  {
   "experience": 505791,
   "level": 79
  },
  {
   "experience": 526260,
   "level": 80
  },
  {
   "experience": 547274,
   "level": 81
  },
  {
   "experience": 568841,
   "level": 82
  },
  {
   "experience": 590969,
   "level": 83
  },
  {
   "experience": 613664,
   "level": 84
  },
  {
   "experience": 636935,
   "level": 85
  },
  {
   "experience": 660787,
   "level": 86
  },
  {
   "experience": 685228,
   "level": 87
  },
  {
   "experience": 710266,
   "level": 88
  },
  {
   "experience": 735907,
   "level": 89
  },
  {
   "experience": 762160,
   "level": 90
  },
  {
   "experience": 789030,
   "level": 91
  },
  {
   "experience": 816525,
   "level": 92
  },
  {
   "experience": 844653,
   "level": 93
  },
  {
   "experience": 873420,
   "level": 94
  },
  {
   "experience": 902835,
   "level": 95
  },
  {
   "experience": 932903,
   "level": 96
  },
  {
   "experience": 963632,
   "level": 97
  },
  {
   "experience": 995030,
   "level": 98
  },
  {
   "experience": 1027103,
   "level": 99
  },
  {
   "experience": 1059860,
   "level": 100
  }
 ],
 "name": "medium-slow",
 "pokemon_species": [
  {
   "name": "bulbasaur",
   "url": "https://pokeapi.co/api/v2/pokemon-species/1/"
  },
  {
   "name": "charmander",
   "url": "https://pokeapi.co/api/v2/pokemon-species/4/"
  },
  {
   "name": "squirtle",
   "url": "https://pokeapi.co/api/v2/pokemon-species/7/"
  },
  {
   "name": "pidgey",
   "url": "https://pokeapi.co/api/v2/pokemon-species/16/"
  },
  {
   "name": "nidoran-f",
   "url": "https://pokeapi.co/api/v2/pokemon-species/29/"
  },
  {
   "name": "nidoran-m",
   "url": "https://pokeapi.co/api/v2/pokemon-species/32/"
  },
  {
   "name": "oddish",
   "url": "https://pokeapi.co/api/v2/pokemon-species/43/"
  },
  {
   "name": "poliwag",
   "url": "https://pokeapi.co/api/v2/pokemon-species/60/"
  },
  {
   "name": "abra",
   "url": "https://pokeapi.co/api/v2/pokemon-species/63/"
  },
  {
   "name": "machop",
   "url": "https://pokeapi.co/api/v2/pokemon-species/66/"
  },
  {
   "name": "bellsprout",
   "url": "https://pokeapi.co/api/v2/pokemon-species/69/"
  },
  {
   "name": "geodude",
   "url": "https://pokeapi.co/api/v2/pokemon-species/74/"
  },
  {
   "name": "venusaur",
   "url": "https://pokeapi.co/api/v2/pokemon-species/3/"
  },
  {
   "name": "charmeleon",
   "url": "https://pokeapi.co/api/v2/pokemon-species/5/"
  },
  {
   "name": "charizard",
   "url": "https://pokeapi.co/api/v2/pokemon-species/6/"
  },
  {
   "name": "wartortle",
   "url": "https://pokeapi.co/api/v2/pokemon-species/8/"
  },
  {
   "name": "blastoise",
   "url": "https://pokeapi.co/api/v2/pokemon-species/9/"
  },
  {
   "name": "pidgeotto",
   "url": "https://pokeapi.co/api/v2/pokemon-species/17/"
  },
  {
   "name": "pidgeot",
   "url": "https://pokeapi.co/api/v2/pokemon-species/18/"
  },
  {
   "name": "nidorina",
   "url": "https://pokeapi.co/api/v2/pokemon-species/30/"
  },
  {
   "name": "nidoqueen",
   "url": "https://pokeapi.co/api/v2/pokemon-species/31/"
  },
  {
   "name": "nidorino",
   "url": "https://pokeapi.co/api/v2/pokemon-species/33/"
  },
  {
   "name": "nidoking",
   "url": "https://pokeapi.co/api/v2/pokemon-species/34/"
  },
  {
   "name": "gloom",
   "url": "https://pokeapi.co/api/v2/pokemon-species/44/"
  },
  {
   "name": "vileplume",
   "url": "https://pokeapi.co/api/v2/pokemon-species/45/"
  },
  {
   "name": "poliwhirl",
   "url": "https://pokeapi.co/api/v2/pokemon-species/61/"
  },
  {
   "name": "poliwrath",
   "url": "https://pokeapi.co/api/v2/pokemon-species/62/"
  },
  {
   "name": "kadabra",
   "url": "https://pokeapi.co/api/v2/pokemon-species/64/"
  },
  {
   "name": "alakazam",
   "url": "https://pokeapi.co/api/v2/pokemon-species/65/"
  },
  {
   "name": "machoke",
   "url": "https://pokeapi.co/api/v2/pokemon-species/67/"
  },
  {
   "name": "machamp",
   "url": "https://pokeapi.co/api/v2/pokemon-species/68/"
  },
  {
   "name": "weepinbell",
   "url": "https://pokeapi.co/api/v2/pokemon-species/70/"
  },
  {
   "name": "victreebel",
   "url": "https://pokeapi.co/api/v2/pokemon-species/71/"
  },
  {
   "name": "graveler",
   "url": "https://pokeapi.co/api/v2/pokemon-species/75/"
  },
  {
   "name": "gastly",
   "url": "https://pokeapi.co/api/v2/pokemon-species/92/"
  },
  {
   "name": "haunter",
   "url": "https://pokeapi.co/api/v2/pokemon-species/93/"
  },
  {
   "name": "gengar",
   "url": "https://pokeapi.co/api/v2/pokemon-species/94/"
  },
  {
   "name": "mew",
   "url": "https://pokeapi.co/api/v2/pokemon-species/151/"
  },
  {
   "name": "chikorita",
   "url": "https://pokeapi.co/api/v2/pokemon-species/152/"
  },
  {
   "name": "cyndaquil",
   "url": "https://pokeapi.co/api/v2/pokemon-species/155/"
  },
  {
   "name": "totodile",
   "url": "https://pokeapi.co/api/v2/pokemon-species/158/"
  },
  {
   "name": "mareep",
   "url": "https://pokeapi.co/api/v2/pokemon-species/179/"
  },
  {
   "name": "hoppip",
   "url": "https://pokeapi.co/api/v2/pokemon-species/187/"
  },
  {
   "name": "sunkern",
   "url": "https://pokeapi.co/api/v2/pokemon-species/191/"
  },
  {
   "name": "murkrow",
   "url": "https://pokeapi.co/api/v2/pokemon-species/198/"
  },
  {
   "name": "gligar",
   "url": "https://pokeapi.co/api/v2/pokemon-species/207/"
  },
  {
   "name": "shuckle",
   "url": "https://pokeapi.co/api/v2/pokemon-species/213/"
  },
  {
   "name": "sneasel",
   "url": "https://pokeapi.co/api/v2/pokemon-species/215/"
  },
  {
   "name": "meganium",
   "url": "https://pokeapi.co/api/v2/pokemon-species/154/"
  },
  {
   "name": "quilava",
   "url": "https://pokeapi.co/api/v2/pokemon-species/156/"
  },
  {
   "name": "typhlosion",
   "url": "https://pokeapi.co/api/v2/pokemon-species/157/"
  },
  {
   "name": "croconaw",
   "url": "https://pokeapi.co/api/v2/pokemon-species/159/"
  },
  {
   "name": "feraligatr",
   "url": "https://pokeapi.co/api/v2/pokemon-species/160/"
  },
  {
   "name": "flaaffy",
   "url": "https://pokeapi.co/api/v2/pokemon-species/180/"
  },
  {
   "name": "ampharos",
   "url": "https://pokeapi.co/api/v2/pokemon-species/181/"
  },
  {
   "name": "bellossom",
   "url": "https://pokeapi.co/api/v2/pokemon-species/182/"
  },
  {
   "name": "politoed",
   "url": "https://pokeapi.co/api/v2/pokemon-species/186/"
  },
  {
   "name": "skiploom",
   "url": "https://pokeapi.co/api/v2/pokemon-species/188/"
  },
  {
   "name": "jumpluff",
   "url": "https://pokeapi.co/api/v2/pokemon-species/189/"
  },
  {
   "name": "sunflora",
   "url": "https://pokeapi.co/api/v2/pokemon-species/192/"
  },
  {
   "name": "celebi",
   "url": "https://pokeapi.co/api/v2/pokemon-species/251/"
  },
  {
   "name": "treecko",
   "url": "https://pokeapi.co/api/v2/pokemon-species/252/"
  },
  {
   "name": "torchic",
   "url": "https://pokeapi.co/api/v2/pokemon-species/255/"
  },
  {
   "name": "mudkip",
   "url": "https://pokeapi.co/api/v2/pokemon-species/258/"
  },
  {
   "name": "lotad",
   "url": "https://pokeapi.co/api/v2/pokemon-species/270/"
  },
  {
   "name": "seedot",
   "url": "https://pokeapi.co/api/v2/pokemon-species/273/"
  },
  {
   "name": "taillow",
   "url": "https://pokeapi.co/api/v2/pokemon-species/276/"
  },
  {
   "name": "whismur",
   "url": "https://pokeapi.co/api/v2/pokemon-species/293/"
  },
  {
   "name": "grovyle",
   "url": "https://pokeapi.co/api/v2/pokemon-species/253/"
  },
  {
   "name": "sceptile",
   "url": "https://pokeapi.co/api/v2/pokemon-species/254/"
  },
  {
   "name": "combusken",
   "url": "https://pokeapi.co/api/v2/pokemon-species/256/"
  },
  {
   "name": "blaziken",
   "url": "https://pokeapi.co/api/v2/pokemon-species/257/"
  },
  {
   "name": "marshtomp",
   "url": "https://pokeapi.co/api/v2/pokemon-species/259/"
  },
  {
   "name": "swampert",
   "url": "https://pokeapi.co/api/v2/pokemon-species/260/"
  },
  {
   "name": "lombre",
   "url": "https://pokeapi.co/api/v2/pokemon-species/271/"
  },
  {
   "name": "ludicolo",
   "url": "https://pokeapi.co/api/v2/pokemon-species/272/"
  },
  {
   "name": "nuzleaf",
   "url": "https://pokeapi.co/api/v2/pokemon-species/274/"
  },
  {
   "name": "shiftry",
   "url": "https://pokeapi.co/api/v2/pokemon-species/275/"
  },
  {
   "name": "swellow",
   "url": "https://pokeapi.co/api/v2/pokemon-species/277/"
  },
  {
   "name": "loudred",
   "url": "https://pokeapi.co/api/v2/pokemon-species/294/"
  },
  {
   "name": "exploud",
   "url": "https://pokeapi.co/api/v2/pokemon-species/295/"
  },
  {
   "name": "sableye",
   "url": "https://pokeapi.co/api/v2/pokemon-species/302/"
  },
  {
   "name": "trapinch",
   "url": "https://pokeapi.co/api/v2/pokemon-species/328/"
  },
  {
   "name": "cacnea",
   "url": "https://pokeapi.co/api/v2/pokemon-species/331/"
  },
  {
   "name": "kecleon",
   "url": "https://pokeapi.co/api/v2/pokemon-species/352/"
  },
  {
   "name": "absol",
   "url": "https://pokeapi.co/api/v2/pokemon-species/359/"
  },
  {
   "name": "spheal",
   "url": "https://pokeapi.co/api/v2/pokemon-species/363/"
  },
  {
   "name": "roselia",
   "url": "https://pokeapi.co/api/v2/pokemon-species/315/"
  },
  {
   "name": "vibrava",
   "url": "https://pokeapi.co/api/v2/pokemon-species/329/"
  },
  {
   "name": "flygon",
   "url": "https://pokeapi.co/api/v2/pokemon-species/330/"
  },
  {
   "name": "cacturne",
   "url": "https://pokeapi.co/api/v2/pokemon-species/332/"
  },
  {
   "name": "sealeo",
   "url": "https://pokeapi.co/api/v2/pokemon-species/364/"
  },
  {
   "name": "walrein",
   "url": "https://pokeapi.co/api/v2/pokemon-species/365/"
  },
  {
   "name": "turtwig",
   "url": "https://pokeapi.co/api/v2/pokemon-species/387/"
  },
  {
   "name": "chimchar",
   "url": "https://pokeapi.co/api/v2/pokemon-species/390/"
  },
  {
   "name": "piplup",
   "url": "https://pokeapi.co/api/v2/pokemon-species/393/"
  },
  {
   "name": "starly",
   "url": "https://pokeapi.co/api/v2/pokemon-species/396/"
  },
  {
   "name": "kricketot",
   "url": "https://pokeapi.co/api/v2/pokemon-species/401/"
  },
  {
   "name": "shinx",
   "url": "https://pokeapi.co/api/v2/pokemon-species/403/"
  },
  {
   "name": "budew",
   "url": "https://pokeapi.co/api/v2/pokemon-species/406/"
  },
  {
   "name": "combee",
   "url": "https://pokeapi.co/api/v2/pokemon-species/415/"
  },
  {
   "name": "chatot",
   "url": "https://pokeapi.co/api/v2/pokemon-species/441/"
  },
  {
   "name": "riolu",
   "url": "https://pokeapi.co/api/v2/pokemon-species/447/"
  },
  {
   "name": "grotle",
   "url": "https://pokeapi.co/api/v2/pokemon-species/388/"
  },
  {
   "name": "torterra",
   "url": "https://pokeapi.co/api/v2/pokemon-species/389/"
  },
  {
   "name": "infernape",
   "url": "https://pokeapi.co/api/v2/pokemon-species/392/"
  },
  {
   "name": "prinplup",
   "url": "https://pokeapi.co/api/v2/pokemon-species/394/"
  },
  {
   "name": "empoleon",
   "url": "https://pokeapi.co/api/v2/pokemon-species/395/"
  },
  {
   "name": "staravia",
   "url": "https://pokeapi.co/api/v2/pokemon-species/397/"
  },
  {
   "name": "staraptor",
   "url": "https://pokeapi.co/api/v2/pokemon-species/398/"
  },
  {
   "name": "kricketune",
   "url": "https://pokeapi.co/api/v2/pokemon-species/402/"
  },
  {
   "name": "luxio",
   "url": "https://pokeapi.co/api/v2/pokemon-species/404/"
  },
  {
   "name": "luxray",
   "url": "https://pokeapi.co/api/v2/pokemon-species/405/"
  },
  {
   "name": "roserade",
   "url": "https://pokeapi.co/api/v2/pokemon-species/407/"
  },
  {
   "name": "vespiquen",
   "url": "https://pokeapi.co/api/v2/pokemon-species/416/"
  },
  {
   "name": "honchkrow",
   "url": "https://pokeapi.co/api/v2/pokemon-species/430/"
  },
  {
   "name": "lucario",
   "url": "https://pokeapi.co/api/v2/pokemon-species/448/"
  },
  {
   "name": "shaymin",
   "url": "https://pokeapi.co/api/v2/pokemon-species/492/"
  },
  {
   "name": "snivy",
   "url": "https://pokeapi.co/api/v2/pokemon-species/495/"
  },
  {
   "name": "tepig",
   "url": "https://pokeapi.co/api/v2/pokemon-species/498/"
  },
  {
   "name": "oshawott",
   "url": "https://pokeapi.co/api/v2/pokemon-species/501/"
  },
  {
   "name": "lillipup",
   "url": "https://pokeapi.co/api/v2/pokemon-species/506/"
  },
  {
   "name": "pidove",
   "url": "https://pokeapi.co/api/v2/pokemon-species/519/"
  },
  {
   "name": "roggenrola",
   "url": "https://pokeapi.co/api/v2/pokemon-species/524/"
  },
  {
   "name": "weavile",
   "url": "https://pokeapi.co/api/v2/pokemon-species/461/"
  },
  {
   "name": "gliscor",
   "url": "https://pokeapi.co/api/v2/pokemon-species/472/"
  },
  {
   "name": "servine",
   "url": "https://pokeapi.co/api/v2/pokemon-species/496/"
  },
  {
   "name": "serperior",
   "url": "https://pokeapi.co/api/v2/pokemon-species/497/"
  },
  {
   "name": "pignite",
   "url": "https://pokeapi.co/api/v2/pokemon-species/499/"
  },
  {
   "name": "emboar",
   "url": "https://pokeapi.co/api/v2/pokemon-species/500/"
  },
  {
   "name": "dewott",
   "url": "https://pokeapi.co/api/v2/pokemon-species/502/"
  },
  {
   "name": "samurott",
   "url": "https://pokeapi.co/api/v2/pokemon-species/503/"
  },
  {
   "name": "stoutland",
   "url": "https://pokeapi.co/api/v2/pokemon-species/508/"
  },
  {
   "name": "tranquill",
   "url": "https://pokeapi.co/api/v2/pokemon-species/520/"
  },
  {
   "name": "unfezant",
   "url": "https://pokeapi.co/api/v2/pokemon-species/521/"
  },
  {
   "name": "boldore",
   "url": "https://pokeapi.co/api/v2/pokemon-species/525/"
  },
  {
   "name": "gigalith",
   "url": "https://pokeapi.co/api/v2/pokemon-species/526/"
  },
  {
   "name": "timburr",
   "url": "https://pokeapi.co/api/v2/pokemon-species/532/"
  },
  {
   "name": "tympole",
   "url": "https://pokeapi.co/api/v2/pokemon-species/535/"
  },
  {
   "name": "sewaddle",
   "url": "https://pokeapi.co/api/v2/pokemon-species/540/"
  },
  {
   "name": "venipede",
   "url": "https://pokeapi.co/api/v2/pokemon-species/543/"
  },
  {
   "name": "sandile",
   "url": "https://pokeapi.co/api/v2/pokemon-species/551/"
  },
  {
   "name": "darumaka",
   "url": "https://pokeapi.co/api/v2/pokemon-species/554/"
  },
  {
   "name": "zorua",
   "url": "https://pokeapi.co/api/v2/pokemon-species/570/"
  },
  {
   "name": "gothita",
   "url": "https://pokeapi.co/api/v2/pokemon-species/574/"
  },
  {
   "name": "solosis",
   "url": "https://pokeapi.co/api/v2/pokemon-species/577/"
  },
  {
   "name": "klink",
   "url": "https://pokeapi.co/api/v2/pokemon-species/599/"
  },
  {
   "name": "litwick",
   "url": "https://pokeapi.co/api/v2/pokemon-species/607/"
  },
  {
   "name": "conkeldurr",
   "url": "https://pokeapi.co/api/v2/pokemon-species/534/"
  },
  {
   "name": "palpitoad",
   "url": "https://pokeapi.co/api/v2/pokemon-species/536/"
  },
  {
   "name": "seismitoad",
   "url": "https://pokeapi.co/api/v2/pokemon-species/537/"
  },
  {
   "name": "swadloon",
   "url": "https://pokeapi.co/api/v2/pokemon-species/541/"
  },
  {
   "name": "leavanny",
   "url": "https://pokeapi.co/api/v2/pokemon-species/542/"
  },
  {
   "name": "whirlipede",
   "url": "https://pokeapi.co/api/v2/pokemon-species/544/"
  },
  {
   "name": "scolipede",
   "url": "https://pokeapi.co/api/v2/pokemon-species/545/"
  },
  {
   "name": "krokorok",
   "url": "https://pokeapi.co/api/v2/pokemon-species/552/"
  },
  {
   "name": "krookodile",
   "url": "https://pokeapi.co/api/v2/pokemon-species/553/"
  },
  {
   "name": "darmanitan",
   "url": "https://pokeapi.co/api/v2/pokemon-species/555/"
  },
  {
   "name": "zoroark",
   "url": "https://pokeapi.co/api/v2/pokemon-species/571/"
  },
  {
   "name": "gothorita",
   "url": "https://pokeapi.co/api/v2/pokemon-species/575/"
  },
  {
   "name": "gothitelle",
   "url": "https://pokeapi.co/api/v2/pokemon-species/576/"
  },
  {
   "name": "duosion",
   "url": "https://pokeapi.co/api/v2/pokemon-species/578/"
  },
  {
   "name": "reuniclus",
   "url": "https://pokeapi.co/api/v2/pokemon-species/579/"
  },
  {
   "name": "klang",
   "url": "https://pokeapi.co/api/v2/pokemon-species/600/"
  },
  {
   "name": "lampent",
   "url": "https://pokeapi.co/api/v2/pokemon-species/608/"
  },
  {
   "name": "mienfoo",
   "url": "https://pokeapi.co/api/v2/pokemon-species/619/"
  },
  {
   "name": "chespin",
   "url": "https://pokeapi.co/api/v2/pokemon-species/650/"
  },
  {
   "name": "fennekin",
   "url": "https://pokeapi.co/api/v2/pokemon-species/653/"
  },
  {
   "name": "froakie",
   "url": "https://pokeapi.co/api/v2/pokemon-species/656/"
  },
  {
   "name": "fletchling",
   "url": "https://pokeapi.co/api/v2/pokemon-species/661/"
  },
  {
   "name": "litleo",
   "url": "https://pokeapi.co/api/v2/pokemon-species/667/"
  },
  {
   "name": "quilladin",
   "url": "https://pokeapi.co/api/v2/pokemon-species/651/"
  },
  {
   "name": "chesnaught",
   "url": "https://pokeapi.co/api/v2/pokemon-species/652/"
  },
  {
   "name": "braixen",
   "url": "https://pokeapi.co/api/v2/pokemon-species/654/"
  },
  {
   "name": "delphox",
   "url": "https://pokeapi.co/api/v2/pokemon-species/655/"
  },
  {
   "name": "frogadier",
   "url": "https://pokeapi.co/api/v2/pokemon-species/657/"
  },
  {
   "name": "greninja",
   "url": "https://pokeapi.co/api/v2/pokemon-species/658/"
  },
  {
   "name": "fletchinder",
   "url": "https://pokeapi.co/api/v2/pokemon-species/662/"
  },
  {
   "name": "talonflame",
   "url": "https://pokeapi.co/api/v2/pokemon-species/663/"
  },
  {
   "name": "pyroar",
   "url": "https://pokeapi.co/api/v2/pokemon-species/668/"
  },
  {
   "name": "rowlet",
   "url": "https://pokeapi.co/api/v2/pokemon-species/722/"
  },
  {
   "name": "litten",
   "url": "https://pokeapi.co/api/v2/pokemon-species/725/"
  },
  {
   "name": "popplio",
   "url": "https://pokeapi.co/api/v2/pokemon-species/728/"
  },
  {
   "name": "bounsweet",
   "url": "https://pokeapi.co/api/v2/pokemon-species/761/"
  },
  {
   "name": "dartrix",
   "url": "https://pokeapi.co/api/v2/pokemon-species/723/"
  },
  {
   "name": "decidueye",
   "url": "https://pokeapi.co/api/v2/pokemon-species/724/"
  },
  {
   "name": "torracat",
   "url": "https://pokeapi.co/api/v2/pokemon-species/726/"
  },
  {
   "name": "incineroar",
   "url": "https://pokeapi.co/api/v2/pokemon-species/727/"
  },
  {
   "name": "primarina",
   "url": "https://pokeapi.co/api/v2/pokemon-species/730/"
  },
  {
   "name": "steenee",
   "url": "https://pokeapi.co/api/v2/pokemon-species/762/"
  },
  {
   "name": "minior",
   "url": "https://pokeapi.co/api/v2/pokemon-species/774/"
  },
  {
   "name": "grookey",
   "url": "https://pokeapi.co/api/v2/pokemon-species/810/"
  },
  {
   "name": "scorbunny",
   "url": "https://pokeapi.co/api/v2/pokemon-species/813/"
  },
  {
   "name": "sobble",
   "url": "https://pokeapi.co/api/v2/pokemon-species/816/"
  },
  {
   "name": "rookidee",
   "url": "https://pokeapi.co/api/v2/pokemon-species/821/"
  },
  {
   "name": "rolycoly",
   "url": "https://pokeapi.co/api/v2/pokemon-species/837/"
  },
  {
   "name": "thwackey",
   "url": "https://pokeapi.co/api/v2/pokemon-species/811/"
  },
  {
   "name": "rillaboom",
   "url": "https://pokeapi.co/api/v2/pokemon-species/812/"
  },
  {
   "name": "raboot",
   "url": "https://pokeapi.co/api/v2/pokemon-species/814/"
  },
  {
   "name": "cinderace",
   "url": "https://pokeapi.co/api/v2/pokemon-species/815/"
  },
  {
   "name": "drizzile",
   "url": "https://pokeapi.co/api/v2/pokemon-species/817/"
  },
  {
   "name": "inteleon",
   "url": "https://pokeapi.co/api/v2/pokemon-species/818/"
  },
  {
   "name": "corvisquire",
   "url": "https://pokeapi.co/api/v2/pokemon-species/822/"
  },
  {
   "name": "corviknight",
   "url": "https://pokeapi.co/api/v2/pokemon-species/823/"
  },
  {
   "name": "carkol",
   "url": "https://pokeapi.co/api/v2/pokemon-species/838/"
  },
  {
   "name": "coalossal",
   "url": "https://pokeapi.co/api/v2/pokemon-species/839/"
  },
  {
   "name": "toxel",
   "url": "https://pokeapi.co/api/v2/pokemon-species/848/"
  },
  {
   "name": "clobbopus",
   "url": "https://pokeapi.co/api/v2/pokemon-species/852/"
  },
  {
   "name": "sprigatito",
   "url": "https://pokeapi.co/api/v2/pokemon-species/906/"
  },
  {
   "name": "fuecoco",
   "url": "https://pokeapi.co/api/v2/pokemon-species/909/"
  },
  {
   "name": "quaxly",
   "url": "https://pokeapi.co/api/v2/pokemon-species/912/"
  },
  {
   "name": "toxtricity",
   "url": "https://pokeapi.co/api/v2/pokemon-species/849/"
  },
  {
   "name": "grapploct",
   "url": "https://pokeapi.co/api/v2/pokemon-species/853/"
  },
  {
   "name": "sneasler",
   "url": "https://pokeapi.co/api/v2/pokemon-species/903/"
  },
  {
   "name": "floragato",
   "url": "https://pokeapi.co/api/v2/pokemon-species/907/"
  },
  {
   "name": "meowscarada",
   "url": "https://pokeapi.co/api/v2/pokemon-species/908/"
  },
  {
   "name": "crocalor",
   "url": "https://pokeapi.co/api/v2/pokemon-species/910/"
  },
  {
   "name": "skeledirge",
   "url": "https://pokeapi.co/api/v2/pokemon-species/911/"
  },
  {
   "name": "quaxwell",
   "url": "https://pokeapi.co/api/v2/pokemon-species/913/"
  },
  {
   "name": "quaquaval",
   "url": "https://pokeapi.co/api/v2/pokemon-species/914/"
  },
  {
   "name": "fidough",
   "url": "https://pokeapi.co/api/v2/pokemon-species/926/"
  },
  {
   "name": "smoliv",
   "url": "https://pokeapi.co/api/v2/pokemon-species/928/"
  },
  {
   "name": "nacli",
   "url": "https://pokeapi.co/api/v2/pokemon-species/932/"
  },
  {
   "name": "wattrel",
   "url": "https://pokeapi.co/api/v2/pokemon-species/940/"
  },
  {
   "name": "maschiff",
   "url": "https://pokeapi.co/api/v2/pokemon-species/942/"
  },
  {
   "name": "shroodle",
   "url": "https://pokeapi.co/api/v2/pokemon-species/944/"
  },
  {
   "name": "toedscool",
   "url": "https://pokeapi.co/api/v2/pokemon-species/948/"
  },
  {
   "name": "flittle",
   "url": "https://pokeapi.co/api/v2/pokemon-species/955/"
  },
  {
   "name": "tinkatink",
   "url": "https://pokeapi.co/api/v2/pokemon-species/957/"
  },
  {
   "name": "cyclizar",
   "url": "https://pokeapi.co/api/v2/pokemon-species/967/"
  },
  {
   "name": "glimmet",
   "url": "https://pokeapi.co/api/v2/pokemon-species/969/"
  },
  {
   "name": "greavard",
   "url": "https://pokeapi.co/api/v2/pokemon-species/971/"
  },
  {
   "name": "flamigo",
   "url": "https://pokeapi.co/api/v2/pokemon-species/973/"
  },
  {
   "name": "cetoddle",
   "url": "https://pokeapi.co/api/v2/pokemon-species/974/"
  },
  {
   "name": "tatsugiri",
   "url": "https://pokeapi.co/api/v2/pokemon-species/978/"
  },
  {
   "name": "dachsbun",
   "url": "https://pokeapi.co/api/v2/pokemon-species/927/"
  },
  {
   "name": "dolliv",
   "url": "https://pokeapi.co/api/v2/pokemon-species/929/"
  },
  {
   "name": "naclstack",
   "url": "https://pokeapi.co/api/v2/pokemon-species/933/"
  },
  {
   "name": "garganacl",
   "url": "https://pokeapi.co/api/v2/pokemon-species/934/"
  },
  {
   "name": "kilowattrel",
   "url": "https://pokeapi.co/api/v2/pokemon-species/941/"
  },
  {
   "name": "mabosstiff",
   "url": "https://pokeapi.co/api/v2/pokemon-species/943/"
  },
  {
   "name": "grafaiai",
   "url": "https://pokeapi.co/api/v2/pokemon-species/945/"
  },
  {
   "name": "toedscruel",
   "url": "https://pokeapi.co/api/v2/pokemon-species/949/"
  },
  {
   "name": "espathra",
   "url": "https://pokeapi.co/api/v2/pokemon-species/956/"
  },
  {
   "name": "tinkatuff",
   "url": "https://pokeapi.co/api/v2/pokemon-species/958/"
  },
  {
   "name": "tinkaton",
   "url": "https://pokeapi.co/api/v2/pokemon-species/959/"
  },
  {
   "name": "glimmora",
   "url": "https://pokeapi.co/api/v2/pokemon-species/970/"
  },
  {
   "name": "houndstone",
   "url": "https://pokeapi.co/api/v2/pokemon-species/972/"
  },
  {
   "name": "cetitan",
   "url": "https://pokeapi.co/api/v2/pokemon-species/975/"
  },
  {
   "name": "ivysaur",
   "url": "https://pokeapi.co/api/v2/pokemon-species/2/"
  },
  {
   "name": "golem",
   "url": "https://pokeapi.co/api/v2/pokemon-species/76/"
  },
  {
   "name": "bayleef",
   "url": "https://pokeapi.co/api/v2/pokemon-species/153/"
  },
  {
   "name": "monferno",
   "url": "https://pokeapi.co/api/v2/pokemon-species/391/"
  },
  {
   "name": "herdier",
   "url": "https://pokeapi.co/api/v2/pokemon-species/507/"
  },
  {
   "name": "gurdurr",
   "url": "https://pokeapi.co/api/v2/pokemon-species/533/"
  },
  {
   "name": "klinklang",
   "url": "https://pokeapi.co/api/v2/pokemon-species/601/"
  },
  {
   "name": "chandelure",
   "url": "https://pokeapi.co/api/v2/pokemon-species/609/"
  },
  {
   "name": "mienshao",
   "url": "https://pokeapi.co/api/v2/pokemon-species/620/"
  },
  {
   "name": "brionne",
   "url": "https://pokeapi.co/api/v2/pokemon-species/729/"
  },
  {
   "name": "tsareena",
   "url": "https://pokeapi.co/api/v2/pokemon-species/763/"
  },
  {
   "name": "arboliva",
   "url": "https://pokeapi.co/api/v2/pokemon-species/930/"
  }
 ]
}
```

`https://pokeapi.co/api/v2/evolution-chain/1` que vai ter um retorno similar a esses:

```json
{
 "baby_trigger_item": null,
 "chain": {
  "evolution_details": [],
  "evolves_to": [
   {
    "evolution_details": [
     {
      "base_form_id": null,
      "gender": null,
      "held_item": null,
      "item": null,
      "known_move": null,
      "known_move_type": null,
      "location": null,
      "min_affection": null,
      "min_beauty": null,
      "min_damage_taken": null,
      "min_happiness": null,
      "min_level": 16,
      "min_move_count": null,
      "min_steps": null,
      "needs_multiplayer": false,
      "needs_overworld_rain": false,
      "party_species": null,
      "party_type": null,
      "region_id": null,
      "relative_physical_stats": null,
      "time_of_day": "",
      "trade_species": null,
      "trigger": {
       "name": "level-up",
       "url": "https://pokeapi.co/api/v2/evolution-trigger/1/"
      },
      "turn_upside_down": false,
      "used_move": null
     }
    ],
    "evolves_to": [
     {
      "evolution_details": [
       {
        "base_form_id": null,
        "gender": null,
        "held_item": null,
        "item": null,
        "known_move": null,
        "known_move_type": null,
        "location": null,
        "min_affection": null,
        "min_beauty": null,
        "min_damage_taken": null,
        "min_happiness": null,
        "min_level": 32,
        "min_move_count": null,
        "min_steps": null,
        "needs_multiplayer": false,
        "needs_overworld_rain": false,
        "party_species": null,
        "party_type": null,
        "region_id": null,
        "relative_physical_stats": null,
        "time_of_day": "",
        "trade_species": null,
        "trigger": {
         "name": "level-up",
         "url": "https://pokeapi.co/api/v2/evolution-trigger/1/"
        },
        "turn_upside_down": false,
        "used_move": null
       }
      ],
      "evolves_to": [],
      "is_baby": false,
      "species": {
       "name": "venusaur",
       "url": "https://pokeapi.co/api/v2/pokemon-species/3/"
      }
     }
    ],
    "is_baby": false,
    "species": {
     "name": "ivysaur",
     "url": "https://pokeapi.co/api/v2/pokemon-species/2/"
    }
   }
  ],
  "is_baby": false,
  "species": {
   "name": "bulbasaur",
   "url": "https://pokeapi.co/api/v2/pokemon-species/1/"
  }
 },
 "id": 1
}
```

Onde a aplicação deve atribuir os atributos e assim montar o objeto `pokemon e seus relascionamentos.

Ao final depois de atribuir os valores da tabela `pokemon`, `pokemon_types`, `pokemon_abilities`, `pokemon_moves`, `pokemon_evolutions`, `pokemon_growth_rates`. Deve alterar o `status` para `COMPLETE` e retornar o pokemon para a página

### 3.3 Ideia de `Modelos`

Os modelos abaixo são somente para se basear na construção das tabélas você tem total liberdade de adicionar novos ou alterar os campos ou a estrutura existe.

`Pokemon`

| Column                   | Type                      | Constraints                                                          |
|--------------------------|---------------------------|----------------------------------------------------------------------|
| `id`                     | UUID                      | Primary key, default `uuid4`, not nullable                           |
| `name`                   | String                    | Not nullable                                                         |
| `order`                  | Integer                   | Not nullable                                                         |
| `external_image`         | String                    | Not nullable                                                         |
| `status`                 | Enum(`PokemonStatusEnum`) | Not nullable — must always be stored hashed                          |
| `hp`                     | Integer                   | Nullable, default `0`                                                |
| `image`                  | String                    | Nullable, default `None`                                             |
| `speed`                  | Integer                   | Nullable, default `0`                                                |
| `height`                 | Integer                   | Nullable, default `0`                                                |
| `weight`                 | Integer                   | Nullable, default `0`                                                |
| `attack`                 | Integer                   | Nullable, default `0`                                                |
| `defense`                | Integer                   | Nullable, default `0`                                                |
| `habitat`                | String                    | Nullable, default `None`                                             |
| `is_baby`                | Boolean                   | Nullable, default `False`                                            |
| `shape_url`              | String                    | Nullable                                                             |
| `shape_name`             | String                    | Nullable                                                             |
| `is_mythical`            | Boolean                   | Nullable, default `False`                                            |
| `gender_rate`            | Integer                   | Nullable, default `0`                                                |
| `is_legendary`           | Boolean                   | Nullable, default `False`                                            |
| `capture_rate`           | Integer                   | Nullable, default `0`                                                |
| `hatch_counter`          | Integer                   | Nullable, default `0`                                                |
| `base_happiness`         | Integer                   | Nullable, default `0`                                                |
| `special_attack`         | Integer                   | Nullable, default `0`                                                |
| `base_experience`        | Integer                   | Nullable, default `0`                                                |
| `special_defense`        | Integer                   | Nullable, default `0`                                                |
| `evolution_chain`        | String                    | Nullable                                                             |
| `evolves_from_species`   | String                    | Nullable                                                             |
| `has_gender_differences` | Boolean                   | Nullable, default `False`                                            |
| `growth_rate_id`         | UUID                      | Foreign key → `pokemon_growth_rates.id`, Nullable                    |
| `created_at`             | DateTime                  | Not nullable, default `utcnow`, **never updated after insert**       |
| `updated_at`             | DateTime                  | Nullable, default `None`, **only set on second or later updates**    |
| `deleted_at`             | DateTime                  | Nullable, default `None`, **only set when soft-delete is triggered** |

**`PokemonStatusEnum`**

- `COMPLETE`
- `INCOMPLETE`

**Behavior rules for `Pokemon`:**

- `growth_rate` → many-to-one relationship to `PokemonGrowthRate` via foreign key `growth_rate_id` on `pokemons` (optional, nullable)
- `moves` → many-to-many relationship to `PokemonMove` via association table `pokemon_pokemon_moves`
- `abilities` → many-to-many relationship to `PokemonAbility` via association table `pokemon_pokemon_abilities`
- `types` → many-to-many relationship to `PokemonType` via association table `pokemon_pokemon_types`
- `evolutions` → many-to-many self-referential relationship on `pokemons` via association table `pokemon_evolutions` (both directions accessible via `pokemon.evolutions`)

`PokemonType`

| Column             | Type     | Constraints                                                   |
|--------------------|----------|---------------------------------------------------------------|
| `id`               | UUID     | Primary key, default `uuid4`, not nullable                    |
| `url`              | String   | Not nullable                                                  |
| `order`            | Integer  | Not nullable                                                  |
| `name`             | String   | Unique, not nullable                                          |
| `text_color`       | String   | Not nullable                                                  |
| `background_color` | String   | Not nullable                                                  |
| `created_at`       | DateTime | Not nullable, default `utcnow`, never updated after insert    |
| `updated_at`       | DateTime | Nullable, default `None`, only set on second or later updates |
| `deleted_at`       | DateTime | Nullable, default `None`, only set on soft-delete             |

**Relationships on `PokemonType`:**

- `pokemons` → many-to-many relationship to `Pokemon` via association table `pokemon_pokemon_types`
- `weaknesses` → many-to-many self-referential relationship to `PokemonType` via association table `pokemon_type_weaknesses` (types that deal double or half damage to this type)
- `strengths` → many-to-many self-referential relationship to `PokemonType` via association table `pokemon_type_strengths` (types this type deals double or half damage to)

`PokemonMove`

| Column          | Type     | Constraints                                                   |
|-----------------|----------|---------------------------------------------------------------|
| `id`            | UUID     | Primary key, default `uuid4`, not nullable                    |
| `pp`            | Integer  | Not nullable                                                  |
| `url`           | String   | Not nullable                                                  |
| `type`          | String   | Not nullable                                                  |
| `name`          | String   | Unique, not nullable                                          |
| `order`         | Integer  | Not nullable                                                  |
| `power`         | Integer  | Not nullable                                                  |
| `target`        | String   | Not nullable                                                  |
| `effect`        | String   | Not nullable                                                  |
| `priority`      | Integer  | Not nullable                                                  |
| `accuracy`      | Integer  | Not nullable                                                  |
| `short_effect`  | String   | Not nullable                                                  |
| `damage_class`  | String   | Not nullable                                                  |
| `effect_chance` | Integer  | Nullable                                                      |
| `created_at`    | DateTime | Not nullable, default `utcnow`, never updated after insert    |
| `updated_at`    | DateTime | Nullable, default `None`, only set on second or later updates |
| `deleted_at`    | DateTime | Nullable, default `None`, only set on soft-delete             |

**Relationships on `PokemonMove`:**

- `pokemons` → many-to-many relationship to `Pokemon` via association table `pokemon_pokemon_moves`

`PokemonAbility`

| Column       | Type     | Constraints                                                   |
|--------------|----------|---------------------------------------------------------------|
| `id`         | UUID     | Primary key, default `uuid4`, not nullable                    |
| `url`        | String   | Not nullable                                                  |
| `order`      | Integer  | Not nullable                                                  |
| `name`       | String   | Unique, not nullable                                          |
| `slot`       | Integer  | Not nullable                                                  |
| `is_hidden`  | Boolean  | Not nullable                                                  |
| `created_at` | DateTime | Not nullable, default `utcnow`, never updated after insert    |
| `updated_at` | DateTime | Nullable, default `None`, only set on second or later updates |
| `deleted_at` | DateTime | Nullable, default `None`, only set on soft-delete             |

**Relationships on `PokemonAbility`:**

- `pokemons` → many-to-many relationship to `Pokemon` via association table `pokemon_pokemon_abilities`

`PokemonGrowthRate`

| Column        | Type     | Constraints                                                   |
|---------------|----------|---------------------------------------------------------------|
| `id`          | UUID     | Primary key, default `uuid4`, not nullable                    |
| `url`         | String   | Not nullable                                                  |
| `name`        | String   | Unique, not nullable                                          |
| `formula`     | String   | Not nullable                                                  |
| `description` | String   | Not nullable                                                  |
| `created_at`  | DateTime | Not nullable, default `utcnow`, never updated after insert    |
| `updated_at`  | DateTime | Nullable, default `None`, only set on second or later updates |
| `deleted_at`  | DateTime | Nullable, default `None`, only set on soft-delete             |

**Relationships on `PokemonGrowthRate`:**

- `pokemons` → one-to-many relationship to `Pokemon` via foreign key `growth_rate_id` on `pokemons`

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
