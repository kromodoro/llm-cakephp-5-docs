# Classe Plugin: Utilitários para Plugins

## Visão Geral

`Cake\Core\Plugin` é uma classe utilitária com métodos estáticos para **localizar plugins**, verificar se estão **carregados**, e acessar seus **caminhos de recursos** (templates, configurações, classes).

!!! info
    Use Plugin para descobrir localizações dinâmicas de plugins sem hardcoding de caminhos.

## Localização de Plugins

### `Plugin::path(string $plugin)`

Retorna o caminho completo absoluto para o diretório do plugin.

```php
use Cake\Core\Plugin;

// Localizar plugin
$debugKitPath = Plugin::path('DebugKit');
// /var/www/myapp/plugins/DebugKit/

$customPath = Plugin::path('MyCustomPlugin');
// /var/www/myapp/plugins/MyCustomPlugin/

// Acessar arquivo dentro do plugin
$configFile = Plugin::path('MyPlugin') . 'config' . DS . 'routes.php';
```

### `Plugin::classPath(string $plugin)`

Retorna caminho para diretório de classes do plugin (src/).

```php
$classPath = Plugin::classPath('Users');
// /var/www/myapp/plugins/Users/src/

// Construir caminho para classe
$modelPath = Plugin::classPath('Users') . 'Model' . DS . 'Table' . DS . 'UsersTable.php';
```

### `Plugin::templatePath(string $plugin)`

Retorna caminho para templates do plugin.

```php
$templatePath = Plugin::templatePath('Admin');
// /var/www/myapp/plugins/Admin/templates/

// Localizar template específico
$layoutPath = Plugin::templatePath('Admin') . 'layout' . DS . 'admin.php';
```

### `Plugin::configPath(string $plugin)`

Retorna caminho para arquivos de configuração do plugin.

```php
$configPath = Plugin::configPath('MyPlugin');
// /var/www/myapp/plugins/MyPlugin/config/

// Carregar arquivo config
$routes = include Plugin::configPath('MyPlugin') . 'routes.php';
$settings = include Plugin::configPath('MyPlugin') . 'settings.php';
```

## Verificação de Status

### `Plugin::isLoaded(string $plugin)`

Verifica se um plugin está carregado na aplicação.

```php
use Cake\Core\Plugin;

if (Plugin::isLoaded('DebugKit')) {
    // DebugKit está carregado
    $debugBar = new DebugKit\DebugBar();
}

// Verificar múltiplos plugins
$requiredPlugins = ['Authentication', 'Authorization', 'Crud'];
foreach ($requiredPlugins as $plugin) {
    if (!Plugin::isLoaded($plugin)) {
        throw new Exception("Required plugin not loaded: {$plugin}");
    }
}
```

### `Plugin::loaded()`

Retorna array com todos os plugins carregados.

```php
$allPlugins = Plugin::loaded();
// ['DebugKit', 'Cake/TinyAuth', 'Crud', 'Search']

// Verificar quantidade
$count = count(Plugin::loaded());

// Iterar plugins
foreach (Plugin::loaded() as $plugin) {
    echo $plugin . ' is loaded' . PHP_EOL;
}
```

## Exemplo Prático Completo

### Sistema de Plugins Dinâmico

```php
<?php
// src/Utility/PluginManager.php
namespace App\Utility;

use Cake\Core\Plugin;

class PluginManager
{
    /**
     * Descobrir todos os comandos de um plugin
     */
    public static function getPluginCommands($pluginName)
    {
        if (!Plugin::isLoaded($pluginName)) {
            return [];
        }

        $path = Plugin::classPath($pluginName) . 'Command' . DS;

        if (!is_dir($path)) {
            return [];
        }

        $commands = [];
        foreach (glob($path . '*.php') as $file) {
            $className = basename($file, '.php');
            $commands[] = $className;
        }

        return $commands;
    }

    /**
     * Copiar assets de plugin para webroot
     */
    public static function publishAssets($pluginName)
    {
        if (!Plugin::isLoaded($pluginName)) {
            return false;
        }

        $sourceDir = Plugin::path($pluginName) . 'webroot' . DS;
        $destDir = WWW_ROOT . 'plugins' . DS . strtolower($pluginName) . DS;

        if (!is_dir($sourceDir)) {
            return false;
        }

        // Copiar recursivamente
        exec("cp -r {$sourceDir} {$destDir}");

        return true;
    }

    /**
     * Carregar configuração de plugin
     */
    public static function loadPluginConfig($pluginName, $configName)
    {
        if (!Plugin::isLoaded($pluginName)) {
            throw new Exception("Plugin {$pluginName} is not loaded");
        }

        $configPath = Plugin::configPath($pluginName) . $configName . '.php';

        if (!file_exists($configPath)) {
            throw new Exception("Config file not found: {$configPath}");
        }

        return include $configPath;
    }

    /**
     * Listar todos os templates de um plugin
     */
    public static function listPluginTemplates($pluginName)
    {
        if (!Plugin::isLoaded($pluginName)) {
            return [];
        }

        $path = Plugin::templatePath($pluginName);
        $templates = [];

        foreach (glob($path . '**/*.php') as $file) {
            $relative = str_replace($path, '', $file);
            $templates[] = $relative;
        }

        return $templates;
    }
}

// Uso
$commands = PluginManager::getPluginCommands('MyPlugin');
$config = PluginManager::loadPluginConfig('Admin', 'settings');
$templates = PluginManager::listPluginTemplates('Frontend');
?>
```

### Plugin com Múltiplos Recursos

```php
<?php
// plugins/MyPlugin/src/Controller/ExampleController.php
namespace MyPlugin\Controller;

use Cake\Core\Plugin;

class ExampleController extends AppController
{
    public function initialize()
    {
        parent::initialize();

        // Carregar templates do plugin
        $this->viewBuilder()->setTemplatePath(
            Plugin::templatePath('MyPlugin')
        );

        // Acessar config do plugin
        $settings = include Plugin::configPath('MyPlugin') . 'settings.php';
        $this->set(compact('settings'));
    }

    public function view()
    {
        // Localizar arquivo no plugin
        $pluginFile = Plugin::path('MyPlugin') . 'data' . DS . 'file.json';

        if (file_exists($pluginFile)) {
            $data = json_decode(file_get_contents($pluginFile), true);
            $this->set(compact('data'));
        }
    }
}
?>
```

## Quick Start

```php
use Cake\Core\Plugin;

// 1. Verificar se carregado
if (Plugin::isLoaded('MyPlugin')) {
    // 2. Obter caminhos
    $pluginPath = Plugin::path('MyPlugin');
    $classPath = Plugin::classPath('MyPlugin');
    $templatePath = Plugin::templatePath('MyPlugin');
    $configPath = Plugin::configPath('MyPlugin');

    // 3. Usar caminhos
    $settings = include $configPath . 'settings.php';
}

// 4. Listar todos plugins
$allPlugins = Plugin::loaded();
```

## Recurso Adicionais

- [API Plugin](https://api.cakephp.org/4.x/class-Cake.Core.Plugin.html)
- [Documentação oficial](https://book.cakephp.org/4/en/plugins.html)
- [Criando Plugins](https://book.cakephp.org/4/en/plugins/how-to-use-plugins.html)
