# Registry Objects: Carregamento Dinâmico de Componentes

## Visão Geral

Registry pattern em CakePHP permite carregar **Components**, **Helpers**, **Behaviors** e **Tasks** **dynamicamente** em runtime com configuração inline. Substitui callbacks por eventos através do **Event System**.

!!! info
    Registry é usado internamente por Controllers, Views, e Models para gerenciar dependências injetáveis.

## Carregar Objetos Dinamicamente

### Métodos de Carregamento

Controllers, Models, Views e Components herdam registries para carregar dependências:

```php
use App\Controller\AppController;

class UsersController extends AppController
{
    public function initialize()
    {
        parent::initialize();

        // Carregar componentes
        $this->loadComponent('Authentication.Authentication');
        $this->loadComponent('Authorization.Authorization');

        // Carregar helpers
        $this->addHelper('Form');
        $this->addHelper('Paginator');
    }

    public function add()
    {
        // Carregar em ação específica
        $this->loadComponent('Email');

        $email = $this->Email;
        $email->send('...');
    }
}
```

### Em Models (Behaviors)

```php
<?php
namespace App\Model\Table;

use Cake\ORM\Table;
use Cake\Validation\Validator;

class UsersTable extends Table
{
    public function initialize(array $config): void
    {
        parent::initialize($config);

        // Carregar behaviors
        $this->addBehavior('Timestamp');
        $this->addBehavior('Search.Search');
        $this->addBehavior('Tree');
    }

    public function validationDefault(Validator $validator): Validator
    {
        // ...
    }
}
?>
```

## Configuração Inline

Passar configuração ao carregar:

```php
// Configuração simples
$this->loadComponent('Cookie', ['name' => 'sweet']);

// Múltiplas opções
$this->loadComponent('Paginator', [
    'limit' => 25,
    'maxLimit' => 100,
    'whitelist' => ['id', 'name', 'created']
]);

// Behaviors com config
$this->addBehavior('Search.Search', [
    'filterClass' => 'Search.Manager',
]);
```

## Aliasing com className

Use `className` para criar aliases ou estender classes core:

```php
// Estender componente core com versão customizada
$this->loadComponent('Flash', [
    'className' => 'App.CustomFlash'
]);

// Agora $this->Flash usa CustomFlash
$this->Flash->success('Message');

// Alias de nome customizado
$this->loadComponent('MyAuth', [
    'className' => 'Authentication.Authentication'
]);

// Usar sob novo nome
$identity = $this->MyAuth->getIdentity();
```

Útil para:
- Estender componentes core sem alterar nome original
- Usar múltiplas instâncias de mesma classe com configs diferentes
- Manter compatibilidade com código legado

## Event System

### Substituição de Callbacks

Em versões antigas, callbacks como `beforeFilter`, `afterFilter` eram disparados em componentes. Agora use o **Event System**:

```php
// Em Component
class MyComponent extends Component
{
    public function __construct(ComponentRegistry $registry, array $config = [])
    {
        parent::__construct($registry, $config);

        // Registrar evento
        $this->getEventManager()->on(
            'Controller.initialize',
            [$this, 'onControllerInitialize']
        );
    }

    public function onControllerInitialize($event)
    {
        // Executar lógica
    }
}

// Em Controller
public function initialize()
{
    parent::initialize();

    // Disparar evento
    $this->dispatchEvent('Controller.initialize');
}
```

### Desabilitar/Reabilitar Componentes

```php
// Desabilitar componente de callbacks
$eventManager = $this->getEventManager();
$eventManager->off($this->MyComponent);

// Reabilitar
$eventManager->on($this->MyComponent);
```

## Exemplo Prático Completo

### Sistema de Autenticação com Registry

