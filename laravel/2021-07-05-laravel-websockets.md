# Laravel WebSockets

Laravel WebSockets is a package for Laravel 5.7 and up that will get your application started with WebSockets in no-time! It has a drop-in Pusher API replacement, has a debug dashboard, realtime statistics and even allows you to create custom WebSocket controllers

# Installation

Laravel WebSockets can be installed via composer:

```
composer require beyondcode/laravel-websockets
```

The package will automatically register a service provider.

This package comes with a migration to store statistic information while running your WebSocket server. You can publish the migration file using:

```
php artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider" --tag="migrations"
```

```
php artisan migrate
```

Next, you need to publish the WebSocket configuration file:

```
php artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider" --tag="config"
```

This is the default content of the config file that will be published as `config/websockets.php`:

```
<?php

use BeyondCode\LaravelWebSockets\Dashboard\Http\Middleware\Authorize;

return [

    /*
     * Set a custom dashboard configuration
     */
    'dashboard' => [
        'port' => env('LARAVEL_WEBSOCKETS_PORT', 6001),
    ],

    /*
     * This package comes with multi tenancy out of the box. Here you can
     * configure the different apps that can use the webSockets server.
     *
     * Optionally you specify capacity so you can limit the maximum
     * concurrent connections for a specific app.
     *
     * Optionally you can disable client events so clients cannot send
     * messages to each other via the webSockets.
     */
    'apps' => [
        [
            'id' => env('PUSHER_APP_ID'),
            'name' => env('APP_NAME'),
            'key' => env('PUSHER_APP_KEY'),
            'secret' => env('PUSHER_APP_SECRET'),
            'path' => env('PUSHER_APP_PATH'),
            'capacity' => null,
            'enable_client_messages' => false,
            'enable_statistics' => true,
        ],
    ],

    /*
     * This class is responsible for finding the apps. The default provider
     * will use the apps defined in this config file.
     *
     * You can create a custom provider by implementing the
     * `AppProvider` interface.
     */
    'app_provider' => BeyondCode\LaravelWebSockets\Apps\ConfigAppProvider::class,

    /*
     * This array contains the hosts of which you want to allow incoming requests.
     * Leave this empty if you want to accept requests from all hosts.
     */
    'allowed_origins' => [
        //
    ],

    /*
     * The maximum request size in kilobytes that is allowed for an incoming WebSocket request.
     */
    'max_request_size_in_kb' => 250,

    /*
     * This path will be used to register the necessary routes for the package.
     */
    'path' => 'laravel-websockets',

    /*
     * Dashboard Routes Middleware
     *
     * These middleware will be assigned to every dashboard route, giving you
     * the chance to add your own middleware to this list or change any of
     * the existing middleware. Or, you can simply stick with this list.
     */
    'middleware' => [
        'web',
        Authorize::class,
    ],

    'statistics' => [
        /*
         * This model will be used to store the statistics of the WebSocketsServer.
         * The only requirement is that the model should extend
         * `WebSocketsStatisticsEntry` provided by this package.
         */
        'model' => \BeyondCode\LaravelWebSockets\Statistics\Models\WebSocketsStatisticsEntry::class,

        /**
         * The Statistics Logger will, by default, handle the incoming statistics, store them
         * and then release them into the database on each interval defined below.
         */
        'logger' => BeyondCode\LaravelWebSockets\Statistics\Logger\HttpStatisticsLogger::class,

        /*
         * Here you can specify the interval in seconds at which statistics should be logged.
         */
        'interval_in_seconds' => 60,

        /*
         * When the clean-command is executed, all recorded statistics older than
         * the number of days specified here will be deleted.
         */
        'delete_statistics_older_than_days' => 60,

        /*
         * Use an DNS resolver to make the requests to the statistics logger
         * default is to resolve everything to 127.0.0.1.
         */
        'perform_dns_lookup' => false,
    ],

    /*
     * Define the optional SSL context for your WebSocket connections.
     * You can see all available options at: http://php.net/manual/en/context.ssl.php
     */
    'ssl' => [
        /*
         * Path to local certificate file on filesystem. It must be a PEM encoded file which
         * contains your certificate and private key. It can optionally contain the
         * certificate chain of issuers. The private key also may be contained
         * in a separate file specified by local_pk.
         */
        'local_cert' => env('LARAVEL_WEBSOCKETS_SSL_LOCAL_CERT', null),

        /*
         * Path to local private key file on filesystem in case of separate files for
         * certificate (local_cert) and private key.
         */
        'local_pk' => env('LARAVEL_WEBSOCKETS_SSL_LOCAL_PK', null),

        /*
         * Passphrase for your local_cert file.
         */
        'passphrase' => env('LARAVEL_WEBSOCKETS_SSL_PASSPHRASE', null),
    ],

    /*
     * Channel Manager
     * This class handles how channel persistence is handled.
     * By default, persistence is stored in an array by the running webserver.
     * The only requirement is that the class should implement
     * `ChannelManager` interface provided by this package.
     */
    'channel_manager' => \BeyondCode\LaravelWebSockets\WebSockets\Channels\ChannelManagers\ArrayChannelManager::class,
];

```

# Pusher Replacement

