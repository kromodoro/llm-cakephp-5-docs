# Tutorial CMS - Instalação do CakePHP

Bem-vindo ao Tutorial CMS! Neste guia completo, você aprenderá a criar um Sistema de Gerenciamento de Conteúdo (CMS) completo usando CakePHP 5. Começaremos pela instalação e configuração inicial.

---

## 📋 O Que Você Vai Construir

Este tutorial ensina a criar um **CMS funcional** com:

<div class="grid cards" markdown>

-   :material-file-document-edit:{ .lg .middle } **Gestão de Artigos**

    ---

    Sistema CRUD completo para criar, ler, atualizar e excluir artigos com slug único e validação

-   :material-account-multiple:{ .lg .middle } **Sistema de Usuários**

    ---

    Autenticação, registro e controle de autoria de artigos

-   :material-tag-multiple:{ .lg .middle } **Tags e Categorização**

    ---

    Relacionamento many-to-many para organizar conteúdo

-   :material-shield-lock:{ .lg .middle } **Autenticação & Autorização**

    ---

    Login seguro e controle de permissões por usuário

</div>

---

## ⏱️ Tempo Estimado

| Perfil | Tempo Total | Etapa 1 (Instalação) |
|--------|-------------|----------------------|
| **🟢 Dev Experiente** | 1-2 horas | **15 min** |
| **🟡 Dev PHP Intermediário** | 3-5 horas | **30 min** |
| **🟠 Iniciante em Frameworks** | 6-8 horas | **45 min** |
| **🔴 Estudante/Primeiro Projeto** | 8-12 horas | **60 min** |

!!! tip "Dica de Produtividade"
    Se você já conhece MVC e tem PHP/Composer instalados, pode completar a instalação em **menos de 15 minutos**!

---

## 📋 Pré-requisitos

Antes de começar, certifique-se de ter:

### 1. Conhecimentos Necessários

- [x] **PHP Básico** - Conhecimento de variáveis, funções, classes e namespaces
- [x] **SQL Básico** - Saber criar tabelas, inserir e consultar dados
- [x] **Linha de Comando** - Conforto com terminal (CMD, PowerShell, Bash)
- [x] **HTTP/HTML** - Entender requisições GET/POST e tags HTML básicas

!!! info "Não Sabe MVC?"
    **Tudo bem!** O tutorial ensina os conceitos conforme avança. Se quiser se preparar, leia primeiro [Introdução ao CakePHP](../../intro.md).

### 2. Requisitos de Software

=== "Requisitos Mínimos"

    | Software | Versão Mínima | Como Verificar |
    |----------|---------------|----------------|
    | **PHP** | 8.1 | `php -v` |
    | **Composer** | 2.0 | `composer --version` |
    | **MySQL** | 5.7 / 8.0 | `mysql --version` |
    | **Navegador** | Moderno | Chrome, Firefox, Edge, Safari |

=== "Requisitos Recomendados"

    | Software | Versão Recomendada | Por Quê? |
    |----------|-------------------|----------|
    | **PHP** | **8.3** | Performance +30%, type system melhorado |
    | **Composer** | **2.7+** | Instalação 50% mais rápida |
    | **MySQL** | **8.0+** ou **PostgreSQL 14+** | Window functions, JSON melhorado |
    | **Node.js** | 18+ LTS | Para ferramentas de build (opcional) |

### 3. Extensões PHP Necessárias

!!! warning "IMPORTANTE: Verifique ANTES de Instalar"
    CakePHP requer estas extensões. Se faltar alguma, você verá erros crípticos!

**Verificar extensões instaladas:**

```bash
php -m
```

**Extensões OBRIGATÓRIAS:**

| Extensão | Propósito | Verificar |
|----------|-----------|----------|
| **pdo_mysql** | Conexão com MySQL/MariaDB | `php -m \| grep pdo_mysql` |
| **mbstring** | Manipulação de strings UTF-8 | `php -m \| grep mbstring` |
| **intl** | Internacionalização (ICU) | `php -m \| grep intl` |
| **simplexml** | Leitura de arquivos XML | `php -m \| grep simplexml` |

