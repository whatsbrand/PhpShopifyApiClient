# shopify.php

Lightweight multi-paradigm PHP (JSON) client for the [Shopify API](http://api.shopify.com/).


## Requirements

* PHP 4 with [cURL support](http://php.net/manual/en/book.curl.php).
* Only compatible OAuth Shopify Apps.  For Legacy Authentication: [Use an old version of ohShopify.php](https://github.com/cmcdonaldca/ohShopify.php/blob/7ee7a344ca83518a0560ba585d4f8deab65bf5cd/shopify.php)


## Getting Started

Basic needs for authorization and redirecting

```php
<?php

	require 'shopify.php';

	/* Define your APP`s key and secret*/
	define('SHOPIFY_API_KEY','');
	define('SHOPIFY_SECRET','');
	
	/* Define requested scope (access rights) - checkout https://docs.shopify.com/api/authentication/oauth#scopes 	*/
	define('SHOPIFY_SCOPE','');	//eg: define('SHOPIFY_SCOPE','read_orders,write_orders');
	
	if (isset($_GET['code'])) { // if the code param has been sent to this page... we are in Step 2
		// Step 2: do a form POST to get the access token
		$shopifyClient = new ShopifyClient($_GET['shop'], "", SHOPIFY_API_KEY, SHOPIFY_SECRET);
		session_unset();
		
		// Now, request the token and store it in your session.
		$_SESSION['token'] = $shopifyClient->getAccessToken($_GET['code']);
		if ($_SESSION['token'] != '')
			$_SESSION['shop'] = $_GET['shop'];
	
		header("Location: index.php");
		exit;		
	}
	// if they posted the form with the shop name
	else if (isset($_POST['shop'])) {
	
		// Step 1: get the shopname from the user and redirect the user to the
		// shopify authorization page where they can choose to authorize this app
		$shop = isset($_POST['shop']) ? $_POST['shop'] : $_GET['shop'];
		$shopifyClient = new ShopifyClient($shop, "", SHOPIFY_API_KEY, SHOPIFY_SECRET);
	
		// get the URL to the current page
		$pageURL = 'http';
		if ($_SERVER["HTTPS"] == "on") { $pageURL .= "s"; }
		$pageURL .= "://";
		if ($_SERVER["SERVER_PORT"] != "80") {
			$pageURL .= $_SERVER["SERVER_NAME"].":".$_SERVER["SERVER_PORT"].$_SERVER["SCRIPT_NAME"];
		} else {
			$pageURL .= $_SERVER["SERVER_NAME"].$_SERVER["SCRIPT_NAME"];
		}
	
		// redirect to authorize url
		header("Location: " . $shopifyClient->getAuthorizeUrl(SHOPIFY_SCOPE, $pageURL));
		exit;
	}
	
	// first time to the page, show the form below
?>
	<p>Install this app in a shop to get access to its private admin data.</p> 
 
	<p style="padding-bottom: 1em;">
		<span class="hint">Don&rsquo;t have a shop to install your app in handy? <a href="https://app.shopify.com/services/partners/api_clients/test_shops">Create a test shop.</a></span>
	</p> 
	 
	<form action="" method="post">
	  <label for='shop'><strong>The URL of the Shop</strong> 
	    <span class="hint">(enter it exactly like this: myshop.myshopify.com)</span> 
	  </label> 
	  <p> 
	    <input id="shop" name="shop" size="45" type="text" value="" /> 
	    <input name="commit" type="submit" value="Install" /> 
	  </p> 
	</form>

```

Once you have authorized and stored the token in the session, you can make API calls

Making API calls:

```php
<?php

	require 'shopify.php';

	$sc = new ShopifyClient($_SESSION['shop'], $_SESSION['token'], $api_key, $secret);

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

When receiving requests from the Shopify API, validate the signature value:

```php
<?php

    $sc = new ShopifyClient($_GET['shop'], '', SHOPIFY_API_KEY, SHOPIFY_SECRET);

    if(!$sc->validateSignature($_GET))
        die('Error: invalid signature.');

?>
```
