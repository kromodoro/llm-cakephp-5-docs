# Manipulação Avançada de Arrays com Hash

## Visão Geral

A classe `Hash` do CakePHP é um poderoso utilitário para manipulação de arrays multidimensionais. Oferece métodos para **extrair**, **inserir**, **combinar**, **mesclar**, **filtrar** e **transformar** dados usando uma sintaxe de caminho intuitiva e expressiva.

!!! info
    A classe Hash está disponível em `Cake\Utility\Hash` e todos os métodos são estáticos.

## Sintaxe de Caminho (Path Syntax)

A sintaxe de caminho é o coração da classe Hash. Usa expressões e matchers para navegar arrays complexos.

### Expressões de Traversal

| Expressão | Descrição | Exemplo |
|-----------|-----------|---------|
| `{n}` | Qualquer chave numérica ou string | `users.{n}.name` |
| `{s}` | String value (não numérico) | `config.{s}` |
| `{*}` | Qualquer valor | `data.{*}.id` |
| `Chave` | Chave exata (literal) | `users.name` |

### Matchers de Atributo

| Matcher | Descrição | Exemplo |
|---------|-----------|---------|
| `[id]` | Elemento tem a chave id | `users.{n}[id]` |
| `[id=5]` | Elemento onde id=5 | `users.{n}[id=5]` |
| `[id!=5]` | Elemento onde id!=5 | `users.{n}[id!=5]` |
| `[id>5]` | Elemento onde id>5 | `users.{n}[id>5]` |
| `[id>=5]` | Elemento onde id>=5 | `users.{n}[id>=5]` |
| `[id<5]` | Elemento onde id<5 | `users.{n}[id<5]` |
| `[text=/.../]` | Regex matching | `users.{n}[text=/^admin/]` |

## Extração de Dados

### `Hash::extract(array $data, string $path)`

Extrai valores de um array usando path syntax. Suporta todas as expressões e matchers.

```php
use Cake\Utility\Hash;

$users = [
    ['id' => 1, 'name' => 'João', 'status' => 'active'],
    ['id' => 2, 'name' => 'Maria', 'status' => 'inactive'],
    ['id' => 3, 'name' => 'Pedro', 'status' => 'active'],
];

// Extrair todos os IDs
$ids = Hash::extract($users, '{n}.id');
// Resultado: [1, 2, 3]

// Extrair nomes
$names = Hash::extract($users, '{n}.name');
// Resultado: ['João', 'Maria', 'Pedro']

// Usuários ativos
$active = Hash::extract($users, '{n}[status=active]');
// Resultado: [usuário 1, usuário 3]
```

### `Hash::get(array $data, string $path, mixed $default = null)`

Versão simplificada que retorna um único valor. Não suporta matchers complexos.

```php
$user = [
    'id' => 1,
    'profile' => [
        'name' => 'João',
        'email' => 'joao@example.com'
    ]
];

// Acesso simples
$name = Hash::get($user, 'profile.name');
// Resultado: 'João'

// Com valor padrão
$phone = Hash::get($user, 'profile.phone', '000-0000');
// Resultado: '000-0000'
```

## Inserção e Remoção

### `Hash::insert(array $data, string $path, mixed $values = null)`

Insere valores em múltiplos pontos do array usando path expressions.

```php
$data = [
    'user' => ['id' => 1, 'name' => 'João']
];

// Inserir novo campo
$result = Hash::insert($data, 'user.email', 'joao@example.com');

// Inserir em múltiplos usuários
$users = [
    ['id' => 1, 'name' => 'João'],
    ['id' => 2, 'name' => 'Maria'],
];
$updated = Hash::insert($users, '{n}.role', 'user');
// Resultado: ambos têm role='user'

// Com matchers
$updated = Hash::insert($users, '{n}[id>=2].role', 'admin');
// Apenas usuário 2 recebe role='admin'
```

### `Hash::remove(array $data, string $path)`

