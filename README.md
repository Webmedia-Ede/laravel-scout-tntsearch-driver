# TNTSearch Driver for Laravel Scout - Laravel 5.3/5.4/5.5/5.6


This package makes it easy to add full text search support to your models with Laravel 5.3/5.4/5.5/5.6.
## Contents

- [Installation](#installation)
- [Usage](#usage)
- [Contributing](#contributing)
- [Credits](#credits)
- [License](#license)


## Installation

You can install the package via composer:

``` bash
composer require webmedia-ede/laravel-scout-tntsearch-driver
```

Add the service provider:

```php
// config/app.php
'providers' => [
    // ...
    TeamTNT\Scout\TNTSearchScoutServiceProvider::class,
],
```

Ensure you have Laravel Scout as a provider too otherwise you will get an "unresolvable dependency" error

```php
// config/app.php
'providers' => [
    // ...
    Laravel\Scout\ScoutServiceProvider::class,
],
```

Add  `SCOUT_DRIVER=tntsearch` to your `.env` file

Then you should publish `scout.php` configuration file to your config directory

```bash
php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
```

In your `config/scout.php` add:

```php

'tntsearch' => [
    'storage'  => storage_path(), //place where the index files will be stored
    'fuzziness' => env('TNTSEARCH_FUZZINESS', false),
    'fuzzy' => [
        'prefix_length' => 2,
        'max_expansions' => 50,
        'distance' => 2
    ],
    'asYouType' => false,
    'searchBoolean' => env('TNTSEARCH_BOOLEAN', false),
],
```
To prevent your search indexes being commited to your project repository,
add the following line to your `.gitignore` file.

```/storage/*.index```

The `asYouType` option can be set per model basis, see the example below.

## Usage

After you have installed scout and the TNTSearch driver, you need to add the
`Searchable` trait to your models that you want to make searchable. Additionaly,
define the fields you want to make searchable by defining the `toSearchableArray` method on the model:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;

    public $asYouType = true;

    /**
     * Get the indexable data array for the model.
     *
     * @return array
     */
    public function toSearchableArray()
    {
        $array = $this->toArray();

        // Customize array...

        return $array;
    }
}
```

Then, sync the data with the search service like:

`php artisan scout:import App\\Post`

If you have a lot of records and want to speed it up you can run:

`php artisan tntsearch:import App\\Post`

After that you can search your models with:

`Post::search('Bugs Bunny')->get();`

## Constraints

Additionally to `where()` statements as conditions, you're able to use Eloquent queries to constrain your search. This allows you to take relationships into account.

If you make use of this, the search command has to be called after all queries have been defined in your controller.

The `where()` statements you already know can be applied everywhere.

```php
namespace App\Http\Controllers;

use App\Post;

class PostController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index(Request $request)
    {
        $post = new Post;

        // filter out posts to which the given topic is assigned
        if($request->topic) {
            $post = $post->whereNotIn('id', function($query){
                $query->select('assigned_to')->from('comments')->where('topic','=', request()->input('topic'));
            });
        }

        // only posts from people that are no moderators
        $post = $post->byRole('moderator','!=');

        // when user is not admin filter out internal posts
        if(!auth()->user()->hasRole('admin'))
        {
            $post= $post->where('internal_post', false);
        }

        if ($request->searchTerm) {
            $constraints = $post; // not necessary but for better readability
            $post = Department::search($request->searchTerm)->constrain($constraints);
        }

        $post->where('deleted', false);

        $paginator = $department->paginate(10);
        $posts = $paginator->getCollection();

        // return posts
    }
}
```

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
