# Pagination (Paginação)

!!! info "Listagens Paginadas Eficientes"
    A paginação no CakePHP 5 permite dividir grandes conjuntos de dados em páginas menores e navegáveis, melhorando a performance e a experiência do usuário. O framework oferece o método `paginate()` nos controllers e o `PaginatorHelper` nas views.

---

## 📋 Conteúdo

- **Básico** - Configuração simples de paginação
- **Configuração Avançada** - Opções detalhadas do `$paginate`
- **Múltiplos Models** - Paginar várias tabelas na mesma página
- **Ordenação Customizada** - Sorting dinâmico
- **PaginatorHelper** - Links de navegação nas views
- **SimplePaginator** - Paginação sem contagem total
- **Tratamento de Erros** - Páginas fora do range

---

## 🎯 Paginação Básica

### Uso Simples

```php
<?php
namespace App\Controller;

class ArticlesController extends AppController
{
    public function index()
    {
        // Paginação automática com configurações padrão
        $articles = $this->paginate($this->Articles);
        $this->set(compact('articles'));
    }
}
```

**Configurações Padrão:**
- **Limite:** 20 registros por página
- **Limite Máximo:** 100 registros (proteção contra sobrecarga)
- **Ordenação:** Por chave primária

---

## ⚙️ Configuração Avançada

### Propriedade `$paginate`

```php
<?php
namespace App\Controller;

class ArticlesController extends AppController
{
    protected array $paginate = [
        // Registros por página
        'limit' => 25,

        // Número máximo de registros por requisição
        'maxLimit' => 100,

        // Ordenação padrão (DEVE ser array)
        'order' => [
            'Articles.created' => 'DESC'
        ],

        // Relacionamentos a carregar
        'contain' => ['Users', 'Categories'],

        // Campos permitidos para ordenação (whitelist)
        'sortableFields' => [
            'Articles.id',
            'Articles.title',
            'Articles.created',
            'Users.name'
        ],

        // Finder personalizado
        'finder' => 'published',
    ];

    public function index()
    {
        $articles = $this->paginate($this->Articles);
        $this->set(compact('articles'));
    }
}
```

### Configuração por Action

```php
public function index()
{
    // Sobrescrever configurações para esta action específica
    $query = $this->Articles->find('published')
        ->where(['Articles.status' => 'active']);

    $settings = [
        'limit' => 10,
        'order' => ['Articles.view_count' => 'DESC'],
    ];

    $articles = $this->paginate($query, $settings);
    $this->set(compact('articles'));
}
```

---

## 🔍 Finder Personalizado

### Definir Finder no Model

```php
<?php
// src/Model/Table/ArticlesTable.php
namespace App\Model\Table;

use Cake\ORM\Table;

class ArticlesTable extends Table
{
    public function findPublished($query, array $options = [])
    {
        return $query->where([
            'Articles.status' => 'published',
            'Articles.published_date <=' => date('Y-m-d')
        ]);
    }

    public function findTagged($query, array $options = [])
    {
        return $query
            ->matching('Tags', function ($q) use ($options) {
                return $q->where(['Tags.name IN' => $options['tags']]);
            });
    }
}
```

### Usar Finder no Controller

```php
// Finder simples
protected array $paginate = [
    'finder' => 'published',
];

// Finder com parâmetros
public function tagged()
{
    $tags = $this->request->getQuery('tags');

    $this->paginate = [
        'finder' => [
            'tagged' => ['tags' => explode(',', $tags)]
        ]
    ];

    $articles = $this->paginate($this->Articles);
    $this->set(compact('articles'));
}
```

---

## 📊 Paginando Múltiplos Models

### Usar `scope` para Diferenciar

```php
<?php
namespace App\Controller;

class DashboardController extends AppController
{
    public function index()
    {
        // Paginar Artigos
        $this->paginate = [
            'Articles' => [
                'scope' => 'article',  // Query string: ?article[page]=1
                'limit' => 10,
            ]
        ];
        $articles = $this->paginate($this->Articles);

        // Paginar Tags
        $this->paginate = [
            'Tags' => [
                'scope' => 'tag',  // Query string: ?tag[page]=2
                'limit' => 15,
            ]
        ];
        $tags = $this->paginate($this->Tags);

        $this->set(compact('articles', 'tags'));
    }
}
```

**URL Resultante:** `/dashboard?article[page]=1&article[sort]=title&tag[page]=3`

