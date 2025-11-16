# Logging em CakePHP

## Visão Geral

Sistema de logging em CakePHP permite **registrar eventos** em arquivos, syslog ou engines customizados com **múltiplos níveis** de severidade, **scopes** para segregação por subsistema, e **formatação flexível**.

!!! tip
    Use logging para rastrear atividades, erros e eventos importantes em produção.

## Configuração

### Configurar Engines

```php
// config/app.php
use Cake\Log\Engine\FileLog;
use Cake\Log\Engine\SyslogLog;

return [
    // ... outras configurações

    'Log' => [
        'debug' => [
            'className' => FileLog::class,
            'path' => LOGS,
            'levels' => ['notice', 'info', 'debug'],
            'file' => 'debug',
        ],
        'error' => [
            'className' => FileLog::class,
            'path' => LOGS,
            'levels' => ['error', 'critical', 'alert', 'emergency'],
            'file' => 'error',
        ],
        'access' => [
            'className' => FileLog::class,
            'path' => LOGS,
            'levels' => [], // Todos os níveis
            'scopes' => ['access'],  // Apenas scope 'access'
            'file' => 'access',
        ]
    ]
];
```

### Em config/bootstrap.php

```php
use Cake\Log\Log;
use Cake\Log\Engine\FileLog;

// Configurar durante bootstrap
Log::setConfig('info', [
    'className' => FileLog::class,
    'path' => LOGS,
    'levels' => ['info'],
    'file' => 'info',
]);

// Com closure
Log::setConfig('special', function () {
    return new FileLog(['path' => LOGS, 'file' => 'special']);
});

// Ou como DSN
Log::setConfig('error', [
    'url' => 'file:///var/log/app/?levels[]=warning&levels[]=error&file=error',
]);
```

## Níveis de Log

CakePHP suporta níveis POSIX (do mais ao menos grave):

| Nível | Descrição |
|-------|-----------|
| **emergency** | Sistema inutilizável |
| **alert** | Ação imediata necessária |
| **critical** | Condição crítica |
| **error** | Erro comum |
| **warning** | Aviso |
| **notice** | Condição normal mas significativa |
| **info** | Informacional |
| **debug** | Debug-level |

```php
use Cake\Log\Log;

Log::emergency('Sistema crítico!');
Log::alert('Verificação imediata necessária');
Log::critical('Falha crítica detectada');
Log::error('Erro na processamento');
Log::warning('Aviso importante');
Log::notice('Evento normal importante');
Log::info('Informação geral');
Log::debug('Informação de debug');
```

## Escrevendo Logs

### `Log::write(string $level, string $message, mixed $context = [])`

```php
use Cake\Log\Log;

// Log simples
Log::write('info', 'Usuário fez login');
Log::write('error', 'Conexão com BD falhou');

// Com variáveis
Log::write('info', 'User {user} logged in', ['user' => $userId]);

// Com escopo
Log::write('error', 'Payment failed', [
    'scope' => ['payments']
]);
```

### Usando `log()` em classes CakePHP

Classes que usam `LogTrait` (Controllers, Models, Components) têm acesso a `log()`:

```php
namespace App\Controller;

use App\Controller\AppController;

class UsersController extends AppController
{
    public function login()
    {
        // ... autenticação ...

        $this->log('User login successful', 'info');

        // Com scope
        $this->log('User login successful', 'info', ['scope' => ['auth']]);
    }

    public function register()
    {
        try {
            $user = $this->Users->save($this->request->getData());
            $this->log('User registered: ' . $user->id, 'info');
        } catch (Exception $e) {
            $this->log('Registration failed: ' . $e->getMessage(), 'error');
        }
    }
}
```

## Placeholders e Context

Use placeholders para inserir dados dinamicamente:

```php
Log::write('error', 'Could not process order {order_id} for user {user_id}', [
    'order_id' => $order->id,
    'user_id' => $user->id
]);
// Resultado: "Could not process order 123 for user 456"

// Escape com barra invertida
Log::write('error', 'Config: {config} literal', ['config' => 'escaped']);
// {config} não é substituído

// Objetos com __toString(), toArray() ou __debugInfo()
Log::write('info', 'User {user} created', [
    'user' => $userEntity  // Chama __toString()
]);
```

!!! info
    Placeholders não-mapeados são deixados como estão. Sempre mapeie valores.

## Scopes (Escopos)

Segregar logs por subsistema:

```php
// Em config/app.php
Log::setConfig('payments', [
    'className' => FileLog::class,
    'path' => LOGS,
    'scopes' => ['payments'],  // Apenas logs com scope 'payments'
    'file' => 'payments.log',
]);

Log::setConfig('orders', [
    'className' => FileLog::class,
    'path' => LOGS,
    'scopes' => ['orders', 'payments'],  // Ambos scopes
    'file' => 'orders.log',
]);

// Escrever com scope
Log::info('Payment processed', ['scope' => ['payments']]);
Log::info('Order placed', ['scope' => ['orders']]);

// payment.log recebe: "Payment processed"
// orders.log recebe: "Payment processed" e "Order placed"
```

## FileLog com Rotação

