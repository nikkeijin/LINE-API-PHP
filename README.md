# LINE BOT - PHP Starter

This is a simple PHP starter code for building LINE bots using the LINE Messaging API.        
The code is designed to handle Follow Events and Text Message Events.

## Installation

Install the LINE Messaging API SDK using [Composer](https://getcomposer.org/).

For PHP 8.1 or later

```bash
composer require linecorp/line-bot-sdk
```

For PHP 7

```bash
composer require linecorp/line-bot-sdk:7.5.0
```

## Uninstallation
```bash
composer remove linecorp/line-bot-sdk:$version
```

## Starter Code

```php
<?php

require __DIR__ . '/../vendor/autoload.php';

use LINE\LINEBot;
use LINE\LINEBot\HTTPClient\CurlHTTPClient;
use LINE\LINEBot\Event\FollowEvent;
use LINE\LINEBot\Event\MessageEvent\TextMessage;
use LINE\LINEBot\MessageBuilder\MultiMessageBuilder;
use LINE\LINEBot\MessageBuilder\TextMessageBuilder;

$channelAccessToken = 'YOUR_CHANNEL_ACCESS_TOKEN';
$channelSecret = 'YOUR_CHANNEL_SECRET_KEY';

$httpClient = new CurlHTTPClient($channelAccessToken);
$bot = new LINEBot($httpClient, ['channelSecret' => $channelSecret]);

// Handle incoming messages
$signature = $_SERVER['HTTP_X_LINE_SIGNATURE'];
$body = file_get_contents('php://input');

try {
    $events = $bot->parseEventRequest($body, $signature);
} catch (\Exception $e) {
    error_log($e->getMessage());
    http_response_code(500);
    exit();
}

foreach ($events as $event) {

    $userId = $event->getUserId();
    $profile = $bot->getProfile($userId);
    $profile = $profile->getJSONDecodedBody();
    $displayName = $profile['displayName'];
    $multiMessageBuilder = new MultiMessageBuilder();

    // Handle　Follow Event
    if ($event instanceof FollowEvent) {
        // Single Message
        $greetings = "Hello, $displayName!\n\nThanks for following us!";
        $bot->replyText($event->getReplyToken(), $greetings);
    }

    // Handle Message Event
    if ($event instanceof TextMessage) {
        // User Message
        $userMessage = $event->getText();
        // Multiple Message
        $multiMessageBuilder->add(new TextMessageBuilder("Thanks for your message!\n\nYour message:\n$userMessage"));
        $multiMessageBuilder->add(new TextMessageBuilder('You may use the rich menu to interact with me!'));
        $bot->replyMessage($event->getReplyToken(), $multiMessageBuilder);
    }

}

http_response_code(200);

```

## Documentation

See the official API documentation for more information.

[Getting Started](https://github.com/line/line-bot-sdk-php/wiki/Getting-started)

### Useful Guides

[Message Types](https://developers.line.biz/en/docs/messaging-api/message-types/)
