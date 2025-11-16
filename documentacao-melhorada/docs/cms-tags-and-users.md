# Tutorial CMS - Tags e Usuários

!!! info "Parte 5 do Tutorial CMS"
    Esta é a quinta parte do [Tutorial CMS](../tutorials-and-examples.md). Certifique-se de ter completado:

    - ✅ [Parte 1: Instalação](cms-installation.md)
    - ✅ [Parte 2: Banco de Dados](cms-database.md)
    - ✅ [Parte 3: Model](cms-articles-model.md)
    - ✅ [Parte 4: Controller & Views](cms-articles-controller.md)

---

## 📋 O Que Você Vai Aprender

- ⚡ **Bake CLI** - Gerar código automaticamente
- 👥 **CRUD de Users** - Gerenciamento de usuários
- 🏷️ **CRUD de Tags** - Sistema de categorização
- 🔗 **Relacionamento N:N** - Articles ↔ Tags
- 🛣️ **Rotas Customizadas** - `/articles/tagged/*`
- 🔍 **Custom Finders** - `findTagged()`
- ✨ **Virtual Fields** - `tag_string` (comma-separated)
- 🔐 **Password Hashing** - Segurança de senhas

---

## ⚡ Usando Bake CLI

### O Que é Bake?

**Bake** é uma ferramenta de **geração de código** que:

- ✅ Gera Model, Controller, Templates automaticamente
- ✅ Detecta relacionamentos via foreign keys
- ✅ Cria validações baseadas no schema
- ✅ Gera testes unitários
- ✅ Segue todas as convenções do CakePHP

---

## 👥 Gerando CRUD de Users

### Etapa 1: Gerar Model

```bash
cd /home/kromodoro/Documentos/cakephp/app
bin/cake bake model Users
```

**Output:**
```
Baking table class for Users...
Creating file src/Model/Table/UsersTable.php
Wrote src/Model/Table/UsersTable.php

Baking entity class for User...
Creating file src/Model/Entity/User.php
Wrote src/Model/Entity/User.php

Baking test case...
Creating file tests/TestCase/Model/Table/UsersTableTest.php
Wrote tests/TestCase/Model/Table/UsersTableTest.php

Baking test fixture...
Creating file tests/Fixture/UsersFixture.php
Wrote tests/Fixture/UsersFixture.php
```

---

### Etapa 2: Gerar Controller

```bash
bin/cake bake controller Users
```

**Output:**
```
Baking controller class for Users...
Creating file src/Controller/UsersController.php
Wrote src/Controller/UsersController.php
```

**Gerado:** CRUD completo (index, view, add, edit, delete)

---

### Etapa 3: Gerar Templates

```bash
bin/cake bake template Users
```

**Output:**
```
Baking view templates for Users...
Creating file templates/Users/index.php
Creating file templates/Users/view.php
Creating file templates/Users/add.php
Creating file templates/Users/edit.php
```

---

### 🔐 IMPORTANTE: Adicionar Password Hashing

!!! danger "Segurança Crítica"
    O Bake **NÃO** adiciona hash de senha automaticamente!

Edite `src/Model/Entity/User.php`:

```php
<?php
// src/Model/Entity/User.php
namespace App\Model\Entity;

use Cake\Auth\DefaultPasswordHasher;
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

    // Ocultar senha em JSON/arrays
    protected array $_hidden = [
        'password',
    ];

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

**Como funciona:**

```php
$user = $this->Users->newEntity(['password' => 'secret']);
// password salvo como: $2y$10$u5ceCKLTkqG1Y8W0h8F6lOuKQTqJXqQq...

// Verificar senha:
$hasher = new DefaultPasswordHasher();
$hasher->check('secret', $user->password);  // true
```

---

### Adicionar Validação de Email

Edite `src/Model/Table/UsersTable.php`:

```php
public function validationDefault(Validator $validator): Validator
{
    $validator
        ->integer('id')
        ->allowEmptyString('id', null, 'create')

        ->email('email', false, 'E-mail inválido')
        ->requirePresence('email', 'create', 'E-mail obrigatório')
        ->notEmptyString('email')
        ->add('email', 'unique', [
            'rule' => 'validateUnique',
            'provider' => 'table',
            'message' => 'Este e-mail já está cadastrado'
        ])

        ->scalar('password')
        ->maxLength('password', 255)
        ->requirePresence('password', 'create', 'Senha obrigatória')
        ->notEmptyString('password')
        ->minLength('password', 8, 'Senha deve ter no mínimo 8 caracteres');

    return $validator;
}

