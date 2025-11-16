# Utilitários de Segurança

## Visão Geral

`Cake\Utility\Security` fornece métodos estáticos para **criptografia** (AES-256), **hash** (md5, sha1, sha256), e **geração de dados aleatórios seguros** usando fontes criptográficas.

!!! warning
    Nunca use `Security::hash()` para senhas! Use `DefaultPasswordHasher` com bcrypt.

## Criptografia (AES-256)

### `Security::encrypt(string $text, string $key, string $hmacSalt = null)`

Criptografa dados usando AES-256 com checksum HMAC.

```php
use Cake\Utility\Security;

// Gerar chave segura (armazenar em .env)
$key = 'wt1U5MACWJFTXGenFoZoiLwQGrLgdbHA';

// Criptografar
$plaintext = 'Dados sensíveis do usuário';
$encrypted = Security::encrypt($plaintext, $key);

// Armazenar $encrypted no banco de dados
```

### `Security::decrypt(string $cipher, string $key, string $hmacSalt = null)`

Descriptografa dados previamente criptografados.

```php
// Recuperar do banco
$cipher = $user->encrypted_data;

// Descriptografar com mesma chave
$key = 'wt1U5MACWJFTXGenFoZoiLwQGrLgdbHA';
$plaintext = Security::decrypt($cipher, $key);

// Verificar se falhou
if ($plaintext === false) {
    throw new Exception('Falha ao descriptografar (chave incorreta?)');
}
```

!!! info
    Se `$hmacSalt` não for fornecido, usa `Security::getSalt()` automaticamente. Mantém mesma chave e salt para descriptografar!

### HMAC Salt Customizado

```php
$key = 'chave-segura';
$salt = 'meu-salt-customizado';

// Criptografar com salt customizado
$encrypted = Security::encrypt($data, $key, $salt);

// Descriptografar com mesmo salt
$decrypted = Security::decrypt($encrypted, $key, $salt);
```

### Exemplo Prático: Campos Sensíveis

```php
<?php
// src/Model/Entity/User.php
namespace App\Model\Entity;

use Cake\ORM\Entity;
use Cake\Utility\Security;

class User extends Entity
{
    protected array $_hidden = ['ssn'];

    // Getter customizado
    public function _getSSN()
    {
        if (!isset($this->_properties['ssn'])) {
            return null;
        }

        $encrypted = $this->_properties['ssn'];
        $key = env('ENCRYPTION_KEY');

        return Security::decrypt($encrypted, $key);
    }

    // Setter customizado
    public function _setSsn($value)
    {
        $key = env('ENCRYPTION_KEY');
        return Security::encrypt($value, $key);
    }
}

// Uso
$user = $this->Users->newEntity();
$user->ssn = '123-45-6789';  // Será criptografado ao salvar
$this->Users->save($user);

// Recuperar
$user = $this->Users->get($id);
echo $user->ssn;  // Será descriptografado automaticamente
?>
```

## Hashing (Não Criptografado)

### `Security::hash(string $string, string $type = null, mixed $salt = false)`

Hash **unidirecional** (não pode ser revertido). Útil para verificação de integridade, checksums.

!!! warning
    Use **bcrypt/password_hash()** para senhas, não este método!

```php
use Cake\Utility\Security;

// Hash simples (padrão)
$hash = Security::hash('CakePHP Framework');

// Especificar algoritmo
$sha1 = Security::hash('CakePHP', 'sha1');
$sha256 = Security::hash('CakePHP', 'sha256');
$md5 = Security::hash('CakePHP', 'md5');

// Com salt da aplicação
$hash = Security::hash('CakePHP', 'sha1', true);

// Com salt customizado
$hash = Security::hash('CakePHP', 'sha1', 'meu-salt');

// Qualquer algoritmo suportado por PHP
$hash = Security::hash('CakePHP', 'sha512');
```

### Algoritmos Suportados

- md5
- sha1
- sha256
- sha384
- sha512
- ripemd160
- Qualquer outro suportado por `hash()`

### Usar para Checksums

```php
// Verificar integridade de arquivo
$file = fopen('/path/to/file', 'r');
$contents = fread($file, filesize('/path/to/file'));

$checksum = Security::hash($contents, 'sha256');

// Armazenar checksum
$document->checksum = $checksum;
$document->save();

// Verificar integridade depois
$newChecksum = Security::hash($contents, 'sha256');
if ($newChecksum === $document->checksum) {
    echo 'Arquivo íntegro!';
} else {
    echo 'Arquivo foi modificado!';
}
```

## Geração de Dados Aleatórios Seguros

### `Security::randomBytes(int $length)`

Gera `$length` bytes de dados aleatórios de fonte segura.

```php
use Cake\Utility\Security;

// Gerar token de 32 bytes
$token = Security::randomBytes(32);

// Converter para string hexadecimal
$tokenHex = bin2hex($token);

// Usar para session tokens, API keys, etc
```

### `Security::randomString(int $length)`

Gera string aleatória hexadecimal de tamanho específico.

```php
// Gerar string aleatória de 32 caracteres
$randomString = Security::randomString(32);

// Gerar código de verificação de 6 dígitos
$code = substr(Security::randomString(6), 0, 6);

// Gerar token de reset de senha
$resetToken = Security::randomString(64);
```

### Fonte de Aleatoriedade

Usa por ordem de preferência:
1. `random_bytes()` (PHP 7+) - RECOMENDADO
2. `openssl_random_pseudo_bytes()` - Fallback
3. Aviso se nenhuma disponível (inseguro!)

