
[![Latest Stable Version](https://poser.pugx.org/owen-it/laravel-auditing/version)](https://packagist.org/packages/owen-it/laravel-auditing)
[![Total Downloads](https://poser.pugx.org/owen-it/laravel-auditing/downloads)](https://packagist.org/packages/owen-it/laravel-auditing)
[![Latest Unstable Version](https://poser.pugx.org/owen-it/laravel-auditing/v/unstable)](//packagist.org/packages/owen-it/laravel-auditing)
[![License](https://poser.pugx.org/owen-it/laravel-auditing/license.svg)](https://packagist.org/packages/owen-it/laravel-auditing)

Laravel Auditing allows you to record changes to an Eloquent model's set of data by simply adding its trait to your model. Laravel Auditing also provides a simple interface for retreiving an audit trail for a piece of data and allows for a great deal of customization in how that data is provided.

> Auditing is based on the package [revisionable](https://packagist.org/packages/VentureCraft/revisionable)

## Installation

Laravel Auditing can be installed with [Composer](http://getcomposer.org/doc/00-intro.md), more details about this package in Composer can be found [here](https://packagist.org/packages/owen-it/laravel-auditing).

Run the following command to get the latest version package:

```
composer require owen-it/laravel-auditing
```

### Laravel
Open the file ```config/app.php``` and then add the service provider, this step is required.

```php
'providers' => [
    // ...
    OwenIt\Auditing\AuditingServiceProvider::class,
],
```
> Note: This provider is important for the publication of configuration files.

Only after complete the step before, use the following command to publish configuration settings:

```
php artisan vendor:publish --provider="OwenIt\Auditing\AuditingServiceProvider"
```
Finally, execute the migration to create the ```logs``` table in your database. This table is used to save audit the logs.

```
php artisan migrate
```

### Lumen (From version 2.3.6)
Open the file ```bootstrap/app.php``` and then add the service provider, this step is required.

```php
$app->register(OwenIt\Auditing\AuditingServiceProvider::class);
```
You should uncomment the $app->withFacades() and $app->withEloquent() call in your `bootstrap/app.php` file.
```php
// ...
$app->withFacades();

$app->withEloquent();
// ...
```

Then execute the migration to create the ```logs``` table in your database. This table is used to save audit the logs.
```
php artisan migrate
```


## Docs
* [Implementation](#implementation)
* [Configuration](#configuration)
* [Getting the Logs](#getting)
* [Customizing log message](#customizing)
* [Examples](#examples)
* [Contributing](#contributing)
* [Having problems?](#faq)
* [license](#license)

<a name="implementation"></a>
## Implementation

### Implementation using ```Trait```

To register the change log, use the trait `OwnerIt\Auditing\AuditingTrait` in the model you want to audit

```php
// app/Team.php
namespace App;

use Illuminate\Database\Eloquent\Model;
use OwenIt\Auditing\AuditingTrait;

class Team extends Model 
{
    use AuditingTrait;
    //...
}

```

### Base implementation Legacy Class

It is also possible to have your model extend the `OwnerIt\Auditing\Auditing` class to enable auditing. Example:

```php
// app/Team.php
namespace App;

use OwenIt\Auditing\Auditing;

class Team extends Auditing 
{
    //...    
}
```

<a name="configuration"></a>
## Configuration (optional)

### Auditing behavior settings
The Auditing behavior settings are carried out with the declaration of attributes in the model. See the examples below:

* Disable / enable logging: `$auditEnabled = false`
* Clear the oldest records: `$historyLimit = 100`
* Turn off logging for specific fields: `$dontKeepLogOf = ['created_at', 'updated_at']`
* Tell what actions you want to audit: `$auditableTypes = ['created', 'saved', 'deleted']`

> Note: This implementation is optional, you can make these customizations where desired.

```php
// app/Team.php
namespace App;

use Illuminate\Database\Eloquent\Model;

class Team extends Model 
{
    use OwenIt\Auditing\AuditingTrait;
    
    // Disables the log record in this model.
    protected $auditEnabled  = false;
    
    // Clear the oldest records after 100 records.
    protected $historyLimit = 100; 
    
    // Fields you do NOT want to register.
    protected $dontKeepLogOf = ['created_at', 'updated_at'];
    
    // Tell what actions you want to audit.
    protected $auditableTypes = ['created', 'saved', 'deleted'];
}
```
### Auditing settings
Using the configuration file, you can define:

* The Model used to represent the current user of application.
* A different database connection for audit.
* The table name used for log registers.
    
The configuration file can be found at `config/auditing.php`.

```php
// config/auditing.php
return [

    // Authentication Model
    'model' => App\User::class,

    // Database Connection
    'connection' => null,

    // Table Name
    'table' => 'logs',
];
```

<a name="getting"></a>
## Getting the Logs

```php
// app/Http/Controller/MyAppController.php
namespace App\Http\Controllers;

use App\Team;

class MyAppController extends BaseController 
{
    public function index()
    {
        // Get team
        $team = Team::find(1); 
        
        // Get all logs
        $team->logs; 
        
        // Get first log
        $team->logs->first(); 
        
        // Get last log
        $team->logs->last();  
        
        // Selects log
        $team->logs->find(2); 
    }
    //...
}
```
Getting logs with user responsible for the change.
```php
use OwenIt\Auditing\Log;

$logs = Log::with(['user'])->get();

```
or
```php
use App\Team;

$logs = Team::logs->with(['user'])->get();

```

> Note: Remember to properly define the user model in the file ``` config/auditing.php ```.
>```php
> ...
> 'model' => App\User::class,
> ... 
>```

<a name="customizing"></a>
## Customizing log message

You can define your own log messages for presentation. These messages can be defined for both the model as well as for each one of fields.The dynamic part of the message can be done by targeted fields per dot segmented as`{object.property.property} or {object.property|Default value} or {object.property||callbackMethod}`. 

> Note: This implementation is optional, you can make these customizations where desired.

Set messages to the model
```php
// app/models/Post.php

namespace App\Models;
use OwenIt\Auditing\Auditing;

class Post extends Auditing 
{
    
    // with default value
    public static $logCustomMessage = '{user.name|Anonymous} {type} a post {elapsed_time}'; 
    
    // with callback method
    public static $logCustomFields = [
        'title'  => 'The title was defined as "{new.title||getNewTitle}"', 
        'ip' => 'Registered from the address {ip||getAnotherthing}',
        'publish_date' => [
            'created' => 'Publication date: {new.publish_date}',
            'delete' =>  'Post removed from {new.publish_date}'
        ]
    ];
    
    public function getNewTitle($log)
    {
        return $log->old['title'];
    }
    
    public function getAnotherthing($log)
    {
        return ':(';
    }
    
    //...
}
```
Getting change logs 
```php
// app/Http/Controllers/MyAppController.php 
    
    //...
    public function auditing()
    {
        // Get logs of Post
        $logs = Post::find(1)->logs; 
        return view('admin.auditing', compact('logs'));
    }
    //...
    
```
Featuring log records:
```
    // resources/views/admin/auditing.blade.php
    ...
    <ol>
        @forelse ($logs as $log)
            <li>
                {{ $log->customMessage }}
                <ul>
                    @forelse ($log->customFields as $custom)
                        <li>{{ $custom }}</li>
                    @empty
                        <li>No details</li>
                    @endforelse
                </ul>
            </li>
        @empty
            <p>No logs</p>
        @endforelse
    </ol>
    ...
    
```
Result:
<ol>
  <li>Antério Vieira created a post 1 day ago   
    <ul>
      <li>The title was defined as "Did someone say rapid?"</li>
      <li>Registered from the address 192.168.10.1</li>
      <li>Publication date: 2016-05-25 00:49:26.0</li>
    </ul>
  </li>
  <li>Antério Vieira updated a post 1 day ago   
    <ul>
      <li>The title was updated as "Did someone say rapid?"</li>
      <li>Registered from the address 192.168.10.1</li>
      <li>The post publication date has been updated from 2016-05-20 00:49:26.0 to 2016-05-25 00:49:26.0</li>
    </ul>
  </li>
  <li>Raphael França deleted a post 2 day ago   
    <ul>
      <li>Registered from the address 192.168.10.1</li>
    </ul>
  </li>  
  <li>...</li>
</ol>

Database:
![auditing-table](https://cloud.githubusercontent.com/assets/1490347/15525219/b34085d8-21fe-11e6-8729-926513fe3caa.jpg)

<a name="examples"></a>
## Examples

##### Spark Auditing
For convenience we decided to use the [spark](https://github.com/laravel/spark) for this example, the demonstration of auditing is simple and self explanatory. [Click here](https://github.com/owen-it/spark-auditing) and see for yourself.

##### Dreams
Dreams is a developed api to serve as an example or direction for developers using laravel-auditing. You can access the application [here](https://dreams-.herokuapp.com). The back-end (api) was developed in laravel 5.1 and the front-end (app) in angularjs, the detail are these:

* [Link for application](https://dreams-.herokuapp.com) 
* [Source code api-dreams](https://github.com/owen-it/api-dreams)
* [Source code app-dreams](https://github.com/owen-it/app-dreams)

<a name="contributing"></a>
## Contributing

Contributions are welcomed; to keep things organized, all bugs and requests should be opened on github issues tab for the main project in the [owen-it/laravel-auditing/issues](https://github.com/owen-it/laravel-auditing/issues).

All pull requests should be made to the branch Develop, so they can be tested before being merged into the master branch.

<a name="faq"></a>
## Having problems?

If you are having problems with the use of this package, there is likely someone has faced the same problem. You can find common answers to their problems:

* [Github Issues](https://github.com/owen-it/laravel-auditing/issues?page=1&state=closed)

<a name="license"></a>
### License

The laravel-audit package is open source software licensed under the [license MIT](http://opensource.org/licenses/MIT)