public function buildRules(RulesChecker $rules): RulesChecker
{
    $rules->add($rules->isUnique(['email']), [
        'errorField' => 'email',
        'message' => 'E-mail já cadastrado'
    ]);

    return $rules;
}
```

---

## 🏷️ Gerando CRUD de Tags

### Comando Único: bake all

```bash
bin/cake bake all Tags
```

**Gera TUDO de uma vez:**
- ✅ Model (Table + Entity)
- ✅ Controller
- ✅ Templates (index, view, add, edit)
- ✅ Tests

---

### Criar Algumas Tags

Acesse: [http://localhost:8765/tags/add](http://localhost:8765/tags/add)

Crie tags:
- CakePHP
- PHP
- MySQL
- Tutorial
- REST API

---

## 🔗 Relacionamento Articles ↔ Tags

### Atualizar ArticlesTable

Edite `src/Model/Table/ArticlesTable.php`:

```php
public function initialize(array $config): void
{
    parent::initialize($config);

    $this->addBehavior('Timestamp');

    $this->belongsTo('Users', [
        'foreignKey' => 'user_id',
        'joinType' => 'INNER',
    ]);

    // ← ADICIONAR belongsToMany:
    $this->belongsToMany('Tags', [
        'foreignKey' => 'article_id',
        'targetForeignKey' => 'tag_id',
        'joinTable' => 'articles_tags',
        'dependent' => true,  // Deletar relações ao deletar artigo
    ]);
}
```

**Como funciona:**

```php
// Carregar artigo com tags:
$article = $this->Articles
    ->get(1, contain: ['Tags']);

foreach ($article->tags as $tag) {
    echo $tag->title;  // CakePHP, PHP, MySQL
}
```

---

## 📝 Formulário de Tags em Articles

### Controller: add()

Edite `src/Controller/ArticlesController.php`:

```php
public function add()
{
    $article = $this->Articles->newEmptyEntity();

    if ($this->request->is('post')) {
        $article = $this->Articles->patchEntity($article, $this->request->getData());
        $article->user_id = 1;  // Temporário

        if ($this->Articles->save($article)) {
            $this->Flash->success(__('Artigo salvo com sucesso!'));
            return $this->redirect(['action' => 'view', $article->slug]);
        }
        $this->Flash->error(__('Não foi possível salvar o artigo.'));
    }

    // ← ADICIONAR: Buscar tags para select
    $tags = $this->Articles->Tags->find('list')->all();
    $this->set(compact('article', 'tags'));
}
```

!!! info "find('list')"
    ```php
    $tags = $this->Tags->find('list')->all();
    // Retorna: ['1' => 'CakePHP', '2' => 'PHP', '3' => 'MySQL']
    // Formato: [id => displayField]
    ```

---

### Template: add.php

Edite `templates/element/article_form.php`:

```php
<?= $this->Form->create($article) ?>
<fieldset>
    <legend><?= __('Informações do Artigo') ?></legend>

    <?= $this->Form->control('title', [
        'label' => 'Título',
        'required' => true,
    ]) ?>

    <?= $this->Form->control('body', [
        'label' => 'Conteúdo',
        'type' => 'textarea',
        'rows' => 15,
        'required' => true,
    ]) ?>

    <?= $this->Form->control('published', [
        'label' => 'Publicar artigo?',
        'type' => 'checkbox',
    ]) ?>

    <!-- ← ADICIONAR select de tags -->
    <?= $this->Form->control('tags._ids', [
        'label' => 'Tags',
        'options' => $tags,
        'multiple' => 'checkbox',  // ou 'multiple' => true para <select multiple>
    ]) ?>
</fieldset>

<div class="actions">
    <?= $this->Form->button(__('Salvar Artigo'), ['class' => 'button']) ?>
    <?= $this->Html->link(__('Cancelar'), ['action' => 'index'], ['class' => 'button secondary']) ?>
