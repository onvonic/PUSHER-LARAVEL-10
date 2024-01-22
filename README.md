# LARAVEL 10 PUSHER

- [pusher](https://pusher.com/).
- [laravel 10 Broadcasting](https://laravel.com/docs/10.x/broadcasting#pusher-channels).

Console
```
composer require pusher/pusher-php-server
```

## config/broadcasting.php
```php
'pusher' => [
    'driver' => 'pusher',
    'key' => env('PUSHER_APP_KEY'),
    'secret' => env('PUSHER_APP_SECRET'),
    'app_id' => env('PUSHER_APP_ID'),
    'options' => [
        'cluster' => env('PUSHER_APP_CLUSTER'),
        'host' => env('PUSHER_HOST') ?: 'api-'.env('PUSHER_APP_CLUSTER', 'mt1').'.pusher.com',
        'port' => env('PUSHER_PORT', 443),
        'scheme' => env('PUSHER_SCHEME', 'https'),
        'encrypted' => true,
        'useTLS' => env('PUSHER_SCHEME', 'https') === 'https',
    ],
    'client_options' => [
        // Guzzle client options: https://docs.guzzlephp.org/en/stable/request-options.html
    ],
],
```

## config/app.php

```php
'providers' => ServiceProvider::defaultProviders()->merge([
    App\Providers\BroadcastServiceProvider::class,
])->toArray(),
```


## .ENV

```
BROADCAST_DRIVER=pusher

PUSHER_APP_ID=xxxxxx
PUSHER_APP_KEY=xxxxxxxxxxxxx
PUSHER_APP_SECRET=xxxxxxxxxxxxxxxxxxxx
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME=https
PUSHER_APP_CLUSTER=ap1
```

## CLIENT.HTML

```html
<!DOCTYPE html>

<head>
    <title>Pusher Test</title>
    <h1 id="message"></h1>
    <script src="https://js.pusher.com/8.2.0/pusher.min.js"></script>
    <script>
        // Enable pusher logging - don't include this in production
        Pusher.logToConsole = true;

        var pusher = new Pusher('xxxxxxxxxxxxx', {
            cluster: 'ap1'
        });

        var channel = pusher.subscribe('myChannel');
        channel.bind('myEvent', function (data) {
            // alert(JSON.stringify(data));
            document.getElementById('message').textContent = JSON.stringify(data.message);
        });

    </script>
</head>

<body>
    <h1>Pusher Test</h1>
    <p>
        Try publishing an event to channel <code>myChannel</code>
        with event name <code>myEvent</code>.
    </p>
</body>
```

## Membuat Event

```
php artisan make:event MyNotification
```

## MyNotification

```php
class MyNotification "implements ShouldBroadcast"

MyNotification.php

<?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class MyNotification implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;
    public $message;
    /**
     * Create a new event instance.
     */
    public function __construct($message)
    {
        $this->message = $message;
    }

    /**
     * Get the channels the event should broadcast on.
     *
     * @return array<int, \Illuminate\Broadcasting\Channel>
     */
    public function broadcastOn(): array
    {
        return ['myChannel'];
    }

    public function broadcastAs()
    {
        return 'myEvent';
    }
}
```

## route.php

```php
Route::get('test', function(){
    event(new \App\Events\MyNotification('hello world'));
    return "success";
});
```
