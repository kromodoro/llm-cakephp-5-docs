# Request & Response (Requisição e Resposta)

!!! info "PSR-7 HTTP Messages"
    CakePHP usa objetos **ServerRequest** e **Response** compatíveis com PSR-7.

---

## 📋 Conteúdo

- **ServerRequest** - Ler dados da requisição
- **Response** - Criar respostas HTTP
- **Cookies** - Manipular cookies
- **Files** - Upload de arquivos
- **Headers** - Headers HTTP
- **Content-Type** - Negociação de conteúdo

---

## 📥 ServerRequest - Lendo Requisições

### Parâmetros de Rota

```php
// URL: /articles/view/5
$controller = $this->request->getParam('controller');  // 'Articles'
$action = $this->request->getParam('action');          // 'view'
$id = $this->request->getParam('pass')[0];             // 5
```

### Query String

```php
// URL: /search?q=cakephp&page=2
$query = $this->request->getQuery('q');        // 'cakephp'
$page = $this->request->getQuery('page');      // 2
$all = $this->request->getQueryParams();       // ['q' => 'cakephp', 'page' => 2]
```

### POST Data

```php
// Form POST
$title = $this->request->getData('title');
$allData = $this->request->getData();

// Nested data
$tags = $this->request->getData('article.tags');
```

### Método HTTP

```php
if ($this->request->is('post')) {
    // POST request
}

if ($this->request->is(['put', 'patch'])) {
    // PUT ou PATCH
}

// Outros métodos:
$this->request->is('get');
$this->request->is('delete');
$this->request->is('ajax');
$this->request->is('json');
$this->request->is('ssl');
```

### Headers

```php
$contentType = $this->request->getHeaderLine('Content-Type');
$accept = $this->request->getHeaderLine('Accept');
$userAgent = $this->request->getHeaderLine('User-Agent');
```

### Cookies

```php
$value = $this->request->getCookie('name');
$value = $this->request->getCookie('name', 'default');
```

### Environment/Server

```php
$ip = $this->request->clientIp();
$host = $this->request->host();
$scheme = $this->request->scheme();  // 'http' or 'https'
$method = $this->request->getMethod();  // 'GET', 'POST', etc.
```

### Files (Upload)

```php
// <input type="file" name="attachment">
$file = $this->request->getData('attachment');

// Properties:
$file->getClientFilename();  // 'document.pdf'
$file->getClientMediaType(); // 'application/pdf'
$file->getSize();            // 1024000 (bytes)
$file->getError();           // UPLOAD_ERR_OK

// Move uploaded file:
$file->moveTo('/path/to/uploads/document.pdf');
```

---

## 📤 Response - Criando Respostas

### Texto Simples

```php
$response = $this->response->withStringBody('Hello World');
return $response;
```

### JSON

```php
$data = ['status' => 'ok', 'message' => 'Success'];
$response = $this->response
    ->withType('application/json')
    ->withStringBody(json_encode($data));
return $response;
```

### Headers

```php
$response = $this->response
    ->withHeader('X-Custom-Header', 'value')
    ->withAddedHeader('X-Multi', 'value1')
    ->withAddedHeader('X-Multi', 'value2');
```

### Status Code

```php
$response = $this->response->withStatus(404);  // Not Found
$response = $this->response->withStatus(201);  // Created
$response = $this->response->withStatus(500);  // Server Error
```

### Cookies

```php
use Cake\Http\Cookie\Cookie;
use Cake\I18n\DateTime;

$cookie = new Cookie(
    'user_id',
    '123',
    new DateTime('+1 year'),
    '/',
    '',
    false,  // secure
    true    // httponly
);

$response = $this->response->withCookie($cookie);
```

### Download de Arquivo

```php
$response = $this->response
    ->withFile('/path/to/file.pdf')
    ->withDownload('document.pdf');
return $response;
```

### Redirect

