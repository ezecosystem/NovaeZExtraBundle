# Novactive Extra Bundle for eZ Publish Platform

## Installation

### Step 1: Download Nova eZExtra Bundle using composer

Add NovaeZExtraBundle in your composer.json: 

``` js
{
    "require": {
        "novactive/ezextrabundle": "dev-master"
    }
}
```

Now tell composer to download the bundle by running the command:

``` bash
$ composer.phar update novactive/ezextrabundle
```

### Step 2: Enable the bundle

#### Enable the bundle in the kernel:

``` php
<?php
// ezpublish/EzPublishKernel.php

public function registerBundles() {
    $bundles = array(
        // ...
		new Novactive\Bundle\eZExtraBundle\NovaeZExtraBundle(),
    );
}
```

### Step 3: Add the default routes

Activate the "dev" routes:

``` yml
_novaezetraRoutesDev:
    resource: "@NovaeZExtraBundle/Resources/config/routing/dev.yml"
```

Activate the "prod" routes:

``` yml
_novaezextraRoutes:
    resource: "@NovaeZExtraBundle/Resources/config/routing/main.yml"
```

### Step 4: Clear the cache and check

``` bash
php app|ezpublish/console cache:clear --env=dev
```

Go to : */_novaezextra/dev/test*

## Documentation


### Twig Helper


#### eznova_content_by_contentinfo( location.contentInfo )

``` twig
{% set content = eznova_content_by_contentinfo( location.contentInfo ) %}
```

#### eznova_contenttype_by_content( content )

``` twig
{% set contentType = eznova_contenttype_by_content( content ) %}
```

#### eznova_parentcontent_by_contentinfo( content )

``` twig
{% set contentType = eznova_parentcontent_by_contentinfo( content ) %}
```

> Note : you get the content of the parent on the main location

#### eznova_location_by_content( content )

``` twig
{% set contentType = eznova_location_by_content( content ) %}
```

#### eznova_relation_field_to_content( fieldValue )

``` twig
{% set content = eznova_relation_field_to_content( ez_field_value( content, 'internal_link' ) ) %}
```

> Note : return the direct linked content by the relation object FieldType

#### eznova_relationlist_field_to_content_list( fieldValue )

``` twig
{% set content = eznova_relationlist_field_to_content_list( ez_field_value( content, 'internal_links' ) ) %}
```

> Note : return an array of direct linked contents by the relation objects FieldType

### Picture Controller

``` twig
{{ render( controller( "eZNovaExtraBundle:Picture:alias", { "contentId": content.getField('picture').value.destinationContentId, "fieldIdentifier": "image", "alias": "large" })) }}
```

### Content/Location Helper

The goal was to mimic the old Fetch Content List

    public function contentTree( $parentLocationId, $typeIdentifiers = [], $sortClauses = [], $limit = null, $offset = 0, $additionnalCriterion );
    public function contentList( $parentLocationId, $typeIdentifiers = [], $sortClauses = [], $limit = null, $offset = 0, $additionnalCriterion );
    public function nextByAttribute( $locationId, $attributeIdentifier, $locale, $additionnalCriterions = [] );
    public function nextByPriority( $locationId, $aditionnalCriterions = [] )
    public function previousByAttribute( $locationId, $attributeIdentifier, $locale, $additionnalCriterion = [] )
    public function previousByPriority( $locationId, $additionnalCriterion = [] )
    
> Return an array of Result

Usage:

```twig
    {% for child in children %}
        <h2>{{ ez_field_value( child.content, "title" ) }}</h2>
        {{ ez_render_field( child.content, "overview" ) }}
        <a href="{{ path( "ez_urlalias", { "locationId" : child.content.contentInfo.mainLocationId } ) }}">{{ "Learn more" | trans() }}</a>
    {% endfor %}
```

### Children Provider

Simply inject the children ( and potentially other things on a view Full )

Add your provider in a folder of your bundle

```yml
project.home_page.children.provider:
    class: Project\Bundle\GeneralBundle\ChildrenProvider\YOUCONTENTIDENTIFIERPROVIDERCLASS
    parent: novactive.ezextra.abstract.children.provider
    tags:
        -  { name: novactive.ezextra.children.provider, contentTypeIdentifier: YOUCONTENTIDENTIFIER }
```

You class YOUCONTENTIDENTIFIERPROVIDERCLASS must extend Novactive\Bundle\eZExtraBundle\EventListener\Type

After you need to create a method for each view you display if you want to get children in your template
The goal is to have children on each view.

Ex:

```php
namespace Yoochoose\Bundle\GeneralBundle\ChildrenProvider;
use Novactive\Bundle\eZExtraBundle\EventListener\Type;
use eZ\Publish\API\Repository\Values\Content\Query;
class PersonalizationEngine extends Type
{
    //its also use as default to get the full view children
    public function getChildren($viewParameters, SiteAccess $siteAccess = null)
    {
        return $this->contentHelper->contentList( $this->location->id, [ 'article' ], array( new Query\SortClause\Location\Priority( Query::SORT_ASC ) ), 10);
    }
    
    public function getLineChildren( $viewParameters )
    {
        ...
    }
}
```



