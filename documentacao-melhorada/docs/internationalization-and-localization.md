# Internacionalização e Localização (i18n/l10n)

## Visão Geral

CakePHP oferece suporte completo a **múltiplas linguagens** com:
- Funções de tradução como `__()`, `__d()`, `__n()`
- Arquivos `.po` em formato Gettext
- Formatação de datas, números e moedas por locale
- Tradução automática de strings no código

!!! info
    **i18n** (Internacionalização) = capacidade de suportar múltiplos idiomas
    **l10n** (Localização) = adaptação para idioma/região específicos

## Configuração Básica

### Estrutura de Arquivos

```
resources/
    locales/
        en_US/
            default.po
        pt_BR/
            default.po
        es_ES/
            default.po
            validation.po
```

### Definir Locale Padrão

```php
// config/app.php
return [
    'App' => [
        'defaultLocale' => env('APP_DEFAULT_LOCALE', 'pt_BR'),
        // ... outras configurações
    ]
];
```

### Alterar Locale em Runtime

```php
use Cake\I18n\I18n;

// Mudar para português do Brasil
I18n::setLocale('pt_BR');

// Mudar para espanhol
I18n::setLocale('es_ES');

// Afeta tradução E formatação de datas/números
```

## Funções de Tradução

### `__(string $string_id, mixed $args = null)`

Função principal de tradução. Retorna string traduzida ou original se não encontrada.

```php
// Simples
echo __('Welcome');

// Com placeholders numerados
echo __('You have {0} new messages', 5);

// Com placeholders nomeados
echo __('Hello {name}, you are {age} years old', [
    'name' => 'João',
    'age' => 25
]);

// Em templates
<?= __('Popular Articles') ?>
```

### `__d(string $domain, string $msg, mixed $args = null)`

Tradução em domínio específico (útil para plugins).

```php
// Traduzir mensagem de plugin
echo __d('users', 'User not found');

// Com argumentos
echo __d('shop', 'Order {0} created successfully', $orderId);

// Para plugins vendor-namespaced
echo __d('vendor/my_plugin', 'Plugin message');
```

### `__n(string $singular, string $plural, int $count, mixed $args = null)`

Tradução com plurais (formato Gettext).

```php
$count = 5;

echo __n(
    'You have {0} message',
    'You have {0} messages',
    $count,
    $count
);
// Saída: "You have 5 messages"

// Com argumentos nomeados
echo __n(
    '{name} has {0} file',
    '{name} has {0} files',
    $count,
    ['name' => 'João', 0 => $count]
);
```

### `__x(string $context, string $msg, mixed $args = null)`

Tradução com contexto para remover ambiguidade.

```php
// Mesma palavra, contextos diferentes
echo __x('fruit', 'orange');          // cor
echo __x('color', 'orange');          // fruta

// Resulta em entradas separadas no arquivo .po
```

## Arquivos de Tradução (.po)

### Estrutura básica

```po
#: app/View/Users/index.ctp:12
msgid "Welcome"
msgstr "Bem-vindo"

#: app/View/Users/profile.ctp:4
msgid "You have {0} messages"
msgstr "Você tem {0} mensagens"

# Com contexto
#: app/View/Products/view.ctp:8
msgctxt "fruit"
msgid "orange"
msgstr "laranja"
```

### Plurais em Gettext

```po
# Singular
msgid "One user deleted"
# Plural
msgid_plural "{0} users deleted"
# Tradução singular
msgstr[0] "Um usuário deletado"
# Tradução plural
msgstr[1] "{0} usuários deletados"
```

## Plurais com ICU MessageFormatter

CakePHP suporta ICU para plurais complexos:

```php
echo __(
    '{0,plural,=0{No files} =1{One file} other{# files}}',
    0
);
// "No files"

echo __(
    '{0,plural,=0{No files} =1{One file} other{# files}}',
    5
);
// "5 files"
```

## Formatação de Datas e Números

### Mudar formatação por locale

```php
use Cake\I18n\I18n;
use Cake\I18n\DateTime;
use Cake\I18n\Number;

// Definir locale
I18n::setLocale('pt_BR');

// Data agora segue formato brasileiro
$date = new DateTime('2024-01-15 14:30:00');
echo $date; // 15/01/2024 14:30

// Número segue formato brasileiro
echo Number::format(1234.56);  // 1.234,56

// Moeda em real
echo Number::currency(99.99, 'BRL');  // R$ 99,99
```

## Middleware de Locale Automático

Selecionar locale automaticamente pelo header HTTP:

```php
// src/Application.php
use Cake\I18n\Middleware\LocaleSelectorMiddleware;

public function middleware(MiddlewareQueue $middlewareQueue): MiddlewareQueue
{
    // Selecionar automaticamente entre locales válidos
    $middlewareQueue->add(
        new LocaleSelectorMiddleware(['pt_BR', 'es_ES', 'en_US'])
    );

    // Ou aceitar qualquer locale
    // $middlewareQueue->add(new LocaleSelectorMiddleware(['*']));

    return $middlewareQueue;
}
```

## Parsing de Datas Localizadas

Aceitar datas em formato do usuário:

