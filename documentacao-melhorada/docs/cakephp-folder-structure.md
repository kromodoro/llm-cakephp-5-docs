# Estrutura de Pastas do CakePHP

Após instalar o skeleton do CakePHP via Composer, você verá uma estrutura de diretórios organizada seguindo convenções do framework. Esta página explica cada pasta e arquivo, ajudando você a navegar e entender onde colocar seu código.

---

## 📁 Visualização Geral

```bash title="Estrutura do CakePHP 5 Skeleton"
myapp/
├── bin/                # Executáveis do console (cake)
├── config/             # Configurações (database, routes, app)
├── logs/               # Arquivos de log (criado em runtime)
├── plugins/            # Plugins da aplicação
├── resources/          # Recursos (locales para i18n)
├── src/                # Código-fonte da aplicação
├── templates/          # Views, layouts, elements
├── tests/              # Testes automatizados
├── tmp/                # Arquivos temporários e cache
├── vendor/             # Dependências do Composer
├── webroot/            # Document root público
├── composer.json       # Configuração de dependências
├── composer.lock       # Versões exatas instaladas
├── .gitignore          # Arquivos ignorados pelo Git
├── .htaccess           # Regras Apache (redirect to webroot)
└── index.php           # Entry point alternativo
```

!!! tip "Visualizar com Tree"
    Para ver a estrutura completa no terminal:
    ```bash
    tree -L 2 -F --dirsfirst -I 'vendor'
    ```

---

## 📂 Diretórios Principais

### bin/ - Executáveis do Console

Contém os executáveis do CakePHP Console (bake, migrations, etc).

**Conteúdo:**
```bash
bin/
├── cake                # Script executável Unix/Linux/macOS
├── cake.bat            # Script executável Windows
├── cake.php            # Núcleo do console
└── bash_completion.sh  # Autocomplete para bash
```

**Uso:**
```bash
# Unix/Linux/macOS
bin/cake bake model Users

# Windows
bin\cake.bat bake model Users
```

---

### config/ - Configurações

Armazena **todos** os arquivos de configuração do CakePHP.

**Arquivos principais:**
```bash
config/
├── app.php                 # Configuração principal da aplicação
├── app_local.example.php   # Template para config local (não versionado)
├── bootstrap.php           # Código executado na inicialização
├── paths.php               # Definição de caminhos de diretórios
├── routes.php              # Definição de rotas
├── plugins.php             # Carregamento de plugins
└── schema/                 # Schema SQL para desenvolvimento
```

**Veja também:**
📖 [Configuration Guide](/development/configuration)

!!! warning "Nunca Versione Senhas"
    Crie `config/app_local.php` para configurações específicas do ambiente (database, etc).
    Este arquivo está no `.gitignore` por padrão.

**Exemplo app_local.php:**
```php
<?php
return [
    'Datasources' => [
        'default' => [
            'host' => 'localhost',
            'username' => 'meu_usuario',
            'password' => 'senha_segura',  // ⚠️ NÃO versione!
            'database' => 'minha_base',
        ],
    ],
];
```

---

### logs/ - Arquivos de Log

!!! info "Criado Automaticamente"
    O diretório `logs/` é criado automaticamente pelo CakePHP no primeiro acesso quando há algo a ser registrado. **Não precisa ser criado manualmente**.

**Conteúdo típico:**
```bash
logs/
├── error.log       # Erros e exceptions
├── debug.log       # Logs de debug
└── queries.log     # Queries SQL (se habilitado)
```

!!! danger "Permissões Obrigatórias"
    O servidor web **DEVE** ter permissão de escrita:

    === "Ubuntu/Debian"
        ```bash
        sudo chown -R www-data:www-data logs/
        sudo chmod -R 0775 logs/
        ```

    === "CentOS/RHEL"
        ```bash
        sudo chown -R apache:apache logs/
        sudo chmod -R 0775 logs/
        ```

    === "macOS (dev)"
        ```bash
        chmod -R 0777 logs/
        ```

**Performance:**
Sem permissões corretas, a performance da aplicação será **severamente impactada**. Em debug mode, o CakePHP exibirá warnings se os diretórios não forem writable.

---

### plugins/ - Plugins da Aplicação

Armazena plugins instalados localmente (não via Composer).

```bash
plugins/
└── MeuPlugin/
    ├── src/
    ├── templates/
    ├── config/
    └── README.md
```

