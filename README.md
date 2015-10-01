# SymfonyLibrinfoCoreBundle
This is the core of Libre Informatique Symfony2 projects

Installation
============

Prequiresites
-------------

- having a working Symfony2 environment
- having created a working Symfony2 app (including your DB and your DB link)
- having composer installed (here in /usr/local/bin/composer, with /usr/local/bin in the path)

Downloading
-----------

  $ composer require libre-informatique/core-bundle dev-master

This will download and install :
* knplabs/knp-menu
* knplabs/knp-menu-bundle
* cocur/slugify
* sonata-project/core-bundle
* sonata-project/cache
* sonata-project/block-bundle
* sonata-project/exporter
* twig/extensions
* sonata-project/admin-bundle
* sonata-project/doctrine-orm-admin-bundle
* libre-informatique/core-bundle

Twig
----

Then you'll probably need to force a higher version of Twig (≥ 1.22.1):

```
  $ composer require twig/twig ^1.22.1
```

Sonata bundles
--------------

Please refer to the Sonata Project's instructions, foundable here :
https://sonata-project.org/bundles/admin/2-3/doc/reference/installation.html

PostgreSQL
----------

Create the database needed

If you are using PostgreSQL as your main database, you'll need to install postgresql-contrib and load the "uuid-ossp" extension :

```
  $ apt-get install postgresql-contrib
  $ echo 'CREATE EXTENSION "uuid-ossp";' | psql [DB]
```

The "libre-informatique" bundles
--------------------------------

Edit your app/AppKernel.php file and add your "libre-informatique" bundle, for instance the "libre-informatique/core-bundle" :

```php
    // app/AppKernel.php
    // ...
    public function registerBundles()
    {
        $bundles = array(
            // ...
            
            // The libre-informatique bundles
            new Librinfo\CoreBundle\CoreBundle(),
            
            // your personal bundles
        );
    }
```

Usages
======

Configuring your SonataAdminBundle interfaces with YAML properties
------------------------------------------------------------------