---

## 🔢 SimplePaginator (Sem Contagem Total)

Para grandes datasets, usar `SimplePaginator` evita queries COUNT custosas:

```php
use Cake\Datasource\Paging\SimplePaginator;

protected array $paginate = [
    'className' => SimplePaginator::class,  // Sem COUNT query
    'limit' => 20,
];
```

**Diferenças:**
- ✅ **Mais rápido** (não faz COUNT)
- ❌ **Sem total de páginas** (apenas Anterior/Próximo)
- ✅ **Ideal para feeds infinitos**

---

## 🎨 PaginatorHelper - Links de Navegação

### Template Completo

```php
<!-- templates/Articles/index.php -->
<h2>Artigos (<?= $this->Paginator->counter('{{count}} artigos') ?>)</h2>

<table>
    <thead>
        <tr>
            <th><?= $this->Paginator->sort('id', '#') ?></th>
            <th><?= $this->Paginator->sort('title', 'Título') ?></th>
            <th><?= $this->Paginator->sort('Users.name', 'Autor') ?></th>
            <th><?= $this->Paginator->sort('created', 'Data') ?></th>
        </tr>
    </thead>
    <tbody>
        <?php foreach ($articles as $article): ?>
            <tr>
                <td><?= h($article->id) ?></td>
                <td><?= h($article->title) ?></td>
                <td><?= h($article->user->name) ?></td>
                <td><?= h($article->created->format('d/m/Y')) ?></td>
            </tr>
        <?php endforeach; ?>
    </tbody>
</table>

<!-- Navegação de Páginas -->
<div class="paginator">
    <ul class="pagination">
        <?= $this->Paginator->first('<< Primeira') ?>
        <?= $this->Paginator->prev('< Anterior') ?>
        <?= $this->Paginator->numbers() ?>
        <?= $this->Paginator->next('Próxima >') ?>
        <?= $this->Paginator->last('Última >>') ?>
    </ul>
    <p><?= $this->Paginator->counter('Página {{page}} de {{pages}}, mostrando {{current}} registros de {{count}} no total') ?></p>
</div>
```

### Métodos do PaginatorHelper

| Método | Descrição | Exemplo |
|--------|-----------|---------|
| `sort()` | Link de ordenação | `sort('title', 'Título')` |
| `first()` | Link primeira página | `first('<< Primeira')` |
| `prev()` | Link página anterior | `prev('< Anterior')` |
| `numbers()` | Links numéricos | `numbers()` |
| `next()` | Link próxima página | `next('Próxima >')` |
| `last()` | Link última página | `last('Última >>')` |
| `counter()` | Contador de registros | `counter('{{count}} total')` |
| `hasNext()` | Verifica se há próxima | `hasNext()` |
| `hasPrev()` | Verifica se há anterior | `hasPrev()` |

### Customizar Links Numéricos

```php
<?= $this->Paginator->numbers([
    'first' => 2,        // Sempre mostrar primeiras 2 páginas
    'last' => 2,         // Sempre mostrar últimas 2 páginas
    'modulus' => 4,      // Quantas páginas ao redor da atual
    'before' => '<ul>',
    'after' => '</ul>',
]) ?>
```

---

## ⚠️ Tratamento de Erros - Páginas Fora do Range

Por padrão, CakePHP lança `NotFoundException` para páginas inexistentes:

```php
use Cake\Http\Exception\NotFoundException;

public function index()
{
    try {
        $articles = $this->paginate($this->Articles);
        $this->set(compact('articles'));
    } catch (NotFoundException $e) {
        // Redirecionar para primeira página
        return $this->redirect(['action' => 'index', '?' => ['page' => 1]]);
    }
}
```

---

## 🎯 Exemplo Completo: Blog com Filtros

### Controller

```php
<?php
namespace App\Controller;

class ArticlesController extends AppController
{
    protected array $paginate = [
        'limit' => 15,
        'maxLimit' => 100,
        'order' => ['Articles.created' => 'DESC'],
        'contain' => ['Users', 'Categories'],
        'sortableFields' => [
            'Articles.id',
            'Articles.title',
            'Articles.created',
            'Articles.view_count',
            'Users.name',
            'Categories.name'
        ],
    ];

    public function index()
    {
        $query = $this->Articles->find();

        // Filtro por categoria
        if ($categoryId = $this->request->getQuery('category')) {
            $query->where(['Articles.category_id' => $categoryId]);
        }

        // Filtro por busca
        if ($search = $this->request->getQuery('search')) {
            $query->where([
                'OR' => [
                    'Articles.title LIKE' => "%{$search}%",
                    'Articles.body LIKE' => "%{$search}%"
                ]
            ]);
        }

        // Filtro por status
        if ($status = $this->request->getQuery('status')) {
            $query->where(['Articles.status' => $status]);
        }

        $articles = $this->paginate($query);
        $categories = $this->Articles->Categories->find('list');

        $this->set(compact('articles', 'categories'));
    }
}
```

