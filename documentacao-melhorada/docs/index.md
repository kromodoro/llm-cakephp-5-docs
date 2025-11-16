# Bem-vindo ao CakePHP 5! 🎉

<div class="grid cards" markdown>

-   :material-rocket-launch:{ .lg .middle } __Comece Rapidamente__

    ---

    Novo no CakePHP? Comece com nosso tutorial prático e construa sua primeira aplicação em minutos!

    [:octicons-arrow-right-24: Tutorial Quickstart](quickstart.md)

-   :material-book-open-variant:{ .lg .middle } __Aprenda os Fundamentos__

    ---

    Entenda os conceitos principais do framework e como ele funciona internamente.

    [:octicons-arrow-right-24: CakePHP em Resumo](intro.md)

-   :material-help-circle:{ .lg .middle } __Precisa de Ajuda?__

    ---

    Encontre suporte na comunidade, documentação e recursos oficiais.

    [:octicons-arrow-right-24: Onde Obter Ajuda](intro/where-to-get-help.md)

-   :material-github:{ .lg .middle } __Código Aberto__

    ---

    Esta documentação é aberta! Contribua com melhorias via GitHub.

    [:octicons-arrow-right-24: Contribuir](https://github.com/cakephp/docs)

</div>

---

## 🌟 O que é o CakePHP?

O **CakePHP 5** é um framework moderno de desenvolvimento web para PHP que acelera a criação de aplicações robustas e escaláveis.

!!! info "Requisitos do Sistema"
    - **PHP:** 8.1 ou superior
    - **Banco de dados:** MySQL 5.7+, PostgreSQL 9.6+, SQLite 3.x, ou SQL Server 2012+
    - **Extensões PHP:** mbstring, intl, pdo

### Por que Escolher o CakePHP?

<div class="grid" markdown>

<div markdown>
**:fontawesome-solid-bolt: Desenvolvimento Rápido**

Configure menos, desenvolva mais! Convenções inteligentes eliminam configurações repetitivas.
</div>

<div markdown>
**:fontawesome-solid-shield: Segurança Integrada**

Proteção contra CSRF, SQL Injection, XSS e muito mais, ativada por padrão.
</div>

<div markdown>
**:fontawesome-solid-database: ORM Poderoso**

Trabalhe com bancos de dados usando código PHP limpo, sem SQL complexo.
</div>

<div markdown>
**:fontawesome-solid-puzzle-piece: Extensível**

Ecossistema rico de plugins e componentes reutilizáveis.
</div>

</div>

---

## 📚 Formatos Disponíveis

Leia a documentação do jeito que preferir:

<div class="grid cards" markdown>

-   :material-web:{ .lg .middle } **Online**

    Navegue pela documentação interativa com busca e navegação facilitadas.

-   :material-file-pdf-box:{ .lg .middle } **PDF**

    [Baixe o livro completo](https://book.cakephp.org/_downloads/en/CakePHPBook.pdf){ .md-button } em formato PDF para leitura offline.

-   :material-book-variant:{ .lg .middle } **EPUB**

    [Versão EPUB](https://book.cakephp.org/_downloads/en/CakePHP.epub){ .md-button } para leitores de e-books e dispositivos móveis.

</div>

---

## 🚀 Primeiros Passos

Aprender um novo framework pode ser intimidante e empolgante ao mesmo tempo! Criamos um roteiro estruturado para você começar com o pé direito.

### 1️⃣ Tutorial Prático (Recomendado!)

Se você é novo no CakePHP, **comece pelo [Quickstart](quickstart.md)**!

Neste tutorial hands-on, você vai:

- ✅ Criar um sistema de gerenciamento de conteúdo (CMS) completo
- ✅ Aprender a estrutura MVC na prática
- ✅ Trabalhar com banco de dados, validação e templates
- ✅ Ver o CakePHP em ação com exemplos reais

!!! tip "Tempo Estimado: 30-45 minutos"
    Ao final do Quickstart, você terá uma aplicação funcional e entenderá os conceitos fundamentais do CakePHP.

[:material-play-circle: Começar o Tutorial Quickstart](quickstart.md){ .md-button .md-button--primary }

---

### 2️⃣ Entenda os Fundamentos

Após concluir o Quickstart, aprofunde-se nos conceitos principais:

<div class="grid cards" markdown>

-   :material-sync:{ .lg .middle } **Ciclo de Requisição**

    ---

    Entenda como o CakePHP processa requisições HTTP do início ao fim.

    [Ciclo de Requisição →](#){ .md-button }

-   :material-format-list-checks:{ .lg .middle } **Convenções**

    ---

    Aprenda as convenções que tornam o CakePHP produtivo e organizado.

    [Convenções →](intro/conventions.md){ .md-button }

</div>

---

### 3️⃣ Domine o MVC

Explore cada camada da arquitetura Model-View-Controller:

=== "Controllers"

    **Controllers** coordenam o fluxo da aplicação.

    Eles recebem requisições, processam dados através dos Models e escolhem qual View renderizar.

    ```php
    namespace App\Controller;

    class ArticlesController extends AppController
    {
        public function index()
        {
            $articles = $this->Articles->find('all');
            $this->set(compact('articles'));
        }
    }
    ```

    [:octicons-arrow-right-24: Documentação de Controllers](controllers.md)

=== "Views"

    **Views** são a camada de apresentação.

    Elas transformam dados em HTML, JSON, XML ou qualquer outro formato que sua aplicação precise.

    ```php
    <!-- templates/Articles/index.php -->
    <h1>Artigos</h1>
    <?php foreach ($articles as $article): ?>
        <h2><?= h($article->title) ?></h2>
        <p><?= h($article->body) ?></p>
    <?php endforeach; ?>
    ```

    [:octicons-arrow-right-24: Documentação de Views](views.md)

=== "Models"

    **Models** são o coração da aplicação.

    Eles gerenciam dados, validação, regras de negócio e interação com o banco de dados.

    ```php
    namespace App\Model\Table;

    use Cake\ORM\Table;

    class ArticlesTable extends Table
    {
        public function validationDefault($validator)
        {
            $validator
                ->requirePresence('title')
                ->notEmptyString('title');

            return $validator;
        }
    }
    ```

    [:octicons-arrow-right-24: Documentação de Models/ORM](orm.md)

---

## 🎯 Roadmap de Aprendizado Completo

Sugerimos esta sequência para dominar o CakePHP:

```mermaid
graph LR
    A[1. Quickstart] --> B[2. Ciclo de Requisição]
    B --> C[3. Convenções]
    C --> D[4. Controllers]
    D --> E[5. Views]
    E --> F[6. Models/ORM]
    F --> G[7. Conceitos Avançados]

    style A fill:#4CAF50,stroke:#2E7D32,color:#fff
    style B fill:#2196F3,stroke:#1565C0,color:#fff
    style C fill:#2196F3,stroke:#1565C0,color:#fff
    style D fill:#FF9800,stroke:#E65100,color:#fff
    style E fill:#FF9800,stroke:#E65100,color:#fff
    style F fill:#FF9800,stroke:#E65100,color:#fff
    style G fill:#9C27B0,stroke:#6A1B9A,color:#fff
```

| Etapa | Tópico | Tempo | Descrição |
|-------|--------|-------|-----------|
| 1 | [Quickstart](quickstart.md) | 30-45 min | Tutorial prático - crie seu primeiro CMS |
| 2 | [Ciclo de Requisição](#) | 15 min | Entenda o fluxo interno do framework |
| 3 | [Convenções](intro/conventions.md) | 20 min | Padrões de nomenclatura e organização |
| 4 | [Controllers](controllers.md) | 1-2 horas | Gerenciamento de requisições e lógica |
| 5 | [Views](views.md) | 1-2 horas | Templates, helpers e apresentação |
| 6 | [Models/ORM](orm.md) | 2-3 horas | Banco de dados, validação e regras |
| 7 | Avançado | Variável | Autenticação, autorização, testes, etc. |

!!! success "Progresso Gradual"
    Não precisa aprender tudo de uma vez! Vá no seu ritmo e pratique cada conceito antes de avançar.

---

## 💡 Recursos Adicionais

### Comunidade e Suporte

<div class="grid" markdown>

<div markdown>
**:fontawesome-brands-slack: Slack**

Converse com desenvolvedores CakePHP em tempo real.

[Entrar no Slack →](https://cakephp.org/slack)
</div>

<div markdown>
**:fontawesome-brands-discourse: Fórum**

Faça perguntas e compartilhe conhecimento.

[Acessar Fórum →](https://discourse.cakephp.org/)
</div>

<div markdown>
**:fontawesome-brands-github: GitHub**

Reporte bugs, sugira features e contribua com código.

[GitHub →](https://github.com/cakephp/cakephp)
</div>

<div markdown>
**:fontawesome-brands-stack-overflow: Stack Overflow**

Busque respostas para problemas específicos.

[Stack Overflow →](https://stackoverflow.com/questions/tagged/cakephp)
</div>

</div>

---

### Plugins e Ferramentas

Acelere o desenvolvimento com plugins e ferramentas da comunidade:

- **[Awesome CakePHP](https://github.com/FriendsOfCake/awesome-cakephp)** - Lista curada de plugins, tutoriais e recursos
- **[CakePHP Debug Kit](https://github.com/cakephp/debug_kit)** - Ferramenta de debugging visual
- **[CakePHP Bake](https://book.cakephp.org/bake/5/en/)** - Geração automática de código
- **[Authentication Plugin](https://book.cakephp.org/authentication/3/)** - Sistema de autenticação modular
- **[Authorization Plugin](https://book.cakephp.org/authorization/3/)** - Controle de acesso granular

---

### Aprenda Mais

- 📺 **[Vídeos e Tutoriais](https://www.youtube.com/results?search_query=cakephp+tutorial)** - Cursos em vídeo
- 📖 **[Blog Oficial](https://bakery.cakephp.org/)** - Artigos e novidades
- 📰 **[Changelog](https://github.com/cakephp/cakephp/releases)** - Atualizações e releases
- 🎓 **[Certificação CakePHP](https://certification.cakephp.org/)** - Valide seus conhecimentos

---

## ❓ Perguntas Frequentes

??? question "É meu primeiro framework PHP - por onde começo?"

    **Resposta:** Comece pelo [Tutorial Quickstart](quickstart.md)! Ele foi projetado especificamente para iniciantes e ensina os conceitos fundamentais na prática.

    Se você nunca usou um framework antes, recomendamos também:

    1. Entender conceitos básicos de [OOP em PHP](https://www.php.net/manual/pt_BR/language.oop5.php)
    2. Conhecer o [padrão MVC](https://pt.wikipedia.org/wiki/MVC)
    3. Familiarizar-se com [Composer](https://getcomposer.org/)

??? question "Já conheço Laravel/Symfony - quais as diferenças?"

    **CakePHP vs. Laravel:**

    | Aspecto | CakePHP | Laravel |
    |---------|---------|---------|
    | Convenções | Muito forte | Moderado |
    | Configuração | Mínima (convenção) | Mais flexível |
    | ORM | CakePHP ORM | Eloquent |
    | Scaffolding | Bake (CLI) | Artisan |
    | Learning Curve | Rápida | Média |

    **CakePHP vs. Symfony:**

    - CakePHP é mais "opinativo" - menos escolhas, mais produtividade
    - Symfony é mais modular e configurável
    - CakePHP tem convenções mais rígidas (CoC - Convention over Configuration)

??? question "Onde encontro plugins e pacotes?"

    **Recursos principais:**

    - **[Packagist](https://packagist.org/?query=cakephp)** - Repositório oficial do Composer
    - **[Awesome CakePHP](https://github.com/FriendsOfCake/awesome-cakephp)** - Lista curada pela comunidade
    - **[CakePHP Plugins](https://plugins.cakephp.org/)** - Diretório oficial de plugins

    Para instalar um plugin, use o Composer:

    ```bash
    composer require vendor/plugin-name
    bin/cake plugin load PluginName
    ```

??? question "CakePHP é adequado para projetos grandes?"

    **Sim!** O CakePHP é usado em aplicações de grande escala por empresas como:

    - BMW
    - Hyundai
    - MIT
    - Express

    Benefícios para projetos grandes:

    - ✅ Estrutura organizada por convenções
    - ✅ ORM poderoso para queries complexas
    - ✅ Sistema de plugins para modularização
    - ✅ Cache integrado (Redis, Memcached, etc.)
    - ✅ Suporte a múltiplos bancos de dados

??? question "Como atualizo de versões antigas do CakePHP?"

    **Guias de Migração:**

    - [CakePHP 3.x → 4.x](https://book.cakephp.org/4/en/appendices/4-0-migration-guide.html)
    - [CakePHP 4.x → 5.x](https://book.cakephp.org/5/en/appendices/5-0-migration-guide.html)

    **Ferramentas de Upgrade:**

    ```bash
    # Instalar ferramenta de upgrade
    composer require --dev cakephp/upgrade

    # Executar upgrade
    bin/cake upgrade rector --rules cakephp50 /path/to/app
    ```

??? question "CakePHP 5 tem suporte comercial?"

    Sim! Você pode obter:

    - **Suporte Oficial:** Contrate desenvolvedores core via [CakeDC](https://www.cakedc.com/)
    - **Treinamento:** Cursos oficiais e workshops
    - **Consultoria:** Arquitetura, code review e otimizações

    Para projetos de código aberto e aprendizado, a comunidade no Slack e Fórum é muito ativa e responsiva.

---

## 🔖 Compatibilidade de Versões

| Versão CakePHP | PHP Mínimo | Suporte | EOL |
|----------------|------------|---------|-----|
| 5.2.x (atual) | 8.1 | ✅ Ativa | - |
| 5.1.x | 8.1 | ✅ Ativa | - |
| 5.0.x | 8.1 | ⚠️ Segurança apenas | Jun 2025 |
| 4.5.x | 7.4 | ⚠️ Segurança apenas | Dez 2025 |
| 4.4.x | 7.4 | ❌ EOL | Dez 2024 |
| 3.x | 5.6 | ❌ EOL | Dez 2021 |

!!! warning "Versões Antigas"
    Se você ainda usa CakePHP 3.x ou 4.4.x, considere atualizar para receber correções de segurança e novos recursos.

---

## 📝 Contribua com a Documentação

Esta documentação é **aberta e colaborativa**! Você pode contribuir de várias formas:

1. **Correção de erros:** Encontrou um typo ou erro técnico? Clique no ícone de edição e sugira uma correção.
2. **Novos exemplos:** Adicione exemplos de código que ajudaram você.
3. **Tradução:** Ajude a traduzir a documentação para outros idiomas.
4. **Tutoriais:** Escreva guias sobre casos de uso específicos.

[:fontawesome-brands-github: Repositório da Documentação](https://github.com/cakephp/docs){ .md-button }

---

## 🎉 Pronto para Começar?

Escolha seu próximo passo:

<div class="grid cards" markdown>

-   :material-flash:{ .lg .middle } __Tutorial Rápido__

    ---

    Crie sua primeira aplicação CakePHP em 30 minutos!

    [:octicons-arrow-right-24: Começar Quickstart](quickstart.md)

-   :material-compass:{ .lg .middle } __Explorar Conceitos__

    ---

    Entenda a filosofia e os princípios do CakePHP.

    [:octicons-arrow-right-24: CakePHP em Resumo](intro.md)

-   :material-api:{ .lg .middle } __Referência da API__

    ---

    Documentação técnica completa de classes e métodos.

    [:octicons-arrow-right-24: API Reference](https://api.cakephp.org/5.0/)

-   :material-download:{ .lg .middle } __Instalar Agora__

    ---

    Configure seu ambiente e instale o CakePHP.

    ```bash
    composer create-project cakephp/app my_app
    ```

</div>

---

<div class="center" markdown>

**Bem-vindo à comunidade CakePHP! 🍰**

_Feliz codificação!_

</div>

---

!!! note "Sobre esta Documentação"
    **Versão:** CakePHP 5.x
    **Última atualização:** 2025-11-15
    **Licença:** MIT
    **Contribuidores:** Comunidade CakePHP

