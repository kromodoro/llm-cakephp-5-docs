# Formatação de Números em CakePHP

## Visão Geral

`Cake\I18n\Number` fornece métodos estáticos para formatar números de acordo com **locale**, **moeda**, **precisão decimal** e outras formatações. Automaticamente respeita o locale configurado em `I18n::setLocale()`.

!!! tip
    Number é mais poderoso que NumberHelper (sem dependência de View). Use em controllers, services e models.

## Formatação de Moedas

### `Number::currency(mixed $value, string $currency = null, array $options = [])`

Formata valor monetário com símbolo e formatação de locale.

```php
use Cake\I18n\Number;

// USD (padrão em en_US)
echo Number::currency(1234.56, 'USD');
// Saída: $1,234.56

// EUR
echo Number::currency(1234.56, 'EUR');
// Saída: €1.234,56 (em pt_BR)

// GBP
echo Number::currency(1234.56, 'GBP');
// Saída: £1,234.56

// Moeda padrão (sem especificar)
Number::setDefaultCurrency('BRL');
echo Number::currency(1234.56);  // R$ 1.234,56
```

### Opções de Formatação

```php
echo Number::currency(1234.5, 'USD', [
    'before' => '$',              // Antes do número
    'after' => ' USD',            // Depois do número
    'zero' => 'Free!',            // Valor zero customizado
    'places' => 2,                // Casas decimais
    'precision' => 2,             // Máximo de casas decimais
    'locale' => 'en_US',          // Locale específico
    'fractionSymbol' => ' cents',  // Símbolo de fração
    'useIntlCode' => true,        // Usar código ISO ao invés de símbolo
]);

// Com separador de milhares
echo Number::currency(1000000, 'USD');
// $1,000,000.00
```

### Formato Accounting

```php
// Formato contábil (valores negativos em parênteses)
Number::setDefaultCurrencyFormat(Number::FORMAT_CURRENCY_ACCOUNTING);

echo Number::currency(-1234.56, 'USD');
// $(1,234.56)
```

## Precisão Decimal

### `Number::precision(float $value, int $precision = 3, array $options = [])`

Formata número com número específico de casas decimais com arredondamento.

```php
echo Number::precision(456.91873645, 2);
// 456.92 (arredonda)

echo Number::precision(456.91873645, 3);
// 456.919

echo Number::precision(0.3333333, 2);
// 0.33

// Com opciones
echo Number::precision(456.91873645, 2, [
    'before' => 'R$',
    'after' => ' BRL'
]);
// R$456.92 BRL
```

## Percentuais

### `Number::toPercentage(mixed $value, int $precision = 2, array $options = [])`

Formata como percentual com símbolo `%`.

```php
echo Number::toPercentage(0.456);
// 45.60%

echo Number::toPercentage(45.6);
// 45.60%

// Com multiply (para decimais)
echo Number::toPercentage(0.456, 1, ['multiply' => true]);
// 45.6%

// Opções
echo Number::toPercentage(0.456, 2, [
    'locale' => 'pt_BR',
    'before' => '~',
    'after' => ' de desconto'
]);
// ~45.60% de desconto
```

## Tamanhos Legíveis

### `Number::toReadableSize(string|int $size)`

Converte bytes em formato legível (B, KB, MB, GB, TB).

```php
echo Number::toReadableSize(0);           // 0 Byte
echo Number::toReadableSize(1024);        // 1 KB
echo Number::toReadableSize(1321205.76);  // 1.26 MB
echo Number::toReadableSize(1048576);     // 1 MB
echo Number::toReadableSize(5368709120);  // 5 GB
echo Number::toReadableSize(2199023255552); // 2 TB

// Com string
echo Number::toReadableSize('1024');      // 1 KB
echo Number::toReadableSize('5MB');       // 5 MB (já é legível)
```

Útil para exibir tamanho de arquivos:

```php
// Em controller
$file = fopen($path, 'r');
$size = filesize($path);

$this->set('fileSize', Number::toReadableSize($size));

// Em view
<?= __('File size: {0}', $fileSize) ?>
```

## Formatação Genérica

### `Number::format(mixed $value, array $options = [])`

Formatação completa com máximo controle.

```php
echo Number::format(1236.334);
// 1,236 (sem decimais por padrão)

// Com opções
echo Number::format(1236.334, [
    'places' => 2,           // Casas decimais
    'precision' => 2,        // Máximo de casas
    'pattern' => '#,###.00', // Pattern ICU
    'locale' => 'fr_FR',     // Locale
    'before' => 'R$',        // Antes
    'after' => ' BRL'        // Depois
]);
// R$1,236.33 BRL

// Locale brasileiro
echo Number::format(1234567.89, ['locale' => 'pt_BR']);
// 1.234.567,89
```

## Números Ordinais

### `Number::ordinal(mixed $value, array $options = [])`

Converte número para forma ordinal.