</div>
<?= $this->Form->end() ?>
```

!!! tip "tags._ids Sintaxe Especial"
    `tags._ids` diz ao FormHelper para:

    1. Criar input com name `tags[_ids][]`
    2. Pre-selecionar tags associadas ao artigo (em edit)
    3. Salvar relacionamento N:N automaticamente

---

### Controller: edit()

Atualizar para carregar tags existentes:

```php
public function edit(?string $slug = null)
{
    $article = $this->Articles
        ->findBySlug($slug)
        ->contain(['Tags'])  // ← Carregar tags associadas
        ->firstOrFail();

    if ($this->request->is(['post', 'put', 'patch'])) {
        $this->Articles->patchEntity($article, $this->request->getData());

        if ($this->Articles->save($article)) {
            $this->Flash->success(__('Artigo atualizado com sucesso!'));
            return $this->redirect(['action' => 'view', $article->slug]);
        }
        $this->Flash->error(__('Não foi possível atualizar o artigo.'));
    }

    // ← Buscar todas as tags para select
    $tags = $this->Articles->Tags->find('list')->all();
    $this->set(compact('article', 'tags'));
}
```

---

## 🛣️ Rota: /articles/tagged/*

### Criar Rota Customizada

Edite `config/routes.php`:

```php
<?php
use Cake\Routing\Route\DashedRoute;
use Cake\Routing\RouteBuilder;

$routes->setRouteClass(DashedRoute::class);

$routes->scope('/', function (RouteBuilder $builder) {
    $builder->connect('/', ['controller' => 'Pages', 'action' => 'display', 'home']);
    $builder->connect('/pages/*', ['controller' => 'Pages', 'action' => 'display']);

    // ← ADICIONAR rota customizada:
    $builder->scope('/articles', function (RouteBuilder $builder) {
        $builder->connect('/tagged/*', [
            'controller' => 'Articles',
            'action' => 'tags'
        ]);
    });

    $builder->fallbacks();
});
```

**URLs aceitas:**
- `/articles/tagged` → sem tags (artigos sem tag)
- `/articles/tagged/php` → 1 tag
- `/articles/tagged/php/cakephp/mysql` → múltiplas tags

---

### Controller: tags()

Adicione em `ArticlesController.php`:

```php
public function tags(...$tags)
{
    // Buscar artigos com custom finder
    $articles = $this->Articles
        ->find('tagged', tags: $tags)
        ->contain(['Users'])
        ->all();

    $this->set([
        'articles' => $articles,
        'tags' => $tags
    ]);
}
```

!!! info "Variadic Arguments (PHP 8.0+)"
    ```php
    public function tags(...$tags)
    // /articles/tagged/php/mysql
    // $tags = ['php', 'mysql']
    ```

---

### Custom Finder: findTagged()

Edite `src/Model/Table/ArticlesTable.php`:

```php
use Cake\ORM\Query\SelectQuery;

public function findTagged(SelectQuery $query, array $tags = []): SelectQuery
{
    if (empty($tags)) {
        // Artigos SEM tags:
        $query->leftJoinWith('Tags')
            ->where(['Tags.id IS' => null]);
    } else {
        // Artigos COM uma ou mais das tags fornecidas:
        $query->innerJoinWith('Tags')
            ->where(['Tags.title IN' => $tags])
            ->distinct();  // ← Evitar duplicatas
    }

    return $query;
}
```

**SQL gerado:**

```sql
-- /articles/tagged/php/mysql
SELECT DISTINCT articles.*
FROM articles
INNER JOIN articles_tags ON articles.id = articles_tags.article_id
INNER JOIN tags ON articles_tags.tag_id = tags.id
WHERE tags.title IN ('php', 'mysql')
```

---

### Template: tags.php

Crie `templates/Articles/tags.php`:

```php
<!-- File: templates/Articles/tags.php -->

