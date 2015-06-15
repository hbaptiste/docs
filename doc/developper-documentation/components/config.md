# Components

## The Config component

This component is used in BackBee to set services and parameters from YAML files into the Container.

### How the Config component work ?

The BackBee Config Component provide a ``Config`` object able to:

* Parse YAML files from a directory;
* Persist this configuration in file or in database;
* Set this configuration in ``BackBee Container``;
* Save and load configuration from ``BackBee Cache``;


```php
<?php

use BackBee\Cache\AbstractCache;
use BackBee\Config\Config;
use BackBee\DependencyInjection\Container;
use BackBee\DependencyInjection\DispatchTagEventInterface;
use BackBee\DependencyInjection\Dumper\DumpableServiceInterface;

class Config implements DispatchTagEventInterface, DumpableServiceInterface
{
    public function __construct(
        $basedir,
        AbstractCache $cache = null,
        Container $container = null,
        $debug = false,
        array $yml_to_ignore = []
    ){/*..*/}
}

?>
```

*This component is not related and not compatible with Symfony Config component.*

### YAML parsing, filtering and extending

The BackBee Config Component can take a valid directory as argument, and merge the configuration according to the data set
in this YAML files. If you don't know YAML, you can take a look to the [Symfony YAML format documentation](http://symfony.com/doc/current/components/yaml/yaml_format.html).

Moreover, BackBee Config component allow you to ignore some files.

```php
<?php

use BackBee\Config\Config;

class Foo
{
    const DIRECTORY = '/my-valid/path';

    public function returnConfiguration(array $filesToIgnore)
    {
        return new Config(self::DIRECTORY, null, null, false, $filesToIgnore);
    }
}

?>
```

This way, the ignored files won't be parsed.


Thanks to the *BackBee environments*, you can also extend and improve your default configuration.
For instance, let's say you want to add a specific configuration when your are in "testing" environment.

You can set the environment this way:


```php
<?php

use BackBee\Config\Config;

class Foo
{
    const DIRECTORY = '/my-valid/path';

    public function TestingConfiguration()
    {
        $config = new Config('/repository/Config', null, null, false, []);

        return $config->setEnvironment('testing');

        /**
         * Now the object is able to parse YAML files from theses directories:
         * /repository/Config
         * /repository/Config/testing
         * and we are also able to parse specific "environment files":
         * /repository/Config/foo.testing.yml
         */
    }
}

?>
```

This is simple and powerful, you can be very specific on the configuration you want to use.
Configurations keys and parameters are merged recursively, starting from base directory to the environnement directory.


### Configuration persisting

The BackBee Config Component can persist configurations in file or in database, you can also create your owns "persistors"
thanks to the ``BackBee\Config\PersistorInterface``.

For now, two persistors are provided in BackBee CMS:

* the **File** Persistor allow you to write and override the main configuration file (config.yml)
* the **Registry** Persistor is a simple database Key/Value system

If we take a look to the BackBee CMS configuration:

```yaml
config:
    save_in_registry: true # default value: true
    persistor: [BackBee\Config\Persistor\File, BackBee\Config\Persistor\Registry]
```

* ``save_in_registry`` is a particular option: if true, this means BackBee will try to look for an overidden configuration
  stored in Registry.
* ``persistor`` is the list of activated persistor, BackBee will loop on each and try to find configuration. If no one 
  provide the requested configuration, an exception will be throw. 

### Configuration caching

The BackBee Config Component is fully compatible with BackBee Cache component.

When you create your Config object, you can pass an instance of ``BackBee\Cache\AbstractCache`` as a second argument.
This way, in non-debug mode (fourth argument, **false** by default), the second time you get the information, the configuration
will be constructed from cache instead of built again from files or database registry.
