# Convenções do CakePHP

!!! quote "Filosofia: Convenção sobre Configuração"
    Somos grandes fãs de **Convention over Configuration** (Convenção sobre Configuração). Embora leve um tempo para aprender as convenções do CakePHP, você economiza tempo a longo prazo. Ao seguir as convenções, você ganha funcionalidades gratuitamente e se liberta do pesadelo de manter arquivos de configuração. As convenções também proporcionam uma experiência de desenvolvimento uniforme, permitindo que outros desenvolvedores entrem no projeto e ajudem facilmente.

---

## 📋 Índice Rápido

<div class="grid cards" markdown>

-   :material-controller: **[Convenções de Controllers](#convencoes-de-controllers)**

    Nomenclatura de classes, métodos e URLs

-   :material-file-code: **[Convenções de Arquivos](#convencoes-de-arquivos-e-classes)**

    PSR-4, nomes de classes e estrutura

-   :material-database: **[Convenções de Banco de Dados](#convencoes-de-banco-de-dados)**

    Tabelas, colunas, foreign keys, join tables

-   :material-cube: **[Convenções de Models](#convencoes-de-models)**

    Table classes, Entity classes, Enums

-   :material-eye: **[Convenções de Views](#convencoes-de-views)**

    Templates, elements, layouts

-   :material-puzzle: **[Convenções de Plugins](#convencoes-de-plugins)**

    Pacotes Composer e namespaces

-   :material-chart-box: **[Tabela Resumo](#tabela-de-referencia-rapida)**

    Referência rápida de todas as convenções

</div>

---

## 🎯 Por Que Seguir Convenções?

### Benefícios Diretos

=== "Desenvolvimento"
    - ✅ **Funcionalidade automática** - Routing, carregamento de models, views funcionam "magicamente"
    - ✅ **Menos configuração** - Não precisa mapear manualmente cada rota ou relacionamento
    - ✅ **Código padronizado** - Fácil de ler e manter

=== "Manutenção"
    - ✅ **Onboarding rápido** - Novos desenvolvedores entendem a estrutura rapidamente
    - ✅ **Bake funciona** - Ferramentas de geração de código só funcionam se seguir convenções
    - ✅ **Upgrades fáceis** - Migrações entre versões são mais simples

=== "Colaboração"
    - ✅ **Padrão da comunidade** - Outros desenvolvedores CakePHP entendem seu código
    - ✅ **Plugins compatíveis** - Plugins de terceiros funcionam perfeitamente
    - ✅ **Documentação aplicável** - Exemplos da documentação funcionam diretamente

---

## 🎮 Convenções de Controllers

### Nomenclatura de Classes

**Regra:** Plural + CamelCase + sufixo `Controller`

```php
// ✅ CORRETO
class UsersController extends AppController { }
class MenuLinksController extends AppController { }
class ArticlesController extends AppController { }

// ❌ INCORRETO
class UserController { }          // Singular
class usersController { }          // Não é CamelCase
class Users { }                    // Falta sufixo Controller
```

!!! tip "Tratamento de Acrônimos"
    Trate acrônimos como palavras normais:

    ```php
    ✅ CmsController    (não CMSController)
    ✅ ApiController    (não APIController)
    ✅ PdfController    (não PDFController)
    ✅ HtmlHelper       (não HTMLHelper)
    ```

---

### Métodos (Actions)

**Regra:** `camelBack` (primeira letra minúscula)

```php
class ArticlesController extends AppController
{
    // ✅ Métodos públicos (acessíveis via URL)
    public function index() { }
    public function viewAll() { }          // /articles/view-all
    public function addNewArticle() { }    // /articles/add-new-article

    // ❌ Métodos protegidos/privados (NÃO acessíveis via URL)
    protected function _calculateTotal() { }
    private function _sendEmail() { }
}
```

!!! warning "Visibilidade de Métodos"
    **Apenas métodos `public`** podem ser acessados via routing. Use `protected` ou `private` para métodos auxiliares que não devem ser expostos via URLs.

---

### Mapeamento de URLs

**Padrão (com DashedRoute):** lowercase-com-hífens

```php
// routes.php
use Cake\Routing\Route\DashedRoute;

$routes->setRouteClass(DashedRoute::class);
```

#### Exemplos de Mapeamento

| URL | Controller | Método |
|-----|------------|--------|
| `/users` | `UsersController` | `index()` |
| `/users/view-me` | `UsersController` | `viewMe()` |
| `/menu-links/view-all` | `MenuLinksController` | `viewAll()` |
| `/articles/add` | `ArticlesController` | `add()` |

---

### Criando Links

**Padrão para arrays de URL:**

```php
<?php
// Em uma view ou controller
echo $this->Html->link('Ver Artigo', [
    'prefix' => 'Admin',           // CamelCased
    'plugin' => 'Blog',             // CamelCased
    'controller' => 'Articles',     // CamelCased (plural)
    'action' => 'view',             // camelBacked
    123                             // ID do artigo
]);
```

**Resultado:** `/admin/blog/articles/view/123`

!!! info "Mais sobre Routing"
    Para detalhes completos sobre URLs e rotas, consulte [Configuração de Rotas](../development/routing.md).

---

## 📁 Convenções de Arquivos e Classes

### Padrão Geral: PSR-4

CakePHP segue o padrão **PSR-4** para autoloading. Isso significa:

- **Nome do arquivo = Nome da classe** (case-sensitive)
- **Namespace → Caminho do diretório** (direto)

#### Configuração PSR-4 (composer.json)

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

---

### Mapeamento de Arquivos

| Tipo | Classe | Arquivo | Localização |
|------|--------|---------|-------------|
| **Controller** | `LatestArticlesController` | `LatestArticlesController.php` | `src/Controller/` |
| **Component** | `MyHandyComponent` | `MyHandyComponent.php` | `src/Controller/Component/` |
| **Table** | `OptionValuesTable` | `OptionValuesTable.php` | `src/Model/Table/` |
| **Entity** | `OptionValue` | `OptionValue.php` | `src/Model/Entity/` |
| **Behavior** | `SluggableBehavior` | `SluggableBehavior.php` | `src/Model/Behavior/` |
| **View** | `CustomJsonView` | `CustomJsonView.php` | `src/View/` |
| **Helper** | `CurrencyHelper` | `CurrencyHelper.php` | `src/View/Helper/` |

---

### Exemplo Completo de Namespace

```php
<?php
declare(strict_types=1);

// Arquivo: src/Controller/Component/PaginationComponent.php
namespace App\Controller\Component;

use Cake\Controller\Component;

class PaginationComponent extends Component
{
    // ...
}
```

**Namespace `App\Controller\Component` → diretório `src/Controller/Component/`**

!!! warning "CakePHP 5 Requirement"
    Todos os arquivos PHP **devem** começar com `declare(strict_types=1);` no CakePHP 5.

---

## 🗄️ Convenções de Banco de Dados

### Nomes de Tabelas

**Regra:** Plural + underscore (snake_case)

```sql
-- ✅ CORRETO
CREATE TABLE users (...);
CREATE TABLE menu_links (...);              -- Apenas última palavra pluralizada
CREATE TABLE user_favorite_pages (...);     -- Apenas última palavra pluralizada

-- ❌ INCORRETO
CREATE TABLE user (...);                    -- Singular
CREATE TABLE MenuLinks (...);               -- CamelCase
CREATE TABLE menus_links (...);             -- Primeira palavra pluralizada
```

!!! info "Pluralização"
    **Apenas a última palavra** é pluralizada em nomes compostos:

    - `MenuLink` → `menu_links` (não `menus_links`)
    - `UserFavoritePage` → `user_favorite_pages` (não `users_favorites_pages`)

---

### Nomes de Colunas

**Regra:** snake_case (underscore)

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    first_name VARCHAR(100),        -- ✅ Underscore
    last_name VARCHAR(100),          -- ✅ Underscore
    email_address VARCHAR(255),      -- ✅ Underscore
    created DATETIME,
    modified DATETIME
);
```

---

### Foreign Keys

**Regra:** `{singular_table_name}_id`

#### Exemplo: Users hasMany Articles

```sql
-- Tabela users
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

-- Tabela articles
CREATE TABLE articles (
    id INT PRIMARY KEY,
    user_id INT,                     -- ✅ Singular: user_id (não users_id)
    title VARCHAR(200),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

#### Exemplo: Nomes Compostos

```sql
-- Tabela menu_links
CREATE TABLE menu_links (
    id INT PRIMARY KEY,
    title VARCHAR(100)
);

-- Tabela menu_items
CREATE TABLE menu_items (
    id INT PRIMARY KEY,
    menu_link_id INT,                -- ✅ menu_link_id (singular)
    FOREIGN KEY (menu_link_id) REFERENCES menu_links(id)
);
```

---

### Join Tables (BelongsToMany)

**Regras:**
1. ✅ Nomes **pluralizados**
2. ✅ **Ordem alfabética**
3. ✅ Padrão: `{table1}_{table2}`

```sql
-- ✅ CORRETO
CREATE TABLE articles_tags (           -- Ordem alfabética: articles < tags
    article_id INT,
    tag_id INT,
    PRIMARY KEY (article_id, tag_id)
);

-- ❌ INCORRETO
CREATE TABLE tags_articles (...);      -- Ordem errada
CREATE TABLE article_tags (...);       -- Não pluralizada
```

!!! danger "Bake Quebra se Não Seguir"
    O comando `bin/cake bake` **não funcionará** se join tables não seguirem a convenção de ordem alfabética!

---

#### Join Tables com Dados Extras

Se a join table precisa armazenar **dados além das foreign keys**, crie uma **entity/table class concreta**:

```sql
-- articles_tags com campo priority
CREATE TABLE articles_tags (
    id INT PRIMARY KEY,
    article_id INT,
    tag_id INT,
    priority INT DEFAULT 0,          -- Campo extra!
    created DATETIME
);
```

```php
// src/Model/Table/ArticlesTagsTable.php
class ArticlesTagsTable extends Table
{
    public function initialize(array $config): void
    {
        $this->belongsTo('Articles');
        $this->belongsTo('Tags');
    }
}

// src/Model/Entity/ArticlesTag.php
class ArticlesTag extends Entity
{
    protected array $_accessible = [
        'article_id' => true,
        'tag_id' => true,
        'priority' => true,
    ];
}
```

---

### Primary Keys

CakePHP suporta dois tipos de chaves primárias:

=== "Auto-increment (Padrão)"
    ```sql
    CREATE TABLE users (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(100)
    );
    ```

    - ✅ Padrão do CakePHP
    - ✅ Mais simples para começar
    - ✅ Melhor performance em bancos relacionais

=== "UUID"
    ```sql
    CREATE TABLE users (
        id CHAR(36) PRIMARY KEY,
        name VARCHAR(100)
    );
    ```

    ```php
    // src/Model/Table/UsersTable.php
    class UsersTable extends Table
    {
        public function initialize(array $config): void
        {
            $this->setPrimaryKey('id');
        }

        public function beforeMarshal(EventInterface $event, ArrayObject $data, ArrayObject $options)
        {
            if (!isset($data['id'])) {
                $data['id'] = Text::uuid();  // Gera UUID automaticamente
            }
        }
    }
    ```

    - ✅ Ideal para sistemas distribuídos
    - ✅ IDs não sequenciais (mais seguro)
    - ⚠️ Maior uso de espaço (36 bytes vs 4 bytes)

---

## 🧩 Convenções de Models

### Table Classes

**Regra:** Plural + CamelCase + sufixo `Table`

```php
// Tabela: users
// src/Model/Table/UsersTable.php
namespace App\Model\Table;

use Cake\ORM\Table;

class UsersTable extends Table
{
    public function initialize(array $config): void
    {
        $this->setTable('users');          // Opcional (inferido automaticamente)
        $this->setDisplayField('name');
        $this->setPrimaryKey('id');
    }
}
```

#### Exemplos de Mapeamento

| Tabela DB | Table Class | Arquivo |
|-----------|-------------|---------|
| `users` | `UsersTable` | `src/Model/Table/UsersTable.php` |
| `menu_links` | `MenuLinksTable` | `src/Model/Table/MenuLinksTable.php` |
| `user_favorite_pages` | `UserFavoritePagesTable` | `src/Model/Table/UserFavoritePagesTable.php` |

---

### Entity Classes

**Regra:** Singular + CamelCase + **SEM** sufixo

```php
// Tabela: users
// src/Model/Entity/User.php
namespace App\Model\Entity;

use Cake\ORM\Entity;

class User extends Entity
{
    protected array $_accessible = [
        'name' => true,
        'email' => true,
        'password' => true,
    ];

    protected array $_hidden = [
        'password',
    ];
}
```

#### Exemplos de Mapeamento

| Tabela DB | Entity Class | Arquivo |
|-----------|--------------|---------|
| `users` | `User` | `src/Model/Entity/User.php` |
| `menu_links` | `MenuLink` | `src/Model/Entity/MenuLink.php` |
| `user_favorite_pages` | `UserFavoritePage` | `src/Model/Entity/UserFavoritePage.php` |

---

### Enum Classes (CakePHP 5+)

**Regra:** `{Entity}{Column}` + casos em CamelCase

```php
// Tabela: users (coluna: status)
// src/Model/Enum/UserStatus.php
namespace App\Model\Enum;

enum UserStatus: string
{
    case Active = 'active';          // CamelCase
    case Inactive = 'inactive';      // CamelCase
    case Suspended = 'suspended';    // CamelCase
}
```

```php
// Uso na Entity
// src/Model/Entity/User.php
class User extends Entity
{
    protected function _getStatus(): UserStatus
    {
        return UserStatus::from($this->_fields['status']);
    }
}
```

---

## 👁️ Convenções de Views

### Estrutura de Templates

**Regra:** `templates/{Controller}/{underscored_action}.php`

```
templates/
├── Articles/
│   ├── index.php              ← ArticlesController::index()
│   ├── view.php               ← ArticlesController::view()
│   ├── add.php                ← ArticlesController::add()
│   ├── edit.php               ← ArticlesController::edit()
│   └── view_all.php           ← ArticlesController::viewAll()
├── Users/
│   ├── login.php              ← UsersController::login()
│   └── profile.php            ← UsersController::profile()
└── MenuLinks/
    ├── index.php              ← MenuLinksController::index()
    └── get_list.php           ← MenuLinksController::getList()
```

---

### Elements (Reutilizáveis)

**Localização:** `templates/element/`

```
templates/element/
├── header.php
├── footer.php
├── sidebar.php
└── flash/
    ├── success.php
    ├── error.php
    └── warning.php
```

**Uso:**
```php
<?= $this->element('header') ?>
<?= $this->element('flash/success') ?>
```

---

### Layouts

**Localização:** `templates/layout/`

```
templates/layout/
├── default.php                 ← Layout padrão
├── ajax.php                    ← Para requisições AJAX
├── error.php                   ← Para páginas de erro
└── email/
    ├── html/default.php        ← Email HTML
    └── text/default.php        ← Email texto
```

---

## 🧩 Convenções de Plugins

### Nomenclatura de Pacotes Composer

**Regra:** `{vendor}/cakephp-{plugin-name}`

```json
// ✅ CORRETO
{
    "name": "sua-empresa/cakephp-payment-gateway",
    "type": "cakephp-plugin"
}

{
    "name": "joao-silva/cakephp-correios-integration",
    "type": "cakephp-plugin"
}

// ❌ INCORRETO
{
    "name": "cakephp/payment-gateway",     // Namespace "cakephp" é reservado
    "type": "cakephp-plugin"
}

{
    "name": "sua-empresa/PaymentGateway",  // Deve ser lowercase-com-hífens
    "type": "cakephp-plugin"
}
```

!!! danger "Namespace Reservado"
    **NÃO use** `cakephp` como vendor name. Este namespace é **reservado** para plugins oficiais do CakePHP.

---

### Estrutura de Plugin

```
vendor/sua-empresa/cakephp-blog/
├── composer.json
├── src/
│   ├── Plugin.php                      ← Classe principal do plugin
│   ├── Controller/
│   │   └── ArticlesController.php
│   ├── Model/
│   │   ├── Table/ArticlesTable.php
│   │   └── Entity/Article.php
│   └── Template/
│       └── Articles/
│           └── index.php
└── config/
    └── routes.php
```

---

## 📊 Tabela de Referência Rápida

### Exemplo Completo: Sistema de Blog

Veja como todas as convenções se conectam:

| Componente | Exemplo 1 (`articles`) | Exemplo 2 (`menu_links`) | Convenção |
|------------|------------------------|--------------------------|-----------|
| **Tabela DB** | `articles` | `menu_links` | Plural, underscored |
| **Colunas** | `id`, `user_id`, `title`, `body`, `created` | `id`, `parent_id`, `title`, `url` | snake_case |
| **Foreign Key** | `user_id` → `users.id` | `parent_id` → `menu_links.id` | `{singular}_id` |
| **Table Class** | `ArticlesTable` | `MenuLinksTable` | Plural + Table |
| **Arquivo Table** | `src/Model/Table/ArticlesTable.php` | `src/Model/Table/MenuLinksTable.php` | PSR-4 |
| **Entity Class** | `Article` | `MenuLink` | Singular, sem sufixo |
| **Arquivo Entity** | `src/Model/Entity/Article.php` | `src/Model/Entity/MenuLink.php` | PSR-4 |
| **Controller** | `ArticlesController` | `MenuLinksController` | Plural + Controller |
| **Arquivo Controller** | `src/Controller/ArticlesController.php` | `src/Controller/MenuLinksController.php` | PSR-4 |
| **Action** | `viewAll()` | `getList()` | camelBacked |
| **URL** | `/articles/view-all` | `/menu-links/get-list` | lowercase-dashed |
| **Template** | `templates/Articles/view_all.php` | `templates/MenuLinks/get_list.php` | underscored |
| **Helper** | `ArticlesHelper` | `MenuLinksHelper` | Plural + Helper |
| **Component** | `ArticlesComponent` | `MenuLinksComponent` | Plural + Component |
| **Behavior** | `ArticlesBehavior` | `MenuLinksBehavior` | Plural + Behavior |

---

### Relacionamentos BelongsToMany

**Exemplo:** Articles belongsToMany Tags

```sql
-- Tabelas principais
CREATE TABLE articles (id INT PRIMARY KEY, title VARCHAR(200));
CREATE TABLE tags (id INT PRIMARY KEY, name VARCHAR(50));

-- Join table (ORDEM ALFABÉTICA!)
CREATE TABLE articles_tags (
    article_id INT,
    tag_id INT,
    PRIMARY KEY (article_id, tag_id)
);
```

```php
// src/Model/Table/ArticlesTable.php
class ArticlesTable extends Table
{
    public function initialize(array $config): void
    {
        $this->belongsToMany('Tags', [
            'joinTable' => 'articles_tags',  // Inferido automaticamente!
        ]);
    }
}
```

**CakePHP infere automaticamente:**
- ✅ Join table: `articles_tags` (ordem alfabética)
- ✅ Foreign keys: `article_id` e `tag_id`
- ✅ Associação inversa em `TagsTable`

---

## 🌐 Inflections: Adaptando para Português

Por padrão, CakePHP usa **inflections em inglês**. Para tabelas em português:

### Configurar Inflector Customizado

```php
// config/bootstrap.php
use Cake\Utility\Inflector;

// Regras de pluralização em português
Inflector::rules('plural', [
    'rules' => [
        '/ão$/i' => 'ões',              // Opinião → Opiniões
        '/m$/i' => 'ns',                 // Pedido → Pedidos
    ],
    'irregular' => [
        'país' => 'países',
        'cônjuge' => 'cônjuges',
    ],
    'uninflected' => [
        'status',
        'bonus',
    ],
]);

// Regras de singularização
Inflector::rules('singular', [
    'rules' => [
        '/ões$/i' => 'ão',
        '/ns$/i' => 'm',
    ],
    'irregular' => [
        'países' => 'país',
        'cônjuges' => 'cônjuge',
    ],
]);
```

---

### Exemplo Prático: Tabelas em Português

=== "Com Inflector Customizado"
    ```php
    // Tabela: opinioes
    class OpinioesTable extends Table { }    // ✅ Plural correto
    class Opiniao extends Entity { }          // ✅ Singular correto
    class OpinioesController extends AppController { }

    // URL: /opinioes
    // Template: templates/Opinioes/index.php
    ```

=== "Sem Inflector (Manual)"
    ```php
    // Tabela: opinioes (DB)
    class OpinioesTable extends Table
    {
        public function initialize(array $config): void
        {
            $this->setTable('opinioes');     // Explícito
        }
    }

    class Opiniao extends Entity { }
    ```

!!! info "Mais sobre Inflector"
    Para detalhes completos, consulte [Documentação do Inflector](../core-libraries/inflector.md).

---

## ✅ Checklist de Verificação

Use esta checklist para garantir que está seguindo todas as convenções:

### Controllers
- [ ] Nome da classe é **plural**? (`UsersController`)
- [ ] Classe termina com **"Controller"**?
- [ ] Arquivo está em `src/Controller/`?
- [ ] Actions são **public** e **camelBacked**? (`viewAll()`)
- [ ] Métodos auxiliares são **protected/private**?
- [ ] `declare(strict_types=1)` no início do arquivo?

### Models
- [ ] **Table class** é plural + "Table"? (`UsersTable`)
- [ ] **Entity class** é singular sem sufixo? (`User`)
- [ ] Arquivos em `src/Model/Table/` e `src/Model/Entity/`?
- [ ] Enums seguem padrão `{Entity}{Column}`? (`UserStatus`)

### Database
- [ ] Tabelas são **plural** e **underscored**? (`menu_links`)
- [ ] Colunas são **snake_case**? (`first_name`)
- [ ] Foreign keys seguem `{singular}_id`? (`user_id`)
- [ ] Join tables em **ordem alfabética**? (`articles_tags`)
- [ ] Join tables são **pluralizadas**?

### Views
- [ ] Templates em `templates/{Controller}/`?
- [ ] Nomes de arquivos são **underscored**? (`view_all.php`)
- [ ] Elements em `templates/element/`?
- [ ] Layouts em `templates/layout/`?

### Routing
- [ ] `DashedRoute::class` configurado em routes.php?
- [ ] URLs são **lowercase-dashed**? (`/menu-links/view-all`)

### Plugins (se aplicável)
- [ ] Package name: `{vendor}/cakephp-{plugin}`?
- [ ] Vendor name **não é** "cakephp"?
- [ ] Tudo em **lowercase com hífens**?

---

## 🧪 Testando Suas Convenções

### Verificar Routing

```bash
# Listar todas as rotas configuradas
bin/cake routes

# Deve mostrar:
# /articles          → ArticlesController::index()
# /articles/view/:id → ArticlesController::view()
```

---

### Verificar Autoloading (PSR-4)

```bash
# Testar se classes são encontradas
bin/cake console

# No console:
>>> use App\Model\Table\ArticlesTable;
>>> $table = new ArticlesTable();
>>> echo get_class($table);
# Deve retornar: App\Model\Table\ArticlesTable
```

---

### Usar Bake para Validar

```bash
# Bake Model (só funciona se DB seguir convenções!)
bin/cake bake model Articles

# Bake Controller
bin/cake bake controller Articles

# Bake Template
bin/cake bake template Articles

# Se tudo funcionar sem erros, suas convenções estão corretas! ✅
```

---

## 🎓 Exemplo Completo de Aplicação

Vamos criar um sistema de **blog** seguindo **todas** as convenções:

### 1. Estrutura de Banco de Dados

```sql
-- Tabela de usuários
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    password VARCHAR(255),
    name VARCHAR(100),
    created DATETIME,
    modified DATETIME
);

-- Tabela de artigos
CREATE TABLE articles (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,                        -- FK para users
    title VARCHAR(200),
    slug VARCHAR(200) UNIQUE,
    body TEXT,
    published BOOLEAN DEFAULT FALSE,
    created DATETIME,
    modified DATETIME,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- Tabela de tags
CREATE TABLE tags (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) UNIQUE,
    created DATETIME
);

-- Join table (ordem alfabética!)
CREATE TABLE articles_tags (
    article_id INT,
    tag_id INT,
    PRIMARY KEY (article_id, tag_id),
    FOREIGN KEY (article_id) REFERENCES articles(id) ON DELETE CASCADE,
    FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE
);
```

---

### 2. Models

#### Table Classes

```php
// src/Model/Table/UsersTable.php
namespace App\Model\Table;

use Cake\ORM\Table;

class UsersTable extends Table
{
    public function initialize(array $config): void
    {
        $this->setDisplayField('name');
        $this->hasMany('Articles');         // Inferido: foreign_key = user_id
    }
}
```

```php
// src/Model/Table/ArticlesTable.php
namespace App\Model\Table;

use Cake\ORM\Table;

class ArticlesTable extends Table
{
    public function initialize(array $config): void
    {
        $this->belongsTo('Users');          // Inferido: foreign_key = user_id
        $this->belongsToMany('Tags');       // Inferido: join_table = articles_tags

        $this->addBehavior('Timestamp');    // Gerencia created/modified
    }
}
```

```php
// src/Model/Table/TagsTable.php
namespace App\Model\Table;

use Cake\ORM\Table;

class TagsTable extends Table
{
    public function initialize(array $config): void
    {
        $this->belongsToMany('Articles');   // Inferido automaticamente
    }
}
```

---

#### Entity Classes

```php
// src/Model/Entity/User.php
namespace App\Model\Entity;

use Cake\ORM\Entity;

class User extends Entity
{
    protected array $_accessible = [
        'email' => true,
        'password' => true,
        'name' => true,
        'articles' => true,
    ];

    protected array $_hidden = [
        'password',                         // Nunca expor em JSON
    ];
}
```

```php
// src/Model/Entity/Article.php
namespace App\Model\Entity;

use Cake\ORM\Entity;

class Article extends Entity
{
    protected array $_accessible = [
        'user_id' => true,
        'title' => true,
        'slug' => true,
        'body' => true,
        'published' => true,
        'tags' => true,
    ];
}
```

```php
// src/Model/Entity/Tag.php
namespace App\Model\Entity;

use Cake\ORM\Entity;

class Tag extends Entity
{
    protected array $_accessible = [
        'name' => true,
        'articles' => true,
    ];
}
```

---

### 3. Controllers

```php
// src/Controller/ArticlesController.php
namespace App\Controller;

class ArticlesController extends AppController
{
    // URL: /articles
    public function index()
    {
        $articles = $this->Articles->find()
            ->contain(['Users', 'Tags'])
            ->where(['published' => true])
            ->orderBy(['created' => 'DESC']);

        $this->set(compact('articles'));
    }

    // URL: /articles/view/123
    public function view($id)
    {
        $article = $this->Articles->get($id, contain: ['Users', 'Tags']);
        $this->set(compact('article'));
    }

    // URL: /articles/add
    public function add()
    {
        $article = $this->Articles->newEmptyEntity();

        if ($this->request->is('post')) {
            $article = $this->Articles->patchEntity($article, $this->request->getData());
            if ($this->Articles->save($article)) {
                $this->Flash->success('Artigo salvo!');
                return $this->redirect(['action' => 'index']);
            }
            $this->Flash->error('Erro ao salvar.');
        }

        $users = $this->Articles->Users->find('list');
        $tags = $this->Articles->Tags->find('list');
        $this->set(compact('article', 'users', 'tags'));
    }
}
```

---

### 4. Views

```php
<!-- templates/Articles/index.php -->
<h1>Blog Posts</h1>

<?php foreach ($articles as $article): ?>
    <article>
        <h2><?= $this->Html->link($article->title, ['action' => 'view', $article->id]) ?></h2>
        <p>Por: <?= h($article->user->name) ?></p>
        <p>
            Tags:
            <?php foreach ($article->tags as $tag): ?>
                <span class="badge"><?= h($tag->name) ?></span>
            <?php endforeach; ?>
        </p>
        <p><?= $this->Text->truncate($article->body, 200) ?></p>
    </article>
<?php endforeach; ?>
```

```php
<!-- templates/Articles/view.php -->
<article>
    <h1><?= h($article->title) ?></h1>
    <p class="meta">
        Por: <?= h($article->user->name) ?>
        em <?= $article->created->format('d/m/Y') ?>
    </p>

    <div class="tags">
        <?php foreach ($article->tags as $tag): ?>
            <span class="badge"><?= h($tag->name) ?></span>
        <?php endforeach; ?>
    </div>

    <div class="body">
        <?= $this->Text->autoParagraph(h($article->body)) ?>
    </div>
</article>
```

---

### 5. Routing

```php
// config/routes.php
use Cake\Routing\Route\DashedRoute;
use Cake\Routing\RouteBuilder;

return function (RouteBuilder $routes): void {
    $routes->setRouteClass(DashedRoute::class);

    $routes->scope('/', function (RouteBuilder $builder): void {
        // Rotas automáticas (seguindo convenções)
        $builder->connect('/articles', ['controller' => 'Articles', 'action' => 'index']);
        $builder->connect('/articles/view/:id', ['controller' => 'Articles', 'action' => 'view'])
            ->setPass(['id']);

        // Fallbacks (desenvolvimento)
        $builder->fallbacks();
    });
};
```

---

### Resultado: Funcionalidade "Gratuita"

Seguindo as convenções, você ganhou **automaticamente**:

✅ **Relacionamentos funcionam sem configuração:**
```php
$article->user->name          // Funciona!
$article->tags                // Funciona!
$user->articles               // Funciona!
```

✅ **URLs mapeiam automaticamente:**
```
/articles              → ArticlesController::index()
/articles/view/5       → ArticlesController::view(5)
/articles/add          → ArticlesController::add()
```

✅ **Bake gera código correto:**
```bash
bin/cake bake model Articles  # Gera Table, Entity, Tests
bin/cake bake controller Articles  # Gera Controller com CRUD
bin/cake bake template Articles  # Gera todas as views
```

✅ **Plugins funcionam sem configuração:**
```bash
composer require friendsofcake/search
# Plugin entende automaticamente sua estrutura!
```

---

## 🚀 Próximos Passos

Agora que você entende as convenções do CakePHP, experimente:

1. **[Tutorial CMS](../tutorials-and-examples/cms/installation.md)** - Construa um CMS completo seguindo convenções
2. **[ORM do CakePHP](../orm.md)** - Aprenda sobre relacionamentos e queries
3. **[Controllers](../controllers.md)** - Aprofunde em controllers e components
4. **[Views](../views.md)** - Domine templates, helpers e elements

---

## 📚 Recursos Adicionais

- **[Application Skeleton](https://github.com/cakephp/app)** - Código de referência com todas as convenções
- **[Documentação do Inflector](../core-libraries/inflector.md)** - Personalizar inflections
- **[PSR-4 Autoloading](https://www.php-fig.org/psr/psr-4/)** - Padrão PHP de autoloading
- **[FriendsOfCake](https://github.com/FriendsOfCake/awesome-cakephp)** - Convenções de plugins da comunidade

---

**Última atualização:** 2025-11-15 | **CakePHP:** 5.2+
