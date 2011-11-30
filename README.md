# shopify.php

Lightweight multi-paradigm PHP (JSON) client for the [Shopify API](http://api.shopify.com/).


## Requirements

* PHP 4 with [cURL support](http://php.net/manual/en/book.curl.php).


## Getting Started

Basic needs for authorization and redirecting

```php
<?php

	require 'shopify.php';

	$sc = new ShopifyClient($shop_domain, $token, $api_key, $secret);
	
	// this is how to get the Authorize URL
	$url = $sc->getAppInstallUrl();

	// this is how you can check if the currrent request is authorized
	$isAuthorized = $sc->isAppInstalled($timestamp, $sig);
?>
```

Making API calls:

```php
<?php

	require 'shopify.php';

	$sc = new ShopifyClient($shop_domain, $token, $api_key, $secret);
	// For private apps (you should never make private apps anyways):
	// $sc = new ShopifyClient($shop_domain, $token, $api_key, $password, true);

	try
	{
		// Get all products
		$products = $sc->call('GET', '/admin/products.json', array('published_status'=>'published'));


		// Create a new recurring charge
		$charge = array
		(
			"recurring_application_charge"=>array
			(
				"price"=>10.0,
				"name"=>"Super Duper Plan",
				"return_url"=>"http://super-duper.shopifyapps.com",
				"test"=>true
			)
		);

		try
		{
			$recurring_application_charge = $sc->call('POST', '/admin/recurring_application_charges.json', $charge);

			// API call limit helpers
			echo $sc->callsMade(); // 2
			echo $sc->callsLeft(); // 498
			echo $sc->callLimit(); // 500

		}
		catch (ShopifyApiException $e)
		{
			// If you're here, either HTTP status code was >= 400 or response contained the key 'errors'
		}

	}
	catch (ShopifyApiException $e)
	{
		/* 
		 $e->getMethod() -> http method (GET, POST, PUT, DELETE)
		 $e->getPath() -> path of failing request
		 $e->getResponseHeaders() -> actually response headers from failing request
		 $e->getResponse() -> curl response object
		 $e->getParams() -> optional data that may have been passed that caused the failure

		*/
	}
	catch (ShopifyCurlException $e)
	{
		// $e->getMessage() returns value of curl_errno() and $e->getCode() returns value of curl_ error()
	}
?>
```
