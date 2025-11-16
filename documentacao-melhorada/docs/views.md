# Views (Camada de Visualização)

!!! info "A Camada V do MVC"
    Views geram a saída específica para cada requisição: HTML, JSON, XML, PDFs, etc.

---

## 📋 Conteúdo

- **AppView** - Classe base personalizada
- **Templates** - Arquivos PHP com sintaxe alternativa
- **Layouts** - Estrutura comum (header, footer)
- **Elements** - Componentes reutilizáveis
- **View Blocks** - Slots para conteúdo dinâmico
- **Helpers** - Lógica de apresentação
- **Events** - Ganchos no ciclo de renderização

---

## 🏗️ AppView - Classe Base

```php
<?php
// src/View/AppView.php
namespace App\View;

use Cake\View\View;

class AppView extends View
{
    public function initialize(): void
    {
        // Carregar helpers globais:
        $this->loadHelper('Form');
        $this->loadHelper('Html');
        $this->loadHelper('Flash');
    }
}
```

**Por que usar AppView?**
- Configuração centralizada de helpers
- Lógica de view compartilhada
- Evita repetição em todos os templates

---

## 📄 Templates - Arquivos de View

### Localização

```
templates/
├── Articles/
│   ├── index.php       # /articles/index
│   ├── view.php        # /articles/view/1
│   ├── add.php         # /articles/add
│   └── edit.php        # /articles/edit/1
├── layout/
│   └── default.php
└── element/
    └── flash/
        └── default.php
```

### Sintaxe Alternativa PHP

```php
<!-- Forma tradicional (evitar em templates): -->
<?php if ($isAdmin) { ?>
    <h1>Admin Panel</h1>
<?php } ?>

<!-- Sintaxe alternativa (RECOMENDADO): -->
<?php if ($isAdmin): ?>
    <h1>Admin Panel</h1>
<?php endif; ?>
```

### Echo Curto

```php
<!-- Tradicional: -->
<?php echo $title; ?>
<?php echo h($user->name); ?>

<!-- Short tag (RECOMENDADO): -->
<?= $title ?>
<?= h($user->name) ?>
```

### Estruturas de Controle

```php
<!-- foreach -->
<ul>
<?php foreach ($articles as $article): ?>
    <li><?= h($article->title) ?></li>
<?php endforeach; ?>
</ul>

<!-- if/elseif/else -->
<?php if ($role === 'admin'): ?>
    <p>Admin Dashboard</p>
<?php elseif ($role === 'editor'): ?>
    <p>Editor Dashboard</p>
<?php else: ?>
    <p>User Dashboard</p>
<?php endif; ?>

<!-- for -->
<?php for ($i = 0; $i < 5; $i++): ?>
    <p>Iteration <?= $i ?></p>
<?php endfor; ?>

<!-- while -->
<?php while ($hasItems): ?>
    <p>Item</p>
<?php endwhile; ?>
```

---

## 🛡️ Segurança - Escape de Dados

**SEMPRE** escapar dados do usuário com `h()`:

```php
<!-- ❌ VULNERÁVEL a XSS: -->
<p><?= $user->bio ?></p>

<!-- ✅ SEGURO: -->
<p><?= h($user->bio) ?></p>
```

**Quando escapar:**
- Dados de usuários
- Dados de formulários
- Dados de banco de dados
- Query strings
- Cookies

**Quando NÃO escapar:**
- HTML gerado por helpers do CakePHP
- Saída já escapada
- HTML confiável (de admins)

---

## 📦 View Variables

### Passar do Controller

```php
// src/Controller/ArticlesController.php
public function index()
{
    $articles = $this->Articles->find('all')->all();
    $title = 'Lista de Artigos';
    $count = $articles->count();

    // Método 1: set() individual
    $this->set('articles', $articles);
    $this->set('title', $title);

    // Método 2: set() com array
    $this->set([
        'articles' => $articles,
        'title' => $title,
    ]);

    // Método 3: compact() (MAIS COMUM)
    $this->set(compact('articles', 'title', 'count'));
}
```

### Acessar no Template

```php
<!-- templates/Articles/index.php -->
<h1><?= h($title) ?></h1>
<p>Total: <?= $count ?> artigos</p>

<ul>
<?php foreach ($articles as $article): ?>
    <li><?= h($article->title) ?></li>
<?php endforeach; ?>
</ul>
```

### Passar do Template para Layout

```php
<!-- templates/Articles/view.php -->
<?php
// Definir variável para o layout:
$this->set('sidebar', true);
$this->set('activeMenu', 'articles');
?>

<!-- Layout pode usar $sidebar e $activeMenu -->
```

