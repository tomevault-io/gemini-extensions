## laravel-api-idempotency

> > RFC-compliant idempotency support for Laravel APIs - Prevent duplicate operations, ensure safe retries

# Laravel API Idempotency

> RFC-compliant idempotency support for Laravel APIs - Prevent duplicate operations, ensure safe retries

## Project Overview

Ce package Laravel fournit une protection d'idempotence pour les APIs. Il cache les reponses en utilisant un header `Idempotency-Key` unique, garantissant que des requetes identiques retournent des reponses identiques sans re-executer l'operation.

### Fonctionnalites principales
- RFC Draft Compliant (IETF Idempotency-Key Header Draft)
- Multiple Storage Drivers (cache, redis, database, dynamodb)
- Payload Fingerprinting (securite contre la reutilisation de cles)
- Conflict Detection (requetes concurrentes)
- Scoping (user, tenant, ip)
- Testing Helpers

---

## Technical Stack

- **PHP**: 8.3+
- **Laravel**: 11.x, 12.x
- **Testing**: Pest PHP
- **Static Analysis**: PHPStan (Larastan)
- **Code Style**: Laravel Pint
- **Refactoring**: Rector

---

## Project Structure

```
laravel-api-idempotency/
├── src/
│   ├── ApiIdempotencyServiceProvider.php
│   ├── IdempotencyManager.php
│   ├── Attributes/
│   │   ├── Idempotent.php
│   │   └── IdempotentExcept.php
│   ├── Commands/
│   │   ├── IdempotencyStatsCommand.php
│   │   ├── IdempotencyCleanupCommand.php
│   │   ├── IdempotencyForgetCommand.php
│   │   └── IdempotencyListCommand.php
│   ├── Contracts/
│   │   ├── StorageDriverInterface.php
│   │   └── ScopeResolverInterface.php
│   ├── Drivers/
│   │   ├── CacheDriver.php
│   │   ├── RedisDriver.php
│   │   ├── DatabaseDriver.php
│   │   └── DynamoDbDriver.php
│   ├── Events/
│   │   ├── IdempotentRequestProcessed.php
│   │   ├── IdempotentRequestReplayed.php
│   │   ├── IdempotentConflictDetected.php
│   │   └── IdempotentPayloadMismatch.php
│   ├── Exceptions/
│   │   ├── IdempotencyException.php
│   │   ├── MissingKeyException.php
│   │   ├── InvalidKeyException.php
│   │   ├── PayloadMismatchException.php
│   │   └── ConflictException.php
│   ├── Facades/
│   │   └── Idempotency.php
│   ├── Http/
│   │   └── Middleware/
│   │       └── IdempotentMiddleware.php
│   ├── Support/
│   │   ├── IdempotencyKey.php
│   │   ├── IdempotencyRecord.php
│   │   ├── PayloadFingerprint.php
│   │   └── helpers.php
│   └── Testing/
│       └── IdempotencyFake.php
├── config/
│   └── api-idempotency.php
├── database/
│   └── migrations/
│       └── create_idempotency_keys_table.php
├── tests/
│   ├── TestCase.php
│   ├── Feature/
│   └── Unit/
├── composer.json
├── phpunit.xml
├── phpstan.neon
├── pint.json
├── rector.php
└── CLAUDE.md
```

---

## Namespace & Autoload

```php
Grazulex\ApiIdempotency\
```

---

## Key Components

### 1. Middleware (IdempotentMiddleware)
Le coeur du package. Intercepte les requetes, verifie le cache, execute ou retourne la reponse cachee.

**Flow:**
1. Extraire le header Idempotency-Key
2. Verifier si la cle existe dans le storage
3. Si oui: verifier le fingerprint, retourner la reponse cachee
4. Si non: verrouiller la cle, executer la requete, stocker la reponse

### 2. Storage Drivers
Interface `StorageDriverInterface` avec implementations:
- `CacheDriver`: Utilise le cache Laravel
- `RedisDriver`: Connection Redis directe pour performance
- `DatabaseDriver`: Persistance avec table dediee
- `DynamoDbDriver`: AWS DynamoDB pour serverless

