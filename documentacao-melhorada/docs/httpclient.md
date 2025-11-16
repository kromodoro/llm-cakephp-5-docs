# Cliente HTTP do CakePHP

## Visão Geral

`Cake\Http\Client` é um cliente HTTP PSR-18 completo para fazer requisições a APIs externas, webservices e recursos remotos. Suporta GET, POST, PUT, DELETE, PATCH com autenticação múltipla (Basic, Digest, OAuth), gerenciamento automático de cookies, SSL/TLS customizável e eventos.

!!! tip
    Use `Http\Client` para integração com APIs REST, serviços de terceiros e requisições HTTP em geral.

## Requisições Básicas

### GET simples

```php
use Cake\Http\Client;

$http = new Client();

// GET simples
$response = $http->get('https://api.example.com/users/1');

// GET com query string
$response = $http->get('https://api.example.com/search', [
    'q' => 'cakephp',
    'limit' => 10
]);

// GET com headers customizados
$response = $http->get('https://api.example.com/data', [], [
    'headers' => [
        'X-Api-Key' => 'secret-key',
        'Accept' => 'application/json'
    ]
]);
```

### POST e PUT

```php
$http = new Client();

// POST com form-encoded data
$response = $http->post('https://api.example.com/users', [
    'name' => 'João',
    'email' => 'joao@example.com',
    'role' => 'admin'
]);

// POST com JSON
$response = $http->post(
    'https://api.example.com/tasks',
    json_encode(['title' => 'Tarefa 1']),
    ['type' => 'json']
);

// PUT request
$response = $http->put('https://api.example.com/users/1', [
    'name' => 'João Silva',
    'email' => 'joao.silva@example.com'
]);
```

### Outros métodos HTTP

```php
$http = new Client();

// DELETE
$response = $http->delete('https://api.example.com/users/1');

// PATCH
$response = $http->patch('https://api.example.com/posts/1', [
    'status' => 'published'
]);

// HEAD (verifica apenas headers)
$response = $http->head('https://api.example.com/large-file');
```

## Autenticação

### Autenticação Básica

```php
$http = new Client();

$response = $http->get('https://api.example.com/admin', [], [
    'auth' => [
        'username' => 'admin',
        'password' => 'secret'
    ]
]);
```

### Autenticação Digest

```php
$response = $http->get('https://api.example.com/protected', [], [
    'auth' => [
        'type' => 'digest',
        'username' => 'user',
        'password' => 'secret',
        'realm' => 'myapp',
        'nonce' => 'servervalue',
        'qop' => 1
    ]
]);

// Suporta algoritmos:
// - MD5, SHA-256, SHA-512-256
// - MD5-sess, SHA-256-sess, SHA-512-256-sess
```

### OAuth 1.0

```php
$response = $http->get('https://api.twitter.com/1.1/account/verify.json', [], [
    'auth' => [
        'type' => 'oauth',
        'consumerKey' => 'your_consumer_key',
        'consumerSecret' => 'your_consumer_secret',
        'token' => 'access_token',
        'tokenSecret' => 'access_token_secret',
        'realm' => 'twitter'
    ]
]);
```

### OAuth 2.0

```php
$http = new Client([
    'headers' => [
        'Authorization' => 'Bearer ' . $accessToken
    ]
]);

$response = $http->get('https://api.github.com/user');
```

!!! warning
    OAuth 2.0 é apenas um header. Configure o token como header da aplicação.

## Upload de Arquivos

### Upload simples

```php
$http = new Client();

$response = $http->post('https://api.example.com/upload', [
    'avatar' => fopen('/path/to/image.jpg', 'r'),
    'name' => 'João'
]);
```

### Multipart com FormData

```php
use Cake\Http\Client\FormData;

$data = new FormData();

// Adicionar arquivo
$file = $data->addFile('document', fopen('/path/to/doc.pdf', 'r'));
$file->disposition('attachment');

// Adicionar campo XML
$xml = $data->newPart('xml_data', '<root>content</root>');
$xml->type('application/xml');
$data->add($xml);

// Enviar
$response = $http->post(
    'https://api.example.com/documents',
    (string)$data,
    ['headers' => ['Content-Type' => $data->contentType()]]
);
```

## Tipo de Conteúdo

### JSON

```php
$http = new Client();

// POST JSON automaticamente
$response = $http->post(
    'https://api.example.com/api/tasks',
    json_encode(['title' => 'Nova tarefa']),
    ['type' => 'json']
);

// Com query string no GET
$response = $http->get(
    'https://api.example.com/api/search',
    ['q' => 'teste', '_content' => json_encode(['filter' => 'active'])],
    ['type' => 'json']
);
```

### XML

```php
$xml = '<root><item>value</item></root>';

$response = $http->post(
    'https://api.example.com/xml',
    $xml,
    ['type' => 'xml']
);
```

## Configuração e Clientes Scoped

### Pré-configurar cliente

```php
$http = new Client([
    'host' => 'api.example.com',
    'scheme' => 'https',
    'auth' => [
        'username' => 'api_user',
        'password' => 'api_pass'
    ],
    'timeout' => 30,
    'ssl_verify_peer' => true
]);

// Todas requisições usam configuração acima
$response = $http->get('/users/1');
$response = $http->post('/users', ['name' => 'João']);
```

### Criar a partir de URL

```php
$http = Client::createFromUrl('https://api.github.com/v3/repos/cakephp/cakephp');

// Configuração extraída automaticamente:
// - scheme: https
// - host: api.github.com
// - basePath: /v3
```

### Opções Disponíveis

