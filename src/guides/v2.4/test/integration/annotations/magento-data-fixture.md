---
group: testing
title: Data fixture annotation
---

A data fixture is a PHP script that sets data you want to reuse in your test.
The script can be defined in a separate file or as a local test case method.

Use data fixtures to prepare a database for tests.
The Integration Testing Framework (ITF) reverts the database to its initial state automatically.
To set up a date fixture, use the `@magentoDataFixture` annotation.

## Format

`@magentoDataFixture` takes an argument that points to the data fixture as a filename or local method.

```php?start_inline=1
/**
 * @magentoDataFixture <script_filename>|<method_name>|<fully_qualified_class_name> [as:alias | with:{}]
 */
```

-  `<script_filename>` is a filename of the PHP script.
-  `<method_name>` is a name of the method declared in the current class.
-  `<fully_qualified_class_name>` is the fully qualified name of a class that implements
   `Magento\TestFramework\FixtureInterface` or `Magento\TestFramework\RevertibleFixtureInterface`.

## Principles

1. Do not use a direct database connection in fixtures to avoid dependencies on the database structure and vendor.
1. Use an application API to implement your data fixtures.
1. A method that implements a data fixture must be declared as `public` and `static`.
1. Fixtures declared at a test level have a higher priority then fixtures declared at a test case level.
1. Test case fixtures are applied to each test in the test case, unless a test has its own fixtures declared.
1. Annotation declaration at a test case level doesn't affect tests that have their own annotation declarations.

### For Parametrized Data Fixtures
1. Fixture class MUST implement `Magento\TestFramework\FixtureInterface`
1. Fixture class MUST implement `Magento\TestFramework\RevertibleFixtureInterface` if the data created by the fixture
   is revertible. For instance fixture that creates an entity (e.g. product).
1. Fixture class namespace MUST match `Magento\<Module Name>\Fixture`
1. Fixture class SHOULD follow single responsibility principle. See example with decoupled fixtures class (link?).
1. Fixture alias SHOULD be camelcase
1. Fixture JSON parameter MUST be a valid JSON string

## Usage

As mentioned above, there are three ways to declare fixtures:

-  as a legacy PHP script file that is used by other tests and test cases.
-  as a local method that is used by other tests in the test cases.
-  as a Parametrized Fixture class that implements
   `Magento\TestFramework\FixtureInterface` or `Magento\TestFramework\RevertibleFixtureInterface`

### Fixture as a legacy PHP script file

Define the fixture in a separate file when you want to reuse it in different test cases.
To declare the fixture, use the following conventions for a path

-  Relative to `dev/tests/integration/<test suite directory>`
-  With forward slashes `/`
-  No leading slash

   Example: `Magento/Cms/_files/pages.php`

The ITF includes the declared PHP script to your test and executes it during test run.

The following example demonstrates a simple implementation of a Cms module page test from the Magento codebase.

Data fixture to test a Cms module page: [`dev/tests/integration/testsuite/Magento/Cms/_files/pages.php`][].

Test case that uses the above data fixture: [`dev/tests/integration/testsuite/Magento/Cms/Block/PageTest.php`][].

### Fixture as a method

[`dev/tests/integration/testsuite/Magento/Cms/Controller/PageTest.php`][] demonstrates an example of the `testCreatePageWithSameModuleName()` test method that uses data from the `cmsPageWithSystemRouteFixture()` data fixture.

### Parametrized Fixture class

Define the fixture in a separate Parametrized Fixture class when you think it could be reused in different test cases.
Fixture class can be provided with a data provider for additional parameters and customizations.

There are two types of data providers:
-  Inline JSON as data provider
-  Decoupled fixtures class

#### Inline JSON as data provider

Example:

```php?start_inline=1
class ProductsList extends \PHPUnit\Framework\TestCase
{
    /**
    * @magentoDataFixture \Magento\Catalog\Fixture\CreateSimpleProduct with:{"sku": "simple1", "price": 5.0}
    * @magentoDataFixture \Magento\Catalog\Fixture\CreateSimpleProduct with:{"sku": "simple2", "price": 10.0}
    * @magentoDataFixture \Magento\Catalog\Fixture\CreateSimpleProduct with:{"sku": "simple3", "price": 15.0}
    */
    public function testGetProductsCount(): void
    {
    }
}
```

### A class method as data provider

Example:

```php?start_inline=1
class ProductsList extends \PHPUnit\Framework\TestCase
{
   /**
   * @magentoDataFixture Magento\Catalog\Fixture\CreateSimpleProduct as:product1
   * @magentoDataFixture Magento\Catalog\Fixture\CreateSimpleProduct as:product2
   * @magentoDataFixture Magento\Catalog\Fixture\CreateSimpleProduct as:product3
   */
   public function testGetProductsCount(): void
   {
   }

   public function getProductsCountFixtureDataProvider(): array
   {
      return [
         [
             'product1' => [
                 'sku' => 'simple1'
             ],
             'product2' => [
                 'sku' => 'simple2'
             ],
             'product3' => [
                 'sku' => 'simple3',
                 'status' => Status::STATUS_DISABLED,
             ],
         ]
      ];
   }
}
```

### Decoupled fixtures class

Example 1:

