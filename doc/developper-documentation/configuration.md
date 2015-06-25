# Developper documentation

## Configuration

BackBee CMS is powerful but highly configurable.
In this section, all the available configurations are broken down by the key (e.g. doctrine) that defines each possible section of your application configuration.

### Bootstrap configuration (bootstrap.yml)

This configuration file have only two responsabilities:

* ``debug``: set the developper mode: if **true**, the container is generated at each request.
* ``container``: set the container configuration which have 2 parameters
    * ``dump_directory``: set the location of the generated container.
    * ``autogenerate``: if true, the container is generated at each request.

The ``autogenerate`` option is useful when you want to force the generation of the container in "production": don't forget this option have a performance cost.

### Bundles configuration (bundles.yml)

To activate a bundle, you need to register it in this file:

```yaml
# /repository/Config/bundles.yml
demo: BackBee\Bundle\DemoBundle\Demo
toolbar: BackBee\Bundle\ToolbarBundle\Toolbar

```

### Database configuration (doctrine.yml)

We provide a demo sample configuration, you can see all available parameters into the related [Doctrine configuration of Symfony](http://symfony.com/doc/current/reference/configuration/doctrine.html):

```yaml
# /repository/Config/doctrine.yml
dbal:
    driver: pdo_mysql # [mysql|pgsql|sqlite]
    host: localhost
    port: 3306
    dbname: backbee
    user: root
    password: password
    charset: utf8
    collation: utf8_general_ci
    defaultTableOptions: { collate: utf8_general_ci, engine: InnoDB, charset: utf8 }
orm:
    auto_mapping: true
    metadata_cache_driver: apc
    query_cache_driver:
        type: service
        id: my_doctrine_common_cache_service # must be a service or a FQCN
    result_cache_driver:
        type: memcache
        host: localhost
        port: 11211
        instance_class: Memcache

```

### SEO Configuration (metadata.yml)

This is where you set the HTML meta headers you want to configure for each page of BackBee:

```yaml
# /repository/Config/metadata.yml
keywords:
    name: 'keywords'
    content:
        default: 'cms, content manager'
description:
    name: 'description'
    content:
        default: "backbee demo website"
    lang: 'en'
robots:
    name: 'robots'
    content:
        default: ''
        select: ['', 'noindex', 'nofollow', 'noindex, nofollow']

```

This configuration have a real impact in the toolbar, see the generated SEO form in Page context mode:

![SEO form option of BackBee page](http://i.imgur.com/hpUmrob.png)

### Security Configuration (security.yml)

You can configure firewalls and suoders inside this file, a sudoer is a super administrator:

```yaml
# /repository/Config/security.yml
firewalls:
    firewall_name: { pattern: ^/, anonymous: null }
sudoers:
    username: userId

```