```php
// Fonte automática (recomendado)
$secure = Security::randomBytes(32);  // Usa random_bytes()

// Em produção, garantir openssl instalado
if (!function_exists('random_bytes') && !extension_loaded('openssl')) {
    throw new Exception('Random bytes source not available!');
}
```

## Exemplo Prático Completo

```php
<?php
// src/Model/Table/UsersTable.php
namespace App\Model\Table;

use Cake\ORM\Table;
use Cake\Utility\Security;

class UsersTable extends Table
{
    public function createResetToken($userId)
    {
        // Gerar token seguro
        $token = bin2hex(Security::randomBytes(32));

        // Salvar token com hash (não armazenar token em plain)
        $user = $this->get($userId);
        $user->reset_token = Security::hash($token, 'sha256');
        $user->reset_token_expires = date('Y-m-d H:i:s', time() + 3600);

        $this->save($user);

        return $token;  // Enviar ao usuário
    }

    public function resetPassword($token, $newPassword)
    {
        // Hash do token enviado
        $tokenHash = Security::hash($token, 'sha256');

        // Procurar usuário
        $user = $this->find()
            ->where([
                'reset_token' => $tokenHash,
                'reset_token_expires >' => date('Y-m-d H:i:s')
            ])
            ->first();

        if (!$user) {
            throw new Exception('Invalid or expired token');
        }

        // Usar bcrypt para senha (não este método!)
        $user->password = password_hash($newPassword, PASSWORD_BCRYPT);
        $user->reset_token = null;
        $user->reset_token_expires = null;

        return $this->save($user);
    }

    public function generateApiKey($userId)
    {
        // Gerar chave API segura
        $apiKey = bin2hex(Security::randomBytes(32));

        $user = $this->get($userId);
        $user->api_key = $apiKey;
        $user->api_key_created = date('Y-m-d H:i:s');

        $this->save($user);

        return $apiKey;
    }
}

// src/Controller/UsersController.php
namespace App\Controller;

use Cake\Utility\Security;

class UsersController extends AppController
{
    public function resetPasswordRequest()
    {
        if ($this->request->is('post')) {
            $email = $this->request->getData('email');
            $user = $this->Users->find()
                ->where(['email' => $email])
                ->first();

            if ($user) {
                // Gerar token
                $token = $this->Users->createResetToken($user->id);

                // Enviar email
                $this->sendResetEmail($user, $token);

                $this->Flash->success(__('Check your email for reset link'));
            } else {
                $this->Flash->error(__('User not found'));
            }
        }
    }

    public function resetPassword($token)
    {
        if ($this->request->is('post')) {
            try {
                $this->Users->resetPassword(
                    $token,
                    $this->request->getData('password')
                );

                $this->Flash->success(__('Password reset successfully'));
                return $this->redirect('/login');
            } catch (Exception $e) {
                $this->Flash->error($e->getMessage());
            }
        }

        $this->set(compact('token'));
    }

    public function generateApiKey()
    {
        if ($this->request->is('post')) {
            $userId = $this->request->getAttribute('identity')->id;
            $apiKey = $this->Users->generateApiKey($userId);

            $this->Flash->success(__('API key generated'));
            $this->set(compact('apiKey'));
        }
    }

    public function encrypt()
    {
        // Exemplo: criptografar dados sensíveis antes de salvar
        $sensitiveData = 'Informação confidencial';
        $key = env('ENCRYPTION_KEY');

        $encrypted = Security::encrypt($sensitiveData, $key);

        $this->set(compact('encrypted'));
    }

    public function decrypt()
    {
        $cipher = $this->request->getData('cipher');
        $key = env('ENCRYPTION_KEY');

        try {
            $decrypted = Security::decrypt($cipher, $key);
            $this->set(compact('decrypted'));
        } catch (Exception $e) {
            $this->Flash->error(__('Decryption failed'));
        }
    }
}
?>
```

## Configuração de Segurança

```php
// .env
ENCRYPTION_KEY=wt1U5MACWJFTXGenFoZoiLwQGrLgdbHA
SECURITY_SALT=DYhG93b0qyJfIxfs2guVoUubWwvniR2G0FgaC9mi

// config/app.php
return [
    'Security' => [
        'cipherSeed' => env('SECURITY_SALT', 'default-seed'),
    ]
];
```

## Quick Start: Casos de Uso

### 1. Criptografar Dados Sensíveis

```php
$ssn = '123-45-6789';
$key = env('ENCRYPTION_KEY');
$encrypted = Security::encrypt($ssn, $key);
// Salvar $encrypted no banco
```

### 2. Gerar Token Seguro

```php
$token = bin2hex(Security::randomBytes(32));
// Enviar ao usuário, armazenar hash
```

### 3. Hash para Verificação

```php
$hash = Security::hash($data, 'sha256');
// Comparar hashes depois
```

### 4. NÃO usar para Senhas

```php
// ERRADO - usar isto para senhas
// $hash = Security::hash($password, 'sha1');

// CERTO - usar bcrypt
$hash = password_hash($password, PASSWORD_BCRYPT);
$valid = password_verify($password, $hash);
```

## Recursos Adicionais

- [API Security](https://api.cakephp.org/4.x/class-Cake.Utility.Security.html)
- [Documentação oficial](https://book.cakephp.org/4/en/core-libraries/security.html)
- [PHP password_hash()](https://www.php.net/manual/en/function.password-hash.php)
- [OWASP - Segurança de Senhas](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
