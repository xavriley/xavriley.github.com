---
layout: post
title: "Test driving Prestashop with Behat, Mink and Selenium - Step by step"
comments: true
category: [php, prestashop] 
tags: [php, prestashop, TDD, acceptance testing, cucumber, behat, mink, selenium, tutorial]
published: true
---

Whilst I don't do so much PHP development nowdays, I've been watching
the current rennaissance with interest. PHP is taking note of trends
being pushed by the Rails world with "clean and classy" frameworks such
as [Laravel](http://laravel.com/) and best practice manifestos like [PHP the right
way](http://www.phptherightway.com/) coming to light. This is all good 
news for the web because better quality code (whatever language it's 
written in) raises the bar all round. 

That said, there still seems to be a lack of good documentation
concerning proper Behaviour Driven Development (BDD) with PHP and it's a
problem that we came across recently at [Kyan](http://kyan.com). I'm going to try and do a
walkthrough of setting up some simple fetaure testing using a default
install of the [Prestashop](http://www.prestashop.com) ecommerce
framework. It's a fairly sizeable project that doesn't ship with test
coverage - let's fix that!

## What you'll need

- A local install of [Prestashop](http://www.prestashop.com)
- [Firefox](http://www.mozilla.org/en-US/firefox/new/) (just the standard consumer version, not Aurora or nightly)
- [Selenium RC (2.25.0 at the time of writing)](http://selenium.googlecode.com/files/selenium-server-standalone-2.25.0.jar)
- PHP 5.4 (just so we're all on the same page)

- [Behat (a PHP testing framework)](http://behat.org/) 
- [Mink (web acceptance testing for Behat)](http://mink.behat.org/) 
- [Composer (a PHP packaging tool - think Bundler for PHP)](http://getcomposer.org/) 

&nbsp;

### 1.) Install Prestashop

Prestashop has a good set of instructions so I won't duplicate them all
here. Suffice to say you need to set up a MySQL db and change a few
permissions.

### 2.) Install Composer

Make sure PHP is in your path. You can test this by firing up terminal
and typing

```bash
php -v
```

If you're using MAMP, put something like
this at the bottom of your `.bashrc`

```bash
export PATH="/Applications/MAMP/bin/:$PATH"
```

Double check the path to the MAMP binaries. It's different in different
versions of MAMP.

Now `cd` into your newly created prestashop folder and run the following
commands to install composer;

```
mkdir bin
curl -s https://getcomposer.org/installer | php -- --install-dir=bin
``` 

If you're installing using the default OSX apache and you get an error
about "detect_unicode = Off", you'll also need to
run the following;

```
sudo su
echo "detect_unicode = Off" >> /private/etc/php.ini
apachectl restart
exit
```

### 3.) Install Behat and Mink

This part is the reason I'm writing this blog post. There seems to be a
lack of clear information about how to get these up and running from
scratch. Hopefully this will be sorted out as time goes on but I would
suggest people submit their points about what works and doesn't work for
them.

To get started, make a file called `composer.json` in the root of the
Prestashop install and put in the following;

```json
{
    "require": {
        "behat/behat":           "2.4@stable",
        "behat/mink-extension":  "*",
        "behat/mink-goutte-driver":     "*",
        "behat/mink-selenium2-driver":  "*"
    },
    "minimum-stability": "dev",
    "config": {
        "bin-dir": "bin"
    }
}
```

This is the equivalent of a Gemfile if you've used bundler and Ruby. So
to get everything up and running, run composer like so.

```
php bin/composer.phar install
```

All being, well you should see the packages installing into a vendor
folder in the project root.

### 4.) Setting up Behat

```bash
> bin/behat --init                     
+d features - place your *.feature files here
+d features/bootstrap - place bootstrap scripts and static files here
+f features/bootstrap/FeatureContext.php - place your feature related code here
```

This bootstraps the files needed to run the tests. Let's go ahead and
write our first test. Usually in test driven development, we write the
tests before the code, but I'm advocating here that we can add tests to
and exisiting site to give us the confidence to refactor and add
features later on. Start editing /features/basket.feature and put in
the following;

```gherkin
Feature: Basket
  In order to add a product to the basket
  As a website user
  I need to add the product to my basket
  And I should see the product in the basket 

  @javascript
  Scenario: Add a product to basket
    Given I am on a product page
    When I click "Add to cart"
    And I hover over "#shopping_cart a"
    Then I should see the product title 
```

This is an example of a Cucumber test (the natural language syntax is
referred to as Gherkin) which is a popular way of doing acceptance
testing in Rails and other frameworks. We proceed by running the test
like so;

```bash
> bin/behat
Feature: Basket
  In order to add a product to the basket
  As a website user
  I need to add the product to my basket
  And I should see the product in the basket

  @javascript
  Scenario: Add a product to basket     # features/basket.feature:8
    Given I am on a product page
    When I click on "Add to cart"
    And I hover over "#shopping_cart a"
    Then I should see the product title

1 scenario (1 undefined)
4 steps (4 undefined)
0m0.039s

You can implement step definitions for undefined steps with these snippets:

    /**
     * @Given /^I am on a product page$/
     */
    public function iAmOnAProductPage()
    {
        throw new PendingException();
    }

    /**
     * @When /^I click "([^"]*)"$/
     */
    public function iClick($arg1)
    {
        throw new PendingException();
    }

    /**
     * @Given /^I hover over "([^"]*)"$/
     */
    public function iHoverOver($arg1)
    {
        throw new PendingException();
    }

    /**
     * @Then /^I should see the product title$/
     */
    public function iShouldSeeTheProductTitle()
    {
        throw new PendingException();
    }
```

This runs the test and tells you what to do next. Copy the snippets into
the `features/bootstrap/FeatureContext.php` file above the final
curly brace. After saving and running `bin/behat` again, the output should change to include 

```
TODO: write pending definition
```  

### 5.) Enable Mink

This is where the documentation starts to part ways with the latest
versions of Mink and Behat. After struggling with the official run
through at [http://mink.behat.org/](http://mink.behat.org/) trying to
get Zombie working as the Javascript client, I stumbled across the Mink
example repository in the Behat official Github page here 
[https://github.com/Behat/MinkExtension-example](https://github.com/Behat/MinkExtension-example)

The key is swapping out `BehatContext` in the `FeaturesContext` file, to
make sure that it reads;

```php
class FeatureContext extends Behat\MinkExtension\Context\MinkContext
```

and then you need to configure Behat and Mink to be looking in the right
places and choosing the correct browser drivers. Make a file called
`behat.yml` in the root of the project with the following;

```yaml
default:
  context:
    class:  'FeatureContext'
  extensions:
    Behat\MinkExtension\Extension:
      base_url:  'http://tdd.prestashopexample.com/'
      goutte:    ~
      selenium2: ~
```

`goutte` seems to be a requirement as far as I can tell, but who
knows...

Now when you run `bin/behat` again, you should see an error like this;

```
Curl error thrown for http POST to http://localhost:4444/wd/hub/session with params: {"desiredCapabilities":{"browserName":"firefox","version":"8","platform":"ANY","browserVersion":"8","browser":"firefox"},"requiredCapabilities":[]}
```

Enter Selenium...

### 6.) Setting up selenium

This should be quite straightforward. First off, make sure you have the
standard version of Firefox installed. Download Selenium RC from the link
at the top of the page and move the `.jar` file to the `vendor/` folder.
Then run the following command to start the Selenium server;

```
java -jar vendor/selenium-server-standalone-2.25.0.jar > /dev/null &
```

You should see `INFO: Launching a standalone server` and you're good to
try the tests again by running `bin/behat`. This time you'll see Firefox
start up, flash up with the page content and close again. The first test
passed! Now let's implement the others.

### 7.) Finishing the steps

Change the methods in `FeatureContext.php` to look like this;

```php
    /**
     * @Given /^I hover over "([^"]*)"$/
     */
    public function iHoverOver($arg1)
    {
        $this->getSession()->getPage()->find('css', $arg1)->mouseOver();
        $this->getSession()->wait(5000, "$('#cart_block_list').hasClass('expanded')");
    }

    /**
     * @Then /^I should see the product title$/
     */
    public function iShouldSeeTheProductTitle()
    {
        $product_title = $this->getSession()->getPage()->find('css', '.cart_block_product_name');
        $product_title == "iPod Nano";
    }
```

A few things to note here before we proceed - this isn't a very robust
test and could be improved. There's duplication all over the place which
could be refactored (but it's late and I've just got it working...). The
real problem is that it's assuming the title of the product as the
assertion in the last point. A better approch would be to call/mock the
product from Prestashop in the constructor and use that instead. Maybe
in part 2.

Also, it bugged me that the `wait->(...)` call which executes JS is
executed in the context of the session. It makes perfect sense in
hindsight as `getPage` is returning a sort of DOM object but it tripped
me up nonetheless.

Down to business - run `bin/behat` one last time and the output should
look like this;

```bash
> bin/behat                                                                                       1 ↵
Feature: Basket
  In order to add a product to the basket
  As a website user
  I need to add the product to my basket
  And I should see the product in the basket

  @javascript
  Scenario: Add a product to basket     # features/basket.feature:8
    Given I am on a product page        # FeatureContext::iAmOnAProductPage()
    When I click on "Add to cart"       # FeatureContext::iClickOn()
    And I hover over "#shopping_cart a" # FeatureContext::iHoverOver()
    Then I should see the product title # FeatureContext::iShouldSeeTheProductTitle()

1 scenario (1 passed)
4 steps (4 passed)
0m4.056s
```

Happy days!

##Conclusion

There's a ton of PHP apps out there that would benefit from regression
tests like this. They're not that hard to write once you get going and
they give you an invaluable safety net and peace of mind when it comes
to refactoring and adding new features. 

My motivation for writing this was that the current state of documentation 
seems a bit patchy, despite the Behat and Mink projects being well established.
I suppose this is a call to arms to PHP devs to try this stuff out
and get their feedback into the loop. 

I'd love to see more advanced posts dealing with the various drivers 
that are available and also to hear what people think of
this one. I'll finish by saying I'm starting out on this trail so my advice
is only what I've managed to cobble together up to now. If there's
improvements or suggestions in the comments I'll happily fold them back in to
the main post. Happy testing...



