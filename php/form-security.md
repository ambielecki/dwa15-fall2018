# Form security
When you accept user input on your web application, you need to develop a certain level of paranoia&mdash; always be on the lookout for ways in which malicious visitors will try and manipulate your application.

Security in our web applications is a common theme we need to explore this semester, and it starts here with form data...

Consider this example:

VisitorA is shown a form where they can update the description of a book.

When filling out the form, instead of entering a valid description, they enter this nefarious JavaScript code:

```xml
<script>
    alert('Attention; your password has been compromised, visit http://shady.com to secure your account.')
</script>
```

Upon submission, the server saves this data to a database.

Later, VisitorB views the page for that specific book, and the description is fetched from the database and displayed on the page, resulting in an ominous alert message...

<img src='http://making-the-internet.s3.amazonaws.com/php-shady-alert@2x.png' style='max-width:422px;' alt=''>

Eep!

This scenario is called a **Cross-site scripting (XSS)** attack, and it occurs when a malicious visitor attempts to inject client-side scripts into web pages viewed by other visitors.

To avoid an attack like this, any data that originated from a visitor should always be &ldquo;sanitized&rdquo; before it's displayed on the page by converting HTML special characters to their equivalent HTML entities.

E.g. if the user entered this: `<script>` we should render it as `&lt;script&gt;` so that no scripts are actually executed on display.

This sanitization can be accomplished using PHP's built-in [htmlentities](http://php.net/manual/en/function.htmlentities.php) function, e.g.:

```php
<?=htmlentities($description, ENT_QUOTES, "UTF-8"); ?>
```

Which would output this:
```
&lt;script&gt;alert(&#039;Attention; your password has been compromised, visit http://shady.com to secure your account.&#039;)&lt;/script&gt;gt;
```

Looking ahead: Other security issues we'll discuss this semester include:
+ SQL Injection Attacks
+ CSRF (Cross Site Request Forgery)


## Sanitize helper
There's a helper function in [`helpers.php`](https://github.com/susanBuck/dwa15-php/blob/master/helpers.php) (discussed in the notes on Imports) called `sanitize` that can be used to conveniently run htmlentities on a single data entity, e.g.:

```php
<?=htmlentities($description, ENT_QUOTES, "UTF-8"); ?>
```

Or it can recursively iterate through and sanitize an array, e.g.:
```php
$data = sanitize($_POST);
```