```php?start_inline=1
class QuoteTest extends \PHPUnit\Framework\TestCase
{
   /**
   * @magentoDataFixture \Magento\Catalog\Fixture\CreateSimpleProduct with:{"sku": "simple1", "price": 5.0} as:product1
   * @magentoDataFixture \Magento\Catalog\Fixture\CreateSimpleProduct with:{"sku": "simple2", "price": 10.0} as:product2
   * @magentoDataFixture \Magento\Quote\Fixture\CreateEmptyCartForGuest as:cart
   * @magentoDataFixture \Magento\Quote\Fixture\AddSimpleProductToCart with:{"cart": "$cart", "product": "$product1", "qty": 2}
   * @magentoDataFixture \Magento\Quote\Fixture\AddSimpleProductToCart with:{"cart": "$cart", "product": "$product2", "qty": 1}
   */
   public function testGetProductsCount(): void
   {
   }
}
```

Example 2:

```php?start_inline=1
class QuoteTest extends \PHPUnit\Framework\TestCase
{
   /**
   * @magentoApiDataFixture Magento\Customer\Fixture\CreateCustomer as:customer
   * @magentoApiDataFixture Magento\Catalog\Fixture\CreateDropdownAttribute with:{"options":["option_a","option_b"]} as:attr1
   * @magentoApiDataFixture Magento\Catalog\Fixture\CreateDropdownAttribute with:{"options":["option_1","option_2"]} as:attr2
   * @magentoApiDataFixture Magento\Catalog\Fixture\CreateSimpleProduct with:{"sku":"simple1"} as:product1
   * @magentoApiDataFixture Magento\Catalog\Fixture\CreateSimpleProduct with:{"sku":"simple2"} as:product2
   * @magentoApiDataFixture Magento\ConfigurableProduct\Fixture\CreateConfigurableProduct as:configurable
   * @magentoApiDataFixture Magento\Quote\Fixture\CreateEmptyCartForCustomer with:{"customer":"$customer"} as:cart
   * @magentoApiDataFixture Magento\ConfigurableProduct\Fixture\AddConfigurableProductToCart with:{"cart":"$cart","product":"$configurable", "selection":"$product1"}
   * @magentoApiDataFixture Magento\ConfigurableProduct\Fixture\AddConfigurableProductToCart with:{"cart":"$cart","product":"$configurable", "selection":"$product2"}
   * @magentoApiDataFixture Magento\Quote\Fixture\AddShippingAddressToCart with:{"cart":"$cart"}
   */
   public function testCartWithConfigurable()
   {
   }

   public function cartWithConfigurableFixtureDataProvider(): array
   {
      return [
         'configurable' => [
             'attributes' => ['$attr1', '$attr2'],
             'products' => [
                 [
                     'product' => '$product1',
                     'options' => [
                         ['attribute' => '$attr1', 'option' => 'option_a'],
                         ['attribute' => '$attr2', 'option' => 'option_1']
                     ]
                 ],
                 [
                     'product' => '$product2',
                     'options' => [
                         ['attribute' => '$attr1', 'option' => 'option_a'],
                         ['attribute' => '$attr1', 'option' => 'option_2']
                     ]
                 ]
             ]
         ]
      ];
   }
}
```

### Test case and test method scopes

The `@magentoDataFixture` can be specified for a particular test or for an entire test case.
The basic rules for fixture annotation at different levels are:

-  `@magentoDataFixture` at a test case level makes the framework to apply the declared fixtures to each test in the test case.
   When the final test is complete, all class-level fixtures are reverted.
-  `@magentoDataFixture` for a particular test signals the framework to revert the fixtures declared on a test case level and applies the fixtures declared at a test method level instead.
   When the test is complete, the ITF reverts the applied fixtures.

{:.bs-callout-info}
The integration testing framework interacts with a database to revert the applied fixtures.

### Fixture rollback

A fixture that contains database transactions only are reverted automatically.
Otherwise, when a fixture creates files or performs any actions other than database transaction, provide the corresponding rollback logic.
Rollbacks are run after reverting all the fixtures related to database transactions.

A fixture rollback must be of the same format as the corresponding fixture: a script or a method:

-  A rollback script must be named according to the corresponding fixture suffixed with `_rollback` and stored in the same directory.
-  Rollback methods must be of the same class as the corresponding fixture and suffixed with `Rollback`.

Examples:

Fixture/Rollback | Fixture name                                         | Rollback name
-----------------|------------------------------------------------------|-------------------------------------------------------------
Script           | `Magento/Catalog/_files/categories.php`              | `Magento/Catalog/_files/categories_rollback.php`
Method           | `\Magento\Catalog\Model\ProductTest::prepareProduct` | `\Magento\Catalog\Model\ProductTest::prepareProductRollback`

### Restrictions

Do not rely on and do not modify an application state from within a fixture, because [application isolation annotation][magentoAppIsolation] can reset the application state at any moment.

<!-- Link definitions -->

[magentoAppIsolation]: magento-app-isolation.html
[`dev/tests/integration/testsuite/Magento/Cms/_files/pages.php`]: {{ site.mage2bloburl }}/{{ page.guide_version }}/dev/tests/integration/testsuite/Magento/Cms/_files/pages.php
[`dev/tests/integration/testsuite/Magento/Cms/Block/PageTest.php`]: {{ site.mage2bloburl }}/{{ page.guide_version }}/dev/tests/integration/testsuite/Magento/Cms/Block/PageTest.php
[`dev/tests/integration/testsuite/Magento/Cms/Controller/PageTest.php`]: {{ site.mage2bloburl }}/{{ page.guide_version }}/dev/tests/integration/testsuite/Magento/Cms/Controller/PageTest.php