### Template com Filtros

```php
<!-- templates/Articles/index.php -->
<h1>Artigos do Blog</h1>

<!-- Formulário de Filtros -->
<form method="get">
    <input type="text" name="search" placeholder="Buscar..."
           value="<?= h($this->request->getQuery('search')) ?>">

    <select name="category">
        <option value="">Todas Categorias</option>
        <?php foreach ($categories as $id => $name): ?>
            <option value="<?= $id ?>"
                    <?= $this->request->getQuery('category') == $id ? 'selected' : '' ?>>
                <?= h($name) ?>
            </option>
        <?php endforeach; ?>
    </select>

    <select name="status">
        <option value="">Todos Status</option>
        <option value="draft" <?= $this->request->getQuery('status') === 'draft' ? 'selected' : '' ?>>Rascunho</option>
        <option value="published" <?= $this->request->getQuery('status') === 'published' ? 'selected' : '' ?>>Publicado</option>
    </select>

    <button type="submit">Filtrar</button>
</form>

<!-- Tabela de Resultados -->
<table class="table">
    <thead>
        <tr>
            <th><?= $this->Paginator->sort('id', 'ID') ?></th>
            <th><?= $this->Paginator->sort('title', 'Título') ?></th>
            <th><?= $this->Paginator->sort('Categories.name', 'Categoria') ?></th>
            <th><?= $this->Paginator->sort('Users.name', 'Autor') ?></th>
            <th><?= $this->Paginator->sort('view_count', 'Visualizações') ?></th>
            <th><?= $this->Paginator->sort('created', 'Criado em') ?></th>
        </tr>
    </thead>
    <tbody>
        <?php foreach ($articles as $article): ?>
            <tr>
                <td><?= h($article->id) ?></td>
                <td><?= $this->Html->link($article->title, ['action' => 'view', $article->id]) ?></td>
                <td><?= h($article->category->name) ?></td>
                <td><?= h($article->user->name) ?></td>
                <td><?= number_format($article->view_count) ?></td>
                <td><?= $article->created->format('d/m/Y H:i') ?></td>
            </tr>
        <?php endforeach; ?>
    </tbody>
</table>

<!-- Paginação -->
<div class="paginator">
    <ul class="pagination">
        <?= $this->Paginator->first('«', ['class' => 'page-item']) ?>
        <?= $this->Paginator->prev('‹', ['class' => 'page-item']) ?>
        <?= $this->Paginator->numbers(['class' => 'page-item']) ?>
        <?= $this->Paginator->next('›', ['class' => 'page-item']) ?>
        <?= $this->Paginator->last('»', ['class' => 'page-item']) ?>
    </ul>
    <p class="paginator-counter">
        <?= $this->Paginator->counter('Mostrando {{current}} de {{count}} artigos (Página {{page}}/{{pages}})') ?>
    </p>
</div>
```

---

## 🚀 Quick Start

**1. Configurar paginação no Controller:**
```php
protected array $paginate = [
    'limit' => 20,
    'order' => ['Articles.created' => 'DESC']
];
```

**2. Paginar query:**
```php
$articles = $this->paginate($this->Articles);
```

**3. Adicionar links na view:**
```php
<?= $this->Paginator->prev() ?>
<?= $this->Paginator->numbers() ?>
<?= $this->Paginator->next() ?>
```

---

## 📚 Referências

- [Pagination - CakePHP Book](https://book.cakephp.org/5/en/controllers/pagination.html)
- [PaginatorHelper API](https://api.cakephp.org/5.0/class-Cake.View.Helper.PaginatorHelper.html)
- [NumericPaginator API](https://api.cakephp.org/5.0/class-Cake.Datasource.Paging.NumericPaginator.html)

---

**✨ Expandido com [Claude Code](https://claude.com/claude-code)**