```php
<?php
// src/Controller/AppController.php
namespace App\Controller;

use Cake\Controller\Controller;

class AppController extends Controller
{
    public function initialize(): void
    {
        parent::initialize();

        // Carregar componentes essenciais
        $this->loadComponent('RequestHandler');
        $this->loadComponent('Flash');
        $this->loadComponent('FormProtection');
        $this->loadComponent('Authentication.Authentication');
        $this->loadComponent('Authorization.Authorization');

        // Carregar helpers padrão
        $this->loadHelper('Form');
        $this->loadHelper('Html');
        $this->loadHelper('Paginator');
        $this->loadHelper('Url');
    }

    protected function beforeFilter(EventInterface $event)
    {
        parent::beforeFilter($event);

        // Configuração por ação
        $this->FormProtection->setConfig('unlockedActions', [
            'index',
            'view',
            'search'
        ]);
    }
}

// src/Controller/UsersController.php
namespace App\Controller;

class UsersController extends AppController
{
    public function initialize(): void
    {
        parent::initialize();

        // Carregar componentes específicos desta controller
        $this->loadComponent('ApiToken');
        $this->loadComponent('Crud.Crud', [
            'actions' => ['Crud.Index', 'Crud.View', 'Crud.Add'],
            'listeners' => ['Api']
        ]);
    }

    public function login()
    {
        // Desabilitar CSRF para POST de login
        $this->FormProtection->setConfig('unlockedActions', ['login']);

        if ($this->request->is('post')) {
            $result = $this->Authentication->getResult();

            if ($result && $result->isValid()) {
                $this->Authentication->setIdentity($result->getData());
                return $this->redirect(['action' => 'dashboard']);
            }

            $this->Flash->error(__('Invalid credentials'));
        }
    }

    public function logout()
    {
        $this->Authentication->logout();
        return $this->redirect('/');
    }
}

// src/Model/Table/UsersTable.php
namespace App\Model\Table;

use Cake\ORM\Table;
use Cake\Validation\Validator;

class UsersTable extends Table
{
    public function initialize(array $config): void
    {
        parent::initialize($config);

        // Behaviors essenciais
        $this->addBehavior('Timestamp', [
            'events' => [
                'Model.beforeSave' => [
                    'created' => 'new',
                    'modified' => 'always',
                ]
            ]
        ]);

        $this->addBehavior('Search.Search');
        $this->addBehavior('Sluggable', [
            'field' => 'username',
            'slug' => 'slug'
        ]);
    }
}
?>
```

### Custom Component com Registry

```php
<?php
// src/Controller/Component/ApiComponent.php
namespace App\Controller\Component;

use Cake\Controller\Component;
use Cake\Event\EventInterface;

class ApiComponent extends Component
{
    protected array $defaultConfig = [
        'apiKey' => null,
        'baseUrl' => 'https://api.example.com',
        'timeout' => 30
    ];

    public function initialize(array $config): void
    {
        parent::initialize($config);

        // Registrar eventos
        $this->getEventManager()->on(
            'Controller.startup',
            [$this, 'beforeControllerAction']
        );
    }

    public function beforeControllerAction(EventInterface $event)
    {
        // Validar API key
        $request = $this->getController()->getRequest();

        if (!$request->getHeaderLine('X-Api-Key')) {
            throw new UnauthorizedException('API key required');
        }
    }

    public function call($endpoint, $method = 'GET', $data = null)
    {
        // Implementar chamada API
    }
}

// Uso em controller
class ApiController extends AppController
{
    public function initialize(): void
    {
        parent::initialize();

        $this->loadComponent('Api', [
            'apiKey' => env('API_KEY'),
            'baseUrl' => 'https://api.example.com/v1'
        ]);
    }

    public function getData()
    {
        $data = $this->Api->call('/endpoint/data');
        $this->set(compact('data'));
    }
}
?>
```

## Quick Start

```php
// 1. Em controller initialize()
public function initialize()
{
    parent::initialize();

    // 2. Carregar componentes
    $this->loadComponent('Flash');
    $this->loadComponent('Auth');

    // 3. Com configuração
    $this->loadComponent('Paginator', ['limit' => 50]);

    // 4. Com alias
    $this->loadComponent('MyAuth', ['className' => 'Auth']);
}

// 5. Usar evento (ao invés de callback)
$this->getEventManager()->on('Event.name', [$this, 'method']);
```

## Diferenças de Versões

| Versão | Callbacks | Events | Registry |
|--------|-----------|--------|----------|
| 2.x | Sim | Não | Simples |
| 3.x | Sim | Parcial | Melhorado |
| 4.x+ | Não | Sim | Padrão |

## Recursos Adicionais

- [API ComponentRegistry](https://api.cakephp.org/4.x/class-Cake.Controller.ComponentRegistry.html)
- [Event System](https://book.cakephp.org/4/en/core-libraries/events.html)
- [Documentação oficial](https://book.cakephp.org/4/en/controllers/components.html)
