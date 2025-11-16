# Tutorial CMS - Autenticação (Login/Logout)

!!! info "Parte 6 do Tutorial CMS"
    ✅ Pré-requisitos: [Instalação](cms-installation.md) | [Database](cms-database.md) | [Model](cms-articles-model.md) | [Controller](cms-articles-controller.md) | [Tags & Users](cms-tags-and-users.md)

---

## 📋 O Que Você Vai Aprender

- 🔐 **Instalar** plugin Authentication
- 🔒 **Hash de senhas** com bcrypt
- 🚪 **Login** com email/senha
- 👋 **Logout** seguro
- 📝 **Registro** de novos usuários
- 🛡️ **Proteção** de rotas (páginas públicas vs privadas)
- 🔄 **Redirect** automático após login

---

## 🔐 Instalando Authentication Plugin

```bash
cd /home/kromodoro/Documentos/cakephp/app
composer require "cakephp/authentication:^3.0"
```

**Output esperado:**
```
Using version ^3.0 for cakephp/authentication
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 1 install, 0 updates, 0 removals
  - Installing cakephp/authentication (3.0.0)
Writing lock file
Generating autoload files
```

---

## 🔒 Hash de Senhas (CRÍTICO!)

### Adicionar _setPassword() em User Entity

Edite `src/Model/Entity/User.php`:

```php
<?php
namespace App\Model\Entity;

use Authentication\PasswordHasher\DefaultPasswordHasher;
use Cake\ORM\Entity;

class User extends Entity
{
    protected array $_accessible = [
        'email' => true,
        'password' => true,
        'created' => true,
        'modified' => true,
        'articles' => true,
    ];

    protected array $_hidden = ['password'];

    // Hash automático ao setar senha
    protected function _setPassword(string $password): ?string
    {
        if (strlen($password) > 0) {
            return (new DefaultPasswordHasher())->hash($password);
        }
        return null;
    }
}
```

### Testar Hash

