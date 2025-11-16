# Tutorial CMS - Criando o Banco de Dados

!!! info "Parte 2 do Tutorial CMS"
    Esta é a segunda parte do [Tutorial CMS](../tutorials-and-examples.md). Certifique-se de ter completado a [Parte 1: Instalação](cms-installation.md) antes de continuar.

---

## 📋 O Que Você Vai Aprender

Neste tutorial, você aprenderá:

- ✅ Criar banco de dados MySQL, PostgreSQL ou SQLite
- ✅ Projetar schema com relacionamentos (1:N, N:N)
- ✅ Usar convenções do CakePHP para nomes de tabelas
- ✅ Configurar conexão com banco em `app_local.php`
- ✅ Usar Migrations para versionamento de schema
- ✅ Popular banco com Seeds (dados iniciais)
- ✅ Entender UTF-8 (utf8mb4) e índices

---

## 🎯 Estrutura do Banco de Dados

Nosso CMS terá 4 tabelas com os seguintes relacionamentos:

```mermaid
erDiagram
    USERS ||--o{ ARTICLES : "escreve"
    ARTICLES ||--o{ ARTICLES_TAGS : "possui"
    TAGS ||--o{ ARTICLES_TAGS : "categoriza"

    USERS {
        int id PK
        string email
        string password
        datetime created
        datetime modified
    }

    ARTICLES {
        int id PK
        int user_id FK
        string title
        string slug UK "URL-friendly"
        text body
        boolean published
        datetime created
        datetime modified
    }

    TAGS {
        int id PK
        string title UK
        datetime created
        datetime modified
    }

    ARTICLES_TAGS {
        int article_id PK,FK
        int tag_id PK,FK
    }
```

!!! tip "Relacionamentos"
    - **1:N** - Um usuário escreve vários artigos
    - **N:N** - Um artigo tem várias tags, uma tag está em vários artigos
    - **Junction Table** - `articles_tags` implementa relacionamento N:N

---

## 🗄️ Escolhendo o Banco de Dados

