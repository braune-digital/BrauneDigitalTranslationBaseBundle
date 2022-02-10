# BrauneDigitalTranslationBaseBundle

This Symfony2-Bundle provides basic features for translations and offers SonataAdmin integration.
## Requirements
In order to install this bundle you will need:
* KNPDoctrineBehaviors (Basic Translatable functions)
* DoctrineORM (Entity Persistence)  

If you want to use SonataAdmin integration you need a few more:
* SonataAdmin (Backend Management)
* A2LixTranslationFormBundle (Translation Form-Rendering)
* An ckeditor field maybe from `FOSCKEditorBundle`  

## Installation

Just run composer:
```bash
composer require braune-digital/translation-base-bundle
```

And enable the Bundle in AppKernel.php:
```php
public function registerBundles()
    {
        $bundles = array(
          ...
          new BrauneDigital\TranslationBaseBundle\BrauneDigitalTranslationBaseBundle(),
          ...
        );
```
## Configuration
You may want to use an alternative Admin-Layout where you have a Dropdown Menu instead of the translation tabs:  
Just configure SonataAdmin and A2lix to use the extended layouts in your `config.yml`.
```yaml
sonata_admin:
    templates:
        layout: "BrauneDigitalTranslationBaseBundle:admin/translation_layout.html.twig"
a2lix_translation_form:
    templating: "BrauneDigitalTranslationBaseBundle:admin/a2lix_form.html.twig"
```
## Usage

Translatable properties of an entity (*YourEntity*) are moved to a new *YourEntityTranslation* entity: 
  
*Entity*:
```php
<?php

namespace YourBundle\Entity;
use Knp\DoctrineBehaviors\Model as ORMBehaviors;

/**
 * Entity
 */
class Entity
{
    use ORMBehaviors\Translatable\Translatable;
    use \BrauneDigital\TranslationBaseBundle\Model\Translatable\TranslatableMethods {
        \BrauneDigital\TranslationBaseBundle\Model\Translatable\TranslatableMethods::proxyCurrentLocaleTranslation insteadof ORMBehaviors\Translatable\Translatable;
    }

    /**
     * @var int
     */
    private $id;

    /**
     * @param $method
     * @param $arguments
     * @return mixed
     */
    public function __call($method, $arguments)
    {
        return $this->proxyCurrentLocaleTranslation($method, $arguments);
    }

    /**
     * Get id
     *
     * @return int
     */
    public function getId()
    {
        return $this->id;
    }
    

    /**
     * @return mixed
     */
    public function getTitle() {
        $title = $this->translate()->getTitle();
        if ($title) {
            return $title;
        } else {
            return $this->translate($this->getDefaultLocale())->getTitle();
        }
    }

    /**
     * __toString
     *
     * @return string
     */
    public function __toString()
    {
        return $this->getTitle();
    }
}
```

The magic `__call()` method is used to delegate a call to a translation property. For some Bundles (like SonataAdmin) you still need to define the wanted methods (e.g. `getTitle`). In most cases for SonataAdmin a `__toString`-method comes in handy for Relations as well.

*EntityTranslation*:
```php
<?php

namespace YourBundle\Entity;

use Knp\DoctrineBehaviors\Model as ORMBehaviors;
/**
 * EntityTranslation
 */
class EntityTranslation
{

    use ORMBehaviors\Translatable\Translation;

    /**
     * @var string
     */
    protected $title;

    /**
     * @return string
     */
    public function getTitle() {
        return $this->title;
    }

    /**
     * @param string $title
     */
    public function setTitle($title) {
        $this->title = $title;
    }
}
```

You still need to register both Entities in Doctrine, but you do not have to worry about the translation relations as they are dynamically added by DoctrineBehaviors, so you just need to define the normal fields and relations.

## SonataAdmin Integration
This Bundle provides a Basic TranslationAdmin which you might want to extend from:
```php
<?php

namespace YourBundle\Admin;

use BrauneDigital\TranslationBaseBundle\Admin\TranslationAdmin;
use Sonata\AdminBundle\Datagrid\ListMapper;
use Sonata\AdminBundle\Datagrid\DatagridMapper;
use Sonata\AdminBundle\Form\FormMapper;

class EntityAdmin extends TranslationAdmin
{
    // Fields to be shown on create/edit forms
    protected function configureFormFields(FormMapper $formMapper)
    {
		$this->setCurrentLocale();
		$this->buildTranslations($this->subject);

        $formMapper
            ->with('Localization')
			->add('translations', 'a2lix_translations', array(
                'locales' => $this->currentLocale,
                'required_locales' => $this->currentLocale,
				'fields' => array(
					'title' => array(
						'field_type' => 'text',
						'label' => 'Title',
						'empty_data' => ''
					)
				)
			), array(
				'label' => ''
			))->end()
            /*
            ->with('General')
            ...
            other non localized properties
            ...
            ->end()
           */
        ;
    }

    // Fields to be shown on filter forms
    protected function configureDatagridFilters(DatagridMapper $datagridMapper)
    {
    }

    // Fields to be shown on lists
    protected function configureListFields(ListMapper $listMapper)
    {
        $listMapper
            ->addIdentifier('title')
        ;
    }
}
```

## Languages
This Bundle also provides a basic Language Entity, which can be managed through SonataAdmin as well.
## Todo
Add requirements with versions in `composer.json`.
A caching mechanism is needed for the service Router. The Symfony2 router creates own instances and caches them. The service Router however is not able to do that, but maybe a RouteCollection cache would speed things up ;)
