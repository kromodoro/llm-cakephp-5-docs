# Constantes e Funções Globais do CakePHP

## Visão Geral

O CakePHP oferece um conjunto poderoso de **constantes globais** e **funções de conveniência** que facilitam tarefas comuns em aplicações. Essas funções podem ser carregadas globalmente através de `require CAKE . 'functions.php';` em `config/bootstrap.php`, ou acessadas via namespaces específicos.

!!! info
    Funções namespaced são carregadas automaticamente. Para usar aliases globais, adicione `require CAKE . 'functions.php';` em seu `config/bootstrap.php`.

## Funções de Tradução (Internacionalização)

### `__(string $string_id, mixed $formatArgs = null)`

Função principal para tradução em CakePHP. Retorna a string traduzida se encontrada, caso contrário retorna a string original.

```php
use Cake\I18n;

// Tradução simples
echo __('Welcome');

// Com placeholder numérico
echo __('You have {0} unread messages', $number);

// Com placeholder nomeado
echo __('Hello {name}, you are {age} years old', [
    'name' => 'João',
    'age' => 30
]);

// Com múltiplos argumentos
echo __('User {0} created on {1}', ['Maria', new DateTime()]);
```

### `__d(string $domain, string $msg, mixed $args = null)`

Tradução dentro de um domínio específico (útil para plugins e módulos).

```php
// Traduzir mensagem de um plugin
echo __d('users_plugin', 'User profile');

// Com argumentos
echo __d('shop_plugin', 'Order {0} created', $orderId);
```

### `__n(string $singular, string $plural, integer $count, mixed $args = null)`

Tradução com suporte a plurais (formato Gettext).

```php
$count = 5;
echo __n(
    'You have {0} message',
    'You have {0} messages',
    $count,
    $count
);
// Retorna: "You have 5 messages"
```

### `__x(string $context, string $msg, mixed $args = null)`

Tradução com contexto, permitindo mesmos textos com significados diferentes.

```php
// Contextos diferentes para mesma palavra
echo __x('noun', 'letter');      // Letra do alfabeto
echo __x('correspondence', 'letter');  // Correspondência
```

## Funções de Depuração

### `debug(mixed $var, boolean $showHtml = null, $showFrom = true)`

Exibe informações de debug formatadas. Apenas executa se `$debug` for `true` no modo de desenvolvimento.

```php
$user = ['name' => 'João', 'email' => 'joao@example.com'];

// Debug simples
debug($user);

// Sem mostrar HTML (output de texto)
debug($user, false);

// Sem indicar origem da chamada
debug($user, null, false);
```

!!! tip
    Use `debug()` em desenvolvimento para inspecionar variáveis sem interromper a execução. Não afeta produção quando `debug = false`.

### `dd(mixed $var, boolean $showHtml = null)`

**Die-Dump**: Como `debug()`, mas **interrompe a execução**. Útil para depuração rápida.

```php
// Vai exibir e parar execução
dd($userData);  // Nunca chegará ao código após isto
```

### `pr(mixed $var)` e `pj(mixed $var)`

Conveniências para `print_r()` com tags `<pre>` e JSON pretty-print.

```php
pr($array);     // print_r() com formatação HTML
pj($data);      // JSON formatado para debugging
```

## Acesso a Variáveis de Ambiente

### `env(string $key, string $default = null)`

Acessa variáveis de ambiente de forma segura, funcionando como emulador de `$_SERVER` e `$_ENV`.

```php
// Retorna variável de ambiente
$host = env('HTTP_HOST');
$method = env('REQUEST_METHOD');
$dbHost = env('DB_HOST', 'localhost');

// Emula PHP_SELF e DOCUMENT_ROOT
$phpSelf = env('PHP_SELF');
$docRoot = env('DOCUMENT_ROOT');
```

!!! warning
    Use `env()` ao invés de `$_SERVER` ou `getenv()` para código mais portável e compatível.

## Escape de HTML

### `h(string $text, boolean $double = true, string $charset = null)`

Wrapper seguro para `htmlspecialchars()`. Previne XSS escapando caracteres especiais.

```php
$userInput = '<script>alert("XSS")</script>';

// Escape simples
echo h($userInput);
// Saída: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// Sem double-encode
echo h($userInput, false);

// Com charset específico
echo h($userInput, true, 'UTF-8');
```