**Veja também:**
📖 [Plugins Guide](/plugins)

!!! tip "Plugins via Composer"
    A maioria dos plugins modernos é instalada via Composer e fica em `vendor/`, não em `plugins/`.

---

### resources/ - Recursos da Aplicação

Armazena arquivos de recursos como traduções (i18n).

**Estrutura padrão:**
```bash
resources/
└── locales/
    ├── pt_BR/
    │   └── default.po
    ├── en_US/
    │   └── default.po
    └── es_ES/
        └── default.po
```

**Exemplo de tradução (pt_BR/default.po):**
```po
msgid "Welcome"
msgstr "Bem-vindo"

msgid "Hello {0}"
msgstr "Olá {0}"
```

---

### src/ - Código-Fonte da Aplicação

O diretório **mais importante** - onde você desenvolverá a maior parte do seu código.

```bash
src/
├── Application.php       # Classe principal da aplicação
├── Command/              # ⚠️ Não existe por padrão (criar quando necessário)
├── Console/              # Scripts de instalação (Composer)
├── Controller/           # Controllers da aplicação
│   ├── AppController.php
│   └── Component/        # Components reutilizáveis
├── Middleware/           # ⚠️ Não existe por padrão (criar quando necessário)
├── Model/                # Camada de dados (ORM)
│   ├── Behavior/         # Behaviors
│   ├── Entity/           # Entities
│   └── Table/            # Table classes
└── View/                 # Camada de apresentação
    ├── Cell/             # View cells
    └── Helper/           # Helpers
```

