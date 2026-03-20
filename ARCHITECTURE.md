# Architecture: yii2-composer

## Purpose

A Composer plugin for Yii2 framework packages. Automates post-install/update tasks specific to Yii2: writing class maps, updating the Yii extension registry file, and running framework bootstrap actions when dependencies are installed or updated.

## Directory Structure

```
Installer.php   - Composer plugin: handles package install/update/uninstall events
Plugin.php      - Composer Plugin entry point: registers event subscribers
```

## Key Design Decisions

- **Composer plugin model**: Implements `Composer\Plugin\PluginInterface` and subscribes to `POST_PACKAGE_INSTALL`, `POST_PACKAGE_UPDATE`, and `POST_PACKAGE_UNINSTALL` events, keeping framework-specific logic out of application code.
- **Extension registry**: Maintains a `yii\extensions.php` file in the project's `vendor/yiisoft/extensions.php` path. Each installed Yii2 package contributes its name, version, and bootstrap class to this registry, which the Yii2 bootstrap process reads at runtime.
- **Class map generation**: Writes optimised class maps for Yii2 packages so the framework's autoloader can find classes without relying solely on Composer's default autoloader.

## Extension Points

- No public extension points — this is framework infrastructure. Yii2 extension packages declare their bootstrap class in `composer.json` under `extra.bootstrap`; the plugin reads and registers it automatically.

## Dependency Flow

```
composer install / composer update
  └─> Plugin::activate() — register event listeners
  └─> POST_PACKAGE_INSTALL event
        └─> Installer::updatePackage()
              └─> read extra.bootstrap from package composer.json
              └─> update vendor/yiisoft/extensions.php registry
              └─> regenerate class map
```