```php
use Cake\Database\TypeFactory;

// Em controller ou middleware
// Permitir parsing de datas no formato padrão do locale
TypeFactory::build('datetime')->useLocaleParser();

// Com formato customizado
TypeFactory::build('datetime')
    ->useLocaleParser()
    ->setLocaleFormat('dd-MM-yyyy');

// Com constantes IntlDateFormatter
TypeFactory::build('datetime')
    ->useLocaleParser()
    ->setLocaleFormat([IntlDateFormatter::SHORT, -1]);
```

## Conversão de Timezone do Usuário

```php
use Cake\Database\TypeFactory;

// Definir timezone do usuário
TypeFactory::build('datetime')
    ->setUserTimezone($user->timezone);  // 'America/Sao_Paulo'

// ORM agora converte automaticamente entre
// timezone do usuário e timezone da app
```

## Middleware Completo de Datetime

```php
// src/Middleware/DatetimeMiddleware.php
namespace App\Middleware;

use Cake\Database\TypeFactory;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;

class DatetimeMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        $user = $request->getAttribute('identity');

        if ($user && isset($user->timezone)) {
            TypeFactory::build('datetime')
                ->useLocaleParser()
                ->setUserTimezone($user->timezone);
        }

        return $handler->handle($request);
    }
}
```

## Tradutores Customizados

### Carregar de fonte customizada

```php
use Cake\I18n\Package;

I18n::setTranslator('animals', function () {
    $package = new Package('default', 'default');
    $package->setMessages([
        'Dog' => 'Cachorro',
        'Cat' => 'Gato',
        'Bird' => 'Pássaro'
    ]);
    return $package;
}, 'pt_BR');

// Uso
I18n::setLocale('pt_BR');
echo __d('animals', 'Dog');  // 'Cachorro'
```

### Carregar de YAML

```php
use Cake\I18n\MessagesFileLoader as Loader;

// Em config/bootstrap.php
I18n::setTranslator(
    'custom',
    new Loader('messages', 'locales', 'yaml'),
    'pt_BR'
);

// File: resources/locales/pt_BR/messages.yaml
// Welcome: Bem-vindo
// Hello: Olá
```

## Extração de Strings para Tradução

Usar command-line para extrair strings com `__()`:

```bash
# Extrair strings para arquivo .pot
bin/cake i18n extract

# Resultado: resources/locales/default.pot
```

## Cache de Traduções

!!! warning
    Traduções são cacheadas. Limpar cache após mudanças:

```bash
# Limpar cache de traduções
bin/cake cache clear _cake_core_

# Ou manualmente
rm -rf tmp/cache/persistent/*
```

## Exemplo Prático Completo

```php
<?php
// config/bootstrap.php
use Cake\I18n\I18n;

// Definir locale padrão
I18n::setLocale('pt_BR');

// Em src/Controller/ArticlesController.php
namespace App\Controller;

use Cake\I18n\I18n;

class ArticlesController extends AppController
{
    public function index()
    {
        // Obter artigos do model
        $articles = $this->Articles->find('all');

        // Traduzir título da página
        $title = __('Popular Articles');

        $this->set(compact('articles', 'title'));
    }

    public function delete($id)
    {
        $article = $this->Articles->get($id);

        if ($this->Articles->delete($article)) {
            $count = $this->Articles->find()->count();

            // Mensagem pluralizada
            $msg = __n(
                'Article deleted. {0} article remaining.',
                'Article deleted. {0} articles remaining.',
                $count,
                $count
            );

            $this->Flash->success($msg);
        }

        return $this->redirect(['action' => 'index']);
    }

    public function changeLocale($locale)
    {
        // Alterar locale do usuário
        I18n::setLocale($locale);

        // Salvar preferência
        $this->request->getSession()
            ->write('User.locale', $locale);

        return $this->redirect($this->referer());
    }
}

// Em template articles/index.ctp
<h1><?= $title ?></h1>

<table>
<?php foreach ($articles as $article): ?>
    <tr>
        <td><?= h($article->title) ?></td>
        <td>
            <?= __('By') ?> <?= h($article->author) ?>
        </td>
        <td>
            <?= $this->Html->link(
                __('Delete'),
                ['action' => 'delete', $article->id],
                ['method' => 'post', 'confirm' => __('Are you sure?')]
            ) ?>
        </td>
    </tr>
<?php endforeach; ?>
</table>
?>
```

## Quick Start

```php
// 1. Em todo texto visível
echo __('Welcome to our site');

// 2. Em plugins
echo __d('my_plugin', 'Plugin message');

// 3. Com plurais
echo __n('1 user', '{0} users', $count, $count);

// 4. Definir locale
I18n::setLocale('pt_BR');

// 5. Formatar data/número
echo $date;  // Segue locale configurado
echo Number::currency(100, 'BRL');
```

## Recursos Adicionais

- [Guia oficial i18n](https://book.cakephp.org/4/en/core-libraries/internationalization-and-localization.html)
- [API I18n](https://api.cakephp.org/4.x/class-Cake.I18n.I18n.html)
- [CakePHP Localized](https://github.com/cakephp/localized) - traduções core
- [Lista de Locales ICU](https://www.localeplanet.com/icu/)
