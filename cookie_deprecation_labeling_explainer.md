# Cookie Deprecation Labeling

## Motivation 

To prepare for third-party cookie deprecation, it is important to understand the full impact of Chrome’s planned transition from third-party cookies to the Privacy Sandbox Ads APIs.

One important step is to support opt-in server side testing by ad techs, which can examine how systems perform if cookies were not to be used. Given the large number of ad techs involved in these systems, for example in a Protected Audiences auction, the browser is in a unique position to support consistent experimentation across all of these parties.

Browser-determined control and treatment groups would allow all of these parties to run experiments involving traffic from consistent groups of users, allowing meaningful comparison and experimentation of cookie deprecation.

This API is intended to only be offered for a short period of time to support third party cookie phase out, and should not be relied on as a stable web platform API.

## Cookie-Deprecation-Header

The browser can randomly assign an experiment label to a portion of users, that can be used by ad techs to perform coordinated experimentation.

Labels can be sent to servers via a new HTTP request header:

```
Sec-Cookie-Deprecation: label
```

Accessing labels involves accessing information stored on the user’s device. In some jurisdictions (for example, the EU and UK), we understand that this activity is analogous to the use of cookies and thus accessing labels likely requires end user consent. Before requesting labels, we recommend that you seek legal advice as to whether this consent obligation applies to you.

To ensure that servers have the opportunity to not receive labels, origins should need to opt-in to receiving labels. Opt-in can be done by setting a special partitioned cookie:

```
Set-Cookie: receive-cookie-deprecation=1; Secure; HttpOnly; Path=/; SameSite=None; Partitioned;
```

Notably, this means that on first visit to a new top-level site, the initial subresource loads to an origin will not carry the label header.

The Secure, HttpOnly, SameSite, and Partitioned cookie attributes are mandatory. The other attributes: Domain, Expires, and Max-Age may be set as desired.

## JS Access

To allow access to labels prior to setting the cookie (e.g. prior to first request), a JS API is also exposed. Invoking the JS API does not require separate opt-in.

```js
// Feature detect temporary API first
if ('cookieDeprecationLabel' in navigator) {
  // Request value and resolve promise
  navigator.cookieDeprecationLabel.getValue().then((label) => {
    console.log(label);
    // Expected output: "example_label_1"
  });
}
```

If the browser is not part of a group, then the value will be an empty string. The browser may not expose the API to JS if the user is not part of a group, so it will be necessary to feature detect the API.

# Privacy Considerations

The label advertised by the browsers adds new [active fingerprinting](https://www.w3.org/TR/fingerprinting-guidance/#active-0) information to the browser.

The amount of information revealed depends on the size of groups given a specific label. Considering the smallest possible group being .25% of the browser population, this reveals -log2(.0025) =8.64 bits of information.

The [Shannon entropy](https://en.wikipedia.org/wiki/Entropy_(information_theory)) can be calculated given the size and number of all labels.

## Information is only temporary

The label values are only needed to support third party cookie phase out. Once cookie phase out is complete, this API will be removed, which limits the long term risk of exposing this information.

## Users with cookies disabled should not send labels

To ensure we are not introducing new information which is not already obtainable (e.g., via cookies), the labeling APIs should be disabled for users who have explicitly turned off cookies in their browser.

# For developers

For information on how Chrome intends to use labeling, please see: [https://developer.chrome.com/docs/privacy-sandbox/chrome-testing/](https://developer.chrome.com/docs/privacy-sandbox/chrome-testing/).
