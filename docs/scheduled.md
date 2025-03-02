# ⏰ Scheduled Events

- [`ScheduledEvent` Reference](https://developers.cloudflare.com/workers/runtime-apis/scheduled-event)
- [`addEventListener` Reference](https://developers.cloudflare.com/workers/runtime-apis/add-event-listener)

## Cron Triggers

`scheduled` events are automatically dispatched according to the specified cron
triggers:

```shell
$ miniflare --cron "15 * * * *" --cron "45 * * * *" # or -t
```

```toml
# wrangler.toml
[triggers]
crons = ["15 * * * *", "45 * * * *"]
```

```js
const mf = new Miniflare({
  crons: ["15 * * * *", "45 * * * *"],
});
```

## HTTP Triggers

Because waiting for cron triggers is annoying, you can also make HTTP requests
to `/.mf/scheduled` to trigger `scheduled` events, when using the CLI or
starting an HTTP server with `createServer`:

```shell
$ curl "http://localhost:8787/.mf/scheduled"
```

To simulate different values of `scheduledTime` and `cron` in the dispatched
event, use the `time` and `cron` query parameters:

```shell
$ curl "http://localhost:8787/.mf/scheduled?time=1000"
$ curl "http://localhost:8787/.mf/scheduled?cron=* * * * *"
```

## Dispatching Events

When using the API, the `dispatchScheduled` function can be used to dispatch
`scheduled` events to your worker. This can be used for testing responses. It
takes optional `scheduledTime` and `cron` parameters, which default to the
current time and the empty string respectively. It will return a promise which
resolves to an array containing data returned by all waited promises:

```js
import { Miniflare } from "miniflare";

const mf = new Miniflare({
  script: `
  addEventListener("scheduled", (event) => {
    event.waitUntil(Promise.resolve(event.scheduledTime));
    event.waitUntil(Promise.resolve(event.cron));
  });
  `,
});

let waitUntil = await mf.dispatchScheduled();
console.log(waitUntil[0]); // Current time in milliseconds
console.log(waitUntil[1]); // ""

waitUntil = await mf.dispatchScheduled(1000);
console.log(waitUntil[0]); // 1000
console.log(waitUntil[1]); // ""

waitUntil = await mf.dispatchScheduled(1000, "* * * * *");
console.log(waitUntil[0]); // 1000
console.log(waitUntil[1]); // "* * * * *"
```