```php
return $this->redirect(['controller' => 'Articles', 'action' => 'index']);
return $this->redirect('/articles', 301);  // Permanent
```

---

## 🍪 Cookies Avançados

```php
use Cake\Http\Cookie\Cookie;

// Criar cookie:
$cookie = Cookie::create('theme', 'dark')
    ->withExpiry(new DateTime('+1 month'))
    ->withPath('/')
    ->withDomain('.example.com')
    ->withSecure(true)
    ->withHttpOnly(true)
    ->withSameSite('Lax');

$response = $this->response->withCookie($cookie);

// Ler cookie:
$theme = $this->request->getCookie('theme');

// Deletar cookie:
$response = $this->response->withExpiredCookie(new Cookie('theme'));
```

---

## 📁 Upload de Arquivos

```php
public function upload()
{
    if ($this->request->is('post')) {
        $file = $this->request->getData('photo');

        // Validar:
        if ($file->getError() === UPLOAD_ERR_OK) {
            // Gerar nome único:
            $filename = uniqid() . '_' . $file->getClientFilename();

            // Mover para pasta:
            $destination = WWW_ROOT . 'uploads/' . $filename;
            $file->moveTo($destination);

            $this->Flash->success('Upload realizado!');
        } else {
            $this->Flash->error('Erro no upload.');
        }
    }
}
```

---

## 🔍 Content-Type Negotiation

```php
// Controller aceita JSON e XML:
$this->addViewClasses([JsonView::class, XmlView::class]);

// Template:
if ($this->request->is('json')) {
    // Lógica específica para JSON
}
```

---

---

## 🔍 Request Detectors Customizados

### Tipos de Detectors

```php
// 1. Environment detector - Compara $_SERVER
$this->request->addDetector('mobile', [
    'env' => 'HTTP_USER_AGENT',
    'pattern' => '/(iPhone|Android|Mobile)/'
]);

// 2. Pattern detector - Regex em qualquer valor
$this->request->addDetector('apiKey', [
    'pattern' => '/^[a-f0-9]{32}$/',
    'param' => 'api_key'
]);

// 3. Option detector - Compara com lista
$this->request->addDetector('premium', [
    'param' => 'plan',
    'value' => ['premium', 'enterprise']
]);

// 4. Callback detector - Lógica customizada
$this->request->addDetector('weekend', function ($request) {
    $day = date('N');
    return ($day == 6 || $day == 7);
});

// 5. Header detector
$this->request->addDetector('bearer', [
    'header' => 'Authorization',
    'pattern' => '/^Bearer\s+/'
]);
```

### Usar Detectors

```php
if ($this->request->is('mobile')) {
    // Versão mobile
}

if ($this->request->is('weekend')) {
    // Promoção de fim de semana
}

if ($this->request->is('bearer')) {
    $token = str_replace('Bearer ', '', $this->request->getHeaderLine('Authorization'));
    // Validar token JWT
}
```

---

## 🌐 CORS (Cross-Origin Resource Sharing)

### Configurar CORS no Controller

```php
<?php
namespace App\Controller;

class ApiController extends AppController
{
    public function initialize(): void
    {
        parent::initialize();

        // Aplicar CORS em todas as actions
        $this->response = $this->response->cors($this->request)
            ->allowOrigin(['https://app.example.com', 'https://mobile.example.com'])
            ->allowMethods(['GET', 'POST', 'PUT', 'DELETE'])
            ->allowHeaders(['Content-Type', 'Authorization', 'X-Requested-With'])
            ->allowCredentials()
            ->exposeHeaders(['X-Total-Count', 'X-Page-Number'])
            ->maxAge(3600)
            ->build();
    }

    public function index()
    {
        $data = ['status' => 'ok', 'data' => []];
        $this->set(compact('data'));
        $this->viewBuilder()->setOption('serialize', 'data');
    }
}
```

### CORS Wildcard (Desenvolvimento)

