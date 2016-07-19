# Content Size Policy

**Goal: enable developers to set and enforce limits on content size used by nested contexts (i.e. iframes).**

Today the top-level frame is unable to audit or enforce size limits on resources used by a nested context. For example, the developer may take great care to optimize their assets to fit within a specific quota, but the moment they add a third party resource that uses (or injects) a nested context, they have no way to track or enforce limits on the resulting size.

In practice, this can result in mis{behaving, configured} parties fetching large amounts of data, which violate site owners expectations and harm the user experience. To address this, one could imaging enablign site authors to specify expected limits that the browser could then enforce on their behalf...

```html
  <!-- insert vigorous handwaving here -->
  <iframe max-content-size="300kb" src="..."> 
  <iframe max-javascript-size="150kb" src="..."> 
  
  <!-- alternatively, header mechanism to set similar limits -->
```

The above limits intend to enforce the total amount of resources used by the page, regardless whether the resources comes from cache or not. This is to allow site authors to enforce some reasonable limits on amounts of script/other resources that are executed by the nested context. 

## Enforcement

TBD and subject to privacy/security considerations. Some plausible strategies:

- Once threshold is reached, cancel in-flight requests and abort future requests
- Allow in-flight requests to finish, abort new requests


## Privacy & Security

Resource Timing API exposes [encodedBodySize attribute](http://w3c.github.io/resource-timing/#dom-performanceresourcetiming-encodedbodysize) which provides the size of a particular resource prior to removing any content codings. This attribute is exposed by default for same-origin resources, and for third-party resources that provide the TAO opt-in header. However, RT API does not allow the top-level context to introspect inside of a nested context and see which resources it's fetching, or their size. As a result, RT is not sufficient to address this problem. Further...

We can't simply expose the exact number of bytes consumed by the nested context:
- Knowing response size of a resource can reveal information about its contents and/or users state - e.g. authenticated vs log-in page response.
- Forcing coarse thresholds doesn't solve the problem either because it is possible to embed any resource inside an iframe with known “padding” (e.g. another resource whose size you know or control) and then figure out the size of the resource you’re interested in.

_We need to carefully evaluate the privacy and security implications of exposing this mechanism._

---

Discuss: https://github.com/WICG/content-size/issues
