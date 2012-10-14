---
layout: post
title: "Test driving Prestashop with Behat, Mink and Selenium"
comments: true
category: php 
tags: [php, prestashop, TDD, acceptance testing, cucumber, behat, mink, selenium]
published: false
---
#Test driving Prestashop with Behat, Mink and Selenium

Whilst I don't do so much PHP development nowdays, I've been watching
the current rennaissance with interest. PHP is taking note of trends
being pushed by the Rails world with "clean and classy" frameworks such
as [Laravel](http://laravel.com/) and best practice manifestos like [PHP the right
way](http://www.phptherightway.com/) springing up in the past couple of
years. This is all good news for the web because better quality code (whatever 
language it's written in) breeds higher expectations of what's possible. 

That said, there still seems to be a lack of good documentation
concerning proper behaviour driven development with PHP and it's a
problem that we came across recently at Kyan. I'm going to try and do a
walkthrough of setting up some simple fetaure testing using a default
install of the [Prestashop](http://www.prestashop.com) ecommerce
framework. It's a fairly sizeable project that doesn't ship with test
coverage - let's fix that!

## What you'll need

- A local install of [Prestashop](http://www.prestashop.com) (duh...)
- Firefox (just the standard consumer version, not Aurora or nightly)
- [Selenium RC (2.25.0 at the time of writing)](http://selenium.googlecode.com/files/selenium-server-standalone-2.25.0.jar)
- PHP 5.4 (just so we're all on the same page)

- [Behat (a testing framework)](http://behat.org/) 
- [Mink (web acceptance testing)](http://mink.behat.org/) 
- [Composer](http://getcomposer.org/) (a PHP packaging tool - think Bundler if you've used that
  before)

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
write our first test. Start editing /features/basket.feature and put in
the following

```gherkin
Feature: Basket
  In order to add a product to the basket
  As a website user
  I need to be able to search for a word

  Scenario: Searching for a page that does exist
    Given I am on "/wiki/Main_Page"
    When I fill in "search" with "Behavior Driven Development"
    And I press "searchButton"
    Then I should see "software development process"

  Scenario: Searching for a page that does NOT exist
    Given I am on "/wiki/Main_Page"
    When I fill in "search" with "Glory Driven Development"
    And I press "searchButton"
    Then I should see "Search results"

  @javascript
  Scenario: Searching for a page with autocompletion
    Given I am on "/wiki/Main_Page"
    When I fill in "search" with "Behavior Driv"
    And I wait for the suggestion box to appear
    Then I should see "Behavior-Driven Development"
```



//TODO
Run through install
Screenshots
Write a feature
Run it