```php
$http = new Client([
    'host' => 'api.example.com',
    'basePath' => '/v1',
    'scheme' => 'https',
    'auth' => [...],
    'proxy' => [...],
    'port' => 8080,
    'cookies' => [...],
    'timeout' => 30,
    'ssl_verify_peer' => true,
    'ssl_verify_depth' => 5,
    'ssl_verify_host' => true
]);
```

## Segurança SSL/TLS

```php
$http = new Client([
    // Desabilitar verificação (NÃO RECOMENDADO)
    'ssl_verify_peer' => false,

    // Custom CA bundle
    'ssl_cafile' => '/etc/ssl/certs/ca-bundle.crt',

    // Profundidade da cadeia CA
    'ssl_verify_depth' => 10,

    // Validar hostname
    'ssl_verify_host' => true
]);
```

!!! warning
    Sempre verifique SSL em produção. Desabilitar `ssl_verify_peer` é perigoso.

## Gerenciamento de Cookies

```php
$http = new Client([
    'host' => 'api.example.com'
]);

// Primeira requisição define cookies
$response = $http->get('/login');

// Segunda requisição automaticamente envia cookies
$response = $http->get('/dashboard');

// Adicionar cookie manualmente
use Cake\Http\Cookie\Cookie;

$http->addCookie(new Cookie(
    'session_id',
    'abc123def456',
    time() + 3600
));

// Sobrescrever cookies em requisição
$response = $http->get('/api/data', [], [
    'cookies' => ['session_id' => 'new_value']
]);
```

## Tratamento de Resposta

### Verificar status

```php
$response = $http->get('https://api.example.com/user/1');

// Verificações
if ($response->isOk()) {
    // Status 200-299
    $data = $response->getJson();
} elseif ($response->isRedirect()) {
    // Status 300-399
    $location = $response->getHeaderLine('Location');
} else {
    // Erro
    $code = $response->getStatusCode();
    $body = $response->getStringBody();
}
```

### Acessar dados

```php
$response = $http->get('https://api.example.com/data');

// String bruto
$text = $response->getStringBody();

// JSON decodificado
$data = $response->getJson();  // Array

// XML como SimpleXMLElement
$xml = $response->getXml();

// Headers
$contentType = $response->getHeaderLine('Content-Type');
$headers = $response->getHeaders();

// Cookies
$cookies = $response->getCookies();
$sessionId = $response->getCookie('session_id');
```

## Eventos

### beforeSend

Modificar requisição antes de enviar:

```php
$http->getEventManager()->on(
    'HttpClient.beforeSend',
    function($event, $request, $adapterOptions, $redirects) {
        // Adicionar header customizado
        $request = $request->withHeader('X-Request-ID', uniqid());

        $event->setRequest($request);
    }
);
```

### afterSend

Processar resposta após receber:

```php
$http->getEventManager()->on(
    'HttpClient.afterSend',
    function($event, $request, $adapterOptions, $redirects, $requestSent) {
        $response = $event->getResponse();

        // Log de resposta
        if ($response->getStatusCode() >= 400) {
            error_log('HTTP Error: ' . $response->getStatusCode());
        }
    }
);
```

## Exemplo Prático Completo

```php
<?php
namespace App\Service;

use Cake\Http\Client;

class GithubApi
{
    private $http;
    private $token;

    public function __construct(string $token)
    {
        $this->token = $token;
        $this->http = new Client([
            'host' => 'api.github.com',
            'scheme' => 'https',
            'headers' => [
                'Authorization' => 'Bearer ' . $token,
                'Accept' => 'application/vnd.github.v3+json'
            ]
        ]);
    }

    public function getUser(string $username): array
    {
        $response = $this->http->get("/users/$username");

        if ($response->isOk()) {
            return $response->getJson();
        }

        throw new \Exception('GitHub API Error: ' . $response->getStatusCode());
    }

    public function listRepos(string $username): array
    {
        $response = $this->http->get("/users/$username/repos");

        if (!$response->isOk()) {
            return [];
        }

        return $response->getJson();
    }

    public function createIssue(string $owner, string $repo, array $issue): array
    {
        $response = $this->http->post(
            "/repos/$owner/$repo/issues",
            json_encode($issue),
            ['type' => 'json']
        );

        return $response->getJson();
    }
}

// Uso
$github = new GithubApi(env('GITHUB_TOKEN'));
$user = $github->getUser('cakephp');
$repos = $github->listRepos('cakephp');
?>
```

## Testes com HttpClientTrait

```php
<?php
namespace App\Test\TestCase\Service;

use Cake\Http\TestSuite\HttpClientTrait;
use Cake\TestSuite\TestCase;
use App\Service\PaymentService;

class PaymentServiceTest extends TestCase
{
    use HttpClientTrait;

    public function testProcessPayment()
    {
        // Mock resposta da API de pagamento
        $this->mockClientPost(
            'https://payment.provider.com/charge',
            $this->newClientResponse(200, [], json_encode([
                'id' => 'charge_123',
                'status' => 'succeeded',
                'amount' => 9999
            ]))
        );

        $payment = new PaymentService();
        $result = $payment->charge(99.99);

        $this->assertTrue($result['succeeded']);
    }
}
?>
```

## Quick Start

```php
// 1. Importar
use Cake\Http\Client;

// 2. Criar cliente
$http = new Client(['host' => 'api.example.com']);

// 3. Fazer requisição
$response = $http->get('/endpoint', ['param' => 'value']);

// 4. Processar resposta
if ($response->isOk()) {
    $data = $response->getJson();
}
```

## Recursos Adicionais

- [PSR-18 HTTP Client Interface](https://www.php-fig.org/psr/psr-18/)
- [API HttpClient](https://api.cakephp.org/4.x/class-Cake.Http.Client.html)
- [Documentação oficial](https://book.cakephp.org/4/en/core-libraries/httpclient.html)
