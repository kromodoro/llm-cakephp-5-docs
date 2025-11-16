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

---

## 📚 Referências

- [Request & Response - CakePHP Book](https://book.cakephp.org/5/en/controllers/request-response.html)
- [PSR-7: HTTP Messages](https://www.php-fig.org/psr/psr-7/)

---

**✨ Criado com [Claude Code](https://claude.com/claude-code)**