<div class="articles tagged">
    <?php if (empty($tags)): ?>
        <h1><?= __('Artigos Sem Tags') ?></h1>
    <?php else: ?>
        <h1>
            <?= __('Artigos com as tags:') ?>
            <strong><?= $this->Text->toList(h($tags), 'ou') ?></strong>
        </h1>
    <?php endif; ?>

    <?php if ($articles->count()): ?>
        <section class="articles-list">
            <?php foreach ($articles as $article): ?>
                <article class="article-card">
                    <h4>
                        <?= $this->Html->link(
                            h($article->title),
                            ['controller' => 'Articles', 'action' => 'view', $article->slug]
                        ) ?>
                    </h4>
                    <p class="meta">
                        <small>
                            <?= __('Por:') ?> <?= h($article->user->email) ?>
                            | <?= $article->created->format('d/m/Y') ?>
                        </small>
                    </p>
                </article>
            <?php endforeach; ?>
        </section>
    <?php else: ?>
        <p class="notice">
            <?= __('Nenhum artigo encontrado com estas tags.') ?>
        </p>
    <?php endif; ?>

    <p><?= $this->Html->link(__('← Voltar para todos os artigos'), ['action' => 'index']) ?></p>
</div>
```

---

## ✨ Input de Tags (Comma-Separated)

Melhor UX: ao invés de select múltiplo, permitir digitar tags separadas por vírgula.

### Virtual Field: tag_string

Edite `src/Model/Entity/Article.php`:

```php
use Cake\Collection\Collection;

class Article extends Entity
{
    protected array $_accessible = [
        'user_id' => true,
        'title' => true,
        'slug' => true,
        'body' => true,
        'published' => true,
        'created' => true,
        'modified' => true,
        'user' => true,
        'tags' => true,
        'tag_string' => true,  // ← Adicionar
    ];

    // Expor tag_string como virtual field
    protected array $_virtual = ['tag_string'];

    // Getter: converte tags array → string
    protected function _getTagString(): string
    {
        if (isset($this->_fields['tag_string'])) {
            return $this->_fields['tag_string'];
        }

        if (empty($this->tags)) {
            return '';
        }

        // Extrair títulos e juntar com vírgula
        $tagTitles = collection($this->tags)->extract('title')->toArray();
        return implode(', ', $tagTitles);
    }
}
```

---

### Template: Substituir Select por Input Text

Edite `templates/element/article_form.php`:

```php
<!-- ANTES (select múltiplo):
<?= $this->Form->control('tags._ids', [
    'options' => $tags,
    'multiple' => 'checkbox'
]) ?>
-->

<!-- DEPOIS (input text): -->
<?= $this->Form->control('tag_string', [
    'type' => 'text',
    'label' => 'Tags (separadas por vírgula)',
    'placeholder' => 'Ex: php, cakephp, mysql',
]) ?>
```

---

### beforeSave: Criar Tags On-The-Fly

Edite `src/Model/Table/ArticlesTable.php`:

```php
public function beforeSave(EventInterface $event, $entity, $options): void
{
    // Gerar slug (código existente):
    if ($entity->isNew() && !$entity->slug) {
        // ... código de slug ...
    }

    // ← ADICIONAR processamento de tag_string:
    if ($entity->tag_string) {
        $entity->tags = $this->_buildTags($entity->tag_string);
    }
}

protected function _buildTags(string $tagString): array
{
    // Separar por vírgula, limpar espaços, normalizar case:
    $newTags = explode(',', $tagString);
    $newTags = array_map('trim', $newTags);
    $newTags = array_filter($newTags);  // Remover vazios
    $newTags = array_map('strtolower', $newTags);  // Case-insensitive
    $newTags = array_unique($newTags);  // Remover duplicatas

    // Buscar tags existentes (case-insensitive):
    $existingTags = $this->Tags->find()
        ->where(function ($exp) use ($newTags) {
            return $exp->in($exp->func()->lower('Tags.title'), $newTags);
        })
        ->all();

    $out = [];

    // Adicionar tags existentes:
    foreach ($existingTags as $tag) {
        $out[] = $tag;
    }

    // Identificar quais tags são novas:
    $existingTitles = array_map('strtolower', $existingTags->extract('title')->toArray());
    $newTagsOnly = array_diff($newTags, $existingTitles);

    // Criar tags novas:
    foreach ($newTagsOnly as $tagTitle) {
        $out[] = $this->Tags->newEntity([
            'title' => ucfirst($tagTitle)  // Capitalizar primeira letra
        ]);
    }

    return $out;
}
```

**Como funciona:**

```php
// Usuário digita: "php, MySQL, CakePHP, new tag"
// Resultado:
// - php → encontra "PHP" existente → usa existente
// - MySQL → encontra "mysql" existente → usa existente
// - CakePHP → encontra "CakePHP" existente → usa existente
// - new tag → não existe → CRIA "New tag"
```

---

### Atualizar view() e edit()

Carregar tags em todas as actions:

```php
public function view(?string $slug = null)
{
    $article = $this->Articles
        ->findBySlug($slug)
        ->contain(['Users', 'Tags'])  // ← Adicionar Tags
        ->firstOrFail();

    $this->set(compact('article'));
}