### 3. IdempotencyManager
Facade principale pour l'acces programmatique:
- `get($key)`: Recuperer une reponse cachee
- `store($key, $response)`: Stocker manuellement
- `forget($key)`: Supprimer une cle
- `skip()`: Marquer la requete actuelle comme non-cacheable

### 4. Scope Resolvers
Isoler les cles par contexte:
- User scope: cles par utilisateur authentifie
- Tenant scope: cles par tenant
- IP scope: cles par adresse IP
- Custom: callback personnalise

### 5. Payload Fingerprinting
Securite contre la reutilisation de cles avec des donnees differentes.
Hash SHA256 de: method + path + body (configurable).

---

## Configuration Reference

Voir `config/api-idempotency.php` pour toutes les options.

Options principales:
- `enabled`: Activer/desactiver globalement
- `header`: Nom du header (default: Idempotency-Key)
- `driver`: cache, redis, database, dynamodb
- `ttl`: Duree de vie en secondes (default: 86400)
- `key.required`: Forcer la presence du header
- `fingerprint.enabled`: Verification du payload
- `scope.enabled`: Isolation des cles
- `conflict.strategy`: wait ou reject

---

## Commands

| Command | Description |
|---------|-------------|
| `idempotency:stats` | Statistiques d'utilisation |
| `idempotency:cleanup` | Nettoyer les cles expirees |
| `idempotency:forget {key}` | Supprimer une cle specifique |
| `idempotency:list` | Lister les requetes recentes |

---

## Events

| Event | Description |
|-------|-------------|
| `IdempotentRequestProcessed` | Nouvelle requete traitee et cachee |
| `IdempotentRequestReplayed` | Reponse cachee retournee |
| `IdempotentConflictDetected` | Requete concurrente detectee |
| `IdempotentPayloadMismatch` | Tentative de reutilisation avec payload different |

---

## Testing Strategy

### Unit Tests
- IdempotencyKey generation
- PayloadFingerprint calculation
- Driver implementations
- Configuration validation

### Feature Tests
- Middleware behavior
- Response caching and replay
- Conflict handling
- Error responses
- Scoping

### Test Helpers
```php
Idempotency::fake();
Idempotency::assertStored($key);
Idempotency::assertReplayed($key);
Idempotency::assertStoredCount($count);
```

---

## Development Commands

```bash
# Install dependencies
composer install

# Run tests
composer test

# Run specific test suites
composer test:unit
composer test:lint
composer test:types

# Fix code style
composer lint

# Full test suite
composer full
```

---

## Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `IDEMPOTENCY_KEY_MISSING` | 400 | Header requis absent |
| `IDEMPOTENCY_KEY_INVALID` | 400 | Format de cle invalide |
| `IDEMPOTENCY_PAYLOAD_MISMATCH` | 422 | Cle reutilisee avec payload different |
| `IDEMPOTENCY_CONFLICT` | 409 | Requete en cours avec meme cle |

---

## Implementation Notes

### Middleware Registration
```php
// bootstrap/app.php (Laravel 11+)
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'idempotent' => \Grazulex\ApiIdempotency\Http\Middleware\IdempotentMiddleware::class,
    ]);
})
```

### Route Usage
```php
Route::post('/payments', PaymentController::class)->middleware('idempotent');
Route::post('/orders', OrderController::class)->middleware('idempotent:ttl=172800,required');
```

### Attribute Usage
```php
#[Idempotent]
class PaymentController extends Controller
{
    #[IdempotentExcept]
    public function index() { }
}
```

---

## References

- [IETF Idempotency-Key Header Draft](https://datatracker.ietf.org/doc/draft-ietf-httpapi-idempotency-key-header/)
- [Stripe Idempotent Requests](https://stripe.com/docs/api/idempotent_requests)
- [laravel-apiroute](https://github.com/Grazulex/laravel-apiroute) - Reference structure

---
> Source: [Grazulex/laravel-api-idempotency](https://github.com/Grazulex/laravel-api-idempotency) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-09 -->