!!! note "Diretórios Opcionais"
    **Command/** e **Middleware/** não existem por padrão.
    Crie-os quando precisar de console commands ou middleware customizados.

**Criando Command/**:
```bash
mkdir -p src/Command
bin/cake bake command Email
```

**Criando Middleware/**:
```bash
mkdir -p src/Middleware
```

#### Subdire tórios de src/

| Diretório | Propósito | Link |
|-----------|-----------|------|
| **Command/** | Console commands customizados | [Commands Guide](/console-commands/commands) |
| **Console/** | Installation scripts (Composer) | - |
| **Controller/** | Controllers + Components | [Controllers Guide](/controllers) |
| **Middleware/** | Application middleware | [Middleware Guide](/controllers/middleware) |
| **Model/** | Tables, Entities, Behaviors | [ORM Guide](/orm) |
| **View/** | Views, Cells, Helpers | [Views Guide](/views) |

---

### templates/ - Views e Layouts

Contém **todos** os arquivos de apresentação (HTML/PHP).

**Estrutura padrão:**
```bash
templates/
├── layout/             # Layouts da aplicação
│   ├── default.php     # Layout padrão
│   ├── error.php       # Layout de erros
│   └── ajax.php        # Layout para Ajax
├── element/            # Elementos reutilizáveis
│   ├── header.php
│   └── footer.php
├── email/              # Templates de email
│   ├── html/
│   └── text/
├── Error/              # Páginas de erro
│   ├── error400.php
│   └── error500.php
├── Pages/              # Páginas estáticas (PagesController)
│   └── home.php
└── cell/               # Templates de View Cells
```

**Exemplo de estrutura por controller:**
```bash
templates/
└── Users/              # Templates do UsersController
    ├── index.php       # UsersController::index()
    ├── view.php        # UsersController::view()
    ├── add.php         # UsersController::add()
    └── edit.php        # UsersController::edit()
```

**Convenção:**
Action `viewProfile()` → template `templates/Users/view_profile.php` (snake_case)

---

### tests/ - Testes Automatizados

Contém os test cases da aplicação.

**Estrutura:**
```bash
tests/
├── bootstrap.php           # Bootstrap dos testes
├── schema.sql              # Schema de teste
├── Fixture/                # Dados de teste (fixtures)
│   ├── ArticlesFixture.php
│   └── UsersFixture.php
└── TestCase/               # Test cases
    ├── Controller/
    │   └── ArticlesControllerTest.php
    ├── Model/
    │   └── Table/
    │       └── ArticlesTableTest.php
    └── View/
        └── Helper/
            └── MyHelperTest.php
```

**Rodando testes:**
```bash
# Todos os testes
vendor/bin/phpunit

# Teste específico
vendor/bin/phpunit tests/TestCase/Model/Table/ArticlesTableTest.php
```

---

### tmp/ - Dados Temporários e Cache

Armazena cache, sessões e outros dados temporários.

**Estrutura padrão:**
```bash
tmp/
├── cache/              # Cache de metadata, queries, views
│   ├── models/
│   ├── persistent/
│   └── views/
├── sessions/           # Sessões PHP (se configurado)
└── tests/              # Dados temporários dos testes
    └── debug_kit.sqlite
```

!!! danger "Permissões Críticas"
    O diretório `tmp/` **DEVE ser writable** pelo servidor web:

    === "Ubuntu/Debian"
        ```bash
        sudo chown -R www-data:www-data tmp/
        sudo chmod -R 0775 tmp/
        ```

    === "CentOS/RHEL"
        ```bash
        sudo chown -R apache:apache tmp/
        sudo chmod -R 0775 tmp/
        ```

    === "macOS (dev)"
        ```bash
        chmod -R 0777 tmp/
        ```

**Performance:**
Sem permissões corretas, a performance será **severamente impactada**. CakePHP usa `tmp/` intensivamente para cache de metadata do ORM.

**Limpando cache:**
```bash
bin/cake cache clear_all
```

---

### vendor/ - Dependências do Composer

Contém **todas** as bibliotecas instaladas pelo Composer (incluindo o próprio CakePHP).

**Estrutura:**
```bash
vendor/
├── cakephp/
│   ├── cakephp/        # Framework core
│   ├── bake/           # Code generation
│   └── debug_kit/      # Debug toolbar
├── autoload.php        # Autoloader do Composer
├── composer/           # Metadados do Composer
└── [outras bibliotecas]
```

!!! danger "NUNCA Edite vendor/"
    O Composer **sobrescreverá suas alterações** no próximo `composer update`.

    **Se precisar modificar uma biblioteca:**
    ```bash
    # Use patches via composer.json
    {
        "extra": {
            "patches": {
                "vendor/package": {
                    "Fix bug": "patches/fix-bug.patch"
                }
            }
        }
    }
    ```

---

### webroot/ - Pasta Pública (Document Root)

O **único diretório** acessível publicamente pela web.

**Estrutura padrão:**
```bash
webroot/
├── index.php           # Entry point da aplicação
├── .htaccess           # Regras de rewrite (Apache)
├── favicon.ico         # Ícone do site
├── css/                # Arquivos CSS
│   └── cake.css
├── js/                 # Arquivos JavaScript
│   └── empty
├── img/                # Imagens
│   └── cake-logo.png
└── font/               # Fontes customizadas
```

!!! tip "Configuração de Servidor"
    O document root do servidor **DEVE** apontar para `webroot/`, **NÃO** para a raiz do projeto.

    === "Apache (VirtualHost)"
        ```apache
        <VirtualHost *:80>
            ServerName myapp.local
            DocumentRoot /var/www/myapp/webroot

            <Directory /var/www/myapp/webroot>
                Options Indexes FollowSymLinks
                AllowOverride All
                Require all granted
            </Directory>
        </VirtualHost>
        ```

    === "nginx"
        ```nginx
        server {
            listen 80;
            server_name myapp.local;
            root /var/www/myapp/webroot;
            index index.php;

            location / {
                try_files $uri $uri/ /index.php?$args;
            }

            location ~ \.php$ {
                fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
                fastcgi_index index.php;
                include fastcgi_params;
            }
        }
        ```

**Uploads de usuários:**
```bash
webroot/
└── uploads/            # Criar para uploads
    ├── avatars/
    ├── documents/
    └── .gitkeep        # Versionar pasta vazia
```

---

## 📄 Arquivos de Configuração Raiz

### composer.json

Define as **dependências** do projeto e configurações do Composer.

**Exemplo:**
```json
{
    "name": "minha-empresa/meu-app",
    "description": "Aplicação CakePHP 5",
    "type": "project",
    "require": {
        "php": ">=8.1",
        "cakephp/cakephp": "^5.0",
        "cakephp/migrations": "^4.0",
        "cakephp/plugin-installer": "^2.0"
    },
    "require-dev": {
        "cakephp/bake": "^3.0",
        "cakephp/debug_kit": "^5.0",
        "phpunit/phpunit": "^10.0"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

**Comandos úteis:**
```bash
# Instalar dependências
composer install

# Adicionar nova dependência
composer require vendor/package

# Atualizar dependências
composer update
```

---

### composer.lock

Arquivo **gerado automaticamente** pelo Composer contendo as versões **exatas** de todas as dependências instaladas.

!!! success "Sempre Versione"
    O `composer.lock` **DEVE** ser versionado no Git para garantir que todos os ambientes usem as mesmas versões.

---

### .gitignore

Define quais arquivos/diretórios **não** versionar no Git.

**Padrão do CakePHP:**
```gitignore
# CakePHP
/config/app_local.php       # Configurações locais (senhas!)
/config/.env                # Variáveis de ambiente
/logs/*                     # Logs
/tmp/*                      # Cache e temporários
/vendor/*                   # Dependências (Composer)

# OS
.DS_Store                   # macOS
Thumbs.db                   # Windows

# IDE
.idea/                      # PHPStorm
.vscode/                    # VSCode
*.swp                       # Vim

# Testing
.phpunit.cache
tests.sqlite

# Node (se usar)
/node_modules/
```

!!! tip "Uploads de Usuários"
    Adicione uploads ao `.gitignore`:
    ```gitignore
    /webroot/uploads/*
    !/webroot/uploads/.gitkeep
    ```

---

### .htaccess (raiz)

Redireciona todas as requisições para `webroot/` (apenas Apache).

**Conteúdo:**
```apache
<IfModule mod_rewrite.c>
    RewriteEngine on
    RewriteRule ^$ webroot/ [L]
    RewriteRule (.*) webroot/$1 [L]
</IfModule>
```

**Importante:**
Para nginx, este arquivo é **ignorado** (nginx não usa .htaccess).

---

## 🗺️ Onde Colocar Cada Arquivo?

### Tabela de Referência Rápida

| Tipo de Arquivo | Diretório Correto | Exemplo |
|-----------------|-------------------|---------|
| Controller personalizado | `src/Controller/` | `UsersController.php` |
| Entity | `src/Model/Entity/` | `User.php` |
| Table class | `src/Model/Table/` | `UsersTable.php` |
| Behavior | `src/Model/Behavior/` | `SluggableBehavior.php` |
| Template/View | `templates/{Controller}/` | `templates/Users/index.php` |
| Layout | `templates/layout/` | `default.php` |
| Element | `templates/element/` | `header.php` |
| Email template (HTML) | `templates/email/html/` | `welcome.php` |
| Email template (text) | `templates/email/text/` | `welcome.php` |
| Helper | `src/View/Helper/` | `CurrencyHelper.php` |
| View Cell | `src/View/Cell/` | `PopularArticlesCell.php` |
| Cell template | `templates/cell/PopularArticles/` | `display.php` |
| Component | `src/Controller/Component/` | `AuthComponent.php` |
| Middleware | `src/Middleware/` | `CorsMiddleware.php` |
| Console Command | `src/Command/` | `EmailCommand.php` |
| Arquivo CSS | `webroot/css/` | `style.css` |
| Arquivo JavaScript | `webroot/js/` | `app.js` |
| Imagem | `webroot/img/` | `logo.png` |
| Upload de usuário | `webroot/uploads/` | `avatar.jpg` |
| Tradução (i18n) | `resources/locales/pt_BR/` | `default.po` |
| Test case (Controller) | `tests/TestCase/Controller/` | `UsersControllerTest.php` |
| Test case (Table) | `tests/TestCase/Model/Table/` | `UsersTableTest.php` |
| Fixture | `tests/Fixture/` | `UsersFixture.php` |
| Configuração customizada | `config/` | `permissions.php` |
| Schema SQL | `config/schema/` | `initial.sql` |

---

## 🔧 Comparação com Outros Frameworks

### Laravel vs CakePHP

| Recurso | Laravel | CakePHP |
|---------|---------|---------|
| **Document root** | `public/` | `webroot/` |
| **Código-fonte** | `app/` | `src/` |
| **Views** | `resources/views/` | `templates/` |
| **Configuração** | `config/` | `config/` ✅ |
| **Rotas** | `routes/web.php` | `config/routes.php` |
| **Controllers** | `app/Http/Controllers/` | `src/Controller/` |
| **Models** | `app/Models/` | `src/Model/Entity/` + `Table/` |
| **Migrations** | `database/migrations/` | `config/Migrations/` (plugin) |
| **Assets compilados** | `public/css/`, `public/js/` | `webroot/css/`, `webroot/js/` |

### Symfony vs CakePHP

| Recurso | Symfony | CakePHP |
|---------|---------|---------|
| **Document root** | `public/` | `webroot/` |
| **Código-fonte** | `src/` | `src/` ✅ |
| **Templates** | `templates/` | `templates/` ✅ |
| **Configuração** | `config/` | `config/` ✅ |
| **Controllers** | `src/Controller/` | `src/Controller/` ✅ |
| **Entities** | `src/Entity/` | `src/Model/Entity/` |

---

## 🚀 Checklist Pós-Instalação

Use este checklist após instalar o CakePHP para verificar se tudo está configurado corretamente:

### Permissões

- [ ] `tmp/` é writable pelo servidor web
- [ ] `logs/` será criado automaticamente (ou criar com permissões 0775)
- [ ] `webroot/uploads/` é writable (se usar uploads)

### Configuração

- [ ] `config/app_local.php` criado com credenciais de database
- [ ] `.env` configurado (se usar variáveis de ambiente)
- [ ] Timezone configurado em `config/app.php`
- [ ] Locale configurado (pt_BR se necessário)

### Servidor Web

- [ ] Document root aponta para `webroot/`
- [ ] mod_rewrite ativo (Apache) ou try_files configurado (nginx)
- [ ] PHP 8.1+ instalado
- [ ] Extensões PHP necessárias ativas (intl, mbstring, pdo_mysql)

### Git

- [ ] `.gitignore` verificado
- [ ] `composer.lock` versionado
- [ ] `config/app_local.php` NÃO versionado
- [ ] `vendor/` NÃO versionado

### Testes

- [ ] `vendor/bin/phpunit` executando
- [ ] Acesso a `http://localhost:8765/` funcionando
- [ ] Debug mode ativo em desenvolvimento

---

## 🐳 Adaptação para Docker

Se você usa Docker, a estrutura pode ser adaptada com volumes:

**docker-compose.yml exemplo:**
```yaml
version: '3.8'

services:
  app:
    image: php:8.2-fpm
    volumes:
      - ./:/var/www/html
      - ./tmp:/var/www/html/tmp          # Volume para cache
      - ./logs:/var/www/html/logs        # Volume para logs
    working_dir: /var/www/html

  nginx:
    image: nginx:alpine
    volumes:
      - ./:/var/www/html
      - ./docker/nginx.conf:/etc/nginx/conf.d/default.conf
    ports:
      - "8080:80"
    depends_on:
      - app
```

**Permissões em Docker:**
```bash
# Dentro do container
chown -R www-data:www-data tmp logs
chmod -R 0775 tmp logs
```

---

## 🇧🇷 Hostings Brasileiros (cPanel/Plesk)

Em shared hostings, a estrutura precisa ser adaptada:

### cPanel (Locaweb, HostGator BR, UOL Host)

**Estrutura típica:**
```bash
/home/usuario/
├── public_html/        # ⚠️ Document root (mapear para webroot/)
│   ├── .htaccess       # Copiar de webroot/.htaccess
│   ├── index.php       # Copiar de webroot/index.php
│   ├── css/
│   ├── js/
│   └── img/
└── myapp/              # Projeto CakePHP (FORA de public_html)
    ├── bin/
    ├── config/
    ├── src/
    ├── templates/
    ├── tmp/            # ⚠️ Configurar permissões 0775
    └── vendor/
```

**Ajustar webroot/index.php:**
```php
<?php
// Ajustar caminho
define('ROOT', dirname(__DIR__, 2) . '/myapp');
define('APP_DIR', 'src');
define('WEBROOT_DIR', basename(__DIR__));

// ... resto do arquivo
```

---

## ❓ FAQ

??? question "Por que logs/ não existe após instalação?"
    O diretório `logs/` é criado **automaticamente** pelo CakePHP no primeiro acesso quando há algo a ser registrado. Não é necessário criá-lo manualmente.

    Se quiser criar antecipadamente:
    ```bash
    mkdir -p logs
    chmod 0775 logs
    ```

??? question "Posso renomear src/ para app/?"
    Não é recomendado. CakePHP segue PSR-4 e `src/` é o padrão moderno do PHP. Renomear exigiria ajustar:
    - `composer.json` (autoload)
    - `config/paths.php`
    - Configurações do servidor web

??? question "Onde colocar bibliotecas de terceiros não-Composer?"
    Instale via Composer sempre que possível. Se não houver pacote:
    ```bash
    vendor/
    └── minha-empresa/
        └── minha-lib/
    ```
    Ou crie um repositório Composer local.

??? question "Como organizar uploads de usuários?"
    Crie `webroot/uploads/` com subdiretórios por tipo:
    ```bash
    webroot/uploads/
    ├── avatars/
    ├── documents/
    └── images/
    ```
    Configure permissões 0775 e adicione ao `.gitignore`.

??? question "tmp/ pode ficar cheio e travar o servidor?"
    CakePHP gerencia o cache automaticamente. Para limpar manualmente:
    ```bash
    bin/cake cache clear_all
    ```
    Em produção, configure limpeza periódica via cron:
    ```cron
    0 3 * * * cd /var/www/myapp && bin/cake cache clear_all
    ```

??? question "Middleware/ não existe, está faltando algo?"
    Não! Similar a `Command/`, `Middleware/` não é criado por padrão. Crie apenas se precisar de middleware customizado:
    ```bash
    mkdir -p src/Middleware
    ```

??? question "Posso ter múltiplas aplicações compartilhando vendor/?"
    Sim, mas não é recomendado em produção:
    ```bash
    /var/www/
    ├── vendor/            # Compartilhado
    ├── app1/
    │   └── vendor -> ../vendor
    └── app2/
        └── vendor -> ../vendor
    ```
    Problemas: conflitos de versões entre apps.

---

## 📚 Recursos Relacionados

<div class="grid cards" markdown>

-   :material-cog: **Configuração**

    ---

    Aprenda a configurar database, cache, email e mais

    [:octicons-arrow-right-24: Configuration Guide](/development/configuration)

-   :material-routes: **Rotas**

    ---

    Defina rotas customizadas em `config/routes.php`

    [:octicons-arrow-right-24: Routing Guide](/development/routing)

-   :material-controller: **Controllers**

    ---

    Crie controllers em `src/Controller/`

    [:octicons-arrow-right-24: Controllers Guide](/controllers)

-   :material-database: **Models**

    ---

    Trabalhe com Models em `src/Model/`

    [:octicons-arrow-right-24: ORM Guide](/orm)

-   :material-eye: **Views**

    ---

    Organize templates em `templates/`

    [:octicons-arrow-right-24: Views Guide](/views)

-   :material-puzzle: **Plugins**

    ---

    Instale e crie plugins

    [:octicons-arrow-right-24: Plugins Guide](/plugins)

</div>

---

## 🔍 Troubleshooting

### "Class 'App\Controller\MyController' not found"

**Causa:** Arquivo não está no diretório correto ou nome da classe não corresponde ao nome do arquivo.

**Solução:**
```bash
# Verificar estrutura
src/Controller/MyController.php  # ✅ Correto
src/controller/MyController.php  # ❌ Errado (case-sensitive)
src/MyController.php             # ❌ Errado (fora de Controller/)
```

---

### "Unable to write to tmp/ directory"

**Causa:** Permissões incorretas.

**Solução:**
```bash
# Verificar proprietário
ls -la tmp/

# Corrigir
sudo chown -R www-data:www-data tmp/
sudo chmod -R 0775 tmp/
```

---

### "webroot/ assets não carregam (404)"

**Causa:** Document root não aponta para `webroot/`.

**Solução Apache:**
```apache
DocumentRoot /var/www/myapp/webroot  # ✅ Correto
DocumentRoot /var/www/myapp          # ❌ Errado
```

**Solução nginx:**
```nginx
root /var/www/myapp/webroot;  # ✅ Correto
root /var/www/myapp;          # ❌ Errado
```

---

### "Composer alterações perdidas após update"

**Causa:** Editou arquivos em `vendor/` diretamente.

**Solução:** Use patches:
```json
{
    "require": {
        "cweagans/composer-patches": "^1.7"
    },
    "extra": {
        "patches": {
            "vendor/package": {
                "Descrição": "patches/fix.patch"
            }
        }
    }
}
```

---

**Última atualização:** 2025-11-15
**CakePHP:** 5.2+
**Skeleton:** cakephp/app ^5.0
