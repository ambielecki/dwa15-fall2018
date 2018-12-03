# Foobooks: One to Many

The following is a __rough outline__ of the modifications I'll make to Foobooks during Week 13's lectures.

__This should not be considered a stand-alone document; for full details please refer to the lecture video and the Foobooks code source.__


## Author dropdown
To associate Authors with Books we'll use a dropdown filled with authors. I'll start by showing this in the *Add a book* feature, and later add it to the *Edit a book* feature.

To make this happen, we first need an __array of authors__ where the key is the author `id` and the value is the author `name`.

```php
# BookController.php
public function create($id)
{
    # Get all the authors in alphabetical order by last name
    $authors = Author::orderBy('last_name')->get();

    # Organize the authors into an array where the key = author id and value = author name
    $authorsForDropdown = [];
    foreach ($authors as $author) {
        $authorsForDropdown[$author->id] = $author->last_name.', '.$author->first_name;
    }

    return view('book.edit')->with([
        'book' => $book,
        'authorsForDropdown' => $authorsForDropdown # Make $authorsForDropdown available for the view
    ]);
```

Construct the dropdown (`<select>`) in the view using this data:

```html
<label for='author_id'>* Author</label>
<select name='author_id' id='author_id'>
    <option value=''>Choose one...</option>
    @foreach($authorsForDropdown as $id => $authorName)
        <option value='{{ $id }}' {{ ($book->author_id == $id) ? 'selected' : '' }}>{{ $authorName }}</option>
    @endforeach
</select>
```

Update the `BookController@store` method to also save the author details about the book.

One way to do this:
```php
$author = Author::find($request->input('author'));
$book->author()->associate($author);
```

Or, just manually specify the `author_id` since we already have it in the request. Saves us a trip to the database to fetch the Author object.
```php
$book->author_id = $request->input('author');
```


## Custom model methods
We'll want to do the same thing on the *Edit* page.

Rather than duplicate the &ldquo;get authors for dropdown&rdquo; code, we can extract it out of the controller and add it as a method to the `Author` model:

```php
# app/Author.php
/**
 * Return an array of author id => author name
 */
public static function getForDropdown()
{
    $authors = self::orderBy('last_name')->get();

    $authorsForDropdown = [];
    foreach ($authors as $author) {
        $authorsForDropdown[$author->id] = $author->getFullName($reverse = true);
    }

    return $authorsForDropdown;
}
```

Then in `BookController@create`:
```
$authorsForDropdown = Author::getForDropdown();
```


## Validation
```php
$this->validate($request, [
    'title' => 'required|min:3',
    'author_id' => 'required', # <----
    'published' => 'required|min:4|numeric',
    'purchase_link' => 'required|url',
    'cover' => 'required|url',
]);
```