---

## 🔄 Extending Views - Herança

### View Pai (Base)

```php
<!-- templates/Common/view.php -->
<h1><?= h($this->fetch('title')) ?></h1>
<?= $this->fetch('content') ?>

<div class="actions">
    <h3>Actions</h3>
    <ul>
    <?= $this->fetch('sidebar') ?>
    </ul>
</div>
```

### View Filha (Estende)

```php
<!-- templates/Posts/view.php -->
<?php
$this->extend('/Common/view');

$this->assign('title', $post->title);

$this->start('sidebar');
?>
<li><?= $this->Html->link('Edit', ['action' => 'edit', $post->id]) ?></li>
<li><?= $this->Html->link('Delete', ['action' => 'delete', $post->id]) ?></li>
<?php $this->end(); ?>

<!-- Conteúdo não capturado vai para o bloco 'content': -->
<p><?= h($post->body) ?></p>
```

**Como funciona:**
1. View filha executa até o final
2. Blocos são capturados (`sidebar`, `title`)
3. Conteúdo não-capturado → bloco `content`
4. View pai é renderizada com blocos preenchidos

---

## 🧩 View Blocks - Slots Dinâmicos

### Métodos Principais

| Método | O que faz |
|--------|-----------|
| `start('name')` | Inicia captura de bloco |
| `end()` | Finaliza captura |
| `assign('name', $value)` | Atribui valor direto |
| `append('name', $value)` | Adiciona ao final |
| `prepend('name', $value)` | Adiciona no início |
| `fetch('name')` | Exibe bloco |
| `reset('name')` | Limpa bloco |

### Exemplo: Sidebar Dinâmica

```php
<!-- templates/Articles/index.php -->
<?php
$this->start('sidebar');
echo $this->element('sidebar/recent_topics');
echo $this->element('sidebar/recent_comments');
$this->end();
?>

<!-- Ou append() -->
<?php
$this->append('sidebar');
echo $this->element('sidebar/popular_topics');
$this->end();

// Forma curta:
$this->append('sidebar', $this->element('sidebar/popular_topics'));
?>
```

### Exemplo: Scripts e CSS

```php
<!-- templates/Articles/view.php -->
<?php
// Adicionar script específico desta página:
$this->Html->script('carousel', ['block' => true]);
$this->Html->css('carousel', ['block' => true]);
?>

<!-- templates/layout/default.php -->
<!DOCTYPE html>
<html>
<head>
    <title><?= h($this->fetch('title')) ?></title>
    <?= $this->fetch('css') ?>
</head>
<body>
    <?= $this->fetch('content') ?>
    <?= $this->fetch('script') ?>
</body>
</html>
```

### Bloco com Fallback

```php
<!-- templates/layout/default.php -->
<div class="shopping-cart">
    <h3>Seu Carrinho</h3>
    <?= $this->fetch('cart', 'Seu carrinho está vazio') ?>
</div>

<!-- Se bloco 'cart' não existe, exibe mensagem padrão -->
```

### Condicional no Layout

```php
<?php if ($this->fetch('menu')): ?>
<div class="menu">
    <h3>Menu Options</h3>
    <?= $this->fetch('menu') ?>
</div>
<?php endif; ?>
```

---

## 🎨 Layouts - Estrutura Comum

### Layout Padrão

```php
<!-- templates/layout/default.php -->
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title><?= h($this->fetch('title')) ?></title>

    <?= $this->Html->meta('icon') ?>
    <?= $this->fetch('meta') ?>
    <?= $this->fetch('css') ?>
</head>
<body>
    <header>
        <nav>
            <!-- Menu de navegação -->
        </nav>
    </header>

    <main>
        <?= $this->Flash->render() ?>
        <?= $this->fetch('content') ?>
    </main>

    <footer>
        <!-- Rodapé -->
    </footer>

    <?= $this->fetch('script') ?>
</body>
</html>
```

### Trocar Layout no Controller

```php
// src/Controller/UsersController.php
public function admin()
{
    $this->viewBuilder()->setLayout('admin');
}
```

### Trocar Layout no Template

```php
<!-- templates/Users/profile.php -->
<?php
$this->layout = 'sidebar';
?>
```

### Layout de Plugin

```php
public function view()
{
    $this->viewBuilder()->setLayout('Contacts.contact');
}
```

### Layouts Built-in

CakePHP skeleton vem com:

