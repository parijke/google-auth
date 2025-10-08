# Upgrade to PHP 8.2+ and Symfony 7.3

## Summary

This document outlines the changes made to upgrade the application from PHP 7.2.5 and Symfony 5.0 to PHP 8.2+ and Symfony 7.3.

## Major Changes

### PHP Version
- **Before:** PHP ^7.2.5
- **After:** PHP ^8.2

### Symfony Version
- **Before:** Symfony 5.0.*
- **After:** Symfony 7.3.*

### Key Package Updates

#### Two-Factor Authentication Bundle
- **Before:** `scheb/two-factor-bundle` ^4.13
- **After:** `scheb/2fa-bundle` ^7.0 + `scheb/2fa-google-authenticator` ^7.0
- **Note:** The package was renamed and split into separate modules

#### Symfony Components
All Symfony components upgraded to 7.3.*:
- symfony/framework-bundle: 7.3.4
- symfony/security-bundle: 7.3.4
- symfony/console: 7.3.4
- And all other Symfony components

### Code Changes

#### 1. Kernel.php
- Simplified to use only `MicroKernelTrait` without manual configuration methods
- Removed manual route and container configuration (now handled by trait)

#### 2. Controllers
- **Before:** Used Doctrine-style annotations with `@Route`
- **After:** Uses PHP 8 attributes with `#[Route()]`
- Added return type hints to all controller methods

**Example:**
```php
// Before
/**
 * @Route("/login", name="app_login")
 */
public function login(AuthenticationUtils $authenticationUtils): Response

// After
#[Route('/login', name: 'app_login')]
public function login(AuthenticationUtils $authenticationUtils): Response
```

#### 3. User Entity (src/Entity/User.php)
- **Before:** Used Doctrine annotations and old UserInterface methods
- **After:** Uses PHP 8 attributes and new interface methods

Key changes:
- Removed `getUsername()` → Replaced with `getUserIdentifier()`
- Removed `getSalt()` → No longer needed
- Added `PasswordAuthenticatedUserInterface` implementation
- Converted all ORM annotations to attributes
- Added proper type hints to all properties

#### 4. LoginAuthenticator
- **Before:** Extended `AbstractFormLoginAuthenticator` from Guard component
- **After:** Extends new `AbstractLoginFormAuthenticator` using the Passport system

Key changes:
- Removed manual credential checking methods
- Uses new `Passport` with `UserBadge`, `PasswordCredentials`, `CsrfTokenBadge`
- Simplified constructor (no longer needs password encoder or CSRF manager)
- Uses constructor property promotion

#### 5. UserRepository
- **Before:** `upgradePassword(UserInterface $user, string $newEncodedPassword)`
- **After:** `upgradePassword(PasswordAuthenticatedUserInterface $user, string $newHashedPassword)`
- Updated namespace: `Doctrine\Common\Persistence\ManagerRegistry` → `Doctrine\Persistence\ManagerRegistry`

#### 6. RegistrationController
- **Before:** Used `UserPasswordEncoderInterface` and `getDoctrine()`
- **After:** Uses `UserPasswordHasherInterface` and dependency injection of `EntityManagerInterface`
- Replaced `encodePassword()` with `hashPassword()`

#### 7. Form Classes (RegistrationFormType)
- Added void return type to `buildForm()` and `configureOptions()` methods

### Configuration Changes

#### 1. security.yaml
```yaml
# Before:
security:
    encoders:
        App\Entity\User:
            algorithm: auto
    firewalls:
        main:
            anonymous: lazy
            guard:
                authenticators:
                    - App\Security\LoginAuthenticator

# After:
security:
    password_hashers:
        App\Entity\User:
            algorithm: auto
    firewalls:
        main:
            lazy: true
            custom_authenticator: App\Security\LoginAuthenticator
```

- Replaced `encoders` with `password_hashers`
- Replaced `anonymous: lazy` with `lazy: true`
- Replaced `guard.authenticators` with `custom_authenticator`
- Updated access control: `IS_AUTHENTICATED_ANONYMOUSLY` → `PUBLIC_ACCESS`

#### 2. doctrine.yaml
```yaml
# Before:
mappings:
    App:
        type: annotation

# After:
mappings:
    App:
        type: attribute
```

#### 3. doctrine_migrations.yaml
```yaml
# Before:
doctrine_migrations:
    dir_name: '%kernel.project_dir%/src/Migrations'
    namespace: DoctrineMigrations

# After:
doctrine_migrations:
    migrations_paths:
        'DoctrineMigrations': '%kernel.project_dir%/src/Migrations'
    enable_profiler: false
```

#### 4. config/routes/annotations.yaml
Changed type from `annotation` to `attribute` for both controllers and kernel.

#### 5. config/bundles.php
- Removed `Sensio\Bundle\FrameworkExtraBundle\SensioFrameworkExtraBundle` (deprecated/removed in Symfony 6+)

#### 6. Two-Factor Authentication Configuration
- Old file: `config/packages/scheb_two_factor.yaml` (removed)
- New file: `config/packages/scheb_2fa.yaml`
- Updated token configuration to use new Symfony 6+ security tokens

### Removed Files
- `config/packages/sensio_framework_extra.yaml`
- `config/packages/scheb_two_factor.yaml`

### New Files
- `config/packages/scheb_2fa.yaml`
- `config/routes/scheb_2fa.yaml`

## Testing

After upgrade, the following tests were successful:
- ✅ `php bin/console cache:clear` - Cache clears successfully
- ✅ `php bin/console --version` - Shows Symfony 7.3.4
- ✅ `php bin/console about` - Displays PHP 8.3.6 and Symfony 7.3.4
- ✅ `php bin/console debug:router` - All routes are properly registered with attributes
- ✅ `php bin/console list` - All console commands available

## Compatibility

- **PHP:** Requires PHP 8.2 or higher (tested with PHP 8.3.6)
- **Symfony:** Uses Symfony 7.3.* components
- **Database:** Compatible with existing database schema (no migrations needed for upgrade)

## Next Steps

1. Test all application features thoroughly
2. Update any custom code that may use deprecated Symfony features
3. Consider upgrading to PHP 8.4 once all dependencies support it
4. Review and update any third-party bundles for Symfony 7 compatibility

## References

- [Symfony 7.3 Release Notes](https://symfony.com/releases/7.3)
- [Symfony Security Upgrade Guide](https://symfony.com/doc/current/security.html)
- [Scheb 2FA Bundle v7 Documentation](https://symfony.com/bundles/SchebTwoFactorBundle/7.x/index.html)
