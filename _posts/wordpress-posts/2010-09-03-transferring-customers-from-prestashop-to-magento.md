---
layout: post
category : prestashop
tags : [magento, prestashop]
---

There’s always debate about which ecommerce platform is harder/better/faster/stronger, and from experience of Prestashop and Magento I can say they’re both really good for different types of shops. For [Forsyths](http::www.forsyths.co.uk/), I developed a Prestashop solution for their Sheet Music department whilst I was working there and it’s been great so far but…

~~When picking a solution it really pays to do some forward planning and in this case, Prestashop can’t scale up to the kind of multi-store functionality that this music department store needed. That said, it was a lot easier getting a Prestashop store off the ground (I tried both at the start) and it’s served us well. My only gripe is that Prestashop’s module and templating system isn’t flexible enough to make real changes without editing the core – and that screws up future updates. Not what you need for ecommerce purposes.~~

_UPDATE_
Since writing this, it's all change with Prestashop v1.5
They've had a basic overrides system in place for a while now.
Also the new version supports multistore but I haven't used it, yet...

** Making the switch

Swapping out the products from PS to Mage is pretty straightforward – it’s just a case of a mysql query which maps them into the right fields. As with any Magento import the process is more or less similar;

- Add some dummy data into Magento, using all the fields you’re likely to use
- Export the data as a csv using System > Import/Export > Profiles
- Look at the file in your var/export folder and take a look at the headers

Tip for \*nix users working with csv files – in terminal use the following command to save the first two lines to a file;

```shell
head -n 2 your_csv_file.csv > your_csv_file_head.csv
```

That’s if your csv file is really big.

** The problem with users

Now the above works fine for most things, but the problem with users lies in the different authentication that both shops use. Magento uses MD5 with a salt on the end, Prestashop uses a ‘Cookie Key’ prefix to the customer password, which is then MD5 encrypted.
Anyone who’s looked into MD5 knows that this is a pig – you can’t reverse an MD5 into plain text, therefore you can’t get the original password strings in order to re-encode them (which is actually a good thing…).

** How to fix it

Clearly its impossible to convert from one MD5 hash to another so we have to try something different. Make the following file;

```
(magentoroot)/app/code/local/Mage/Customer/Model/Customer.php
```

and in it put the following;

```php
class Mage_Customer_Model_Customer extends Mage_Core_Model_Abstract
{
/**
* Authenticate customer
*
* @param string $login
* @param string $password
* @return true
* @throws Exception
*/
public function authenticate($login, $password)
{

$this->loadByEmail($login);
if ($this->getConfirmation() && $this->isConfirmationRequired()) {
throw Mage::exception('Mage_Core', Mage::helper('customer')->__('This account is not confirmed.'),
self::EXCEPTION_EMAIL_NOT_CONFIRMED
);
}
if (!$this->validatePassword($password) && !$this->validatePassword('hKvthisisyourgibberishcookiestringfromprestashopCM'.$password)) {
throw Mage::exception('Mage_Core', Mage::helper('customer')->__('Invalid login or password.'),
self::EXCEPTION_INVALID_EMAIL_OR_PASSWORD
);
}
Mage::dispatchEvent('customer_customer_authenticated', array(
'model' => $this,
'password' => $password,
));
return true;
}

//end class
}
}
```

Notice the random string prepended to the password variable? That’s your cookie string which you’ll find in your Prestashop install here;

```
(prestshop root)/config/settings.inc.php
```

It’s the line beginning

```php
define('_COOKIE_KEY_', 'ThisIsTheBitYouWant...' 
```

Believe it or not that’s all you need to do. Import your MD5 hashes from Prestashop straight into Magento and it’ll authenticate them as Prestashop would do. I’ll go through the mysql import in another post.