```php
echo Number::ordinal(1);   // 1st
echo Number::ordinal(2);   // 2nd
echo Number::ordinal(3);   // 3rd
echo Number::ordinal(4);   // 4th
echo Number::ordinal(21);  // 21st
echo Number::ordinal(22);  // 22nd
echo Number::ordinal(23);  // 23rd

// Em português
echo Number::ordinal(1, ['locale' => 'pt_BR']);   // 1º
echo Number::ordinal(2, ['locale' => 'pt_BR']);   // 2º
```

## Delta (Mudança de Valor)

### `Number::formatDelta(mixed $value, array $options = [])`

Formata mudanças de valor com sinal +/-.

```php
echo Number::formatDelta(100);    // +100
echo Number::formatDelta(-100);   // -100
echo Number::formatDelta(0);      // 0

// Com opciones
echo Number::formatDelta(1234.56, [
    'places' => 2,
    'before' => '[',
    'after' => ']'
]);
// [+1,234.56]

// Cores em HTML
echo Number::formatDelta(50, ['before' => '<span class="gain">', 'after' => '</span>']);
// <span class="gain">+50</span>
```

## Configuração Global

### `Number::config(string $locale, int $type = NumberFormatter::DECIMAL, array $options = [])`

Configura formatter padrão reutilizável:

```php
// Configurar para locale e tipo
Number::config('pt_BR', \NumberFormatter::CURRENCY, [
    'pattern' => '#,##,##0.00',  // Pattern customizado
]);

// Padrão para formato específico
Number::config('en_US', \NumberFormatter::DECIMAL, [
    'pattern' => '#,##0.##'
]);
```

## Setters e Getters

### Currency Padrão

```php
// Definir moeda padrão
Number::setDefaultCurrency('BRL');

// Obter moeda padrão
$currency = Number::getDefaultCurrency();  // 'BRL'
```

## Exemplo Prático Completo

```php
<?php
// config/bootstrap.php
use Cake\I18n\I18n;
use Cake\I18n\Number;

// Configurar locale padrão
I18n::setLocale('pt_BR');

// Definir moeda padrão
Number::setDefaultCurrency('BRL');

// src/Controller/ProductsController.php
namespace App\Controller;

use Cake\I18n\Number;

class ProductsController extends AppController
{
    public function view($id)
    {
        $product = $this->Products->get($id);

        // Formatar preço
        $product->formatted_price = Number::currency($product->price);

        // Formatar percentual de desconto
        if ($product->discount > 0) {
            $product->discount_percent = Number::toPercentage(
                $product->discount / 100,
                2,
                ['multiply' => false]
            );
        }

        // Tamanho em download (se aplicável)
        if (isset($product->file_size)) {
            $product->file_size_readable = Number::toReadableSize(
                $product->file_size
            );
        }

        $this->set(compact('product'));
    }

    public function sales()
    {
        $sales = $this->Products->Sales->find('all');

        // Formatar dados de vendas
        $formatted = [];
        foreach ($sales as $sale) {
            $formatted[] = [
                'product' => $sale->product->name,
                'price' => Number::currency($sale->price),
                'quantity' => $sale->quantity,
                'total' => Number::currency($sale->total),
                'change' => Number::formatDelta($sale->price_change)
            ];
        }

        $this->set(compact('formatted'));
    }
}

// src/Template/Products/view.html.php
<?php
use Cake\I18n\Number;
?>

<div class="product">
    <h1><?= h($product->name) ?></h1>

    <!-- Preço formatado -->
    <div class="price">
        <strong><?= Number::currency($product->price) ?></strong>
    </div>

    <!-- Desconto com percentual -->
    <?php if ($product->discount > 0): ?>
        <div class="discount">
            <?= __('Discount: ') ?>
            <?= Number::toPercentage($product->discount / 100) ?>
        </div>
    <?php endif; ?>

    <!-- Tamanho do arquivo -->
    <?php if (isset($product->file_size)): ?>
        <div class="download">
            <?= __('File size: ') ?>
            <?= Number::toReadableSize($product->file_size) ?>
        </div>
    <?php endif; ?>

    <!-- Quantidade em estoque -->
    <div class="stock">
        <?= __n(
            '{0} item in stock',
            '{0} items in stock',
            $product->stock,
            $product->stock
        ) ?>
    </div>
</div>
?>
```

## Quick Start

```php
use Cake\I18n\Number;
use Cake\I18n\I18n;

// 1. Definir locale padrão
I18n::setLocale('pt_BR');
Number::setDefaultCurrency('BRL');

// 2. Formatar moeda
echo Number::currency(99.99);        // R$ 99,99

// 3. Formatar percentual
echo Number::toPercentage(0.25);     // 25.00%

// 4. Tamanho legível
echo Number::toReadableSize(1048576); // 1 MB

// 5. Números com precisão
echo Number::precision(3.14159, 2);   // 3.14
```

## Recursos Adicionais

- [API Number](https://api.cakephp.org/4.x/class-Cake.I18n.Number.html)
- [Documentação oficial](https://book.cakephp.org/4/en/core-libraries/number.html)
- [Formatação ICU](https://www.unicode.org/reports/tr35/tr35-numbers.html)