```php
// ⚠️ Apenas para desenvolvimento - NUNCA em produção
$this->response = $this->response->cors($this->request)
    ->allowOrigin(['*'])
    ->allowMethods(['*'])
    ->allowHeaders(['*'])
    ->build();
```

### CORS Middleware (Preferível)

```php
// src/Application.php
public function middleware(MiddlewareQueue $middlewareQueue): MiddlewareQueue
{
    $middlewareQueue->add(new \Cake\Http\Middleware\CorsMiddleware([
        'origin' => ['https://app.example.com'],
        'methods' => ['GET', 'POST'],
        'headers' => ['Content-Type', 'Authorization'],
        'credentials' => true,
        'maxAge' => 3600,
    ]));

    return $middlewareQueue;
}
```

---

## 📡 Streaming de Respostas

### Streaming de Arquivo Grande

```php
use Laminas\Diactoros\Stream;

public function download($id)
{
    $video = $this->Videos->get($id);
    $path = WWW_ROOT . 'files/' . $video->filename;

    // Stream eficiente para arquivos grandes
    $stream = new Stream($path, 'rb');

    return $this->response
        ->withType($video->mime_type)
        ->withHeader('Content-Disposition', 'attachment; filename="' . $video->filename . '"')
        ->withHeader('Content-Length', (string)filesize($path))
        ->withBody($stream);
}
```

### Streaming com Callback (Geração Dinâmica)

```php
use Cake\Http\CallbackStream;

public function exportCsv()
{
    $articles = $this->Articles->find('all');

    $stream = new CallbackStream(function () use ($articles) {
        $output = fopen('php://output', 'w');

        // Header CSV
        fputcsv($output, ['ID', 'Título', 'Criado em']);

        // Dados
        foreach ($articles as $article) {
            fputcsv($output, [
                $article->id,
                $article->title,
                $article->created->format('Y-m-d H:i:s')
            ]);
        }

        fclose($output);
    });

    return $this->response
        ->withType('text/csv')
        ->withDownload('articles.csv')
        ->withBody($stream);
}
```

### Gerar Imagem Dinamicamente

```php
public function generateQrCode($data)
{
    // Biblioteca QR Code (exemplo)
    $qrCode = new \Endroid\QrCode\QrCode($data);

    $stream = new CallbackStream(function () use ($qrCode) {
        echo $qrCode->writeString();
    });

    return $this->response
        ->withType('image/png')
        ->withBody($stream);
}
```

### Gerar PDF em Streaming

```php
public function invoice($id)
{
    $invoice = $this->Invoices->get($id, ['contain' => ['Items']]);

    $stream = new CallbackStream(function () use ($invoice) {
        // Gerar PDF com biblioteca (ex: TCPDF, DomPDF)
        $pdf = new \TCPDF();
        $pdf->AddPage();
        $pdf->writeHTML($this->renderInvoiceHtml($invoice));
        echo $pdf->Output('', 'S'); // String output
    });

    return $this->response
        ->withType('application/pdf')
        ->withDownload("invoice-{$id}.pdf")
        ->withBody($stream);
}
```

---

## 🔒 HTTP Caching (Cache de Navegador)

### Cache com ETag

```php
public function view($id)
{
    $article = $this->Articles->get($id);

    // Gerar ETag baseado em conteúdo
    $etag = md5(serialize($article));

    $this->response = $this->response->withEtag($etag);

    // Se cliente tem cache válido, retornar 304 Not Modified
    if ($this->response->isNotModified($this->request)) {
        return $this->response;
    }

    $this->set(compact('article'));
}
```

### Cache com Last-Modified

```php
public function index()
{
    $articles = $this->Articles->find('all');

    // Última modificação
    $lastModified = $this->Articles->find()
        ->select(['modified'])
        ->orderBy(['modified' => 'DESC'])
        ->first()
        ->modified;

    $this->response = $this->response->withModified($lastModified);

    if ($this->response->isNotModified($this->request)) {
        return $this->response;
    }

    $this->set(compact('articles'));
}
```