Remove elementos do array.

```php
$users = [
    ['id' => 1, 'name' => 'João', 'deleted' => false],
    ['id' => 2, 'name' => 'Maria', 'deleted' => true],
    ['id' => 3, 'name' => 'Pedro', 'deleted' => false],
];

// Remover usuários deletados
$active = Hash::remove($users, '{n}[deleted=true]');

// Remover campo específico
$noEmail = Hash::remove($users, '{n}.email');
```

## Combinação de Dados

### `Hash::combine(array $data, string $keyPath, string $valuePath = null, string $groupPath = null)`

Cria arrays associativos com chaves e valores extraídos de dados complexos.

```php
$records = [
    [
        'user' => ['id' => 1, 'name' => 'João'],
        'role' => 'admin'
    ],
    [
        'user' => ['id' => 2, 'name' => 'Maria'],
        'role' => 'user'
    ],
];

// Criar mapa ID => Nome
$names = Hash::combine($records, '{n}.user.id', '{n}.user.name');
// Resultado: [1 => 'João', 2 => 'Maria']

// Com agrupamento por role
$byRole = Hash::combine(
    $records,
    '{n}.user.id',
    '{n}.user.name',
    '{n}.role'
);
// Resultado:
// [
//     'admin' => [1 => 'João'],
//     'user' => [2 => 'Maria']
// ]

// Concatenar valores
$formatted = Hash::combine(
    $records,
    '{n}.user.id',
    ['%s (%s)', '{n}.user.name', '{n}.role']
);
// Resultado: [1 => 'João (admin)', 2 => 'Maria (user)']
```

## Mesclagem de Arrays

### `Hash::merge(array $data, array $merge, ...)`

Mescla arrays recursivamente (híbrido entre `array_merge` e `array_merge_recursive`).

```php
$config1 = [
    'debug' => true,
    'database' => [
        'host' => 'localhost',
        'port' => 3306
    ]
];

$config2 = [
    'database' => [
        'username' => 'root',
        'password' => 'secret'
    ],
    'cache' => ['enabled' => true]
];

$merged = Hash::merge($config1, $config2);
// Resultado:
// [
//     'debug' => true,
//     'database' => [
//         'host' => 'localhost',
//         'port' => 3306,
//         'username' => 'root',
//         'password' => 'secret'
//     ],
//     'cache' => ['enabled' => true]
// ]
```

!!! info
    `merge()` preserva valores de arrays vazios, ao contrário de `array_merge_recursive`.

## Conversão de Dimensionalidade

### `Hash::flatten(array $data, string $separator = '.')`

Converte array multidimensional em array simples com chaves separadas.

```php
$data = [
    [
        'user' => ['id' => 1, 'name' => 'João'],
        'role' => 'admin'
    ],
    [
        'user' => ['id' => 2, 'name' => 'Maria'],
        'role' => 'user'
    ],
];

$flat = Hash::flatten($data);
// Resultado:
// [
//     '0.user.id' => 1,
//     '0.user.name' => 'João',
//     '0.role' => 'admin',
//     '1.user.id' => 2,
//     '1.user.name' => 'Maria',
//     '1.role' => 'user'
// ]
```

### `Hash::expand(array $data, string $separator = '.')`

Inverte `flatten()`. Converte chaves separadas de volta em array aninhado.

```php
$flat = [
    '0.user.id' => 1,
    '0.user.name' => 'João',
    '1.user.id' => 2,
    '1.user.name' => 'Maria'
];

$nested = Hash::expand($flat);
// Resultado: array aninhado original
```

## Filtragem e Verificação

### `Hash::filter(array $data, callable $callback = null)`

Remove elementos vazios ou que não passam no callback.

```php
$data = [
    '0',
    false,
    true,
    0,
    ['one', 'I can tell you', 'is you got to be', false]
];

$filtered = Hash::filter($data);
// Remove false e null, mantém '0' e 0

// Com callback customizado
$filtered = Hash::filter($data, function($val) {
    return !is_numeric($val) || $val > 10;
});
```

