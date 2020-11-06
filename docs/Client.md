Client • [Server](Server.md) • [Changelog](Changelog.md) • [Contributing](Contributing.md) • [License](COPYING.md)

# Websocket: Client

The client can read and write on a WebSocket stream.
It internally supports Upgrade handshake and implicit close and ping/pong operations.

##  Class synopsis

```php
WebSocket\Client {

    public __construct(string $uri, array $options = [])
    public __destruct()

    public send(mixed $payload, string $opcode = 'text', bool $masked = true) : void
    public receive() : mixed
    public close(int $status = 1000, mixed $message = 'ttfn') : mixed

    public getLastOpcode() : string
    public getCloseStatus() : int
    public isConnected() : bool
    public setTimeout(int $seconds) : void
    public setFragmentSize(int $fragment_size) : self
    public getFragmentSize() : int
    public setLogger(Psr\Log\LoggerInterface $logger = null) : void
}
```

## Examples

### Simple send-receive operation

This example send a single message to a server, and output the response.

```php
$client = new WebSocket\Client("ws://echo.websocket.org/");
$client->send("Hello WebSocket.org!");
echo $client->receive();
$client->close();
```

### Listening to a server

To continuously listen to incoming messages, you need to put the receive operation within a loop.
Note that these functions **always** throw exception on any failure, including recoverable failures such as connection time out.
By consuming exceptions, the code will re-connect the socket in next loop iteration.

```php
$client = new WebSocket\Client("ws://echo.websocket.org/");
while (true) {
    try {
        $message = $client->receive();
        // Act on received message
        // Break while loop to stop listening
    } catch (\WebSocket\ConnectionException $e) {
        // Possibly log errors
    }
}
$client->close();
```

## Constructor options

The `$options` parameter in constructor accepts an associative array of options.

* `timeout` - Time out in seconds. Default 5 seconds.
* `fragment_size` - Maximum payload size. Default 4096 chars.
* `context` - A stream context created using [stream_context_create](https://www.php.net/manual/en/function.stream-context-create).
* `headers` - Additional headers as associative array name => content.
* `logger` - A [PSR-3](https://www.php-fig.org/psr/psr-3/) compatible logger.
* `persistent` - Connection is re-used between requests until time out is reached. Default false.

```php
$context = stream_context_create();
stream_context_set_option($context, 'ssl', 'verify_peer', false);
stream_context_set_option($context, 'ssl', 'verify_peer_name', false);

$client = new WebSocket\Client("ws://echo.websocket.org/", [
    'timeout' => 60, // 1 minute time out
    'context' => $context,
    'headers' => [
        'Sec-WebSocket-Protocol' => 'soap',
        'origin' => 'localhost',
    ],
]);
```

## Exceptions

* `WebSocket\BadOpcodeException` - Thrown if provided opcode is invalid.
* `WebSocket\BadUriException` - Thrown if provided URI is invalid.
* `WebSocket\ConnectionException` - Thrown on any socket I/O failure.
* `WebSocket\TimeoutExeception` - Thrown when the socket experiences a time out.