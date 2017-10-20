# Transfer Size Policy

**Goal: enable developers to set and enforce limits on network usage by nested contexts (i.e. iframes).**

Today the top-level frame is unable to audit or enforce size limits on resources used by a nested context. For example, the developer may take great care to optimize their assets to fit within a specific quota, but the moment they add a third party resource that uses (or injects) a nested context, they have no way to track or enforce limits on the resulting size.

In practice, this can result in mis{behaving, configured} parties fetching large amounts of data, which violate site owners expectations and harm the user experience. To address this, frames could trigger an event once they've consumed at least a specified number of bytes over the network. The page could then handle the event in any way it chooses, such as logging it, displaying a notice to the user, [pausing](https://github.com/jkarlin/pause-frame) it, unloading it, etc. There might even be a future API to cancel all current and future network requests for a frame as a response.

Here is an example which limits an iframe to 300KB total:

```html
  <script>
    function onBigFrame(event) {
      console.log("Frame id: " + event.target.id + " exceeded its threshold bytes in category: " + event.category);
    }
  </script>

  <iframe id="foo" ontransferexceeded="onBigFrame(event)" transfer-threshold="300" src="...">
```

Here is an example that limits the total size of the page to 500KB, of which CSS can use 50KB max:
```
  <iframe id="foo" ontransferexceeded="onBigFrame(event)" transfer-threshold="style:50, total:500" src="...">
```

And here is an example that only limits the size of videos on the frame to 10MB:
```
  <iframe id="foo" ontransferexceeded="onBigFrame(event)" transfer-threshold="video:10240" src="...">
```

Transfer Size Policy can also be configured via response headers. This is especially useful if third-party script creates the frame such that you can't add the attribute to the frame yourself. In this example, the default threshold for a frame is 100KB, the site's origin is unrestricted, and \*.example.com is limited to 500KB and 50KB of style:

```http
  Transfer-Thresholds: {"default":"100", "self":"*", "*.example.com":"style:50, total:500"}
```

and in your script:
```javascript
document.ontransferexceeded=onBigFrame;
```

## Use Cases
* Enforcing size policies on content, such as AMP's 50,000 byte limit on CSS
* Logging how often your own site grows larger than desired
* Monitoring (or even enforcing policy upon) how often ads get too large

## Details
Each frame with a `transfer-threshold` will count the number of over-the-wire bytes that it and its child frames (transitively) receive from requests that utilize the network (cached responses are not counted). Once a threshold is crossed, the bubbling `transferexceeded` event (a new event type which includes the resource category that was exceeded) is fired on the frame element in the parent frame.

If any of the network bytes are from origins other than the embedding frame's origin, a random pad is added to `transfer-threshold` so as not to reveal the exact size of the resources in the frame.

There will be a cap on the number of `transferexceeded` events that will be fired per top-level pageload in order to help [preserve privacy](#Privacy-&-Security).

## Response
It is up to the event handler to determine what to do about the frame exceeding its set size. Examples include:

 * Logging that the frame exceeded its threshold
 * [Pausing](https://github.com/jkarlin/pause-frame) the frame
 * Unloading the frame
 * Informing the user

## Privacy & Security
According to the [same origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy), it should not be possible to determine the response size of a cross-origin resource. It's impossible for this API to exist and not leak some amount of size information. In particular, it leaks information about the full size of a page. The best we can do is to limit the leak to as small amount as is practically possible. That is what is done for other APIs that leak size information. 

There are two mitigations to help minimize size leakage:
 1. Frames with network responses that are cross-origin with respect to the embedding page have a random amount of bytes added to that component of their `transfer-threshold`. The developer can be certain that the `transferexceeded` event will only fire if at least the threshold bytes were observed, but it may have been even more in a cross-origin situation.
 1. There is a cap on the number of `transferexceeded` events that can be fired per top-level page load to stop adversaries from taking lots of samples in order to statistically defeat the random padding. (e.g., 10-100)
 
Specifics about the random pad distribution and the size of the event cap will be provided in the specification.

---

- Discuss: https://github.com/WICG/content-size/issues
