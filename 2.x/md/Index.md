# Index
---

- First of We Will Discuss how Framework works when HTTP request hits the our web server.
- Every Request in our application is handled by `index.php` placed in `public/`.
- `index.php` looks like:

```php
<?php 
use Bundle\Bundler;

try {
	require __DIR__ . "/../bundle/Bundler.php";

	// Bundler class bundles your application with routes and other neccesary things
	$app = new Bundler();
	$app->bundle(); // Starting our app
} catch (Exception $err) {
	echo "<pre>$err</pre>";
}

```

## Let's Understand Line by Line:

- `Bundle\Bundler` class is our real bootstrapper which runs every neccesary code which should be run upon receiving an HTTP request, we understand it after some time, for now just keep in mind it runs important framework codes like loading .env files, setting up Development env, Loading Helper Functions, etc.
- Then, we create instance of `Bundler` class and run it's bundle method.
- also, if any type of Exception is thrown, then exception is handled by `catch` keyword, and it shows the error in readble format
> It is nice to know that if any static file is requested, then any HTML, JavaScript or CSS file could be accessible by End User directly, so keep sensitive files out of `public/` folder
- now let's take a look at `Bundler` Class: [](./Bundler.md)