| Layout | Uso |
|--------|-----|
| `default.php` | Layout padrão HTML |
| `ajax.php` | Respostas AJAX (vazio) |
| `rss/default.php` | Feeds RSS |

---

## 🔗 Elements - Componentes Reutilizáveis

### Criar Element

```php
<!-- templates/element/helpbox.php -->
<div class="helpbox">
    <h4>Ajuda</h4>
    <p><?= $helptext ?></p>
</div>
```

### Usar Element

```php
<!-- templates/Articles/view.php -->
<?= $this->element('helpbox', [
    'helptext' => 'Este artigo foi publicado em 2025.',
]) ?>
```

### Element com Subfolder

```
templates/element/
└── sidebar/
    ├── recent_topics.php
    └── popular_topics.php
```

```php
<?= $this->element('sidebar/recent_topics') ?>
```

### Element de Plugin

```php
<?= $this->element('Contacts.helpbox') ?>
```

### Cache de Element

```php
<!-- Cache por 1 hora: -->
<?= $this->element('helpbox', [], [
    'cache' => ['config' => 'short', 'key' => 'unique_key_1'],
]) ?>

<!-- Cache simples (usa config padrão): -->
<?= $this->element('helpbox', [], ['cache' => true]) ?>
```

**Importante:** Se renderizar o mesmo element várias vezes, use `key` diferente!

```php
<?= $this->element('helpbox', ['var' => $var1], [
    'cache' => ['key' => 'first_use', 'config' => 'view_long'],
]) ?>

<?= $this->element('helpbox', ['var' => $var2], [
    'cache' => ['key' => 'second_use', 'config' => 'view_long'],
]) ?>
```

### Element com Lógica Complexa?

Use **View Cells** em vez de Elements! (ver [views/cells](/views/cells))

---

## 🎯 Exemplo Completo: Blog CRUD

### Controller

```php
<?php
// src/Controller/PostsController.php
namespace App\Controller;

class PostsController extends AppController
{
    public function index()
    {
        $posts = $this->Posts->find('all')
            ->contain(['Users', 'Categories'])
            ->order(['Posts.created' => 'DESC'])
            ->all();

        $this->set(compact('posts'));
    }

    public function view($id)
    {
        $post = $this->Posts->get($id, contain: ['Users', 'Categories', 'Comments']);
        $this->set(compact('post'));
    }
}
```

### Layout

```php
<!-- templates/layout/blog.php -->
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <title><?= $this->fetch('title', 'Meu Blog') ?></title>
    <?= $this->Html->css('blog') ?>
</head>
<body>
    <header>
        <h1>Meu Blog</h1>
        <nav>
            <?= $this->Html->link('Home', '/') ?>
            <?= $this->Html->link('Sobre', '/about') ?>
        </nav>
    </header>

    <?= $this->Flash->render() ?>

    <main>
        <?= $this->fetch('content') ?>
    </main>

    <aside>
        <?= $this->fetch('sidebar') ?>
    </aside>

    <footer>
        <p>&copy; 2025 Meu Blog</p>
    </footer>

    <?= $this->fetch('script') ?>
</body>
</html>
```

### Template Index

```php
<!-- templates/Posts/index.php -->
<?php
$this->assign('title', 'Todos os Posts');
$this->layout = 'blog';

$this->start('sidebar');
echo $this->element('sidebar/categories');
echo $this->element('sidebar/recent_posts');
$this->end();
?>

<h2>Posts Recentes</h2>

<?php foreach ($posts as $post): ?>
    <article>
        <h3><?= $this->Html->link($post->title, ['action' => 'view', $post->id]) ?></h3>
        <p class="meta">
            Por <?= h($post->user->name) ?>
            em <?= $post->created->format('d/m/Y H:i') ?>
        </p>
        <p><?= h($post->excerpt) ?></p>
        <?= $this->Html->link('Ler mais →', ['action' => 'view', $post->id]) ?>
    </article>
<?php endforeach; ?>
```

### Template View

```php
<!-- templates/Posts/view.php -->
<?php
$this->assign('title', $post->title);
$this->layout = 'blog';

$this->start('sidebar');
echo $this->element('sidebar/author_bio', ['user' => $post->user]);
echo $this->element('sidebar/related_posts', ['category' => $post->category]);
$this->end();
?>

<article>
    <h2><?= h($post->title) ?></h2>

    <p class="meta">
        Por <?= h($post->user->name) ?>
        em <?= $post->created->format('d/m/Y H:i') ?>
        | Categoria: <?= h($post->category->name) ?>
    </p>

    <div class="content">
        <?= h($post->body) ?>
    </div>

    <section class="comments">
        <h3><?= $post->comments->count() ?> Comentários</h3>
        <?php foreach ($post->comments as $comment): ?>
            <?= $this->element('comment', compact('comment')) ?>
        <?php endforeach; ?>
    </section>
</article>
```

