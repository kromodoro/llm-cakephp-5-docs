# Components (Componentes Reutilizáveis)

!!! info "Lógica Compartilhada entre Controllers"
    Components encapsulam funcionalidades reutilizáveis entre múltiplos controllers.

---

## 📋 Resumo

**Components:** Pacotes de lógica compartilhada

**Built-in:**
- FlashComponent - Mensagens flash
- FormProtectionComponent - CSRF/tampering
- CheckHttpCacheComponent - HTTP caching

---

## 🔧 Usando Components

```php
class PostsController extends AppController
{
    public function initialize(): void
    {
        parent::initialize();
        $this->loadComponent('Flash');
        $this->loadComponent('FormProtection', [
            'unlockedActions' => ['index'],
        ]);
    }

    public function delete($id)
    {
        $post = $this->Posts->get($id);
        if ($this->Posts->delete($post)) {
            $this->Flash->success('Post deletado!');
            return $this->redirect(['action' => 'index']);
        }
        $this->Flash->error('Erro ao deletar.');
    }
}
```

---

## 🛠️ Criando Components

```php
<?php
// src/Controller/Component/MathComponent.php
namespace App\Controller\Component;

use Cake\Controller\Component;

class MathComponent extends Component
{
    public function add($a, $b)
    {
        return $a + $b;
    }
}
```

**Usar:**
```php
$this->loadComponent('Math');
$result = $this->Math->add(5, 3);  // 8
```

---

## 📚 Referências
- [Components - CakePHP Book](https://book.cakephp.org/5/en/controllers/components.html)