!!! info
    Sempre use `h()` ao exibir dados de usuários para evitar ataques XSS.

## Utilitários de Manipulação

### `pluginSplit(string $name, boolean $dotAppend = false, string $plugin = null)`

Divide notação de plugin em componentes.

```php
list($plugin, $name) = pluginSplit('Users.User');
// $plugin = 'Users'
// $name = 'User'

list($plugin, $name) = pluginSplit('User');
// $plugin = null
// $name = 'User'
```

### `namespaceSplit(string $class)`

Separa namespace do nome da classe.

```php
list($namespace, $className) = namespaceSplit('Cake\Core\App');
// $namespace = 'Cake\Core'
// $className = 'App'
```

### `collection(mixed $items)`

Cria instância de Collection para manipulação funcional de arrays.

```php
use Cake\Collection\Collection;

$data = ['apple', 'banana', 'cherry'];
$fruits = collection($data);

// Operações funcional
$upper = $fruits->map(fn($f) => strtoupper($f))->toArray();
```

## Constantes de Aplicação

As constantes abaixo definem caminhos essenciais da aplicação:

| Constante | Descrição | Exemplo |
|-----------|-----------|---------|
| `APP` | Caminho até diretório `app/` | `/var/www/myapp/app/` |
| `APP_DIR` | Nome do diretório da app | `app` |
| `CONFIG` | Caminho para `config/` | `/var/www/myapp/config/` |
| `CACHE` | Caminho para cache | `/var/www/myapp/tmp/cache/` |
| `LOGS` | Caminho para logs | `/var/www/myapp/logs/` |
| `ROOT` | Caminho raiz do projeto | `/var/www/myapp/` |
| `WWW_ROOT` | Caminho webroot público | `/var/www/myapp/webroot/` |
| `DS` | Separador de diretório | `/` (Linux) ou `\` (Windows) |
| `CAKE` | Caminho CakePHP | `/var/www/myapp/vendor/cakephp/cakephp/` |

### Exemplo de Uso

```php
// Ler arquivo de configuração
$config = include CONFIG . 'database.php';

// Salvar arquivo no cache
file_put_contents(CACHE . 'data.json', json_encode($data));

// Registrar log
file_put_contents(LOGS . 'app.log', $message);

// Servir arquivo estático
$file = WWW_ROOT . 'img/logo.png';
```

## Constante de Tempo

### `TIME_START`

Timestamp em microsegundos quando a aplicação foi iniciada. Útil para benchmarking.

```php
$startTime = TIME_START;

// ... código para medir ...

$elapsedTime = microtime(true) - $startTime;
echo "Tempo decorrido: {$elapsedTime}ms";
```

## Exemplo Prático Completo

```php
<?php
// config/bootstrap.php

use Cake\I18n\I18n;
use Cake\Log\Log;

// Carregar funções globais
require CAKE . 'functions.php';

// Configurar locale
I18n::setLocale('pt_BR');

// Mensagem traduzida
echo __('Welcome to CakePHP');

// Variáveis de ambiente
$dbHost = env('DB_HOST', 'localhost');
$dbPort = env('DB_PORT', 3306);

// Escapar entrada de usuário
$name = h($_GET['name'] ?? 'Guest');
echo "Hello, {$name}";

// Debug em desenvolvimento
if (env('DEBUG', false)) {
    debug($dbConfig);
}

// Log de eventos
Log::info('Application started', [
    'time' => date('Y-m-d H:i:s'),
    'memory' => memory_get_usage()
]);
?>
```

## Quick Start: Melhores Práticas

!!! tip
    1. **Use `__()` em todo texto visível** para permitir tradução futura
    2. **Sempre use `h()`** ao exibir dados do usuário
    3. **Use `env()`** ao invés de acessar `$_SERVER` diretamente
    4. **Use constantes** ao invés de caminhos hardcoded
    5. **Use `debug()`** apenas em desenvolvimento, nunca em produção

## Recursos Adicionais

- Documentação oficial: [CakePHP Global Functions](https://api.cakephp.org)
- Guia de Internacionalização: `/core-libraries/internationalization-and-localization`
- Guia de Depuração: `/development/debugging`