1. Acesse: [http://localhost:8765/users/edit/1](http://localhost:8765/users/edit/1)
2. Mude senha para: `senha123`
3. Salvar
4. Verifique no banco: senha agora é hash bcrypt (`$2y$10$...`)

---

## ⚙️ Configurar Authentication

### Etapa 1: Atualizar Application.php

Edite `src/Application.php`:

```php
<?php
namespace App;

use Authentication\AuthenticationService;
use Authentication\AuthenticationServiceInterface;
use Authentication\AuthenticationServiceProviderInterface;
use Authentication\Middleware\AuthenticationMiddleware;
use Cake\Http\BaseApplication;
use Cake\Http\MiddlewareQueue;
use Cake\Routing\Router;
use Psr\Http\Message\ServerRequestInterface;

class Application extends BaseApplication implements AuthenticationServiceProviderInterface
{
    public function middleware(MiddlewareQueue $middlewareQueue): MiddlewareQueue
    {
        $middlewareQueue
            ->add(new \Cake\Error\Middleware\ErrorHandlerMiddleware(null, $this->config))
            ->add(new \Cake\Http\Middleware\AssetMiddleware([
                'cacheTime' => '+1 year',
            ]))
            ->add(new \Cake\Routing\Middleware\RoutingMiddleware($this))
            ->add(new \Cake\Http\Middleware\BodyParserMiddleware())

            // ← ADICIONAR AuthenticationMiddleware
            ->add(new AuthenticationMiddleware($this));

        return $middlewareQueue;
    }

    public function getAuthenticationService(ServerRequestInterface $request): AuthenticationServiceInterface
    {
        $service = new AuthenticationService([
            'unauthenticatedRedirect' => Router::url('/users/login'),
            'queryParam' => 'redirect',
        ]);

        // 1. Session authenticator (verifica sessão)
        $service->loadAuthenticator('Authentication.Session');

        // 2. Form authenticator (verifica email/senha)
        $service->loadAuthenticator('Authentication.Form', [
            'fields' => [
                'username' => 'email',
                'password' => 'password',
            ],
            'loginUrl' => Router::url('/users/login'),
        ]);

        // Identifier (como buscar usuário)
        $service->loadIdentifier('Authentication.Password', [
            'fields' => [
                'username' => 'email',
                'password' => 'password',
            ],
        ]);

        return $service;
    }
}
```

---

### Etapa 2: Carregar AuthenticationComponent

Edite `src/Controller/AppController.php`:

```php
public function initialize(): void
{
    parent::initialize();

    $this->loadComponent('RequestHandler');
    $this->loadComponent('Flash');

    // ← ADICIONAR Authentication Component
    $this->loadComponent('Authentication.Authentication');
}

// Permitir index/view sem login
public function beforeFilter(\Cake\Event\EventInterface $event): void
{
    parent::beforeFilter($event);

    // Páginas públicas (não requer login):
    $this->Authentication->addUnauthenticatedActions(['index', 'view']);
}
```

---

## 🚪 Implementar Login

### Controller: UsersController.php

```php
<?php
// src/Controller/UsersController.php
namespace App\Controller;

class UsersController extends AppController
{
    public function beforeFilter(\Cake\Event\EventInterface $event): void
    {
        parent::beforeFilter($event);

        // Permitir login e registro sem autenticação
        $this->Authentication->addUnauthenticatedActions(['login', 'add']);
    }

    public function login()
    {
        $this->request->allowMethod(['get', 'post']);
        $result = $this->Authentication->getResult();

        // Se já logado, redirecionar
        if ($result && $result->isValid()) {
            // Redirect para página solicitada ou articles/index
            $redirect = $this->request->getQuery('redirect', [
                'controller' => 'Articles',
                'action' => 'index',
            ]);

            return $this->redirect($redirect);
        }

        // Se POST e falhou, mostrar erro
        if ($this->request->is('post') && !$result->isValid()) {
            $this->Flash->error(__('E-mail ou senha inválidos'));
        }
    }

    public function logout()
    {
        $result = $this->Authentication->getResult();

        if ($result && $result->isValid()) {
            $this->Authentication->logout();
            return $this->redirect(['controller' => 'Users', 'action' => 'login']);
        }
    }

    // ... outras actions (index, view, add, edit, delete) ...
}
```

---

### Template: login.php

Crie `templates/Users/login.php`:

```php
<!-- File: templates/Users/login.php -->

<div class="users form content">
    <h3><?= __('Login') ?></h3>

    <?= $this->Form->create() ?>
    <fieldset>
        <legend><?= __('Digite seu e-mail e senha') ?></legend>

        <?= $this->Form->control('email', [
            'label' => 'E-mail',
            'required' => true,
            'autofocus' => true,
            'autocomplete' => 'email',
        ]) ?>

        <?= $this->Form->control('password', [
            'label' => 'Senha',
            'required' => true,
            'autocomplete' => 'current-password',
        ]) ?>
    </fieldset>

    <div class="actions">
        <?= $this->Form->button(__('Entrar'), ['class' => 'button']) ?>
    </div>
    <?= $this->Form->end() ?>

    <p class="text-center">
        <?= __('Não tem conta?') ?>
        <?= $this->Html->link(__('Registre-se'), ['action' => 'add']) ?>
    </p>
</div>
```

---

## ✅ Testando Login

1. **Acesse:** [http://localhost:8765/users/login](http://localhost:8765/users/login)
2. **Digite:**
   - Email: `admin@example.com`
   - Senha: `senha123`
3. **Clique em "Entrar"**
4. **Resultado:** Redirecionado para `/articles` ✅

---

## 📝 Habilitar Registro (Sign Up)

O método `add()` já existe (gerado pelo Bake). Apenas garantir que está acessível sem login:

```php
// src/Controller/UsersController.php
public function beforeFilter(\Cake\Event\EventInterface $event): void
{
    parent::beforeFilter($event);

    // add já está na lista de unauthenticated actions:
    $this->Authentication->addUnauthenticatedActions(['login', 'add']);
}
```

**Testar:**
1. Logout: [http://localhost:8765/users/logout](http://localhost:8765/users/logout)
2. Registrar: [http://localhost:8765/users/add](http://localhost:8765/users/add)
3. Preencher email/senha
4. Salvar
5. Login com nova conta! ✅

---

## 🎨 Melhorias de UX

### Mostrar Usuário Logado no Layout

Edite `templates/layout/default.php`:

```php
<!-- Dentro de <body>, antes do conteúdo -->

<nav class="top-nav">
    <div class="top-nav-title">
        <?= $this->Html->link('CMS Tutorial', ['controller' => 'Articles', 'action' => 'index']) ?>
    </div>

    <div class="top-nav-links">
        <?php
        $identity = $this->request->getAttribute('identity');
        if ($identity):
        ?>
            <span>👤 <?= h($identity->email) ?></span>
            <?= $this->Html->link('Meus Artigos', ['controller' => 'Articles', 'action' => 'index']) ?>
            <?= $this->Html->link('Novo Artigo', ['controller' => 'Articles', 'action' => 'add']) ?>
            <?= $this->Html->link('Sair', ['controller' => 'Users', 'action' => 'logout']) ?>
        <?php else: ?>
            <?= $this->Html->link('Login', ['controller' => 'Users', 'action' => 'login']) ?>
            <?= $this->Html->link('Registrar', ['controller' => 'Users', 'action' => 'add']) ?>
        <?php endif; ?>
    </div>
</nav>

<?= $this->Flash->render() ?>
<?= $this->fetch('content') ?>
```

---

### Usar user_id da Sessão em add()

Atualizar `ArticlesController::add()`:

```php
public function add()
{
    $article = $this->Articles->newEmptyEntity();

    if ($this->request->is('post')) {
        $article = $this->Articles->patchEntity($article, $this->request->getData());

        // ← SUBSTITUIR hardcode por sessão:
        $article->user_id = $this->Authentication->getIdentity()->getIdentifier();

        if ($this->Articles->save($article)) {
            $this->Flash->success(__('Artigo salvo com sucesso!'));
            return $this->redirect(['action' => 'view', $article->slug]);
        }
        $this->Flash->error(__('Não foi possível salvar o artigo.'));
    }

    $tags = $this->Articles->Tags->find('list')->all();
    $this->set(compact('article', 'tags'));
}
```

---

## 🛡️ Rotas Públicas vs Privadas

### Configuração Atual

| Página | Requer Login? |
|--------|---------------|
| `/articles` | ❌ Público (index) |
| `/articles/view/{slug}` | ❌ Público (view) |
| `/articles/add` | ✅ **Requer login** |
| `/articles/edit/{slug}` | ✅ **Requer login** |
| `/articles/delete/{slug}` | ✅ **Requer login** |
| `/users/login` | ❌ Público |
| `/users/add` | ❌ Público (registro) |
| `/users/logout` | ✅ Requer login |

---

## ❓ Problemas Comuns

??? danger "Erro: Infinite redirect loop"
    **Causa:** `addUnauthenticatedActions(['login'])` não configurado.

    **Solução:** Ver `UsersController::beforeFilter()` acima.

---

??? danger "Erro: session.cookie_secure"
    **Causa:** App em HTTP mas sessão configurada para HTTPS.

    **Solução (`config/app_local.php`):**
    ```php
    'Session' => [
        'defaults' => 'php',
        'cookie_secure' => false,  // ← Adicionar
    ],
    ```

---

??? warning "Senha não hasheada"
    **Causa:** Esqueceu `_setPassword()` na Entity.

    **Solução:** Ver seção "Hash de Senhas" acima.

---

## 🎓 Conceitos Avançados

??? abstract "Como funciona o Middleware?"
    **Fluxo de requisição:**

    ```
    Request → AuthenticationMiddleware
             ↓
             Verifica Session (logado?)
             ↓
             Se não, verifica Form POST
             ↓
             Injeta identity no request
             ↓
             → Controller → View → Response
    ```

---

??? abstract "Diferença Session vs Form Authenticator"
    | Authenticator | Quando executa | O que faz |
    |---------------|----------------|-----------|
    | **Session** | Toda requisição | Verifica se usuário está na sessão |
    | **Form** | POST em /users/login | Verifica email/senha do formulário |

---

## 🚀 Próximos Passos

✅ **Autenticação completa!** Próximo: **Autorização** (quem pode editar/deletar o quê)

👉 **[Implementar Autorização](cms-authorization.md)** - Controle de acesso por usuário

---

## 📚 Referências

- 🔐 [Authentication Plugin](https://book.cakephp.org/authentication/3/)
- 🔑 [Password Hashing](https://book.cakephp.org/5/en/core-libraries/security.html)
- 🛡️ [Middleware](https://book.cakephp.org/5/en/controllers/middleware.html)
- 🍪 [Sessions](https://book.cakephp.org/5/en/development/sessions.html)

---

**✨ Criado com [Claude Code](https://claude.com/claude-code)**