The easiest way to get started with Laravel WebSockets is by using it as a [Pusher](https://pusher.com/) replacement. The integrated WebSocket and HTTP Server has complete feature parity with the Pusher WebSocket and HTTP API. In addition to that, this package also ships with an easy to use debugging dashboard to see all incoming and outgoing WebSocket requests

```
composer require pusher/pusher-php-server "~3.0"
```

Next, you should make sure to use Pusher as your broadcasting driver. This can be achieved by setting the `BROADCAST_DRIVER` environment variable in your `.env` file:

```
BROADCAST_DRIVER=pusher
```

## Pusher Configuration

When broadcasting events from your Laravel application to your WebSocket server, the default behavior is to send the event information to the official Pusher server. But since the Laravel WebSockets package comes with its own Pusher API implementation, we need to tell Laravel to send the events to our own server.

To do this, you should add the `host` and `port` configuration key to your `config/broadcasting.php` and add it to the `pusher` section. The default port of the Laravel WebSocket server is 6001.

```
'pusher' => [
    'driver' => 'pusher',
    'key' => env('PUSHER_APP_KEY'),
    'secret' => env('PUSHER_APP_SECRET'),
    'app_id' => env('PUSHER_APP_ID'),
    'options' => [
        'cluster' => env('PUSHER_APP_CLUSTER'),
        'encrypted' => true,
        'host' => '127.0.0.1',
        'port' => 6001,
        'scheme' => 'http'
    ],
],
```

## Configuring WebSocket Apps

The Laravel WebSocket Pusher replacement server comes with multi-tenancy support out of the box. This means that you could host it independently from your current Laravel application and serve multiple WebSocket applications with one server.

To make the move from an existing Pusher setup to this package as easy as possible, the default app simply uses your existing Pusher configuration.

You may add additional apps in your `.env` file. 

```
PUSHER_APP_ID=local
PUSHER_APP_KEY=local
PUSHER_APP_SECRET=local
PUSHER_APP_CLUSTER=mt1
```

Go go ` config/app.php`   and fine the provider then enable command

```
'providers' => [
        App\Providers\BroadcastServiceProvider::class,
    ],
```

# Starting the WebSocket server

Once you have configured your WebSocket apps and Pusher settings, you can start the Laravel WebSocket server by issuing the artisan command:

```
php artisan websockets:serve
```

![](https://i.loli.net/2021/07/05/mLOK786S9h3uNFt.png)

we are already to config websocket successfully, so now we need to test with an event in laravel following bellow code:

```
php artisan make:event WebsocketEvent
```

The laravel will generate package and laravel class in app folder. then we need to implements broadcasting

```
class WebSocketEvent implements ShouldBroadcast {

}
```

change some code : full code

```
<?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class WebSocketEvent implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    /**
     * Create a new event instance.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    public function broadcastWith()
    {
        return [
            'hello' => 'there'
        ];
    }

    /**
     * Get the channels the event should broadcast on.
     *
     * @return \Illuminate\Broadcasting\Channel|array
     */
    public function broadcastOn()
    {
        return new Channel('channel');
    }
}

```

when we already we need to route event, you can route web.php or api.php

after you finish you can run serve http://127.0.0.1:8000/broadcast  this is depend on your route:

```
 Route::get('/broadcast',function (){
        broadcast(new \App\Events\WebSocketEvent());
 });
```

![](https://i.loli.net/2021/07/05/97sLaMFqRBh6Gpc.png)

```
php artisan websockets:serve
```

# **Web Sockets in Vuejs** 

At first you need to install `laravel-echo`

```
npm install laravel-echo pusher-js

or 

yarn add laravel-echo pusher-js
```

when you already install in the vuejs you go to your laravel project go to **resource->js->boostrap.js** open it, you will see below code:

```
// import Echo from 'laravel-echo';

// window.Pusher = require('pusher-js');

// window.Echo = new Echo({
//     broadcaster: 'pusher',
//     key: process.env.MIX_PUSHER_APP_KEY,
//     cluster: process.env.MIX_PUSHER_APP_CLUSTER,
//     forceTLS: true
// });
```



copy and go to your vuejs project and **main.js** pass your code to enable your comment code like below and change some code:

```
import Echo from 'laravel-echo'

window.Pusher = require('pusher-js')

window.Echo = new Echo({
  broadcaster: 'pusher',
  key: process.env.VUE_APP_WEBSOCKETS_KEY,
  wsHost: process.env.VUE_APP_WEBSOCKETS_SERVER,
  wsPort: 6001,
  forceTLS: false,
  disableState: true
})
```

Change key, wsHost, wsPort in the .env.development file and  npm run serve to inspect your browser to see like below:

```
VUE_APP_WEBSOCKETS_KEY = local
VUE_APP_WEBSOCKETS_SERVER = 127.0.0.1
```

![](https://i.loli.net/2021/07/05/QYRwjeztM5cXJga.png)

we also check websocket event below just use code anywhere you want to test:

```

<script>
export default {
  name: 'App',
  mounted() {
    window.Echo.channel('channel')
      .listen('WebSocketEvent', (e) => {
        console.log(e)
      })
  }
}
</script>
```

inspect and go to console in your browsers

![](https://i.loli.net/2021/07/05/oyW2NDeiahfKcb8.png)

# Laravel Websockets documents

https://beyondco.de/docs/laravel-websockets/getting-started/introduction

https://freek.dev/1228-introducing-laravel-websockets-an-easy-to-use-websocket-server-implemented-in-php

