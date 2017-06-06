# Content Size Policy

**Goal: enable developers to set and enforce limits on content size used by nested contexts (i.e. iframes).**

Today the top-level frame is unable to audit or enforce size limits on resources used by a nested context. For example, the developer may take great care to optimize their assets to fit within a specific quota, but the moment they add a third party resource that uses (or injects) a nested context, they have no way to track or enforce limits on the resulting size.

In practice, this can result in mis{behaving, configured} parties fetching large amounts of data, which violate site owners expectations and harm the user experience. To address this, one could imaging enabling site authors to specify expected limits that the browser could then enforce on their behalf...

```html
  <!-- insert vigorous handwaving here -->
  <iframe max-content-size="300kb" src="...">
  <!-- alternatively, header mechanism to set similar limits -->
```

The above limits intend to enforce the total amount of resources used by the frame, regardless whether the resources comes from cache or not. This is to allow site authors to enforce some reasonable limits on amounts of resources that are executed by the nested context.

## Enforcement
Once a frame's threshold has been reached, the frame is marked as a violating frame and all of its (and its children's) in-flight resource requests are cancelled and future requests are aborted.

For privacy purposes (as described below) there is a limit to the number of violations that can occur per top-level page navigation. After the limit has been reached, all frames with a max-content-size (and their children) will abort requests.

As an example, suppose the violation limit is 10 per navigation. This means that if you have 11 iframes with size policy and 10 of them exceed their max size, then the 11th will also stop loading.

*Be very careful when using size policy to constrain critical components of your page.*


## Notification
TBD - Should the frame be informed that it has a size constraint? Should it be informed (à la Reporting API, or JavaScript) if it violates the size constraint?

## Privacy & Security

The browser can't enforce the exact max-content-size as provided to it, as that could be used to expose the precise size of cross-origin resources. Knowing response size of a cross-origin resource can reveal information about its contents and/or user state (e.g., authenticated vs log-in page response).

In order to prevent size policy from leaking the precise size of cross-origin resources, the enforcement is fuzzy and the number of violations is limited. The actual enforced max size for a frame is hidden from the page, but it's a function of the max size that includes a random number of bytes (the random pad). The random distribution is chosen to statistically reveal no more information than simple [network timing](https://www.igvita.com/2016/08/26/stop-cross-site-timing-attacks-with-samesite-cookies/) given a limited number of samples.

To prevent statistical defeat of the random pad, the number of size policy violations that may occur per top-level navigation will be fixed to a small number, such as 10. After the maximum number of size policy violations have occurred, all frames with size policies will be considered in violation and won't be able to load resources.

_We need to carefully evaluate the privacy and security implications of exposing this mechanism._

---

- Discuss: https://github.com/WICG/content-size/issues
- [blink-dev] [Intent to implement: content size policy](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/N0HybdIpKBs)
 - [Technical design doc (blink)](https://docs.google.com/document/d/1dg3zblqRjNMcM-xUno-q1dLZz9vGP7qukzD9EEuFAC4/edit#heading=h.2uabi8vktqox)
