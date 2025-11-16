# Inflector: Manipulação de Palavras

## Visão Geral

`Cake\Utility\Inflector` transforma strings para diferentes formas (plurais, singulares, camelCase, underscored, etc.). Essencial para convenções CakePHP como nomes de tabelas, classes e propriedades.

!!! tip
    Inflector permite sua aplicação seguir automaticamente convenções de nomenclatura do CakePHP.

## Pluralização e Singularização

### `Inflector::pluralize(string $singular)`

Converte palavra singular para plural.

```php
use Cake\Utility\Inflector;

// Exemplos simples
Inflector::pluralize('apple');      // 'apples'
Inflector::pluralize('person');     // 'people'
Inflector::pluralize('potato');     // 'potatoes'

// Casos especiais já configurados
Inflector::pluralize('child');      // 'children'
Inflector::pluralize('goose');      // 'geese'
Inflector::pluralize('fish');       // 'fish' (irregular)

// Com camelCase
Inflector::pluralize('BigApple');   // 'BigApples'
```

### `Inflector::singularize(string $plural)`

Converte palavra plural para singular.

```php
Inflector::singularize('apples');   // 'apple'
Inflector::singularize('people');   // 'person'
Inflector::singularize('children'); // 'child'

// Com underscore
Inflector::singularize('big_apples');  // 'big_apple'
```

!!! warning
    Não use `pluralize()` em palavra já plural, nem `singularize()` em palavra já singular.

## Transformação CamelCase e Underscore

### `Inflector::camelize(string $underscored)`

Converte string underscored para CamelCase (PascalCase).

```php
Inflector::camelize('big_apple');         // 'BigApple'
Inflector::camelize('user_profile');      // 'UserProfile'
Inflector::camelize('big apple');         // 'BigApple'
Inflector::camelize('some-dashed-text');  // 'SomeDashedText'
```

### `Inflector::underscore(string $camelCase)`

Converte CamelCase para underscore.

```php
Inflector::underscore('BigApple');        // 'big_apple'
Inflector::underscore('UserProfile');     // 'user_profile'
Inflector::underscore('HTTPResponse');    // 'http_response'

// Não converte espaços em underscores
Inflector::underscore('Big Apple');       // 'big apple' (apenas minúsculas)
```

!!! info
    `underscore()` funciona melhor com CamelCase. Espaços são apenas minusculizados.

## Texto Legível

### `Inflector::humanize(string $underscored)`

Converte underscore para formato "Title Case" legível.

```php
Inflector::humanize('big_apple');       // 'Big Apple'
Inflector::humanize('user_profile');    // 'User Profile'
Inflector::humanize('first_name');      // 'First Name'

// Com camelCase também funciona
Inflector::humanize('bigApple');        // 'BigApple' (mantém maiúsculas)
```

Útil para gerar labels em formulários:

```php
// Em view
<?php
$fields = ['first_name', 'last_name', 'email_address'];
foreach ($fields as $field) {
    echo Inflector::humanize($field);  // Gera labels automaticamente
}
?>
```

## Nomes de Tabelas e Classes

### `Inflector::classify(string $underscored)`

Gera nome de classe (singular, CamelCase) a partir de underscore.

```php
Inflector::classify('users');               // 'User'
Inflector::classify('user_profiles');       // 'UserProfile'
Inflector::classify('admin_settings');      // 'AdminSetting'

// Segue convenção de tabela plural → classe singular
```

### `Inflector::tableize(string $camelCase)`

Gera nome de tabela (plural, underscore) a partir de CamelCase.

```php
Inflector::tableize('User');                // 'users'
Inflector::tableize('UserProfile');         // 'user_profiles'
Inflector::tableize('AdminSetting');        // 'admin_settings'

// Inverso de classify()
```

### `Inflector::dasherize(string $camelCase)`

Converte para kebab-case (hífens).

```php
Inflector::dasherize('BigApple');           // 'big-apple'
Inflector::dasherize('UserProfile');        // 'user-profile'

// Apenas CamelCase sem espaços
Inflector::dasherize('Big Apple');          // 'big apple' (não converte espaço)
```

Útil para URLs amigáveis:

```php
$slug = Inflector::dasherize(
    Inflector::underscore('ProductCategory')
);
// 'product-category'
```

## Nomes de Variáveis

### `Inflector::variable(string $underscored)`

