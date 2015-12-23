Configuring your SonataAdminBundle interfaces with YAML properties
==================================================================

This feature is provided by Librinfo\CoreBundle to ease the deployment of new bundles & entities. It works quite closely to the  [Sonata\AdminBundle\MapperBaseMapper](https://github.com/sonata-project/SonataAdminBundle/blob/master/Mapper/BaseMapper.php) & [Sonata\AdminBundle\MapperBaseGroupedMapper](https://github.com/sonata-project/SonataAdminBundle/blob/master/Mapper/BaseGroupedMapper.php) (and their children like [Sonata\AdminBundle\Form\FormMapper](https://github.com/sonata-project/SonataAdminBundle/blob/master/Form/FormMapper.php)) ```add()```, ```with()```/```tab()``` & ```remove()``` methods.

So it is structured as :

```
# app/config/config.yml (or any other file that is loaded by your bundle)
parameters:
    librinfo:
        AcmeBundle\Admin\DemoAdmin:               # The Admin class extension
            managedCollections: []                # Array of collections that need to be managed, in relation with the embeded objects (e.g. House::$doors -> [doors])
                                                  # An other way to do the same thing automagically is to use the trait Librinfo\CoreBundle\Admin\Traits\Embedding within your Sonata Admin form instead of the Librinfo\CoreBundle\Admin\Traits\Base            Sonata\AdminBundle\Form\FormMapper:   # The class of objects that needs to be configured (here the edit/create form)
                remove: [name, id]                # The fields that need to be removed from inheritance (array)
                add:                              # What we want to display (associative array)
                    text:                         # The name of a field that needs to be directly injected (without any tab)
                        type: textarea            # The type of field to display
                        required: false           # Other options refering to the BaseMapper super-class used
                    gfx_tab:                      # A first tab
                        _options:                 # ... with its options (cf. BaseGroupedMapper::with() options)
                            description: tab
                            groupsOrder: [gfx_group2, gfx_group]
                            hideTitle: false      # remove graphically the title of the tab (false by default)
                        gfx_group:                # A first group inside the "tab"
                            _options:             # ... with its options (cf. BaseGroupedMapper::with() options)
                                description: with
                                fieldsOrder:      # You can defined fields order in this key.
                                    - text
                                    - otherField
                                hideTitle: false  # remove graphically the title of the group (false by default)
                            title: ~              # Adding a field, with no option
                            description:
                                type: textarea
                                label: Descriiiiiiption
                                _options:         # Extra options ("fieldDescriptionOptions" in the BaseMapper::add super-class)
                                    translation_domain: fr
                        gfx_group2:
                            field2: ~
                    _options:
                        tabsOrder: []
            Sonata\AdminBundle\Show\ShowMapper:   # The class of objects that needs to be configured (here the "show" view)
                _copy: Sonata\AdminBundle\Form\FormMapper # indicates to take the configuration of an other class of the current Admin class extension (including its parents configuration)
            Sonata\AdminBundle\Datagrid\DatagridMapper:   # The class of objects that needs to be configured (here the "show" view)
                add:
                    _options:
                        orderFields: [title, name]
                    name: ~
                    title:
                        type: XXX
                        filterOption1: xxx
                        filterOption2: yyy
                        field_options:
                            fieldOption1: value1
                            fieldOption2: value2
                        fieldDescriptionOptions1: aaa
                        fieldDescriptionOptions2: bbb
                        field_type: fieldType
                        #_option: fieldType # can replace "field_type"
```

How to use the Librinfo\CoreBundle features ?
---------------------------------------------

After having installed properly the bundle, and learning the configuration reference, just use the ```Librinfo\CoreBundle\Admin\CoreAdmin``` as the parent class of your ```Admin/*Admin.php``` modules:

```php
<?php
// src/AcmeBundle/Admin/DemoAdmin.php
// ...
use Librinfo\CoreBundle\Admin\CoreAdmin;
// ...
class DemoAdmin extends CoreAdmin
{
    // ... empty everything original... and if you want to extend those methods, always call parent::METHOD(); somewhere
}
```

Then you will have to create a "Concrete" (or any other keyword) Admin :

```php
<?php
// src/AcmeBundle/Admin/DemoAdminConcrete.php
// ...
use Librinfo\CoreBundle\Admin\Trait\Base as BaseAdmin;
// ...
class DemoAdminConcrete extends DemoAdmin
{
    use BaseAdmin;
}
```

To finish this, register your service properly in your ```admin.yml``` file:
```
services:
    acme.demo:
        class: AcmeBundle\Admin\DemoAdminConcrete
        arguments: [~, AcmeBundle\Entity\Demo, SonataAdminBundle:CRUD]
        tags:
            - {name: sonata.admin, manager_type: orm}
```

[Back to the README file](../../README.md)