public function edit(?string $slug = null)
{
    $article = $this->Articles
        ->findBySlug($slug)
        ->contain(['Tags'])  // ← Já tem
        ->firstOrFail();

    // ... resto do código
}
```

---

### Mostrar Tags em view.php

Edite `templates/Articles/view.php`:

```php
<div class="articles view">
    <h1><?= h($article->title) ?></h1>

    <div class="article-meta">
        <p>
            <strong><?= __('Autor:') ?></strong>
            <?= h($article->user->email) ?>
        </p>
        <p>
            <strong><?= __('Criado em:') ?></strong>
            <?= $article->created->format('d/m/Y \à\s H:i') ?>
        </p>

        <!-- ← ADICIONAR exibição de tags -->
        <?php if ($article->tag_string): ?>
        <p>
            <strong><?= __('Tags:') ?></strong>
            <?php foreach ($article->tags as $tag): ?>
                <?= $this->Html->link(
                    $tag->title,
                    ['action' => 'tags', $tag->title],
                    ['class' => 'badge']
                ) ?>
            <?php endforeach; ?>
        </p>
        <?php endif; ?>
    </div>

    <div class="article-body">
        <?= nl2br(h($article->body)) ?>
    </div>

    <!-- ... ações (editar, deletar) ... -->
</div>
```

---

## 🎯 Testando o Sistema Completo

### 1. Criar Usuários

Acesse: [http://localhost:8765/users/add](http://localhost:8765/users/add)

- Email: `admin@example.com`
- Password: `senha123`

---

### 2. Criar Tags

Acesse: [http://localhost:8765/tags/add](http://localhost:8765/tags/add)

Criar: `PHP`, `CakePHP`, `MySQL`

---

### 3. Criar Artigo com Tags

Acesse: [http://localhost:8765/articles/add](http://localhost:8765/articles/add)

- Título: `Tutorial CakePHP`
- Tags: `php, cakephp, mysql`
- Publicar: ✅

---

### 4. Buscar por Tag

Acesse: [http://localhost:8765/articles/tagged/php](http://localhost:8765/articles/tagged/php)

Deve mostrar todos os artigos com tag "PHP"!

---

## ❓ Problemas Comuns

??? danger "Senha salva em plain text"
    **Causa:** Esqueceu `_setPassword()` na Entity.

    **Solução:** Ver seção "Password Hashing" acima.

---

??? danger "Erro: `Unknown method 'distinct'`"
    **Causa:** distinct() com parâmetros (sintaxe antiga).

    **Solução:**
    ```php
    // ❌ Errado:
    ->distinct($columns)

    // ✅ Correto:
    ->distinct()
    ```

---

??? warning "Tags duplicadas (case-sensitive)"
    **Causa:** `_buildTags()` não normaliza case.

    **Solução:** Ver código completo de `_buildTags()` acima com `strtolower()`.

---

## 🚀 Próximos Passos

✅ **CRUD completo + Tags funcionando!** Agora você pode:

1. 🔐 **[Implementar Autenticação](cms-authentication.md)** - Login/Logout real
2. 🛡️ **[Implementar Autorização](cms-authorization.md)** - Controle de acesso

---

## 📚 Referências

- ⚡ [Bake Console](https://book.cakephp.org/bake/2/en/usage.html)
- 🔗 [BelongsToMany Associations](https://book.cakephp.org/5/en/orm/associations.html#belongstomany-associations)
- 🛣️ [Routing](https://book.cakephp.org/5/en/development/routing.html)
- 🔍 [Custom Finders](https://book.cakephp.org/5/en/orm/retrieving-data-and-resultsets.html#custom-finder-methods)
- ✨ [Virtual Fields](https://book.cakephp.org/5/en/orm/entities.html#creating-virtual-fields)
- 🔐 [Password Hashing](https://book.cakephp.org/5/en/core-libraries/security.html#hashing-passwords)

---

**✨ Criado com [Claude Code](https://claude.com/claude-code)**