Gera nome de variável (camelCase minúsculo) a partir de underscore.

```php
Inflector::variable('big_apple');       // 'bigApple'
Inflector::variable('user_profile');    // 'userProfile'
Inflector::variable('first_name');      // 'firstName'

// Útil para propriedades de classe
Inflector::variable('big apples');      // 'bigApples'
```

## Tabela de Transformações

| Entrada | pluralize | singularize | camelize | underscore | humanize | classify | tableize | variable |
|---------|-----------|-------------|----------|-----------|----------|----------|----------|----------|
| apple | apples | apple | Apple | apple | Apple | Apple | apples | apple |
| big_apple | big_apples | big_apple | BigApple | big_apple | Big Apple | BigApple | big_apples | bigApple |
| BigApple | BigApples | BigApple | BigApple | big_apple | BigApple | BigApple | big_apples | bigApple |
| users | users | user | Users | users | Users | User | users | users |

## Regras Customizadas

Para idiomas que não sejam inglês ou plurais irregulares, configure regras customizadas:

### `Inflector::rules(string $type, array $rules, boolean $reset = false)`

```php
// Em config/bootstrap.php

// Regras singulares
Inflector::rules('singular', [
    '/^(bil)er$/i' => '\1',           // biler → bil
    '/^(inflec|contribu)tors$/i' => '\1ta'  // inflectors → inflecta
]);

// Palavras não inflectáveis
Inflector::rules('uninflected', [
    'singularis',
    'pluralis',
    'sheep',
    'fish'
]);

// Formas irregulares
Inflector::rules('irregular', [
    'phylum' => 'phyla',              // singular => plural
    'alumnus' => 'alumni',
    'bacillus' => 'bacilli'
]);
```

### Português Brasileiro

```php
// Adicionar suporte a português
Inflector::rules('singular', [
    '/ões$/i' => 'ão',      // creações → creação
    '/ães$/i' => 'ão',      // pães → pão
    '/is$/i' => 'il',       // brasileiros → brasileiro
]);

Inflector::rules('irregular', [
    'pão' => 'pães',
    'irmão' => 'irmãos',
    'capitão' => 'capitães'
]);
```

### Limpar regras customizadas

```php
// Restaurar estado original
Inflector::reset();
```

## Exemplo Prático Completo

```php
<?php
namespace App\Utility;

use Cake\Utility\Inflector;

class ModelNamer
{
    /**
     * Gera nome de classe a partir de nome de tabela
     */
    public static function getClassName(string $table): string
    {
        // users → User
        return Inflector::classify(Inflector::singularize($table));
    }

    /**
     * Gera nome de tabela a partir de nome de classe
     */
    public static function getTableName(string $class): string
    {
        // User → users
        return Inflector::tableize($class);
    }

    /**
     * Gera slug URL-friendly
     */
    public static function getSlug(string $title): string
    {
        $slug = Inflector::dasherize(
            Inflector::underscore($title)
        );

        // users profile → user-profile
        return strtolower($slug);
    }

    /**
     * Gera label legível para campo
     */
    public static function getFieldLabel(string $fieldName): string
    {
        return Inflector::humanize(
            Inflector::underscore($fieldName)
        );
    }
}

// Uso
echo ModelNamer::getClassName('admin_users');      // 'AdminUser'
echo ModelNamer::getTableName('UserProfile');      // 'user_profiles'
echo ModelNamer::getSlug('My Great Product');      // 'my-great-product'
echo ModelNamer::getFieldLabel('userFirstName');   // 'User First Name'
?>
```

## Quick Start

```php
use Cake\Utility\Inflector;

// Converter nome de tabela para classe
$class = Inflector::classify('user_profiles');
// 'UserProfile'

// Converter nome de classe para tabela
$table = Inflector::tableize($class);
// 'user_profiles'

// Humanizar para label
$label = Inflector::humanize('first_name');
// 'First Name'
```

## Testes Online

CakePHP oferece teste interativo de Inflector:
- https://inflector.cakephp.org/
- https://sandbox.dereuromark.de/sandbox/inflector

## Recursos Adicionais

- [API Inflector](https://api.cakephp.org/4.x/class-Cake.Utility.Inflector.html)
- [Documentação oficial](https://book.cakephp.org/4/en/core-libraries/inflector.html)
- Convenções CakePHP: `/intro/conventions`