**Extensões OPCIONAIS (mas recomendadas):**

| Extensão | Propósito |
|----------|-----------|
| **opcache** | Cache de bytecode PHP (performance) |
| **apcu** | Cache de dados em memória |
| **redis** | Cache distribuído (produção) |

??? tip "Como Instalar Extensões PHP"

    === "Ubuntu/Debian"

        ```bash
        # PHP 8.1
        sudo apt-get update
        sudo apt-get install -y \
          php8.1-mysql \
          php8.1-mbstring \
          php8.1-intl \
          php8.1-xml \
          php8.1-opcache \
          php8.1-apcu

        # Reiniciar servidor (se Apache)
        sudo systemctl restart apache2
        ```

    === "CentOS/RHEL/Rocky"

        ```bash
        # PHP 8.1 (requer repositório Remi)
        sudo dnf install -y \
          php81-php-mysqlnd \
          php81-php-mbstring \
          php81-php-intl \
          php81-php-xml \
          php81-php-opcache \
          php81-php-pecl-apcu

        sudo systemctl restart php-fpm
        ```

    === "macOS (Homebrew)"

        ```bash
        brew install php@8.1

        # Extensões já incluídas no Homebrew PHP
        # Verificar:
        php -m
        ```

    === "Windows (XAMPP/WAMP)"

        **XAMPP:**
        1. Abra `C:\xampp\php\php.ini`
        2. Procure por linhas como `;extension=pdo_mysql`
        3. Remova o `;` para descomentar:
           ```ini
           extension=pdo_mysql
           extension=mbstring
           extension=intl
           extension=simplexml
           ```
        4. Reinicie Apache no painel de controle do XAMPP

        **WAMP:**
        - Clique no ícone WAMP na bandeja
        - PHP → PHP extensions → Marque as extensões necessárias

---

## 📦 Instalação do CakePHP

Existem três métodos principais de instalação. Escolha o que melhor se adapta ao seu ambiente:

### Método 1: Via Composer (Recomendado)

=== "Linux/macOS"

    **Passo 1: Instalar o Composer (se ainda não tiver)**

    ```bash
    # Download do instalador
    curl -sS https://getcomposer.org/installer | php

    # Mover para o PATH (opcional, mas recomendado)
    sudo mv composer.phar /usr/local/bin/composer
    sudo chmod +x /usr/local/bin/composer

    # Verificar instalação
    composer --version
    # Composer version 2.7.1 ...
    ```

    **Passo 2: Criar Projeto CakePHP**

    ```bash
    # Navegar até o diretório onde quer criar o projeto
    cd ~/projetos  # ou qualquer diretório

    # Criar projeto CMS com CakePHP 5
    composer create-project cakephp/app:5 cms

    # Aguardar instalação (1-3 minutos)
    # ✓ Downloading cakephp/cakephp (5.2.0)
    # ✓ Installing dependencies...
    # ✓ Setting file permissions...
    # ✓ Creating config/app.php...
    ```

    **Passo 3: Acessar o Diretório**

    ```bash
    cd cms
    ls -la
    # bin/  config/  src/  templates/  webroot/  ...
    ```

