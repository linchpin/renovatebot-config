# Linchpin Renovate Mend Custom Default Config

The default.json is used as a base config for all WordPress based Website projects Linchpin builds or maintains.

## Goals of this configuration file

### General Config
- Group all pull requests into a `maintenance/MM-YYYY` named branch

### Project Build Config

#### npm
- Group all [minor, patch, bump] **build** related `npm` packages into a single grouped pull request
- Create individual [major] **build** related `npm` packages into single pull requests

#### composer 
- Group all [minor, patch, bump] **build** related `composer` packages into a single grouped pull request
- Create individual [major] **build** related `composer` packages into single pull requests

### wordpress.org, wpackagist, packagist.linchpin.com maintenance config

#### WordPress Plugins
- Group all [minor, patch, bump] **plugin** updates from `wpackagist` (wordpress.org) and `packagist.linchpin.com` into a single pull request
- Create individual pull requests for any [major] **plugin** updates from `wpackagist` (wordpress.org) and `packagist.linchpin.com` reviewed by a human before merging

#### WordPress Themes
- Group all [minor, patch, bump] **theme** updates from `wpackagist` (wordpress.org) and `packagist.linchpin.com` into a single pull request
- Create individual pull requests for any [major] **theme** updates from `wpackagist` (wordpress.org) and `packagist.linchpin.com` reviewed by a human before merging

### Linchpin Built Custom WordPress Plugins

#### npm
- Group all [minor, patch, bump] **plugin** related `npm` packages into a single grouped pull request
- Create individual [major] **plugin** related `npm` packages into a single group pull request

#### composer
- Group all [minor, patch, bump] **plugin** related `composer` packages into a single grouped pull request
- Create individual [major] **plugin** related `composer` packages into a single group pull request

### Linchpin Built Custom WordPress Themes 
#### npm
- Group all [minor, patch, bump] **theme** related `npm` packages into a single grouped pull request
- Create individual [major] **theme** related `npm` packages into a single group pull request

#### composer
- Group all [minor, patch, bump] **theme** related `composer` packages into a single grouped pull request
- Create individual [major] **theme** related `composer` packages into a single group pull request
