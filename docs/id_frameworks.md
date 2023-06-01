# Optable ID Frameworks

Passing specially encrypted identifiers in the bidstream for specific bidders 

## Prerequisites
# Business prerequisites

Optable ID frameworks are a way to pass encrypted identifiers to specific bidders in the programmating bidstream. As such, it is required to have a commercial [data collaboration](https://docs.optable.co/optable-documentation/clean-rooms/overview) established within Optable with one or more [partner](https://docs.optable.co/optable-documentation/partnerships/overview), and for relevant [ID framework(s)](https://docs.optable.co/optable-documentation/settings/id-frameworks) to be enabled. 

# Technical prerequisites

Optable [web SDK](https://github.com/Optable/optable-web-sdk) has to be installed and configured on your web properties, and must use both the [identify](https://github.com/Optable/optable-web-sdk/blob/master/README.md#identify-api) API, and the [targeting](https://github.com/Optable/optable-web-sdk/blob/master/README.md#targeting-api) API. 

The `identify` API enables you to associate a user's browser with a known identifier, such as an email address or a publisher assigned user ID.

The `targeting` API retrieves targeting data, including the ID `frameworks` ids when available, and stores it in browser local storage for subsequent retrieval.

## Additional configuration
# UID2.0 - The Trade Desk

No additional configuration required

# Loblaw Media Private ID (LMPID) - Through GPT Secure Signals

You must call the `installGPTSecureSignals()` SDK API shortly after Optable SDK instantiation. This API will configure GPT to pass `lmpid` to Google Ad Manager (GAM), when it is available in browser local storage.

Try to call the Optable SDK `targeting()` API as early as possible. This API will retrieve the latest lmpid, when it is enabled and available for the current user, and store it in browser local storage for retrieval by GPT as previously described.

# Loblaw Media Private ID (LMPID) - Through prebid.js

For Prebid.js installations, you must make sure to include the `mabidder` bidder adapter and the `lmpid` user ID module. Note that you must include `mabidder` in the `addAdUnits()` call. See [Prebid.js download instructions](https://docs.prebid.org/download.html) and [Prebid.js user ID modules](https://docs.prebid.org/dev-docs/modules/userId.html#user-id-sub-modules) for details.

## Prebid.js and GPT integration example
The code snippet below shows an example integration with GPT on page. Note that `{OPTABLE_DCN_HOST}` and `{OPTABLE_DCN_SOURCE_SLUG}` must be replaced with your Optable Data Collaboration Node (DCN) hostname and Javascript SDK source slug, respectively.

```
// Setup both Optable and GPT SDKs.
window.optable = window.optable || { cmd: [] };
window.googletag = window.googletag || { cmd: [] };

// When optable SDK is loaded, initialize it and install GPT Secure Signals provider.
optable.cmd.push(function () {
  optable.instance = new optable.SDK({
    host: "{OPTABLE_DCN_HOST}",
    site: "{OPTABLE_DCN_SOURCE_SLUG}",
  });

  optable.instance.installGPTSecureSignals();
  optable.instance.targeting()
});

// When GPT SDK is loaded, define and prepare an ad slot.
// Note that we disableInitialLoad() in order to defer the first ad request
// until secure signals are installed.
googletag.cmd.push(function() {
  googletag.pubads().disableInitialLoad();

  googletag
    .defineSlot('/my/slot', [[728, 90]], "ad")
    .addService(googletag.pubads());

  googletag.pubads().enableSingleRequest();
  googletag.enableServices();
  googletag.display("ad");
});

// Explicitly refresh ads when everything is loaded and the above setup is done.
googletag.cmd.push(function () {
  optable.cmd.push(function () {
    googletag.pubads().refresh();
  });
});
```

## Try it
Check our demo pages for [GAM Secure Signals]([https://demo.optable.co/integration/lmpid-prebid-gpt.html?cookies=yes](https://demo.optable.co/integration/lmpid-signal-gpt.html?cookies=yes) and [Prebid.js](https://demo.optable.co/integration/lmpid-prebid-gpt.html?cookies=yes) integrations.
