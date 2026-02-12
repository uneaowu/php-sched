## PHP Sched
This is an experimental cooperative scheduler implemented in PHP for educational and recreational purposes.

The context swithing managed through the functions `go`, `stream_read`, `stream_write`, `socket_accept_`, `delay`.

Not intended for production use.

### Example
```php
...

/** @var Channel<string> */
$chan = chan();

// The `go` function dispatches a routine to run in the background.

go(static function (string $id, Channel $chan): void {
    /** @var Channel<string> $chan */

    for ($i = 0; $i < 2; ++$i) {
        delay(Duration::milliseconds(500));
        dprintfn('worker %s has woken up', $id);
        $chan->send($id);
    }

    $chan->close();

    dprintfn('worker %s has terminated', $id);
}, '01', $chan);

go(static function (string $id): void {
    for ($i = 0; $i < 5; ++$i) {
        delay(Duration::milliseconds(200));
        dprintfn('worker %s has woken up', $id);
    }

    dprintfn('worker %s has terminated', $id);
}, '02');

go(static function (string $id, Channel $chan): void {
    /** @var Channel<string> $chan */

    while (! $chan->isClosed()) {
        $msg = $chan->receive();
        dprintfn("worker %s has received from channel: '%s'", $id, $msg);
    }

    dprintfn('worker %s has terminated', $id);
}, '03', $chan);

// Runtime will block the current execution until all pending tasks have been completed.
```