### Cache Público vs Privado

```php
// Cache público (CDN pode cachear)
$this->response = $this->response->withSharable(true)
    ->withCache('-1 minute', '+1 hour');

// Cache privado (apenas navegador)
$this->response = $this->response->withSharable(false)
    ->withCache('-1 minute', '+10 minutes');

// Desabilitar cache completamente
$this->response = $this->response->withDisabledCache();
```

### Vary Header (Variar cache por condição)

```php
// Cache diferente para cada User-Agent
$this->response = $this->response->withVary('User-Agent');

// Múltiplos critérios
$this->response = $this->response->withVary('Accept-Language, Accept-Encoding');
```

---

## 🔐 Segurança em Request/Response

### Validar Método HTTP

```php
public function delete($id)
{
    // Aceitar apenas DELETE e POST
    $this->request->allowMethod(['delete', 'post']);

    $article = $this->Articles->get($id);
    if ($this->Articles->delete($article)) {
        $this->Flash->success('Deletado!');
    }
    return $this->redirect(['action' => 'index']);
}
```

### Bearer Token Authentication

```php
public function beforeFilter(\Cake\Event\EventInterface $event)
{
    parent::beforeFilter($event);

    if ($this->request->is('api')) {
        $token = $this->request->getHeaderLine('Authorization');

        if (!str_starts_with($token, 'Bearer ')) {
            throw new \Cake\Http\Exception\UnauthorizedException('Token inválido');
        }

        $jwt = str_replace('Bearer ', '', $token);
        // Validar JWT...
    }
}
```

### Sanitizar Input

```php
// Usar getData() que já escapa HTML
$title = $this->request->getData('title');  // Seguro

// NUNCA use $_POST diretamente
// $title = $_POST['title'];  // ❌ Inseguro!

// Para query strings, use getQuery()
$search = $this->request->getQuery('q');  // Seguro
```

### Content Security Policy

```php
public function beforeRender(\Cake\Event\EventInterface $event)
{
    parent::beforeRender($event);

    $this->response = $this->response->withHeader(
        'Content-Security-Policy',
        "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'"
    );
}
```

---

## 🌍 Trusted Proxies

Para aplicações atrás de proxies (Nginx, CloudFlare, Load Balancer):

```php
// config/bootstrap.php ou AppController::initialize()
use Cake\Http\ServerRequest;

ServerRequest::trustProxy(true);

// Ou especificar proxies confiáveis
ServerRequest::setTrustedProxies(['192.168.1.10', '10.0.0.0/8']);
```

Isso permite ler corretamente:
- `clientIp()` - IP real do cliente (de X-Forwarded-For)
- `scheme()` - HTTPS original (de X-Forwarded-Proto)
- `host()` - Host original (de X-Forwarded-Host)

---

## 📋 Type Casting Seguro

CakePHP 5 oferece helpers para converter dados com segurança:

```php
use function Cake\Utility\toBool;
use function Cake\Utility\toInt;
use function Cake\Utility\toString;
use function Cake\Utility\toFloat;

// Converter query string para tipos específicos
$page = toInt($this->request->getQuery('page'), 1);          // Default: 1
$active = toBool($this->request->getQuery('active'), true);  // Default: true
$search = toString($this->request->getQuery('q'), '');       // Default: ''
$price = toFloat($this->request->getQuery('max'), 100.0);    // Default: 100.0

// Datas
use function Cake\Utility\toDate;
use function Cake\Utility\toDateTime;

$date = toDate($this->request->getQuery('date'));      // \Cake\I18n\Date
$datetime = toDateTime($this->request->getData('created')); // \Cake\I18n\DateTime
```

---

## 🎯 Exemplo Completo: API REST com CORS e Caching

