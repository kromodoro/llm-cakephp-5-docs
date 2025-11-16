# Pagination (Paginação)

!!! info "Listagens Paginadas"
    CakePHP oferece paginação automática com PaginatorComponent e PaginatorHelper.

---

## 🎯 Paginação Completa

### Controller
```php
class ArticlesController extends AppController
{
    protected array $paginate = [
        'limit' => 20,
        'order' => ['Articles.created' => 'DESC'],
        'contain' => ['Users'],
    ];

    public function index()
    {
        $articles = $this->paginate($this->Articles);
        $this->set(compact('articles'));
    }
}
```

### Template
```php
<?php foreach ($articles as $article): ?>
    <h3><?= h($article->title) ?></h3>
<?php endforeach; ?>

<div class="paginator">
    <?= $this->Paginator->first('<<') ?>
    <?= $this->Paginator->prev('<') ?>
    <?= $this->Paginator->numbers() ?>
    <?= $this->Paginator->next('>') ?>
    <?= $this->Paginator->last('>>') ?>
    <p><?= $this->Paginator->counter() ?></p>
</div>
```

---

## 📚 Referências
- [Pagination - CakePHP Book](https://book.cakephp.org/5/en/controllers/pagination.html)