=== "Windows (CMD)"

    **Passo 1: Instalar o Composer**

    1. Baixe o instalador: [Composer-Setup.exe](https://getcomposer.org/Composer-Setup.exe)
    2. Execute e siga o assistente de instalação
    3. Reinicie o terminal (CMD ou PowerShell)

    **Passo 2: Verificar Instalação**

    ```cmd
    composer --version
    # Composer version 2.7.1 ...
    ```

    **Passo 3: Criar Projeto**

    ```cmd
    REM Navegar até pasta de projetos
    cd C:\xampp\htdocs

    REM Atualizar Composer (recomendado)
    composer self-update

    REM Criar projeto CMS
    composer create-project cakephp/app:5.* cms

    REM Acessar diretório
    cd cms
    ```

=== "Windows (PowerShell)"

    ```powershell
    # Navegar até pasta de projetos
    cd C:\xampp\htdocs

    # Atualizar Composer
    composer self-update

    # Criar projeto
    composer create-project cakephp/app:5.* cms

    # Acessar diretório
    cd cms

    # Listar arquivos
    Get-ChildItem
    ```

=== "Windows (Git Bash)"

    ```bash
    # Funciona igual ao Linux/macOS
    cd /c/xampp/htdocs
    composer create-project cakephp/app:5 cms
    cd cms
    ```

!!! success "Instalação Completa!"
    O Composer automaticamente:

    - ✅ Baixou o CakePHP 5.2
    - ✅ Instalou todas as dependências
    - ✅ Configurou permissões de arquivos
    - ✅ Criou `config/app.php` a partir do template
    - ✅ Gerou salt de segurança aleatório

---

### Método 2: Via Docker (Moderno)

Para quem prefere ambientes isolados e reproduzíveis:

=== "Docker Compose (Recomendado)"

    **Passo 1: Criar docker-compose.yml**

    ```yaml
    # docker-compose.yml
    version: '3.8'

    services:
      app:
        image: php:8.3-apache
        container_name: cms-app
        ports:
          - "8765:80"
        volumes:
          - ./:/var/www/html
        environment:
          - APACHE_DOCUMENT_ROOT=/var/www/html/webroot
        depends_on:
          - db

      db:
        image: mysql:8.0
        container_name: cms-mysql
        environment:
          MYSQL_ROOT_PASSWORD: secret
          MYSQL_DATABASE: cms
          MYSQL_USER: cakephp
          MYSQL_PASSWORD: password
        ports:
          - "3306:3306"
        volumes:
          - mysql-data:/var/lib/mysql

    volumes:
      mysql-data:
    ```

    **Passo 2: Iniciar Containers**

    ```bash
    # Criar projeto primeiro
    composer create-project cakephp/app:5 cms
    cd cms

    # Salvar docker-compose.yml acima no diretório

    # Iniciar ambiente
    docker-compose up -d

    # Ver logs
    docker-compose logs -f app
    ```

    **Passo 3: Acessar**

    Abra: [http://localhost:8765](http://localhost:8765)

=== "Docker Run (Manual)"

    ```bash
    # Criar projeto usando container do Composer
    docker run --rm -v $(pwd):/app composer \
      create-project cakephp/app:5 cms

    cd cms

    # Rodar servidor PHP
    docker run -d \
      --name cms-app \
      -p 8765:8080 \
      -v $(pwd):/app \
      -w /app \
      php:8.3-cli \
      php -S 0.0.0.0:8080 -t webroot

    # Acessar: http://localhost:8765
    ```

!!! tip "Por Que Usar Docker?"
    - ✅ **Zero configuração:** Não precisa instalar PHP/MySQL no host
    - ✅ **Isolamento:** Cada projeto tem seu próprio ambiente
    - ✅ **Reproduzível:** Mesma configuração em dev/staging/prod
    - ✅ **Windows-friendly:** Evita problemas do XAMPP/WAMP

---

### Método 3: Via Download Manual (Não Recomendado)

Para situações onde não há acesso ao Composer:

!!! warning "Apenas Se Necessário"
    Este método **não é recomendado** porque você precisará gerenciar dependências manualmente.

```bash
# 1. Baixar código-fonte
wget https://github.com/cakephp/app/archive/refs/tags/5.0.0.tar.gz
tar -xzf 5.0.0.tar.gz
mv app-5.0.0 cms
cd cms

# 2. Instalar dependências via Composer
composer install

# 3. Criar config/app.php
cp config/app_local.example.php config/app_local.php
```

Alternativamente, veja [métodos alternativos de instalação](../../installation.md).

---

## 🗂️ Estrutura de Diretórios

Após a instalação, seu projeto terá esta estrutura:

```
cms/
├── bin/                    # Scripts executáveis
│   └── cake                # CLI do CakePHP (bin/cake)
├── config/                 # Arquivos de configuração
│   ├── app.php             # Configuração principal
│   ├── app_local.php       # Configurações locais (NÃO COMMITAR)
│   ├── bootstrap.php       # Inicialização da aplicação
│   ├── paths.php           # Definições de caminhos
│   └── routes.php          # Rotas da aplicação
├── logs/                   # Logs da aplicação
├── plugins/                # Plugins instalados
├── resources/              # Recursos (locales, etc)
├── src/                    # ⭐ SEU CÓDIGO FICA AQUI
│   ├── Controller/         # Controllers (lógica de requisição/resposta)
│   ├── Model/              # Models (lógica de negócio e dados)
│   │   ├── Entity/         # Entities (representam linhas do DB)
│   │   └── Table/          # Tables (representam tabelas do DB)
│   └── View/               # View classes customizadas
├── templates/              # ⭐ VIEWS (ARQUIVOS .php DE VISUALIZAÇÃO)
│   ├── layout/             # Layouts mestres
│   └── Pages/              # Templates de páginas
├── tests/                  # Testes automatizados
│   ├── Fixture/            # Dados de teste
│   └── TestCase/           # Casos de teste
├── tmp/                    # Arquivos temporários (cache, sessions)
│   ├── cache/              # Cache da aplicação
│   └── logs/               # Logs
├── vendor/                 # Dependências do Composer (NÃO COMMITAR)
├── webroot/                # Arquivos públicos
│   ├── css/                # Arquivos CSS
│   ├── js/                 # Arquivos JavaScript
│   ├── img/                # Imagens
│   ├── .htaccess           # Config Apache (rewrite)
│   └── index.php           # Ponto de entrada da aplicação
├── .gitignore              # Arquivos ignorados pelo Git
├── composer.json           # Dependências PHP
├── index.php               # Fallback (aponta para webroot/)
├── phpunit.xml.dist        # Configuração PHPUnit
└── README.md               # Documentação do projeto
```

??? info "Explicação Detalhada dos Diretórios"

    | Diretório | Você Edita? | Propósito |
    |-----------|-------------|-----------|
    | **src/** | ✅ SIM | Seu código PHP (Models, Controllers, Commands) |
    | **templates/** | ✅ SIM | Suas views (HTML com PHP) |
    | **webroot/** | ✅ SIM | Arquivos públicos (CSS, JS, imagens) |
    | **config/** | ✅ SIM | Configurações (rotas, database, app) |
    | **tests/** | ✅ SIM | Testes automatizados (PHPUnit) |
    | **bin/** | ⚠️ RARAMENTE | Scripts CLI (bin/cake) |
    | **vendor/** | ❌ NUNCA | Dependências Composer (auto-gerenciado) |
    | **tmp/** | ❌ NUNCA | Cache e logs (auto-gerenciado) |
    | **logs/** | ❌ NUNCA | Logs da aplicação (auto-gerenciado) |

!!! danger "NUNCA Commite Estes Arquivos"
    Adicione ao `.gitignore`:

    ```
    /vendor/
    /tmp/
    /logs/
    /config/app_local.php
    .env
    ```

---

## ✅ Verificando a Instalação

Agora vamos confirmar que tudo está funcionando!

### Passo 1: Iniciar Servidor de Desenvolvimento

=== "Linux/macOS"

    ```bash
    cd /caminho/para/cms
    bin/cake server
    ```

    **Saída esperada:**
    ```
    Welcome to CakePHP v5.2 Console
    ---------------------------------------------------------------
    App : src
    Path: /caminho/para/cms/
    PHP : 8.3.0
    ---------------------------------------------------------------
    Built-in server is running in http://localhost:8765/
    You can exit with CTRL-C
    ```

=== "Windows (CMD)"

    ```cmd
    cd C:\xampp\htdocs\cms
    bin\cake server
    ```

    !!! warning "Atenção ao Backslash"
        No Windows, use **backslash** (`\`) em vez de forward slash (`/`):

        - ✅ Correto: `bin\cake server`
        - ❌ Errado: `bin/cake server`

=== "Windows (PowerShell)"

    ```powershell
    cd C:\xampp\htdocs\cms
    .\bin\cake server
    ```

=== "Windows (Git Bash)"

    ```bash
    cd /c/xampp/htdocs/cms
    bin/cake server
    ```

!!! tip "Servidor Rodando em Background"
    Para liberar o terminal:

    ```bash
    # Linux/macOS
    bin/cake server > /dev/null 2>&1 &

    # Windows (PowerShell)
    Start-Process -NoNewWindow bin\cake server
    ```

??? question "Porta 8765 Já Está em Uso?"
    Se ver erro `port 8765 already in use`, use porta diferente:

    ```bash
    bin/cake server -p 8080
    # Ou
    bin/cake server --port 9000
    ```

---

### Passo 2: Acessar Welcome Page

1. Abra seu navegador
2. Acesse: **[http://localhost:8765](http://localhost:8765)**

Você deverá ver a página inicial do CakePHP com vários **chef hats** (ícones de chapéus de chef):

![CakePHP Welcome Page](https://book.cakephp.org/5/en/_images/welcome-page.png)

---

### Passo 3: Interpretar os "Chef Hats"

A welcome page mostra status de várias configurações:

| Ícone | Status | Significado |
|-------|--------|-------------|
| 🟢 **Chef Hat Verde** | ✅ OK | Configuração correta |
| 🔴 **Chef Hat Vermelho** | ❌ Problema | Requer atenção |

**Verificações mostradas:**

1. **PHP Version** - Versão do PHP é compatível (8.1+)
2. **Temporary directory** - `tmp/` é gravável
3. **Logs directory** - `logs/` é gravável
4. **Debug Mode** - Modo debug ativado (desenvolvimento)
5. **DebugKit** - Plugin DebugKit carregado
6. **Database** - Conexão com banco de dados (inicialmente vermelho - normal!)
7. **Sessions** - Configuração de sessão

!!! success "Tudo Verde Exceto Database?"
    **Perfeito!** É esperado que "Database" esteja vermelho neste momento porque ainda não configuramos o MySQL. Vamos fazer isso na [próxima etapa](database.md)!

!!! warning "Algum Outro Item Vermelho?"
    Veja a seção [Troubleshooting](#troubleshooting) abaixo.

---

### Passo 4: Explorar o DebugKit (Opcional)

Se o DebugKit estiver ativo (chef hat verde), você verá uma barra de ferramentas no canto superior direito:

- 📊 **Request** - Dados da requisição (GET/POST params, cookies, headers)
- 🗄️ **SQL Log** - Queries executadas (útil para performance)
- ⏱️ **Timer** - Tempo de renderização da página
- 📁 **Variables** - Variáveis de view disponíveis
- 🛠️ **Environment** - Configurações PHP e CakePHP

!!! tip "DebugKit é Seu Amigo"
    Use intensivamente durante desenvolvimento. Ele revela tudo que está acontecendo nos bastidores!

---

## ❌ Troubleshooting

Problemas comuns e soluções:

??? danger "Erro: `composer: command not found`"

    **Causa:** Composer não está no PATH ou não foi instalado.

    **Solução (Linux/macOS):**
    ```bash
    # Download do instalador
    curl -sS https://getcomposer.org/installer | php

    # Mover para PATH
    sudo mv composer.phar /usr/local/bin/composer
    sudo chmod +x /usr/local/bin/composer

    # Testar
    composer --version
    ```

    **Solução (Windows):**
    1. Baixe [Composer-Setup.exe](https://getcomposer.org/Composer-Setup.exe)
    2. Execute o instalador
    3. Reinicie o terminal

??? danger "Erro: `Your version of PHP is too low`"

    **Causa:** PHP instalado é anterior a 8.1.

    **Solução (Ubuntu/Debian):**
    ```bash
    # Adicionar repositório PPA
    sudo add-apt-repository ppa:ondrej/php
    sudo apt-get update

    # Instalar PHP 8.3
    sudo apt-get install php8.3 php8.3-cli php8.3-mysql \
      php8.3-mbstring php8.3-xml php8.3-intl

    # Verificar
    php -v
    ```

    **Solução (Windows - XAMPP):**
    1. Desinstale XAMPP antigo
    2. Baixe [XAMPP 8.3+](https://www.apachefriends.org/)
    3. Reinstale

??? danger "Erro: `tmp/ is not writable`"

    **Causa:** Permissões incorretas no diretório `tmp/`.

    **Solução (Linux/macOS):**
    ```bash
    cd /caminho/para/cms

    # Dar permissão de escrita
    chmod -R 777 tmp/
    chmod -R 777 logs/

    # Alternativa mais segura (Linux):
    sudo chown -R www-data:www-data tmp/ logs/
    sudo chmod -R 775 tmp/ logs/
    ```

    **Solução (Windows):**
    1. Clique com botão direito em `tmp/`
    2. Propriedades → Segurança → Editar
    3. Adicione permissões de "Gravação" para seu usuário

??? danger "Erro: `port 8765 already in use`"

    **Causa:** Outro processo está usando a porta 8765.

    **Solução 1: Usar outra porta**
    ```bash
    bin/cake server -p 8080
    # Acesse: http://localhost:8080
    ```

    **Solução 2: Encontrar e matar o processo (Linux/macOS)**
    ```bash
    # Descobrir PID usando porta 8765
    lsof -i :8765
    # PID 12345

    # Matar processo
    kill 12345
    ```

    **Solução 2: Encontrar e matar o processo (Windows)**
    ```cmd
    REM Descobrir PID
    netstat -ano | findstr :8765

    REM Matar processo (substitua 12345 pelo PID real)
    taskkill /PID 12345 /F
    ```

??? danger "Erro: `Class 'PDO' not found`"

    **Causa:** Extensão PDO não está habilitada.

    **Solução (Linux):**
    ```bash
    sudo apt-get install php8.3-mysql
    sudo systemctl restart apache2
    ```

    **Solução (Windows - XAMPP):**
    1. Abra `C:\xampp\php\php.ini`
    2. Procure por `;extension=pdo_mysql`
    3. Remova o `;` (descomente):
       ```ini
       extension=pdo_mysql
       ```
    4. Reinicie Apache no painel XAMPP

??? danger "Erro: `intl extension is not loaded`"

    **Causa:** Extensão intl (ICU) não está instalada.

    **Solução (Ubuntu/Debian):**
    ```bash
    sudo apt-get install php8.3-intl
    sudo systemctl restart apache2
    ```

    **Solução (Windows - XAMPP):**
    1. Abra `C:\xampp\php\php.ini`
    2. Descomente:
       ```ini
       extension=intl
       ```
    3. Reinicie Apache

??? danger "Página em Branco (Nada Aparece)"

    **Causa:** Erro fatal do PHP sem display_errors habilitado.

    **Solução:**
    1. Verifique logs:
       ```bash
       # Linux/macOS
       tail -f logs/error.log

       # Windows
       type logs\error.log
       ```

    2. Habilite debug em `config/app.php`:
       ```php
       'debug' => true,
       ```

    3. Verifique PHP error log:
       ```bash
       php -i | grep error_log
       ```

??? danger "Erro 404 em `localhost/cms`"

    **Causa:** Tentando acessar sem servidor de desenvolvimento rodando.

    **Solução:**
    - ❌ **Não acesse:** `http://localhost/cms`
    - ✅ **Acesse:** `http://localhost:8765` (com `bin/cake server` rodando)

    **Alternativa (Configurar Apache):**

    Se quiser usar `http://localhost/cms`, configure um VirtualHost Apache (veja [guia de instalação](../../installation.md)).

---

## 🇧🇷 Recursos para Desenvolvedores Brasileiros

### XAMPP/WAMP no Windows

Muitos desenvolvedores brasileiros usam XAMPP ou WAMP para desenvolvimento local. Aqui estão configurações específicas:

??? tip "Configurar XAMPP para CakePHP"

    **Passo 1: Criar Projeto no htdocs**

    ```cmd
    cd C:\xampp\htdocs
    composer create-project cakephp/app:5 cms
    ```

    **Passo 2: Configurar Virtual Host (Opcional)**

    Edite `C:\xampp\apache\conf\extra\httpd-vhosts.conf`:

    ```apache
    <VirtualHost *:80>
        DocumentRoot "C:/xampp/htdocs/cms/webroot"
        ServerName cms.local

        <Directory "C:/xampp/htdocs/cms/webroot">
            Options Indexes FollowSymLinks
            AllowOverride All
            Require all granted
        </Directory>
    </VirtualHost>
    ```

    Edite `C:\Windows\System32\drivers\etc\hosts`:

    ```
    127.0.0.1    cms.local
    ```

    Reinicie Apache e acesse: `http://cms.local`

??? tip "Configurar WAMP para CakePHP"

    **Similar ao XAMPP:**

    - Diretório: `C:\wamp64\www\cms`
    - VirtualHost: `C:\wamp64\bin\apache\apache2.x.x\conf\extra\httpd-vhosts.conf`

### Hospedagens Brasileiras

| Provedor | Plano Mínimo | PHP 8.1+ | Composer | SSH | Preço Aprox. |
|----------|--------------|----------|----------|-----|--------------|
| **Locaweb** | Hospedagem 3 | ✅ | ✅ | ✅ | R$ 25/mês |
| **Hostgator BR** | Plano Business | ✅ | ✅ | ✅ | R$ 30/mês |
| **UOL Host** | Hospedagem 2 | ✅ | ❌ | ❌ | R$ 20/mês |
| **KingHost** | KingHost II | ✅ | ✅ | ✅ | R$ 35/mês |
| **HostMídia** | Plano Pro | ✅ | ✅ | ✅ | R$ 28/mês |

!!! warning "Verifique SSH e Composer"
    Para instalar CakePHP via Composer, você **precisa de acesso SSH**. Planos básicos geralmente não têm SSH.

### Timezone Brasil

Configure timezone em `config/app.php`:

```php
'App' => [
    'defaultTimezone' => 'America/Sao_Paulo', // Horário de Brasília
    'defaultLocale' => 'pt_BR', // Português brasileiro
],
```

---

## ❓ Perguntas Frequentes

??? question "Preciso instalar o MySQL antes de criar o projeto?"

    **Não necessariamente.** Você pode criar o projeto CakePHP sem MySQL instalado. A welcome page mostrará "Database" em vermelho, mas isso é normal.

    Você só precisa ter o MySQL instalado e rodando **antes da próxima etapa** (configuração de banco de dados).

??? question "Qual a diferença entre `app:5` e `app:5.*` no comando Composer?"

    - `cakephp/app:5` → Instala última versão **estável** da linha 5.x (ex: 5.2.0)
    - `cakephp/app:5.*` → Sempre atualiza para **última** versão 5.x, mesmo patch releases

    **Recomendação:** Use `app:5` para projetos em produção (mais previsível).

??? question "Posso usar PostgreSQL em vez de MySQL?"

    **Sim!** CakePHP suporta:

    - MySQL / MariaDB
    - PostgreSQL
    - SQLite
    - SQL Server

    O tutorial usa MySQL para simplicidade, mas você pode adaptar facilmente.

??? question "Funciona no Mac/Linux/Windows?"

    **Sim**, CakePHP é **multiplataforma**. A principal diferença é:

    - **Linux/macOS:** `bin/cake` (forward slash)
    - **Windows:** `bin\cake` (backslash)

??? question "Posso usar MAMP/XAMPP/WAMP?"

    **Sim!** XAMPP (Windows), MAMP (macOS) e WAMP (Windows) funcionam perfeitamente.

    **Importante:** Certifique-se de que incluem:
    - PHP 8.1+
    - Composer
    - MySQL 5.7+

??? question "Preciso saber MVC antes de começar?"

    **Não!** O tutorial ensina MVC conforme avança. Mas se quiser se preparar, leia primeiro:

    - [Introdução ao CakePHP](../../intro.md)
    - [Convenções do CakePHP](../../intro/conventions.md)

??? question "O projeto criado já tem autenticação?"

    **Não.** O skeleton básico (`cakephp/app`) vem vazio. Você vai adicionar autenticação manualmente na **Etapa 6** do tutorial.

??? question "Por que `vendor/` não deve ser commitado?"

    A pasta `vendor/` contém **dependências do Composer** (20-50 MB). Essas dependências são instaladas automaticamente via:

    ```bash
    composer install
    ```

    **Boas práticas:**
    - ✅ Commitar `composer.json` e `composer.lock`
    - ❌ **NUNCA** commitar `vendor/`

    Adicione ao `.gitignore`:
    ```
    /vendor/
    ```

??? question "Posso renomear o diretório `cms`?"

    **Sim!** O nome do diretório não afeta o funcionamento:

    ```bash
    composer create-project cakephp/app:5 meu-projeto
    cd meu-projeto
    ```

??? question "Como paro o servidor de desenvolvimento?"

    Pressione **Ctrl + C** no terminal onde `bin/cake server` está rodando.

??? question "Posso usar PHP 7.4 ou 8.0?"

    **Não.** CakePHP 5 requer **PHP 8.1+**.

    Se você está preso ao PHP 7.4 ou 8.0, use CakePHP 4:
    ```bash
    composer create-project cakephp/app:4 cms
    ```

---

## 🚀 Próximos Passos

Parabéns! Você completou a instalação do CakePHP. 🎉

**Checklist de Conclusão:**

- [x] PHP 8.1+ instalado e verificado
- [x] Composer instalado e funcional
- [x] Projeto CMS criado via `composer create-project`
- [x] Servidor de desenvolvimento rodando (`bin/cake server`)
- [x] Welcome page exibindo com chef hats verdes (exceto Database)

### Próxima Etapa: Configuração do Banco de Dados

Na próxima etapa, você vai:

1. Criar o banco de dados MySQL
2. Configurar conexão em `config/app_local.php`
3. Criar tabelas (`users`, `articles`, `tags`, `articles_tags`)
4. Popular com dados de teste

<div class="grid cards" markdown>

-   :material-database:{ .lg .middle } **Etapa 2: Database**

    ---

    Configure o banco de dados MySQL e crie as tabelas do CMS

    **Tempo estimado:** 30 minutos

    [:octicons-arrow-right-24: Configurar Database](database.md)

-   :material-github:{ .lg .middle } **Código Completo**

    ---

    Se ficar preso, compare com o repositório final:

    [:octicons-arrow-right-24: GitHub: cms-tutorial](https://github.com/cakephp/cms-tutorial)

</div>

---

## 📚 Recursos Adicionais

- [Documentação Oficial - Instalação](../../installation.md)
- [Estrutura de Pastas Detalhada](../../intro/cakephp-folder-structure.md)
- [Configuração do CakePHP](../../development/configuration.md)
- [Slack da Comunidade CakePHP](https://cakesf.herokuapp.com/)
- [Forum CakePHP](https://discourse.cakephp.org/)

---

!!! quote "Dica Final"
    **Não pule etapas!** Cada etapa do tutorial ensina conceitos fundamentais que serão usados nas próximas. Se encontrar problemas, revise a seção de [Troubleshooting](#troubleshooting) antes de continuar.

**Bom desenvolvimento! 🎂**