This feature is provided by Librinfo\CoreBundle to ease the deployment of new bundles & entities. It works quite closely to the  [Sonata\AdminBundle\MapperBaseMapper](https://github.com/sonata-project/SonataAdminBundle/blob/master/Mapper/BaseMapper.php) & [Sonata\AdminBundle\MapperBaseGroupedMapper](https://github.com/sonata-project/SonataAdminBundle/blob/master/Mapper/BaseGroupedMapper.php) (and their children like [Sonata\AdminBundle\Form\FormMapper](https://github.com/sonata-project/SonataAdminBundle/blob/master/Form/FormMapper.php)) ```add()```, ```with()```/```tab()``` & ```remove()``` methods.

So it is structured as :

```
# app/config/config.yml (or any other file that is loaded by your bundle)
parameters:
    librinfo:
        AcmeBundle\Admin\DemoAdmin:               # The Admin class extension
            Sonata\AdminBundle\Form\FormMapper:   # The class of objects that needs to be configured (here the edit/create form)
                remove: [name, id]                # The fields that need to be removed from inheritance (array)
                add:                              # What we want to display (associative array)
                    text:                         # The name of a field that needs to be directly injected (without any tab)
                        type: textarea            # The type of field to display
                        required: false           # Other options refering to the BaseMapper super-class used
                    gfx_tab:                      # A first tab
                        _options:                 # ... with its options (cf. BaseGroupedMapper::with() options)
                            description: tab
                        gfx_group:                # A first group inside the "tab"
                            _options:             # ... with its options (cf. BaseGroupedMapper::with() options)
                                description: with
                            title: ~              # Adding a field, with no option
                            description:
                                type: textarea
                                label: Descriiiiiiption
                                _options:         # Extra options ("fieldDescriptionOptions" in the BaseMapper::add super-class)
                                    translation_domain: fr
                        gfx_group2:
                            field2: ~
            Sonata\AdminBundle\Show\ShowMapper:   # The class of objects that needs to be configured (here the "show" view)
                _copy: Sonata\AdminBundle\Form\FormMapper # indicates to take the configuration of an other class of the current Admin class extension (including its parents configuration)
```

How to use the Librinfo\CoreBundle features ?
---------------------------------------------

After having installed properly the bundle, and learning the configuration reference, just use it in the ```Admin/*Admin.php``` files of your own bundles, and remove the overloaded methods generated (or add in each generated method a call to ```parent::METHOD()```):

```php
<?php
// src/AcmeBundle/Admin/BlogAdmin.php
// ...
use Librinfo\CoreBundle\Admin\AddressableAdmin;
// ...
class DemoAdmin extends BaseAdmin
{
    // ... empty everything original... and if you want to extend those methods, always call parent::METHOD(); somewhere
}
```

Then add, anywhere in your configuration files, the previous kind of configuration, as it will fit your needs... And be sure this file is correctly loaded by Symfony.

Going further...
----------------

#### Configuring a standalone bundle

If you want a standalone bundle, eventually published for composer, and deployed in your vendor directory, you can incorporate the configuration of your ```DemoAdmin``` component within the bundle. Here is an example taken from the [libre-informatique/crm-bundle](https://github.com/libre-informatique/SymfonyLibrinfoCRMBundle) :

```php
<?php
// vendor/libre-informatique/crm-bundle/DependencyInjection/CRMExtension.php
namespace Librinfo\CRMBundle\DependencyInjection;

use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\Config\FileLocator;
use Symfony\Component\HttpKernel\DependencyInjection\Extension;
use Symfony\Component\DependencyInjection\Loader;
use Librinfo\CoreBundle\DependencyInjection\CoreExtension;

/**
 * This is the class that loads and manages your bundle configuration
 *
 * To learn more see {@link http://symfony.com/doc/current/cookbook/bundles/extension.html}
 */
class CRMExtension extends CoreExtension
{
    /**
     * {@inheritdoc}
     */
    public function load(array $configs, ContainerBuilder $container)
    {
        $configuration = new Configuration();
        $config = $this->processConfiguration($configuration, $configs);

        $loader = new Loader\YamlFileLoader($container, new FileLocator(__DIR__.'/../Resources/config'));
        $loader->load('services.yml');
        $loader->load('admin.yml');
        
        $this->mergeParameter('librinfo', $container, __DIR__.'/../Resources/config');
    }
}
```

Then create a ```Resources/config/librinfo.yml``` file in your bundle, matching the previous specifications.

You'll notice the ```use Librinfo\CoreBundle\DependencyInjection\CoreExtension;```, the ```class CRMExtension extends CoreExtension``` and the ```$this->mergeParameter('librinfo', $container, __DIR__.'/../Resources/config');``` that loads the new configuration file, overloading the configuration of the parent bundle (here, ```CoreBundle```).

#### Keeping the original/default SonataAdmin configuration (fields)

In fact when you generate an ```Admin``` component, it comes with the full list of fields representing the object you want to create/edit/list/show... So if you want, for any reason, to keep this configuration available, the best practice with ```Librinfo\CoreBundle``` is to extend this ```Admin``` component as: ```DemoAdmin``` -> ```DemoAdminConcrete```.

```php
<?php
// src/AcmeBundle/Admin/DemoAdminConcrete.php
namespace AcmeBundle\Admin;

use Sonata\AdminBundle\Datagrid\DatagridMapper;
use Sonata\AdminBundle\Datagrid\ListMapper;
use Sonata\AdminBundle\Form\FormMapper;
use Sonata\AdminBundle\Show\ShowMapper;

class DemoAdminConcrete extends DemoAdmin
{
    /**
     * @param DatagridMapper $datagridMapper
     */
    protected function configureDatagridFilters(DatagridMapper $mapper)
    {
        $this->configureFields(__FUNCTION__, $mapper, $this->getGrandParentClass());
        // ...
    }

    /**
     * @param ListMapper $listMapper
     */
    protected function configureListFields(ListMapper $mapper)
    {
        $this->configureFields(__FUNCTION__, $mapper, $this->getGrandParentClass());
        // ...
    }

    /**
     * @param FormMapper $formMapper
     */
    protected function configureFormFields(FormMapper $mapper)
    {
        $this->configureFields(__FUNCTION__, $mapper, $this->getGrandParentClass());
        // ...
    }

    /**
     * @param ShowMapper $showMapper
     */
    protected function configureShowFields(ShowMapper $mapper)
    {
        $this->configureFields(__FUNCTION__, $mapper, $this->getGrandParentClass());
        // ...
    }
}
```

Then to use your ```*AdminConcrete``` class, simply change your ```services``` file as:

```
# app/config/services.yml
services:
# ...
    app.admin.demo:
        class: AcmeBundle\Admin\DemoAdminConcrete
        # ...
```