```php
<?php
namespace App\Controller\Api;

use App\Controller\AppController;
use Cake\Http\Exception\MethodNotAllowedException;

class ArticlesController extends AppController
{
    public function initialize(): void
    {
        parent::initialize();

        // Configurar CORS
        $this->response = $this->response->cors($this->request)
            ->allowOrigin(['https://app.example.com'])
            ->allowMethods(['GET', 'POST', 'PUT', 'DELETE'])
            ->allowHeaders(['Content-Type', 'Authorization'])
            ->allowCredentials()
            ->exposeHeaders(['X-Total-Count'])
            ->maxAge(3600)
            ->build();

        // JSON view
        $this->viewBuilder()->setClassName('Json');
    }

    public function index()
    {
        // Apenas GET
        $this->request->allowMethod(['get']);

        $query = $this->Articles->find('all')
            ->contain(['Users', 'Categories'])
            ->limit(20);

        // Total para header
        $total = $this->Articles->find()->count();

        // Cache com ETag
        $etag = md5(serialize($query->toArray()) . $total);
        $this->response = $this->response
            ->withEtag($etag)
            ->withHeader('X-Total-Count', (string)$total);

        if ($this->response->isNotModified($this->request)) {
            return $this->response;
        }

        $articles = $query->toArray();
        $this->set(compact('articles'));
        $this->viewBuilder()->setOption('serialize', 'articles');
    }

    public function view($id)
    {
        $this->request->allowMethod(['get']);

        $article = $this->Articles->get($id, ['contain' => ['Users', 'Categories']]);

        // Cache por 1 hora
        $this->response = $this->response
            ->withModified($article->modified)
            ->withCache('-1 minute', '+1 hour')
            ->withSharable(true);

        if ($this->response->isNotModified($this->request)) {
            return $this->response;
        }

        $this->set(compact('article'));
        $this->viewBuilder()->setOption('serialize', 'article');
    }

    public function add()
    {
        $this->request->allowMethod(['post']);

        $article = $this->Articles->newEntity($this->request->getData());

        if ($this->Articles->save($article)) {
            $this->response = $this->response->withStatus(201);
            $message = 'Artigo criado';
        } else {
            $this->response = $this->response->withStatus(422);
            $message = 'Erro ao criar artigo';
        }

        $this->set(compact('article', 'message'));
        $this->viewBuilder()->setOption('serialize', ['article', 'message']);
    }
}
```

---

## 📚 Resumo Rápido

### Request

| Método | O que faz |
|--------|-----------|
| `getParam('name')` | Parâmetro de rota |
| `getQuery('name')` | Query string |
| `getData('name')` | POST/PUT data |
| `is('post')` | Verificar método HTTP |
| `getHeaderLine('Name')` | Ler header |
| `getCookie('name')` | Ler cookie |
| `clientIp()` | IP do cliente |
| `addDetector()` | Criar detector customizado |
| `allowMethod()` | Validar método HTTP |
| `getUploadedFile()` | Arquivo de upload |

### Response

| Método | O que faz |
|--------|-----------|
| `withStringBody($text)` | Definir corpo |
| `withType('json')` | Content-Type |
| `withStatus(404)` | Status HTTP |
| `withHeader('Name', 'val')` | Adicionar header |
| `withCookie($cookie)` | Definir cookie |
| `withFile($path)` | Servir arquivo |
| `withDownload($name)` | Forçar download |
| `withEtag()` | Cache com ETag |
| `withModified()` | Cache com Last-Modified |
| `cors()` | Configurar CORS |
| `withBody(StreamInterface)` | Streaming |

---

## 📚 Referências

- [Request & Response - CakePHP Book](https://book.cakephp.org/5/en/controllers/request-response.html)
- [PSR-7: HTTP Messages](https://www.php-fig.org/psr/psr-7/)
- [Cookie Collection](https://book.cakephp.org/5/en/controllers/request-response.html#cookie-collections)
- [CORS Middleware](https://book.cakephp.org/5/en/controllers/middleware.html#cross-origin-request-sharing-cors-middleware)

---

**✨ Expandido com [Claude Code](https://claude.com/claude-code)**
