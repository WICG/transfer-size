# Transfer Size Policy

**Goal: enable developers to set and enforce limits on network usage by nested contexts (i.e. iframes).**

Today the top-level frame is unable to audit or enforce size limits on resources used by a nested context. For example, the developer may take great care to optimize their assets to fit within a specific quota, but the moment they add a third party resource that uses (or injects) a nested context, they have no way to track or enforce limits on the resulting size.

In practice, this can result in mis{behaving, configured} parties fetching large amounts of data, which violate site owners expectations and harm the user experience. To address this, frames could trigger an event once they've consumed more than a specified number of bytes over the network. The page could then handle the event in any way it chooses, such as logging it, displaying a notice to the user, [pausing](https://github.com/jkarlin/pause-frame) it, unloading it, etc. There might even be a future API to cancel all current and future network requests for a frame as a response.

Here is an example:

```html
  <script>
    function onBigFrame(event) {
      console.log("Frame with id: " + event.target.id + " exceeded its threshold bytes");
    }
  </script>

  <iframe id="foo" ontransferexceeded="onBigFrame(event)" transfer-threshold="300KB" src="...">
```

Transfer Size Policy can also be configured via response headers. This is especially useful if third-party script creates the frame such that you can't add the attribute to the frame yourself:

```http
  Transfer-Thresholds: {"default":"100KB", "self":"Infinite", "*.example.com":"200KB"}
```

and in your script:
```javascript
document.ontransferexceeded=onBigFrame;
```

## Details
Each frame with a `transfer-threshold` will count the number of bytes that it and its child frames (transitively) receive from requests that utilize the network (cached responses are not counted). Each response from an origin that differs from the embedding frame's origin will have a random number of bytes added to its size. Once the threshold is crossed, the `transferexceeded` event is fired. The event bubbles and has a target of the frame element in the embedding page.

There will be a cap on the number of `transferexceeded` events that will be fired per top-level pageload in order to help [preserve privacy](#Privacy-&-Security).

## Response
It is up to the event handler to determine what to do about the frame exceeding its set size. Examples include:

 * Logging that the frame exceeded its threshold
 * [Pausing](https://github.com/jkarlin/pause-frame) the frame
 * Unloading the frame
 * Informing the user

## Privacy & Security
According to the [same origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy), it should not be possible to determine the response size of a cross-origin resource. It's impossible for this API to exist and not leak some amount of size information. The best we can do is to limit it to as small amount as is practically possible. That is what is done for other APIs that leak size information. 

There are two mitigations to help minimize size leakage:
 1. Each cross-origin resource's size is padded with a random value (e.g., -1 * rand(0KB, 20KB)). Because the pad is negative, the event will only fire if the frame is at least threshold size, though it may be larger at the time of firing.
 1. The is a cap on the number of `transferexceeded` events that can be fired per top-level page load to stop adversaries from taking lots of samples in order to statistically defeat the random padding. (e.g., 100)
 
Specifics about the random pad distribution and the size of the event cap will be provided in the specification.

---

- Discuss: https://github.com/WICG/content-size/issues