### Element: Comment

```php
<!-- templates/element/comment.php -->
<div class="comment">
    <p class="author"><?= h($comment->author) ?></p>
    <p class="date"><?= $comment->created->format('d/m/Y H:i') ?></p>
    <p><?= h($comment->text) ?></p>
</div>
```

---

## ⚙️ View Events

CakePHP dispara eventos durante o ciclo de renderização:

| Evento | Quando dispara |
|--------|----------------|
| `View.beforeRender` | Antes de renderizar a view |
| `View.beforeRenderFile` | Antes de renderizar cada arquivo |
| `View.afterRenderFile` | Após renderizar cada arquivo |
| `View.afterRender` | Após renderizar a view |
| `View.beforeLayout` | Antes de renderizar o layout |
| `View.afterLayout` | Após renderizar o layout |

### Listener de Event

```php
<?php
// src/Event/ViewListener.php
namespace App\Event;

use Cake\Event\EventListenerInterface;
use Cake\Event\EventInterface;

class ViewListener implements EventListenerInterface
{
    public function implementedEvents(): array
    {
        return [
            'View.beforeRender' => 'beforeRender',
        ];
    }

    public function beforeRender(EventInterface $event)
    {
        $view = $event->getSubject();
        // Adicionar variável global a todas as views:
        $view->set('siteTitle', 'Meu Site');
    }
}
```

---

## 🔧 Custom View Classes

Para criar tipos customizados de views (PDF, Excel, etc.):

```php
<?php
// src/View/PdfView.php
namespace App\View;

use Cake\View\View;

class PdfView extends View
{
    public function render($view = null, $layout = null)
    {
        // Lógica customizada para gerar PDF
        $html = parent::render($view, $layout);

        // Converter HTML para PDF com biblioteca externa
        return $this->generatePdf($html);
    }

    protected function generatePdf($html)
    {
        // Usar Dompdf, mPDF, TCPDF, etc.
    }
}
```

**Usar no Controller:**

```php
public function invoice($id)
{
    $this->viewBuilder()->setClassName('Pdf');
    $invoice = $this->Invoices->get($id);
    $this->set(compact('invoice'));
}
```

**Convenções:**
- Arquivo: `src/View/PdfView.php`
- Classe: `PdfView` (sufixo `View`)
- Referência: `'Pdf'` (sem sufixo)

---

## 🌐 Views para JSON/XML

Ver documentação específica: [JSON e XML Views](/views/json-and-xml-views)

**Exemplo rápido:**

```php
// Controller
public function index()
{
    $this->viewBuilder()->setClassName('Json');
    $articles = $this->Articles->find('all')->all();
    $this->set('articles', $articles);
    $this->viewBuilder()->setOption('serialize', ['articles']);
}
```

---

## 📚 Resumo Rápido

### Template Syntax

```php
<?= $variable ?>                          // Echo curto
<?php foreach ($items as $item): ?>       // Foreach alternativo
<?php endforeach; ?>

<?php if ($condition): ?>                 // If alternativo
<?php endif; ?>

<?= h($user->name) ?>                     // Escape XSS
```

### Blocks

```php
$this->start('name');                     // Iniciar bloco
$this->end();                             // Finalizar bloco
$this->assign('name', $value);            // Atribuir direto
$this->append('name', $value);            // Adicionar ao final
$this->prepend('name', $value);           // Adicionar no início
$this->fetch('name', 'default');          // Exibir bloco
```

### Layouts

```php
$this->viewBuilder()->setLayout('admin'); // Controller
$this->layout = 'admin';                  // Template
$this->extend('/Common/view');            // View inheritance
```

### Elements

```php
$this->element('name');                           // Renderizar
$this->element('name', ['var' => $value]);        // Com dados
$this->element('name', [], ['cache' => true]);    // Com cache
$this->element('Plugin.name');                    // De plugin
```

---

## 📚 Referências

- [Views - CakePHP Book](https://book.cakephp.org/5/en/views.html)
- [Helpers](/views/helpers)
- [Cells](/views/cells)
- [Themes](/views/themes)
- [JSON & XML Views](/views/json-and-xml-views)

---

**✨ Criado com [Claude Code](https://claude.com/claude-code)**