```php
Log::setConfig('default', [
    'className' => FileLog::class,
    'path' => LOGS,
    'file' => 'app.log',

    // Rotação automática quando arquivo atinge tamanho
    'size' => '10MB',        // ou 10485760 bytes
    'rotate' => 5,           // manter 5 arquivos rotacionados
    'mask' => 0644,          // permissões do arquivo
]);

// Arquivo rotacionado: app.log.2024-01-15_12-30-45
// Suporta: '1MB', '100KB', '5MB', etc.
```

## SyslogLog em Produção

```php
use Cake\Log\Engine\SyslogLog;

Log::setConfig('default', [
    'className' => SyslogLog::class,
    'format' => '%s - Web Server 1 - %s',  // Deprecated, use formatter
    'prefix' => 'MyApp',
    'flag' => LOG_ODELAY,
    'facility' => LOG_USER,
]);
```

!!! tip
    Use syslog em produção para melhor performance e integração com monitoramento.

## Formatters Customizados

```php
use Cake\Log\Formatter\AbstractFormatter;

class JsonFormatter extends AbstractFormatter
{
    public function format($level, $message, $context)
    {
        return json_encode([
            'timestamp' => date('Y-m-d H:i:s'),
            'level' => $level,
            'message' => $message,
            'context' => $context
        ]);
    }
}

// Configurar
Log::setConfig('json', [
    'className' => FileLog::class,
    'path' => LOGS,
    'file' => 'app.json',
    'formatter' => JsonFormatter::class,
]);
```

## Monolog Integration

Usar Monolog como logger ao invés de FileLog:

```php
use Monolog\Logger;
use Monolog\Handler\StreamHandler;

Log::setConfig('default', function () {
    $log = new Logger('app');
    $log->pushHandler(
        new StreamHandler(LOGS . 'combined.log', Logger::DEBUG)
    );
    return $log;
});

// Remover loggers padrão redundantes
Log::drop('debug');
Log::drop('error');
```

## Exemplo Prático Completo

```php
<?php
// config/app.php
use Cake\Log\Engine\FileLog;

return [
    // ...
    'Log' => [
        // Logs de debug para desenvolvimento
        'debug' => [
            'className' => FileLog::class,
            'path' => LOGS,
            'levels' => ['notice', 'info', 'debug'],
            'file' => 'debug',
        ],

        // Erros para alertas
        'error' => [
            'className' => FileLog::class,
            'path' => LOGS,
            'levels' => ['error', 'critical', 'alert'],
            'file' => 'error',
            'size' => '10MB',
            'rotate' => 10,
        ],

        // Logs de segurança
        'security' => [
            'className' => FileLog::class,
            'path' => LOGS,
            'scopes' => ['security', 'auth'],
            'file' => 'security',
        ],

        // Logs de pagamento
        'payments' => [
            'className' => FileLog::class,
            'path' => LOGS,
            'scopes' => ['payments'],
            'file' => 'payments',
            'size' => '5MB',
        ]
    ]
];

// src/Controller/PaymentsController.php
namespace App\Controller;

use Cake\Log\Log;

class PaymentsController extends AppController
{
    public function process()
    {
        try {
            $result = $this->paymentGateway->charge(
                $this->request->getData('amount')
            );

            Log::info('Payment processed: ${amount} for user {user}', [
                'scope' => ['payments'],
                'amount' => $result['amount'],
                'user' => $this->request->getAttribute('identity')->id
            ]);

            $this->Flash->success(__('Payment successful'));
        } catch (Exception $e) {
            Log::error('Payment failed: {error}', [
                'scope' => ['payments'],
                'error' => $e->getMessage()
            ]);

            $this->Flash->error(__('Payment failed'));
        }
    }

    public function failed()
    {
        Log::critical('Multiple payment failures detected', [
            'scope' => ['security']
        ]);
    }
}
?>
```

## Testing com LogTestTrait

```php
<?php
namespace App\Test\TestCase\Controller;

use Cake\TestSuite\LogTestTrait;
use Cake\TestSuite\TestCase;

class UsersControllerTest extends TestCase
{
    use LogTestTrait;

    public function setUp(): void
    {
        parent::setUp();

        // Configurar logs para capture
        $this->setupLog([
            'error' => ['scopes' => ['app.security']]
        ]);
    }

    public function testFailedLogin()
    {
        $this->post('/users/login', [
            'email' => 'user@example.com',
            'password' => 'wrong'
        ]);

        // Verificar se erro foi logado
        $this->assertLogMessageContains(
            'error',
            'Login failed',
            'app.security'
        );
    }

    public function testSuccessfulLogin()
    {
        $this->post('/users/login', [
            'email' => 'user@example.com',
            'password' => 'correct'
        ]);

        // Verificar ausência de erros
        $this->assertLogAbsent('error');
    }
}
?>
```

## Quick Start

```php
use Cake\Log\Log;

// 1. Configurar em config/app.php (já feito por padrão)

// 2. Escrever logs
Log::info('Usuário fez login');
Log::error('Erro ao conectar BD');

// 3. Em controllers/models
$this->log('Evento importante', 'info');

// 4. Com escopo
Log::warning('Pagamento falhou', ['scope' => ['payments']]);
```

## Recursos Adicionais

- [API Log](https://api.cakephp.org/4.x/class-Cake.Log.Log.html)
- [Documentação oficial](https://book.cakephp.org/4/en/core-libraries/logging.html)
- [Monolog Documentation](https://seldaek.github.io/monolog/)
