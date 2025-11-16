# PagesController (Páginas Estáticas)

!!! info "Servir Páginas Estáticas"
    PagesController serve páginas estáticas sem lógica de negócio.

---

## 🎯 Uso

**URL:** `/pages/about` → `templates/Pages/about.php`

**Rotas:**
```php
$routes->connect('/about', ['controller' => 'Pages', 'action' => 'display', 'about']);
```

---

## 📚 Referências
- [PagesController - CakePHP Book](https://book.cakephp.org/5/en/controllers/pages-controller.html)