### `Hash::check(array $data, string $path)`

Verifica se um caminho existe no array.

```php
$user = [
    'profile' => [
        'name' => 'João',
        'contact' => [
            'email' => 'joao@example.com'
        ]
    ]
];

Hash::check($user, 'profile.name');           // true
Hash::check($user, 'profile.contact.email');  // true
Hash::check($user, 'profile.phone');          // false
```

## Operações Avançadas

### `Hash::sort(array $data, string $path, string $dir = 'asc', string $type = 'regular')`

Ordena array por caminho específico.

```php
$users = [
    ['id' => 3, 'name' => 'Pedro'],
    ['id' => 1, 'name' => 'João'],
    ['id' => 2, 'name' => 'Maria'],
];

// Ordenar por ID
$sorted = Hash::sort($users, '{n}.id', 'asc');

// Natural sort (foo2 antes de foo10)
$sorted = Hash::sort($users, '{n}.name', 'asc', 'natural');

// Tipos: 'regular', 'numeric', 'string', 'natural'
```

### `Hash::diff(array $data, array $compare)`

Retorna diferenças entre dois arrays.

```php
$original = [
    ['id' => 1, 'name' => 'João'],
    ['id' => 2, 'name' => 'Maria']
];

$updated = [
    ['id' => 1, 'name' => 'João'],
    ['id' => 2, 'name' => 'Maria Silva'],
    ['id' => 3, 'name' => 'Pedro']
];

$diff = Hash::diff($original, $updated);
// Retorna apenas elementos novos/modificados
```

### `Hash::map(array $data, string $path, callable $function)`

Aplica função a cada elemento extraído pelo path.

```php
$users = [
    ['id' => 1, 'name' => 'João'],
    ['id' => 2, 'name' => 'Maria'],
];

$userIds = Hash::map($users, '{n}.id', function($id) {
    return $id * 2;
});
// Resultado: [2, 4]
```

## Quick Start: Exemplos Práticos

### Extrair dados de API

```php
$response = [
    'status' => 'ok',
    'data' => [
        'users' => [
            ['id' => 1, 'name' => 'João', 'active' => true],
            ['id' => 2, 'name' => 'Maria', 'active' => false],
            ['id' => 3, 'name' => 'Pedro', 'active' => true],
        ]
    ]
];

// Usuários ativos
$active = Hash::extract($response, 'data.users.{n}[active=true]');

// IDs dos ativos
$activeIds = Hash::extract($response, 'data.users.{n}[active=true].id');
```

### Normalizar dados de formulário

```php
$form = [
    'user' => [
        'name' => 'João',
        'email' => 'joao@example.com',
        'role' => 'admin'
    ]
];

// Flattened para banco de dados
$flat = Hash::flatten($form);
// [
//     'user.name' => 'João',
//     'user.email' => 'joao@example.com',
//     'user.role' => 'admin'
// ]

// De volta ao aninhado
$nested = Hash::expand($flat);
```

### Combinar resultados de múltiplas queries

```php
$users = [
    ['id' => 1, 'name' => 'João'],
    ['id' => 2, 'name' => 'Maria'],
];

$roles = [
    ['user_id' => 1, 'role' => 'admin'],
    ['user_id' => 2, 'role' => 'user'],
];

// Criar mapa user_id => role
$userRoles = Hash::combine($roles, '{n}.user_id', '{n}.role');
// [1 => 'admin', 2 => 'user']

// Adicionar ao usuário
$users = Hash::insert($users, '{n}.role', null);
foreach ($userRoles as $userId => $role) {
    $users = Hash::insert($users, "{n}[id=$userId].role", $role);
}
```

## Recursos Adicionais

- [API Hash](https://api.cakephp.org/4.x/class-Cake.Utility.Hash.html)
- Documentação oficial: `core-libraries/hash`