=== "MySQL 8.0+ (Recomendado)"

    ### ✅ Vantagens
    - Mais usado em produção (Locaweb, Hostgator, AWS RDS)
    - Melhor performance para leitura
    - Comunidade gigante

    ### 📦 Instalação

    **Ubuntu/Debian:**
    ```bash
    sudo apt update
    sudo apt install mysql-server
    sudo mysql_secure_installation
    ```

    **macOS (Homebrew):**
    ```bash
    brew install mysql
    brew services start mysql
    ```

    **Windows (XAMPP):**
    - Baixar [XAMPP](https://www.apachefriends.org/pt_br/index.html)
    - Iniciar MySQL no painel de controle

    ### 🔧 Verificar Instalação
    ```bash
    mysql --version
    # mysql  Ver 8.0.35 for Linux on x86_64
    ```

=== "PostgreSQL 14+"

    ### ✅ Vantagens
    - Melhor para dados complexos (JSON, arrays)
    - Transações mais robustas
    - Open source 100%

    ### 📦 Instalação

    **Ubuntu/Debian:**
    ```bash
    sudo apt update
    sudo apt install postgresql postgresql-contrib
    sudo systemctl start postgresql
    ```

    **macOS (Homebrew):**
    ```bash
    brew install postgresql@14
    brew services start postgresql@14
    ```

    ### 🔧 Verificar Instalação
    ```bash
    psql --version
    # psql (PostgreSQL) 14.10
    ```

=== "SQLite (Desenvolvimento)"

    ### ✅ Vantagens
    - Zero configuração (arquivo único)
    - Perfeito para testes e protótipos
    - Não precisa de servidor

    ### 📦 Instalação

    **Já vem no PHP 8.1+!**
    ```bash
    php -m | grep pdo_sqlite
    # pdo_sqlite
    ```

    ### ⚠️ Limitações
    - Não recomendado para produção
    - Sem controle de acesso (permissões por arquivo)
    - Performance inferior em concorrência

---

## 📊 Criando o Banco de Dados

### Etapa 1: Criar Database

=== "MySQL"

    **Opção 1: Via linha de comando**
    ```bash
    mysql -u root -p
    ```

    ```sql
    CREATE DATABASE cake_cms CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    SHOW DATABASES;
    EXIT;
    ```

    **Opção 2: Via phpMyAdmin**
    - Acessar [http://localhost/phpmyadmin](http://localhost/phpmyadmin)
    - Aba "Databases" → Criar database `cake_cms`
    - Collation: `utf8mb4_unicode_ci`

=== "PostgreSQL"

    ```bash
    sudo -u postgres psql
    ```

    ```sql
    CREATE DATABASE cake_cms WITH ENCODING 'UTF8' LC_COLLATE='pt_BR.UTF-8' LC_CTYPE='pt_BR.UTF-8';
    \l
    \q
    ```

=== "SQLite"

    **Nenhum comando necessário!**

    O arquivo será criado automaticamente em:
    ```
    /home/kromodoro/Documentos/cakephp/app/tmp/cake_cms.sqlite
    ```

---

### Etapa 2: Criar Tabelas

=== "MySQL"

    ```sql
    USE cake_cms;

    -- Tabela de Usuários
    CREATE TABLE users (
        id INT AUTO_INCREMENT PRIMARY KEY,
        email VARCHAR(255) NOT NULL UNIQUE,
        password VARCHAR(255) NOT NULL,
        created DATETIME,
        modified DATETIME,
        INDEX idx_email (email)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

    -- Tabela de Artigos
    CREATE TABLE articles (
        id INT AUTO_INCREMENT PRIMARY KEY,
        user_id INT NOT NULL,
        title VARCHAR(255) NOT NULL,
        slug VARCHAR(191) NOT NULL,
        body TEXT,
        published BOOLEAN DEFAULT FALSE,
        created DATETIME,
        modified DATETIME,
        UNIQUE KEY uk_slug (slug),
        INDEX idx_published (published),
        INDEX idx_user_id (user_id),
        FOREIGN KEY fk_user (user_id)
            REFERENCES users(id)
            ON DELETE CASCADE
            ON UPDATE CASCADE
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

    -- Tabela de Tags
    CREATE TABLE tags (
        id INT AUTO_INCREMENT PRIMARY KEY,
        title VARCHAR(191) NOT NULL,
        created DATETIME,
        modified DATETIME,
        UNIQUE KEY uk_title (title)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

    -- Tabela de Junção (N:N)
    CREATE TABLE articles_tags (
        article_id INT NOT NULL,
        tag_id INT NOT NULL,
        PRIMARY KEY (article_id, tag_id),
        FOREIGN KEY fk_article (article_id)
            REFERENCES articles(id)
            ON DELETE CASCADE,
        FOREIGN KEY fk_tag (tag_id)
            REFERENCES tags(id)
            ON DELETE CASCADE
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
    ```

=== "PostgreSQL"

    ```sql
    \c cake_cms;

    -- Tabela de Usuários
    CREATE TABLE users (
        id SERIAL PRIMARY KEY,
        email VARCHAR(255) NOT NULL UNIQUE,
        password VARCHAR(255) NOT NULL,
        created TIMESTAMP,
        modified TIMESTAMP
    );
    CREATE INDEX idx_users_email ON users(email);

    -- Tabela de Artigos
    CREATE TABLE articles (
        id SERIAL PRIMARY KEY,
        user_id INT NOT NULL,
        title VARCHAR(255) NOT NULL,
        slug VARCHAR(191) NOT NULL UNIQUE,
        body TEXT,
        published BOOLEAN DEFAULT FALSE,
        created TIMESTAMP,
        modified TIMESTAMP,
        FOREIGN KEY (user_id)
            REFERENCES users(id)
            ON DELETE CASCADE
            ON UPDATE CASCADE
    );
    CREATE INDEX idx_articles_published ON articles(published);
    CREATE INDEX idx_articles_user_id ON articles(user_id);

    -- Tabela de Tags
    CREATE TABLE tags (
        id SERIAL PRIMARY KEY,
        title VARCHAR(191) NOT NULL UNIQUE,
        created TIMESTAMP,
        modified TIMESTAMP
    );

    -- Tabela de Junção
    CREATE TABLE articles_tags (
        article_id INT NOT NULL,
        tag_id INT NOT NULL,
        PRIMARY KEY (article_id, tag_id),
        FOREIGN KEY (article_id) REFERENCES articles(id) ON DELETE CASCADE,
        FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE
    );
    ```

=== "SQLite"

    ```sql
    -- SQLite cria o arquivo automaticamente
    -- Executar via PHP:

    CREATE TABLE users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        email TEXT NOT NULL UNIQUE,
        password TEXT NOT NULL,
        created TEXT,
        modified TEXT
    );

    CREATE TABLE articles (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        user_id INTEGER NOT NULL,
        title TEXT NOT NULL,
        slug TEXT NOT NULL UNIQUE,
        body TEXT,
        published INTEGER DEFAULT 0,
        created TEXT,
        modified TEXT,
        FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
    );

    CREATE TABLE tags (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL UNIQUE,
        created TEXT,
        modified TEXT
    );

    CREATE TABLE articles_tags (
        article_id INTEGER NOT NULL,
        tag_id INTEGER NOT NULL,
        PRIMARY KEY (article_id, tag_id),
        FOREIGN KEY (article_id) REFERENCES articles(id) ON DELETE CASCADE,
        FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE
    );
    ```

---

## 🔍 Entendendo as Decisões de Design

### Por que VARCHAR(191) ao invés de VARCHAR(255)?

!!! warning "Limite de Índice MySQL"
    MySQL/MariaDB com `utf8mb4` usa **4 bytes por caractere**.

    **Limite do índice:** 767 bytes (InnoDB padrão)

    **Cálculo:**
    ```
    767 bytes ÷ 4 bytes/char = 191.75 caracteres
    ```

    **Solução:** Usar `VARCHAR(191)` em campos com `UNIQUE KEY` ou índices.

**Campos afetados:**
- ✅ `articles.slug` - URL única (ex: `meu-primeiro-artigo`)
- ✅ `tags.title` - Nome da tag único (ex: `cakephp`)

**Campos que podem usar VARCHAR(255):**
- ✅ `users.email` - Tem índice, mas 191 é suficiente
- ✅ `articles.title` - Sem índice único, pode ser 255

---

### Por que CHARSET=utf8mb4?

!!! success "Suporte Total a UTF-8"
    - **utf8mb4** = UTF-8 de 4 bytes
    - **utf8** (antigo) = UTF-8 de 3 bytes (INCOMPLETO!)

**O que utf8mb4 suporta:**
- ✅ Emojis (😀 🎉 💻 🚀)
- ✅ Caracteres especiais (汉字, العربية, हिन्दी)
- ✅ Símbolos matemáticos (∑ ∫ √ π)
- ✅ Acentuação completa (é, ñ, ç, ü)

**Exemplo prático:**
```sql
INSERT INTO articles (title, slug, body) VALUES (
    'Lançamento CakePHP 5.0 🎉',
    'lancamento-cakephp-5',
    'Novo recurso incrível! 🚀'
);
```

---

### Por que Composite Primary Key?

!!! tip "Chave Primária Composta"
    A tabela `articles_tags` usa **2 campos como chave primária**:
    ```sql
    PRIMARY KEY (article_id, tag_id)
    ```

**Vantagens:**
- ✅ **Evita duplicatas** - Mesmo artigo não pode ter mesma tag 2 vezes
- ✅ **Performance** - Índice automático nas 2 colunas
- ✅ **Schema simples** - Não precisa de coluna `id` extra

**Exemplo de uso:**
```sql
-- ✅ Permitido
INSERT INTO articles_tags VALUES (1, 1); -- Artigo 1 tem Tag 1
INSERT INTO articles_tags VALUES (1, 2); -- Artigo 1 tem Tag 2

-- ❌ ERRO: Duplicate entry
INSERT INTO articles_tags VALUES (1, 1); -- Artigo 1 JÁ tem Tag 1
```

---

### Foreign Keys e ON DELETE

!!! danger "Integridade Referencial"
    **Problema:** O que acontece quando deletar um usuário que tem artigos?

    **Opções:**

    | Ação | Comportamento | Quando Usar |
    |------|---------------|-------------|
    | `CASCADE` | Deleta artigos também | **Recomendado** - Dados órfãos inúteis |
    | `RESTRICT` | **Bloqueia** deleção | Quando precisa proteger dados |
    | `SET NULL` | Define `user_id = NULL` | Quando precisa manter histórico |
    | `NO ACTION` | Igual a RESTRICT | Padrão SQL |

**Exemplo CASCADE:**
```sql
DELETE FROM users WHERE id = 1;
-- Resultado:
-- ✅ Usuário deletado
-- ✅ Todos os artigos do usuário deletados (CASCADE)
-- ✅ Todas as relações em articles_tags deletadas (CASCADE)
```

**Exemplo RESTRICT:**
```sql
FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE RESTRICT;

DELETE FROM users WHERE id = 1;
-- ❌ ERROR: Cannot delete - user has articles
```

---

## 🌱 Populando com Dados Iniciais

### Dados de Teste (INSERT)

!!! warning "Senha Hash BCrypt"
    **NUNCA** salve senhas em plain text!

    O hash abaixo representa a senha `secret` usando bcrypt:
    ```php
    password_hash('secret', PASSWORD_DEFAULT);
    // $2y$10$u5ceCKLTkqG1Y...
    ```

=== "MySQL"

    ```sql
    -- Usuários (senha: "secret")
    INSERT INTO users (email, password, created, modified) VALUES
    ('admin@example.com', '$2y$10$u5ceCKLTkqG1Y8W0h8F6lOuKQTqJXqQq1XqZqXqZqXqZqXqZqXq', NOW(), NOW()),
    ('editor@example.com', '$2y$10$u5ceCKLTkqG1Y8W0h8F6lOuKQTqJXqQq1XqZqXqZqXqZqXqZqXq', NOW(), NOW()),
    ('autor@example.com', '$2y$10$u5ceCKLTkqG1Y8W0h8F6lOuKQTqJXqQq1XqZqXqZqXqZqXqZqXq', NOW(), NOW());

    -- Tags
    INSERT INTO tags (title, created, modified) VALUES
    ('CakePHP', NOW(), NOW()),
    ('PHP', NOW(), NOW()),
    ('MySQL', NOW(), NOW()),
    ('Tutorial', NOW(), NOW()),
    ('REST API', NOW(), NOW());

    -- Artigos
    INSERT INTO articles (user_id, title, slug, body, published, created, modified) VALUES
    (1, 'Bem-vindo ao CakePHP 5.0', 'bem-vindo-cakephp-5',
     'Este é nosso primeiro artigo usando CakePHP 5! 🎉\n\nVamos explorar os novos recursos...',
     1, NOW(), NOW()),
    (1, 'Guia de Migrations', 'guia-migrations',
     'Migrations permitem versionar o schema do banco de dados.\n\nVeja como usar...',
     1, NOW(), NOW()),
    (2, 'Criando REST APIs', 'criando-rest-apis',
     'CakePHP facilita criar APIs RESTful.\n\nVamos criar uma API completa...',
     0, NOW(), NOW()),
    (3, 'Rascunho - Teste', 'rascunho-teste',
     'Este artigo ainda não foi publicado.',
     0, NOW(), NOW());

    -- Relacionamentos (Articles ↔ Tags)
    INSERT INTO articles_tags (article_id, tag_id) VALUES
    (1, 1), -- "Bem-vindo" tem tag "CakePHP"
    (1, 2), -- "Bem-vindo" tem tag "PHP"
    (1, 4), -- "Bem-vindo" tem tag "Tutorial"
    (2, 1), -- "Migrations" tem tag "CakePHP"
    (2, 3), -- "Migrations" tem tag "MySQL"
    (2, 4), -- "Migrations" tem tag "Tutorial"
    (3, 1), -- "REST APIs" tem tag "CakePHP"
    (3, 5); -- "REST APIs" tem tag "REST API"
    ```

=== "PostgreSQL"

    ```sql
    -- Usuários
    INSERT INTO users (email, password, created, modified) VALUES
    ('admin@example.com', '$2y$10$u5ceCKLTkqG1Y8W0h8F6lOuKQTqJXqQq1XqZqXqZqXqZqXqZqXq', NOW(), NOW()),
    ('editor@example.com', '$2y$10$u5ceCKLTkqG1Y8W0h8F6lOuKQTqJXqQq1XqZqXqZqXqZqXqZqXq', NOW(), NOW()),
    ('autor@example.com', '$2y$10$u5ceCKLTkqG1Y8W0h8F6lOuKQTqJXqQq1XqZqXqZqXqZqXqZqXq', NOW(), NOW());

    -- Tags
    INSERT INTO tags (title, created, modified) VALUES
    ('CakePHP', NOW(), NOW()),
    ('PHP', NOW(), NOW()),
    ('PostgreSQL', NOW(), NOW()),
    ('Tutorial', NOW(), NOW()),
    ('REST API', NOW(), NOW());

    -- Artigos
    INSERT INTO articles (user_id, title, slug, body, published, created, modified) VALUES
    (1, 'Bem-vindo ao CakePHP 5.0', 'bem-vindo-cakephp-5',
     'Este é nosso primeiro artigo usando CakePHP 5! 🎉',
     TRUE, NOW(), NOW()),
    (1, 'Guia de Migrations', 'guia-migrations',
     'Migrations permitem versionar o schema do banco de dados.',
     TRUE, NOW(), NOW()),
    (2, 'Criando REST APIs', 'criando-rest-apis',
     'CakePHP facilita criar APIs RESTful.',
     FALSE, NOW(), NOW());

    -- Relacionamentos
    INSERT INTO articles_tags (article_id, tag_id) VALUES
    (1, 1), (1, 2), (1, 4),
    (2, 1), (2, 3), (2, 4),
    (3, 1), (3, 5);
    ```

---

## ⚙️ Configurando a Conexão

### Etapa 1: Editar config/app_local.php

Abra o arquivo `config/app_local.php` e configure a seção `Datasources`:

=== "MySQL"

    ```php
    <?php
    return [
        'Datasources' => [
            'default' => [
                'className' => 'Cake\Database\Connection',
                'driver' => 'Cake\Database\Driver\Mysql',
                'persistent' => false,
                'host' => 'localhost',
                'port' => 3306,
                'username' => 'root',
                'password' => 'sua_senha_aqui',
                'database' => 'cake_cms',
                'encoding' => 'utf8mb4',
                'timezone' => 'America/Sao_Paulo',
                'flags' => [],
                'cacheMetadata' => true,
                'log' => false,
                'quoteIdentifiers' => false,
                'url' => env('DATABASE_URL', null),
            ],
        ],
    ];
    ```

=== "PostgreSQL"

    ```php
    <?php
    return [
        'Datasources' => [
            'default' => [
                'className' => 'Cake\Database\Connection',
                'driver' => 'Cake\Database\Driver\Postgres',
                'persistent' => false,
                'host' => 'localhost',
                'port' => 5432,
                'username' => 'postgres',
                'password' => 'sua_senha_aqui',
                'database' => 'cake_cms',
                'encoding' => 'utf8',
                'timezone' => 'America/Sao_Paulo',
                'schema' => 'public',
                'url' => env('DATABASE_URL', null),
            ],
        ],
    ];
    ```

=== "SQLite"

    ```php
    <?php
    return [
        'Datasources' => [
            'default' => [
                'className' => 'Cake\Database\Connection',
                'driver' => 'Cake\Database\Driver\Sqlite',
                'persistent' => false,
                'database' => ROOT . DS . 'tmp' . DS . 'cake_cms.sqlite',
                'encoding' => 'utf8',
                'url' => env('DATABASE_URL', null),
            ],
        ],
    ];
    ```

---

### Etapa 2: Verificar Conexão

Acesse [http://localhost:8765](http://localhost:8765) e verifique:

✅ **Sucesso:**
```
CakePHP is able to connect to the database
```

❌ **Erro:**
```
CakePHP is NOT able to connect to the database
```

!!! tip "Troubleshooting"
    Veja a seção [Problemas Comuns](#-problemas-comuns) abaixo.

---

## 🚀 Usando Migrations (Recomendado)

Migrations são **versionamento Git para banco de dados**. Vantagens:

- ✅ **Reversível** - Voltar para versão anterior (`rollback`)
- ✅ **Colaborativo** - Time sincroniza schemas via Git
- ✅ **Multiplataforma** - Mesmo código funciona MySQL/PostgreSQL/SQLite
- ✅ **Histórico** - Cada alteração fica documentada

### Etapa 1: Verificar Plugin

O plugin Migrations já vem instalado no skeleton:

```bash
composer show | grep migrations
# cakephp/migrations  4.5.0  Database Migration plugin for CakePHP
```

---

### Etapa 2: Gerar Migrations

```bash
cd /home/kromodoro/Documentos/cakephp/app

# Tabela Users
bin/cake bake migration CreateUsers \
  email:string \
  password:string \
  created \
  modified

# Tabela Articles
bin/cake bake migration CreateArticles \
  user_id:integer \
  title:string \
  slug:string[191]:unique \
  body:text \
  published:boolean \
  created \
  modified

# Tabela Tags
bin/cake bake migration CreateTags \
  title:string[191]:unique \
  created \
  modified

# Tabela ArticlesTags (Junction)
bin/cake bake migration CreateArticlesTags \
  article_id:integer:primary \
  tag_id:integer:primary \
  created \
  modified
```

**Resultado:**
```
Created file: config/Migrations/20251115120000_CreateUsers.php
Created file: config/Migrations/20251115120001_CreateArticles.php
Created file: config/Migrations/20251115120002_CreateTags.php
Created file: config/Migrations/20251115120003_CreateArticlesTags.php
```

---

### Etapa 3: Corrigir Bug do ArticlesTags

!!! danger "Bug Conhecido"
    O Bake gera migrations com `autoIncrement => true` em **composite primary keys**, causando erro de foreign key.

Abra `config/Migrations/20251115120003_CreateArticlesTags.php` e **substitua** o método `change()`:

**❌ Código gerado (ERRADO):**
```php
public function change(): void
{
    $table = $this->table('articles_tags');
    $table->addColumn('article_id', 'integer', [
        'autoIncrement' => true,  // ❌ REMOVER
        'default' => null,
        'limit' => 11,
        'null' => false,
    ]);
    $table->addColumn('tag_id', 'integer', [
        'autoIncrement' => true,  // ❌ REMOVER
        'default' => null,
        'limit' => 11,
        'null' => false,
    ]);
    $table->create();
}
```

**✅ Código correto:**
```php
public function change(): void
{
    $table = $this->table('articles_tags', [
        'id' => false,
        'primary_key' => ['article_id', 'tag_id']
    ]);

    $table->addColumn('article_id', 'integer', [
        'default' => null,
        'limit' => 11,
        'null' => false,
    ]);

    $table->addColumn('tag_id', 'integer', [
        'default' => null,
        'limit' => 11,
        'null' => false,
    ]);

    $table->addForeignKey('article_id', 'articles', 'id', [
        'delete' => 'CASCADE',
        'update' => 'CASCADE'
    ]);

    $table->addForeignKey('tag_id', 'tags', 'id', [
        'delete' => 'CASCADE',
        'update' => 'CASCADE'
    ]);

    $table->create();
}
```

---

### Etapa 4: Executar Migrations

```bash
bin/cake migrations migrate
```

**Saída esperada:**
```
using migration paths
 - /home/kromodoro/Documentos/cakephp/app/config/Migrations
using seed paths
 - /home/kromodoro/Documentos/cakephp/app/config/Seeds
using environment default
using adapter mysql
using database cake_cms

 == 20251115120000 CreateUsers: migrating
 == 20251115120000 CreateUsers: migrated 0.0234s

 == 20251115120001 CreateArticles: migrating
 == 20251115120001 CreateArticles: migrated 0.0312s

 == 20251115120002 CreateTags: migrating
 == 20251115120002 CreateTags: migrated 0.0189s

 == 20251115120003 CreateArticlesTags: migrating
 == 20251115120003 CreateArticlesTags: migrated 0.0267s

All Done. Took 0.1045s
```

---

### Etapa 5: Verificar Tabelas

```bash
# MySQL
mysql -u root -p cake_cms -e "SHOW TABLES;"

# PostgreSQL
psql -U postgres -d cake_cms -c "\dt"

# SQLite
sqlite3 tmp/cake_cms.sqlite ".tables"
```

**Saída esperada:**
```
+------------------+
| Tables_in_cake_cms |
+------------------+
| articles         |
| articles_tags    |
| phinxlog         |
| tags             |
| users            |
+------------------+
```

!!! info "Tabela phinxlog"
    O Phinx (biblioteca de migrations) cria esta tabela automaticamente para rastrear quais migrations já foram executadas.

---

## 🌱 Seeds (Dados Iniciais)

Seeds populam o banco com dados de teste/produção.

### Etapa 1: Gerar Seeds

```bash
bin/cake bake seed Users
bin/cake bake seed Tags
bin/cake bake seed Articles
bin/cake bake seed ArticlesTags
```

**Resultado:**
```
Created file: config/Seeds/UsersSeed.php
Created file: config/Seeds/TagsSeed.php
Created file: config/Seeds/ArticlesSeed.php
Created file: config/Seeds/ArticlesTagsSeed.php
```

---

### Etapa 2: Editar UsersSeed.php

Abra `config/Seeds/UsersSeed.php` e substitua o método `run()`:

```php
<?php
declare(strict_types=1);

use Migrations\AbstractSeed;

class UsersSeed extends AbstractSeed
{
    public function run(): void
    {
        $data = [
            [
                'email' => 'admin@example.com',
                'password' => '$2y$10$u5ceCKLTkqG1Y8W0h8F6lOuKQTqJXqQq1XqZqXqZqXqZqXqZqXq', // "secret"
                'created' => date('Y-m-d H:i:s'),
                'modified' => date('Y-m-d H:i:s'),
            ],
            [
                'email' => 'editor@example.com',
                'password' => '$2y$10$u5ceCKLTkqG1Y8W0h8F6lOuKQTqJXqQq1XqZqXqZqXqZqXqZqXq',
                'created' => date('Y-m-d H:i:s'),
                'modified' => date('Y-m-d H:i:s'),
            ],
            [
                'email' => 'autor@example.com',
                'password' => '$2y$10$u5ceCKLTkqG1Y8W0h8F6lOuKQTqJXqQq1XqZqXqZqXqZqXqZqXq',
                'created' => date('Y-m-d H:i:s'),
                'modified' => date('Y-m-d H:i:s'),
            ],
        ];

        $table = $this->table('users');
        $table->insert($data)->save();
    }
}
```

---

### Etapa 3: Editar TagsSeed.php

```php
<?php
declare(strict_types=1);

use Migrations\AbstractSeed;

class TagsSeed extends AbstractSeed
{
    public function run(): void
    {
        $data = [
            ['title' => 'CakePHP', 'created' => date('Y-m-d H:i:s'), 'modified' => date('Y-m-d H:i:s')],
            ['title' => 'PHP', 'created' => date('Y-m-d H:i:s'), 'modified' => date('Y-m-d H:i:s')],
            ['title' => 'MySQL', 'created' => date('Y-m-d H:i:s'), 'modified' => date('Y-m-d H:i:s')],
            ['title' => 'Tutorial', 'created' => date('Y-m-d H:i:s'), 'modified' => date('Y-m-d H:i:s')],
            ['title' => 'REST API', 'created' => date('Y-m-d H:i:s'), 'modified' => date('Y-m-d H:i:s')],
        ];

        $table = $this->table('tags');
        $table->insert($data)->save();
    }
}
```

---

### Etapa 4: Editar ArticlesSeed.php

```php
<?php
declare(strict_types=1);

use Migrations\AbstractSeed;

class ArticlesSeed extends AbstractSeed
{
    public function run(): void
    {
        $data = [
            [
                'user_id' => 1,
                'title' => 'Bem-vindo ao CakePHP 5.0',
                'slug' => 'bem-vindo-cakephp-5',
                'body' => "Este é nosso primeiro artigo usando CakePHP 5! 🎉\n\nVamos explorar os novos recursos...",
                'published' => true,
                'created' => date('Y-m-d H:i:s'),
                'modified' => date('Y-m-d H:i:s'),
            ],
            [
                'user_id' => 1,
                'title' => 'Guia de Migrations',
                'slug' => 'guia-migrations',
                'body' => "Migrations permitem versionar o schema do banco de dados.\n\nVeja como usar...",
                'published' => true,
                'created' => date('Y-m-d H:i:s'),
                'modified' => date('Y-m-d H:i:s'),
            ],
            [
                'user_id' => 2,
                'title' => 'Criando REST APIs',
                'slug' => 'criando-rest-apis',
                'body' => "CakePHP facilita criar APIs RESTful.\n\nVamos criar uma API completa...",
                'published' => false,
                'created' => date('Y-m-d H:i:s'),
                'modified' => date('Y-m-d H:i:s'),
            ],
        ];

        $table = $this->table('articles');
        $table->insert($data)->save();
    }
}
```

---

### Etapa 5: Editar ArticlesTagsSeed.php

```php
<?php
declare(strict_types=1);

use Migrations\AbstractSeed;

class ArticlesTagsSeed extends AbstractSeed
{
    public function getDependencies(): array
    {
        return [
            'ArticlesSeed',
            'TagsSeed',
        ];
    }

    public function run(): void
    {
        $data = [
            // Artigo "Bem-vindo" tem tags: CakePHP, PHP, Tutorial
            ['article_id' => 1, 'tag_id' => 1],
            ['article_id' => 1, 'tag_id' => 2],
            ['article_id' => 1, 'tag_id' => 4],

            // Artigo "Migrations" tem tags: CakePHP, MySQL, Tutorial
            ['article_id' => 2, 'tag_id' => 1],
            ['article_id' => 2, 'tag_id' => 3],
            ['article_id' => 2, 'tag_id' => 4],

            // Artigo "REST APIs" tem tags: CakePHP, REST API
            ['article_id' => 3, 'tag_id' => 1],
            ['article_id' => 3, 'tag_id' => 5],
        ];

        $table = $this->table('articles_tags');
        $table->insert($data)->save();
    }
}
```

---

### Etapa 6: Executar Seeds

```bash
bin/cake migrations seed
```

**Saída esperada:**
```
using migration paths
 - /home/kromodoro/Documentos/cakephp/app/config/Migrations
using seed paths
 - /home/kromodoro/Documentos/cakephp/app/config/Seeds
using environment default
using adapter mysql
using database cake_cms

 == UsersSeed: seeding
 == UsersSeed: seeded 0.0123s

 == TagsSeed: seeding
 == TagsSeed: seeded 0.0089s

 == ArticlesSeed: seeding
 == ArticlesSeed: seeded 0.0145s

 == ArticlesTagsSeed: seeding
 == ArticlesTagsSeed: seeded 0.0098s

All Done. Took 0.0512s
```

---

## 🔍 Verificar Dados Populados

```sql
-- Verificar usuários
SELECT id, email, created FROM users;

-- Verificar artigos com nome do autor
SELECT a.id, a.title, u.email AS autor, a.published
FROM articles a
JOIN users u ON a.user_id = u.id;

-- Verificar tags de cada artigo
SELECT a.title AS artigo, t.title AS tag
FROM articles a
JOIN articles_tags at ON a.id = at.article_id
JOIN tags t ON at.tag_id = t.id
ORDER BY a.title;
```

**Resultado esperado:**
```
+----+-----------------------------+---------------------+
| id | title                       | autor               |
+----+-----------------------------+---------------------+
|  1 | Bem-vindo ao CakePHP 5.0    | admin@example.com   |
|  2 | Guia de Migrations          | admin@example.com   |
|  3 | Criando REST APIs           | editor@example.com  |
+----+-----------------------------+---------------------+
```

---

## 📚 Convenções do CakePHP

!!! success "Naming Conventions"
    Seguindo as convenções, o CakePHP gera automaticamente:
    - Model classes
    - Controller actions
    - View templates
    - Associations (relacionamentos)

### Regras de Nomenclatura

| Elemento | Convenção | Exemplo |
|----------|-----------|---------|
| **Tabelas** | Plural, underscore | `users`, `articles_tags` |
| **Colunas** | Singular, underscore | `user_id`, `created` |
| **Models** | Singular, CamelCase | `User`, `Article`, `Tag` |
| **Controllers** | Plural, CamelCase + "Controller" | `UsersController` |
| **Primary Key** | `id` | `users.id`, `articles.id` |
| **Foreign Key** | `{singular}_id` | `user_id`, `article_id` |
| **Pivot Table** | Alfabética, underscore | `articles_tags` (não `tags_articles`) |
| **Timestamps** | `created`, `modified` | Populados automaticamente |

---

### Auto-Magic do CakePHP

Quando você seguir as convenções, o CakePHP automaticamente:

✅ **Detecta relacionamentos:**
```php
// Model/Table/ArticlesTable.php
$this->belongsTo('Users'); // Detecta user_id
$this->belongsToMany('Tags'); // Detecta articles_tags
```

✅ **Popula timestamps:**
```php
$article = $this->Articles->newEntity(['title' => 'Novo']);
$this->Articles->save($article);
// created e modified preenchidos automaticamente!
```

✅ **Gera rotas:**
```
/articles           → ArticlesController::index()
/articles/view/1    → ArticlesController::view(1)
/articles/add       → ArticlesController::add()
```

✅ **Valida dados:**
```php
// Validação automática baseada em schema:
$this->Form->control('email'); // Valida email
$this->Form->control('published'); // Checkbox booleano
```

---

## ❓ Problemas Comuns

??? danger "Erro: `SQLSTATE[HY000] [2002] Connection refused`"
    **Causa:** Servidor MySQL não está rodando.

    **Soluções:**

    === "Linux"
        ```bash
        sudo systemctl start mysql
        sudo systemctl status mysql
        ```

    === "macOS"
        ```bash
        brew services start mysql
        ```

    === "Windows (XAMPP)"
        - Abrir XAMPP Control Panel
        - Clicar "Start" em MySQL

---

??? danger "Erro: `SQLSTATE[HY000] [1045] Access denied for user`"
    **Causa:** Senha incorreta ou usuário sem permissões.

    **Solução 1: Resetar senha root (MySQL)**
    ```bash
    sudo mysql
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'nova_senha';
    FLUSH PRIVILEGES;
    EXIT;
    ```

    **Solução 2: Criar usuário dedicado**
    ```sql
    CREATE USER 'cakephp'@'localhost' IDENTIFIED BY 'senha123';
    GRANT ALL PRIVILEGES ON cake_cms.* TO 'cakephp'@'localhost';
    FLUSH PRIVILEGES;
    ```

    Atualizar `config/app_local.php`:
    ```php
    'username' => 'cakephp',
    'password' => 'senha123',
    ```

---

??? danger "Erro: `SQLSTATE[42000]: Syntax error ... 'slug'`"
    **Causa:** `slug` é palavra reservada em alguns bancos (PostgreSQL).

    **Solução:** Habilitar quote identifiers:
    ```php
    // config/app_local.php
    'Datasources' => [
        'default' => [
            'quoteIdentifiers' => true, // ✅
        ],
    ],
    ```

---

??? warning "Aviso: `Specified key was too long; max key length is 767 bytes`"
    **Causa:** `VARCHAR(255)` com utf8mb4 = 1020 bytes (255 × 4).

    **Solução:** Usar `VARCHAR(191)`:
    ```sql
    ALTER TABLE articles MODIFY slug VARCHAR(191);
    ```

---

??? warning "Foreign key constraint fails ao deletar usuário"
    **Causa:** Artigos órfãos com `user_id` inválido.

    **Solução 1: Usar CASCADE**
    ```sql
    ALTER TABLE articles
    DROP FOREIGN KEY fk_user;

    ALTER TABLE articles
    ADD CONSTRAINT fk_user
    FOREIGN KEY (user_id) REFERENCES users(id)
    ON DELETE CASCADE;
    ```

    **Solução 2: Deletar artigos primeiro**
    ```sql
    DELETE FROM articles WHERE user_id = 1;
    DELETE FROM users WHERE id = 1;
    ```

---

??? info "Como verificar charset/collation do banco?"
    **MySQL:**
    ```sql
    SELECT
        table_name,
        table_collation
    FROM information_schema.tables
    WHERE table_schema = 'cake_cms';
    ```

    **Resultado esperado:**
    ```
    +---------------+--------------------+
    | table_name    | table_collation    |
    +---------------+--------------------+
    | articles      | utf8mb4_unicode_ci |
    | users         | utf8mb4_unicode_ci |
    +---------------+--------------------+
    ```

---

??? tip "Como popular banco em produção (Heroku, AWS)?"
    **Opção 1: Usar DATABASE_URL**
    ```bash
    # .env ou variável de ambiente
    DATABASE_URL="mysql://user:pass@host:3306/cake_cms?encoding=utf8mb4"
    ```

    **Opção 2: Executar migrations remotamente**
    ```bash
    # Via SSH (VPS)
    ssh usuario@meusite.com
    cd /var/www/html/cakephp
    bin/cake migrations migrate
    bin/cake migrations seed
    ```

    **Opção 3: Dump SQL**
    ```bash
    # Gerar SQL
    mysqldump -u root -p cake_cms > backup.sql

    # Restaurar em produção
    mysql -u user -p cake_cms_prod < backup.sql
    ```

---

## 🎓 Conceitos Avançados

??? abstract "Diferença entre DATETIME e TIMESTAMP"
    | Aspecto | DATETIME | TIMESTAMP |
    |---------|----------|-----------|
    | **Fuso horário** | Ignora timezone | Converte para UTC |
    | **Intervalo** | 1000-01-01 a 9999-12-31 | 1970-01-01 a 2038-01-19 |
    | **Tamanho** | 8 bytes | 4 bytes |
    | **Auto-update** | Não | Sim (ON UPDATE CURRENT_TIMESTAMP) |

    **Recomendação CakePHP:** Usar `DATETIME` + configurar timezone:
    ```php
    // config/app_local.php
    'Datasources' => [
        'default' => [
            'timezone' => 'America/Sao_Paulo',
        ],
    ],
    ```

---

??? abstract "InnoDB vs MyISAM"
    | Feature | InnoDB | MyISAM |
    |---------|--------|--------|
    | **Transações** | ✅ ACID | ❌ |
    | **Foreign Keys** | ✅ | ❌ |
    | **Row-level lock** | ✅ | ❌ (Table-level) |
    | **Performance** | Escrita rápida | Leitura rápida |
    | **Crash recovery** | ✅ | ❌ |

    **Recomendação:** Sempre usar InnoDB (padrão desde MySQL 5.5).

---

??? abstract "Índices: Quando usar?"
    **Criar índice quando:**
    - ✅ Coluna usada em `WHERE` frequentemente
    - ✅ Coluna usada em `JOIN`
    - ✅ Coluna com `UNIQUE` constraint
    - ✅ Foreign keys

    **Não criar índice quando:**
    - ❌ Tabela pequena (< 1000 rows)
    - ❌ Coluna com poucos valores únicos (ex: `published` boolean)
    - ❌ Coluna raramente consultada

    **Exemplo:**
    ```sql
    -- ✅ BOM: slug usado em URLs
    CREATE INDEX idx_slug ON articles(slug);

    -- ❌ RUIM: published tem apenas 2 valores (0, 1)
    -- (Já existe índice composto user_id + published)
    ```

---

## 🚀 Próximos Passos

✅ **Banco configurado!** Agora você pode:

1. 📖 **[Criar Models](cms-articles-model.md)** - Definir regras de validação e associações
2. 🎮 **[Criar Controllers](cms-articles-controller.md)** - Lógica de CRUD
3. 🎨 **[Criar Views](cms-tags-and-users.md)** - Interface HTML
4. 🔐 **[Adicionar Autenticação](cms-authentication.md)** - Login/Logout
5. 🛡️ **[Adicionar Autorização](cms-authorization.md)** - Controle de acesso

---

## 📚 Referências

- 📖 [CakePHP Database Basics](https://book.cakephp.org/5/en/orm/database-basics.html)
- 🔧 [Migrations Plugin](https://book.cakephp.org/migrations/4/)
- 🏗️ [Schema Conventions](https://book.cakephp.org/5/en/intro/conventions.html#database-conventions)
- 🔑 [Foreign Keys Best Practices](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html)
- 🌐 [UTF-8 vs UTF8MB4](https://mathiasbynens.be/notes/mysql-utf8mb4)
- 📊 [Database Design Patterns](https://www.vertabelo.com/blog/database-design-patterns/)

---

**✨ Criado com [Claude Code](https://claude.com/claude-code)**
