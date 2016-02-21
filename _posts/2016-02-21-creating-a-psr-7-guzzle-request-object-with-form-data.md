---
layout: post
title: "Creating a PSR-7 Guzzle Request object with form-data"
comments: true
disqus_identifier: creating-a-psr-7-guzzle-request-object-with-form-data
---

Uploading files with [Guzzle 6](http://docs.guzzlephp.org/en/v6/) using a `multipart/form-data` request is pretty straightforward. But creating a request object with a file upload independently from the actual sending process turns out to not be that obvious.

When wrapping everything within one step, performing a file upload with Guzzle 6 using `multipart/form-data` is as simple as:

```php
<?php

use GuzzleHttp\Client;

$client = new Client();
$response = $client->request(
    'POST', 
    'https://xyz.acme.dev/api/upload', 
    [
        'multipart' => [
            [
                'name' => 'upload_file',
                'contents' => fopen('path/to/file', 'r')
            ],
            // ... more fields
        ]
    ]
);
```

But recently I stumbled upon the requirement to create the request object decoupled from actually sending it with the client. Having only a simple string payload in the message body sending as a `POST` request would still be quite easy:

```php
<?php

use GuzzleHttp\Psr7\Request;

$payload = 'My payload';
$request = new Request(
    'POST', 
    'https://xyz.acme.dev/api/upload', 
    [], 
    $payload
);
$response = $client->send($request);
```

But how to a create `multipart/form-data` payload? Because files can get huge they need to be fed into the request by a stream. Guzzle provides [a lot of different stream types](http://docs.guzzlephp.org/en/v6/psr7.html#streams). Unfortunately the one we need is currently not linked from the docs, but you can find it in the supporting [guzzle/psr7](https://github.com/guzzle/psr7#multipartstream) library. Using a `MultipartStream`, the request object can now be created decoupled from the client:

```php
<?php

use GuzzleHttp\Psr7\Request;
use GuzzleHttp\Psr7\MultipartStream;

$multipart = new MultipartStream([
    [
        'name' => 'upload_file',
        'contents' => fopen('path/to/file', 'r')
    ],
    // ... more fields
]);
$request = new Request(
    'POST', 
    'https://xyz.acme.dev/api/upload', 
    [], 
    $multipart
);
$response = $client->send($request);
```

If required, you can now pass around this `$request` object as needed before actually sending it with the Guzzle client.
