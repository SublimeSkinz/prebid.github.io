---
layout: page_v2
page_type: module
title: Module - User ID
description: Supports multiple cross-vendor user IDs
module_code : userId
display_name : User ID
enable_download : false
sidebarType : 1
---

# User ID Module
{:.no_toc}

* TOC
{:toc}

## Overview

The User ID module supports multiple ways of establishing pseudonymous IDs for users, which is an important way of increasing the value of header bidding. Instead of having several exchanges sync IDs with dozens of demand sources, a publisher can choose to integrate with one of these ID schemes:

* **BritePool ID** - Britepool Identity Resolution userId submodule. Universal Identity resolution which does not depend on 3rd party cookies.
* **Criteo ID for Exchanges** –  specific id for Criteo and its partners that enables optimal take rate on all web browsers.
* **ID+** - a universal identity solution that aims to empower the marketing ecosystem with a privacy-compliant mechanism to overcome identity resolution challenges arising from ID fragmentation, cookie restrictions, and increasingly stringent privacy and security regulation. Powered by zeotap and built upon the extensive marketing identity graph, ID+ offers a solution for the future of marketing.
* **ID5 Universal ID** - a neutral identifier for digital advertising that can be used by publishers, brands and ad tech platforms (SSPs, DSPs, DMPs, Data Providers, etc.) to eliminate the need for cookie matching.
* **Identity Link** – provided by LiveRamp, this module calls out to the ATS (Authenticated Traffic Solution) library or a URL to obtain the user’s IdentityLink envelope.
* **IntentIQ ID** – An identity resolution pioneer, Intent IQ enables its partners to confidently identify clients and prospects who interact with their sites, apps and their brick and mortar establishments, whether across their various screens or in person.
* **LiveIntent ID** – a user identifier tied to an active, encrypted email in our graph and functions in cookie-challenged environments and browsers.
* **Lotame Panorama ID** - a people-based identifier available across all browsers -- including when third-party cookies are not available -- to connect and activate first, second, and third party data for programmatic advertising.
* **Parrable ID** - an encrypted pseudonymous ID that is consistent across all browsers and webviews on a device for every publisher the device visits.  This module contacts Parrable to obtain the Parrable EID belonging to the specific device which can then be used by the bidder.
* **PubCommon ID** – an ID is generated on the user’s browser and stored for later use on this publisher’s domain.
* **Quantcast ID** - an ID independent of third-party cookies for publishers with Quantcast Measure tag.      
* **Unified ID** – a simple cross-vendor approach – it calls out to a URL that responds with that user’s ID in one or more ID spaces (e.g. adsrvr.org).
* **netID** – provides an open, standardized, EU-GDPR compliant, IAB TCF aware, cross-device enabled Advertising Identifier Framework, which can be leveraged by publishers and advertisers (and vendors supporting them) to efficiently deliver targeted advertising bought through programmatic systems.

## How It Works

1. The publisher determines which user ID modules to add to their Prebid.js package and consults with their legal counsel to determine the appropriate user disclosures.
1. The publisher builds Prebid.js by specifying one or more ID sub-modules they would like to include. e.g. "gulp build --modules=____IdSystem". You also need to add the `userId` module to your Prebid.js distribution.
1. The page defines User ID configuration in `pbjs.setConfig()`
1. When `setConfig()` is called, and if the user has consented to storing IDs locally, the module is invoked to call the URL if needed
   1. If the relevant local storage is present, the module doesn't call the URL and instead parses the scheme-dependent format, injecting the resulting ID into bidRequest.userId.
1. An object containing one or more IDs (bidRequest.userId) is made available to Prebid.js adapters and Prebid Server S2S adapters.
1. In addition to bidRequest.userId, bidRequest.userIdAsEids is made available to Prebid.js adapters and Prebid Server S2S adapters. bidRequest.userIdAsEids has userIds in ORTB EIDS format.

Note that User IDs aren't needed in the mobile app world because device ID is available in those ad serving scenarios.

Also note that not all bidder adapters support all forms of user ID. See the tables below for a list of which bidders support which ID schemes.

As of Prebid 4.0, this module will attempt storage in the main domain of the publisher's website instead of a subdomain, unless this behavior is overriden by a submodule.

## User ID, GDPR, and Opt-Out

When paired with the [Consent Management](/dev-docs/modules/consentManagement.html) module, privacy rules are enforced:

* The module checks the GDPR consent string
* If no consent string is available OR if the user has not consented to Purpose 1 (local storage):
  * Calls to an external user ID vendor are not made.
  * Nothing is stored to cookies or HTML5 local storage.

In addition, individual users may opt-out of receiving cookies and HTML5 local storage by setting these values:

* `_pbjs_id_optout` cookie or HTML5 local storage
* `_pubcid_optout` cookie or HTML5 local storage (for backwards compatibility with the original PubCommonID module.

## Basic Configuration

By including this module and one or more of the sub-modules, a number of new options become available in `setConfig()`,
all of them under the `userSync` object as attributes of the `userIds` array
of sub-objects. The table below has the options that are common across ID systems. See the sections below for specific configuration needed by each system and examples.

{: .table .table-bordered .table-striped }
| Param under userSync.userIds[] | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| name | Required | String | May be: `"britepoolId"`, `"criteo"`, `"id5id"`, `identityLink`, `"intentIqId"`, `"liveIntentId"`, `"lotamePanoramaId"`, `"parrableId"`, `"quantcastId"`, `"netId"`, `"pubCommonId"`, `"unifiedId"`, `"zeotapIdPlus"` | `"unifiedId"` |
| params | Based on User ID sub-module | Object | | |
| storage | Optional | Object | The publisher can specify some kind of local storage in which to store the results of the call to get the user ID. This can be either cookie or HTML5 storage. This is not needed when `value` is specified or the ID system is managing its own storage | |
| storage.type | Required | String | Must be either `"cookie"` or `"html5"`. This is where the results of the user ID will be stored. | `"cookie"` |
| storage.name | Required | String | The name of the cookie or html5 local storage where the user ID will be stored. | `"_unifiedId"` |
| storage.expires | Strongly Recommended | Integer | How long (in days) the user ID information will be stored. If this parameter isn't specified, session cookies are used in cookie-mode, and local storage mode will create new IDs on every page. | `365` |
| storage.refreshInSeconds | Optional | Integer | The amount of time (in seconds) the user ID should be cached in storage before calling the provider again to retrieve a potentially updated value for their user ID. If set, this value should equate to a time period less than the number of days defined in `storage.expires`. By default the ID will not be refreshed until it expires.
| value | Optional | Object | Used only if the page has a separate mechanism for storing a User ID. The value is an object containing the values to be sent to the adapters. | `{"tdid": "1111", "pubcid": {2222}, "IDP": "IDP-2233", "id5id": "ID5-12345" }` |

## User ID Sub-Modules

### BritePool

BritePool ID, provided by [BritePool](https://britepool.com) is a Universal Identity resolution which does not depend on 3rd party cookies. 

Add it to your Prebid.js package with:

{: .alert.alert-info :}
gulp build --modules=britepoolIdSystem

#### BritePool Registration

Please reach out to [prebid@britepool.com](mailto:prebid@britepool.com) and request your `api_key`. 

The BritePool privacy policy is at [https://britepool.com/services-privacy-notice/](https://britepool.com/services-privacy-notice/).

#### BritePool Configuration

{: .table .table-bordered .table-striped }
| Param under userSync.userIds[] | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| name | Required | String | `"britepoolId"` | `"britepoolId"` |
| params | Required | Object | Details for britepool initialization. | |
| params.api_key | Required | String | BritePool API Key provided by BritePool | "458frgde-djd7-3ert-gyhu-12fghy76dnmko" |
| params.url | Optional | String | BritePool API url | "https://sandbox-api.britepool.com/v1/britepool/id" |
| params.identifier | Required | String | Where identifier in the params object is the key name. At least one identifier is required. Available Identifiers `aaid` `dtid` `idfa` `ilid` `luid` `mmid` `msid` `mwid` `rida` `ssid` `hash` | `params.ssid` `params.aaid` |

#### BritePool Examples

1) Individual params may be set for the BritePool User ID Submodule. At least one identifier must be set in the params.

{% highlight javascript %}
   pbjs.setConfig({
       userSync: {
           userIds: [{
               name: "britepoolId",
               storage: {
                   name: "britepoolid",
                   type: "cookie",
                   expires: 30
               },
               params: {
                   url: "https://sandbox-api.britepool.com/v1/britepool/id", // optional. used for testing
                   api_key: "xxx", // provided by britepool
                   hash: "yyyy", // example identifier
                   ssid: "r894hvfnviurfincdejkencjcv" // example identifier
               }
           }],
           syncDelay: 3000 // 3 seconds after the first auction
       }
   });
{% endhighlight %}

### Criteo ID for Exchanges

Criteo is the leading advertising platform for the Open Internet. The Criteo ID for Exchanges module enables publishers to access Criteo’s unique demand - more than 20.000 advertisers & brands -  to monetize their exchange inventory with an optimal take rate across all browsing environments.
Note that direct access to that demand is also available through [Criteo Direct Bidder](https://www.criteo.com/products/criteo-direct-bidder/), in which case this module is unnecessary.

The Criteo privacy policy is at [https://www.criteo.com/privacy/](https://www.criteo.com/privacy/).

Add it to your Prebid.js package with:

{: .alert.alert-info :}
gulp build --modules=criteoIdSystem

#### Criteo ID Configuration

The Criteo ID module does not require any configuration parameters. It should work as-is provided that bidders use it in their adapters.
When calling Criteo RTB, partners should forward this id in the field `user.ext.prebid_criteoid`.

{: .alert.alert-info :}
NOTE: For optimal performance, the Criteo Id module should be called at every opportunity. It embeds its own optimal caching mechanism. It's best not to use `params.storage` with this module as it may only lower the performances. If you are using multiple id systems, however, you may use it for the other id systems that supports it.

#### Criteo ID Example

{% highlight javascript %}
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: "criteo",
        }]
    }
});
{% endhighlight %}

### ID+

ID+, powered by zeotap, enables the marketing ecosystem to overcome challenges posed by the demise of identifiers and a fast-changing regulatory landscape. ID+ is an open invitation to the entire industry to build the future of identity together.

This sub-module enables the user’s ID+ to be available in the bid request.

More information on ID+ can be found here: [https://idplus.io/](https://idplus.io/)

Add it to your Prebid.js package with:

{: .alert.alert-info :}
gulp build --modules=zeotapIdPlusIdSystem

#### ID+ Registration

You can set up your ID+ account by contacting our support team at [support.idplus@zeotap.com](mailto:support.idplus@zeotap.com) or via [https://idplus.io/contact-us](https://idplus.io/contact-us.html) and we will get back to you.

ID+ is covered under zeotap privacy policy: [Zeotap Privacy Policy](https://zeotap.com/website-privacy-policy).

#### ID+ Example

{% highlight javascript %}
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: "zeotapIdPlus"
        }]
    }
});
{% endhighlight %}


### ID5 Universal ID

The ID5 Universal ID is a shared, neutral identifier that publishers and ad tech platforms can use to recognise users even in environments where 3rd party cookies are not available. The ID5 Universal ID is designed to respect users' privacy choices and publishers’ preferences throughout the advertising value chain. For more information about the ID5 Universal ID and detailed integration docs, please visit [our documentation](https://console.id5.io/docs/public/prebid). We also recommend that you sign up for our [release notes](https://id5.io/universal-id/release-notes) to stay up-to-date with any changes to the implementation of the ID5 Universal ID in Prebid.

#### ID5 Universal ID Registration

The ID5 Universal ID is free to use, but requires a simple registration with ID5. Please visit [id5.io/universal-id](https://id5.io/universal-id) to sign up and request your ID5 Partner Number to get started.

The ID5 privacy policy is at [https://www.id5.io/platform-privacy-policy](https://www.id5.io/platform-privacy-policy).

#### ID5 Universal ID Configuration

First, make sure to add the ID5 submodule to your Prebid.js package with:

{: .alert.alert-info :}
gulp build --modules=id5IdSystem,userId

The following configuration parameters are available:

{: .table .table-bordered .table-striped }
| Param under userSync.userIds[] | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| name | Required | String | The name of this module: `"id5Id"` | `"id5Id"` |
| params | Required | Object | Details for the ID5 Universal ID. | |
| params.partner | Required | Number | This is the ID5 Partner Number obtained from registering with ID5. | `173` |
| params.pd | Optional | String | Publisher-supplied data used for linking ID5 IDs across domains. See [our documentation](https://wiki.id5.io/display/PD/Prebid.js+UserId+Module) for details on generating the string. Omit the parameter or leave as an empty string if no data to supply | `"MT1iNTBjY..."` |

{: .alert.alert-info :}
**NOTE:** The ID5 Universal ID that is delivered to Prebid will be encrypted by ID5 with a rotating key to avoid unauthorized usage and to enforce privacy requirements. Therefore, we strongly recommend setting `storage.refreshInSeconds` to `8` hours (`8*3600` seconds) to ensure all demand partners receive an ID that has been encrypted with the latest key, has up-to-date privacy signals, and allows them to transact against it.

#### ID5 Universal ID Examples

1) Publisher wants to retrieve the ID5 Universal ID through Prebid.js

{% highlight javascript %}
pbjs.setConfig({
  userSync: {
    userIds: [{
      name: "id5Id",
      params: {
        partner: 173,            // change to the Partner Number you received from ID5
        pd: "MT1iNTBjY..."       // optional, see param table above for a link to how to generate this string
      },
      storage: {
        type: "cookie",
        name: "id5id.1st",       // create a cookie with this name
        expires: 90,             // cookie lasts for 90 days
        refreshInSeconds: 8*3600 // refresh ID every 8 hours to ensure it's fresh
      }
    }],
    auctionDelay: 50             // 50ms maximum auction delay
  }
});
{% endhighlight %}

2) Publisher has integrated with ID5 on their own (e.g. via the [ID5 API](https://github.com/id5io/id5-api.js)) and wants to pass the ID5 Universal ID directly through to Prebid.js

{% highlight javascript %}
pbjs.setConfig({
  userSync: {
    userIds: [{
      name: "id5Id",
      value: { "id5id": "ID5-8ekgswyBTQqnkEKy0ErmeQ1GN5wV4pSmA-RE4eRedA" }
    }]
  }
});
{% endhighlight %}

### IdentityLink

IdentityLink, provided by [LiveRamp](https://liveramp.com) is a single person-based identifier which allows marketers, platforms and publishers to perform personalized segmentation, targeting and measurement use cases that require a consistent, cross-channel view of the user in anonymous spaces.

Add it to your Prebid.js package with:

{: .alert.alert-info :}
gulp build --modules=identityLinkIdSystem

#### IdentityLink Registration

Please reach out to [prebid@liveramp.com](mailto:prebid@liveramp.com) and request your `placementId`.

The IdentityLink privacy policy is at [https://liveramp.com/privacy/service-privacy-policy/](https://liveramp.com/privacy/service-privacy-policy/).

#### IdentityLink Configuration

{: .table .table-bordered .table-striped }
| Param under userSync.userIds[] | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| name | Required | String | `"identityLink"` | `"identityLink"` |
| params | Required for Id Link | Object | Details for identityLink initialization. | |
| params.pid | This parameter is required for IdentityLink | String | This is the placementId, value needed for obtaining user’s IdentityLink envelope


#### IdentityLink Examples

1) Publisher passes a placement ID and elects to store the IdentityLink envelope in a cookie.


{% highlight javascript %}
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: "identityLink",
            params: {
                pid: '999'             // Set your real identityLink placement ID here
            },
            storage: {
                type: "cookie",
                name: "idl_env",       // create a cookie with this name
                expires: 30            // cookie can last for 30 days
            }
        }],
        syncDelay: 3000              // 3 seconds after the first auction
    }
});
{% endhighlight %}

2) Publisher passes a placement ID and elects to store the IdentityLink envelope in HTML5 localStorage.

{% highlight javascript %}
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: "identityLink",
            params: {
                pid: '999'          // Set your real identityLink placement ID here
            },
            storage: {
                type: "html5",
                name: "idl_env",    // set localstorage with this name
                expires: 30
            }
        }],
        syncDelay: 3000
    }
});
{% endhighlight %}

### IntentIQ ID

The IntentIQ ID solution is provided by intentiq.com.

Add it to your Prebid.js package with:

{: .alert.alert-info :}
gulp build --modules=intentIqIdSystem

#### IntentIQ ID Registration

You can set up IntentIQ ID by contacting our operations team at [IntentIQ Contact Us] (https://www.intentiq.com/contact-us) and getting your partner id.

The IntentIQ ID privacy is covered under the [IntentIQ Privacy Policy](https://www.intentiq.com/technology-privacy-policy).

#### IntentIQ ID Configuration

{: .table .table-bordered .table-striped }
| Param under userSync.userIds[] | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| name | Required | String | `"intentIqId"` | `"intentIqId"` |
| params | Required for IntentIqId | Object | Details for IntentIqId initialization. | |
| params.partner | Required | String | This is the partner ID value obtained from registering with IntentIQ. | `"1177538"` |

#### IntentIQ ID Examples

1) Publisher has a partner ID from IntentIQ and cookies.

{: .alert.alert-warning :}
{% highlight javascript %}
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: "intentIqId",
            params: {
                parnter: 123456			// valid partner id
            },
            storage: {
                type: "cookie",
                name: "intentIqId",       // create a cookie with this name
                expires: 60                   // cookie can last for 60 days
            }
        }],
        syncDelay: 3000              // 3 seconds after the first auction
    }
});
{% endhighlight %}

2) Publisher supports IntentIQ and HTML5 local storage.

{% highlight javascript %}
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: "intentIqId",
            params: {
                parnter: 123456			// valid partner id
            },
            storage: {
                type: "html5",
                name: "intentIqId",    // set localstorage with this name
                expires: 60
            }
        }],
        syncDelay: 3000
    }
});
{% endhighlight %}

### LiveIntent ID

LiveIntent offers audience resolution by leveraging our next-generation identity solutions. The LiveIntent identity graph is built around a people-based set of data that is authenticated daily through active engagements with email newsletters and media across the web. The LiveIntent ID is a user identifier tied to an active, encrypted email in our graph and functions in cookie-challenged environments and browsers.

Add LiveIntent ID to your Prebid.js package with:

{: .alert.alert-info :}
gulp build --modules=userId,liveIntentIdSystem

The `request.userId.lipb` object would look like:
```
{
  "lipbid": "T7JiRRvsRAmh88",
  "segments": ["999"]
}
```

The adapters can be implemented to use the lipibid as the identifier and segments to which that identifier is associated with. To enable identity resolution for a specific publisher, LiveIntent builds a model on the backend with data collected via an additional call issued on each page load.

#### LiveIntent ID Registration

Please register with us if you’re not a LiveIntent customer already: [https://www.liveintent.com/prebid-registration/](https://www.liveintent.com/prebid-registration/)

When adding LiveIntent’s ID to your Prebid.js package, you disclose or make available Personal Information to LiveIntent.  This information made available to LiveIntent is used to (i) connect with and reference data that already exists in LiveIntent’s graph (the “LiveIntent Graph”), and (ii) authenticate and validate data in aggregate to improve the performance of the services LiveIntent provides. By activating LiveIntent’s module, you hereby confirm that with regard to all individuals to whom Personal Information relates, you have, at or before the point of collecting Personal Information or making Personal Information available to LiveIntent, provided any legally required notices, obtained any legally required consents, and provided individuals with an opportunity to opt-out of the sharing of Personal Information, if such an opt-out is required under applicable laws, such that LiveIntent can provide service to you as described here and in LiveIntent’s privacy policies for the services which can be found at [https://www.liveintent.com/services-privacy-policy/](https://www.liveintent.com/services-privacy-policy/) 

#### How does LiveIntent ID work

The LiveIntent ID sub-module resolves the identity of audiences by connecting impression opportunities to a stable identifier (nonID). In order to provide resolution one or more first-party cookies are used to create a stable identifier. 

How does LiveIntent ID sub-module decide, which first-party cookies to use:
1. By default LiveIntent ID sub-module generates its own first-party identifier on the publisher’s domain. Publishers have the option to disable the cookie generation when configuring the LiveIntent ID sub-module.
2. A publisher can also define in the configuration which additional first-party cookies should be used. These can be used in a combination with the LiveIntent first-party cookie.

The LiveIntent ID sub-module sends the defined identifiers to the identity graph, which processes them and creates a stable identifier (nonID). The detailed description of the parameters being sent is described here: [Collector parameters](https://github.com/liveintent-berlin/live-connect/blob/HEAD/COLLECTOR_PARAMS.md)

For the identity resolution the LiveIntent ID sub-module makes a request to the LiveIntent’s identity resolution API, which returns a stable identifier and the audience segment(s) a user belongs to. The identifier and the segment are then exposed by the Prebid User ID Module to Prebid adapters to be sent out in a bid request. An SSP can then make the impression opportunity available to any buyers targeting the segment via a deal.

The first-party cookie generation and identity resolution functionality is provided by the LiveConnect JS library, included within the LiveIntent ID sub-module. LiveIntent has created a shared library that is open source, available at [https://www.npmjs.com/package/live-connect-js](https://www.npmjs.com/package/live-connect-js)

The LiveIntent ID sub-module follows the standard Prebid.js initialization based on the GDPR consumer opt-out choices. With regard to CCPA, the LiveConnect JS receives a us_privacy string from the Prebid US Privacy Consent Management Module and respects opt-outs.


#### LiveIntent ID configuration

{: .table .table-bordered .table-striped }
| Param under userSync.userIds[] | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| name | Required | String | The name of this module. | `'liveIntentId'` |
| params | Required | Object | Container of all module params. ||
| params.publisherId |Optional| String | The unique identifier for each publisher (for existing LiveIntent customers)|`'12432415'`|
| params.ajaxTimeout |Optional| Number |This configuration parameter defines the maximum duration of a call to the IdentityResolution endpoint. By default, 1000 milliseconds.|`1000`|
| params.partner | Optional| String |The name of the partner whose data will be returned in the response.|`'prebid'`|
| params.identifiersToResolve |Optional| Array[String] |Used to send additional identifiers in the request for LiveIntent to resolve against the LiveIntent ID.|`['my-id']`|
| params.url | Optional| String |Use this to change the default endpoint URL if you can call the LiveIntent Identity Exchange within your own domain.|`'https://idx.my-domain.com'`|
| params.liCollectConfig |Optional| Object |Container of all collector params.||
| params.liCollectConfig.fpiStorageStrategy |Optional| String |This parameter defines whether the first party identifiers that LiveConnect creates and updates are stored in a cookie jar, or in local storage. If nothing is set, default behaviour would be `cookie`. Allowed values: [`cookie`, `ls`, `none`]|`'cookie'`|
| params.liCollectConfig.fpiExpirationDays |Optional| Number |The expiration time of an identifier created and updated by LiveConnect.By default, 730 days.|`729`|
| params.liCollectConfig.collectorUrl |Optional| String |The parameter defines where the signal pixels are pointing to. The params and paths will be defined subsequently. If the parameter is not set, LiveConnect will by default emit the signal towards `https://rp.liadm.com`.|`'https://rp.liadm.com'`|
| params.liCollectConfig.appId |Optional| String |LiveIntent's media business entity application id.|`'a-0012'`|
| storage | Required | Object | This object defines where and for how long the results of the call to get a user ID will be stored. | |
| storage.type | Required | String | This parameter defines where the resolved user ID will be stored (either `'cookie'` or `'html5'` localstorage).| `'cookie'` |
| storage.name | Required | String | The name of the cookie or html5 localstorage where the resolved user ID will be stored. | `'pbjs_li_nonid'` |
| storage.expires | Recommended | Integer | How long (in days) the user ID information will be stored. The recommended value is `1` | `1` |

#### LiveIntent ID examples

1. To receive the LiveIntent ID, the setup looks like this.
```
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: "liveIntentId",
            params: {
              publisherId: "9896876"
            },
            storage: {             
            type: “cookie”,             
            name: “pbjs_li_nonid”,    //create a cookie with this name             
            expires: 1                // cookie is stored for 1 day          
            } 
        }]
    }
})
```

2. If you are passing additional identifiers that you want to resolve to the LiveIntent ID, add those under the `identifiersToResolve` array in the configuration parameters.
```
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: "liveIntentId",
            params: {
              publisherId: "9896876",
              identifiersToResolve: ["my-own-cookie"]
            },
            storage: {             
            type: “cookie”,             
            name: “pbjs_li_nonid”,    //create a cookie with this name             
            expires: 1                // cookie is stored for 1 day          
            } 
        }]
    }
})
```

3. If all the supported configuration params are passed, then the setup looks like this.
```
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: "liveIntentId",
            params: {
              publisherId: "9896876",
              identifiersToResolve: ["my-own-cookie"],
              url: "https://publisher.liveintent.com/idex",
              partner: "prebid",
              ajaxTimeout: 1000,
              liCollectConfig: {
                fpiStorageStrategy: "cookie",
                fpiExpirationDays: 730,
                collectorUrl: "https://rp.liadm.com",
                appId: "a-0012"
              }
            },
            storage: {             
            type: “cookie”,             
            name: “pbjs_li_nonid”,    //create a cookie with this name             
            expires: 1                // cookie is stored for 1 day          
            } 
        }]
    }
})
```

### netID

The [European netID Foundation (EnID)](https://developerzone.netid.de/index.html) aims to establish with the netID an independent European alternative in the digital market for Demand and Supply side. With the netID Single-Sign-On, the EnID established an open standard for consumer logins for services of Buyers and Brands, that also includes user-centric consent management capabilities that results in a standardized, EU-GDPR compliant, IAB TCF aware, cross-device enabled Advertising Identifier, which can be leveraged by publishers and advertisers (and vendors supporting them) to efficiently deliver targeted advertising through programmatic systems to already more than 38 million Europeans on mobile and desktop devices.

The EnID is a non-profit organization which is open to any contributing party on both, the demand and supply side to make identity work for consumers as well as the advertising ecosystem.

#### netID Examples

1) Publisher stores netID via his own logic

{% highlight javascript %}
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: "netId",
            value: {
               "netId":"fH5A3n2O8_CZZyPoJVD-eabc6ECb7jhxCicsds7qSg"
            }
        }]
    }
});
{% endhighlight %}

### Lotame Panorama ID

Lotame Panorama is a suite of data-enrichment solutions for digital advertising that empowers marketers, agencies, publishers and media companies to transform consumer personas into addressable audiences. At the heart of Lotame Panorama is the Panorama ID, a people-based identifier powered by deterministic and probabilistic data, available across the cookie-challenged web and all browsers.

The Lotame privacy policy is at [https://www.lotame.com/about-lotame/privacy/](https://www.lotame.com/about-lotame/privacy/).

Add it to your Prebid.js package with:

{: .alert.alert-info :}
gulp build --modules=lotamePanoramaId

#### Lotame Panorama ID Configuration

The Lotame Panorama ID module does not require any configuration parameters. It should work as-is provided that bidders use it in their adapters.

{: .alert.alert-info :}
NOTE: For optimal performance, the Lotame Panorama Id module should be called at every opportunity. It is best not to use `params.storage` with this module as the module has its own optimal caching mechanism.

#### Lotame Panorama ID Example

{% highlight javascript %}
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: "lotamePanoramaId",
        }]
    }
});
{% endhighlight %}

### Parrable ID

The Parrable ID is a Full Device Identifier that can be used to identify a device across different browsers and webviews on a single device including browsers that have third party cookie restrictions.

Add it to your Prebid.js package with:

{: .alert.alert-info :}
gulp build --modules=parrableIdSystem

#### Parrable ID Registration

Please contact Parrable to obtain a Parrable Partner Client ID and/or use the Parrable Partner Client ID provided by the vendor for each Parrable-aware bid adapter you will be using.  Note that if you are working with multiple Parrable-aware bid adapters you may use multiple Parrable Partner Client IDs.

The Parrable privacy policy as at [https://www.parrable.com/privacy-policy/](https://www.parrable.com/privacy-policy/).

#### Parrable ID Configuration

In addition to the parameters documented above in the Basic Configuration section the following Parrable specific configuration is required:

{: .table .table-bordered .table-striped }
| Param under userSync.userIds[] | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| params | Required | Object | Details for the Parrable ID. | |
| params.partner | Required | String | A list of one or more comma-separated Parrable Partner Client IDs for the Parrable-aware bid adapters you are using.  Please obtain Parrable Partner Client IDs from them and/or obtain your own. | `'30182847-e426-4ff9-b2b5-9ca1324ea09b'` |
| params.timezoneFilter | Optional | Object | Configures a timezone or timezone offset filter | |
| params.timezoneFilter.allowedZones | Optional | Array[String] | description | `[ 'America/Anchorage' ]` |
| params.timezoneFilter.allowedOffsets | Optional | Array[Number] | description | `[ -4 ]` |
| params.timezoneFilter.blockedZones | Optional | Array[String] | description | `[ 'America/New_York' ]` |
| params.timezoneFilter.blockedOffsets | Optional | Array[Number] | description | `[ -5 ]` |


{: .alert.alert-info :}
NOTE: The Parrable ID that is delivered to Prebid is encrypted by Parrable with a time-based key and updated frequently in the browser to enforce consumer privacy requirements and thus will be different on every page view, even for the same user.

The Parrable ID system manages a cookie with the name of `_parrable_id` containing the ID and optout states of the user.
This cookie is used also by standalone Parrable integrations outside of Prebid.
It is for this reason that the cookie name is not configurable for the Parrable ID system.

#### Timezone and Timezone Offset Filtering

The Parrable ID system enables a publisher to configure lists of **allowed** timezones (eg. `Europe/Dublin`) and/or timezone offsets (eg. `-4`) as well as a lists of **blocked** timezones and timezone offsets.

- With no configuration (`params.timezoneFilter` not set, or all of the lists are empty) all impressions are permitted.
- With only allow lists configured a browser must match either a timezone or timezone offset for it to not be filtered.
- With only block lists configured an impression will be filtered only if it is from a browser matching a blocked timezone or timezone offset.
- An impression from a browser that matches any allowed timezone or timezone offset, but does not match a blocked timezone or timezone offset will engage in the Parrable ID syncronization process.
- If a browser has a stored Parrable ID then it will not be filtered even if the browser is in a timezone or timezone offset that is blocked.

#### Parrable ID Examples

{% highlight javascript %}
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: `'parrableId'`,
            params: {
                partner: `'30182847-e426-4ff9-b2b5-9ca1324ea09b'`  // change to the Parrable Partner Client ID(s) you received from the Parrable Partners you are using
            }
        }],
        syncDelay: 1000
    }
});
{% endhighlight %}

### PubCommon ID

This module stores an unique user id in the first party domain and makes it accessible to all adapters. Similar to IDFA and AAID, this is a simple UUID that can be utilized to improve user matching, especially for iOS and MacOS browsers, and is compatible with ITP (Intelligent Tracking Prevention). It’s lightweight and self contained. Adapters that support Publisher Common ID will be able to pick up the user ID and return it for additional server-side cross device tracking.

There is no special registration or configuration for PubCommon ID. Each publisher's privacy policy should take
PubCommon ID into account.

Add it to your Prebid.js package with:

{: .alert.alert-info :}
gulp build --modules=pubCommonIdSystem

#### PubCommon ID Examples

1) Publisher supports PubCommonID and first party domain cookie storage

{% highlight javascript %}
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: "pubCommonId",
            storage: {
                type: "cookie",
                name: "_pubcid",         // create a cookie with this name
                expires: 365             // expires in 1 years
            }
        }]
    }
});
{% endhighlight %}

2) Publisher supports both UnifiedID and PubCommonID and first party domain cookie storage

{% highlight javascript %}
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: "unifiedId",
            params: {
                partner: "myTtdPid"
            },
            storage: {
                type: "cookie",
                name: "pbjs-unifiedid",       // create a cookie with this name
                expires: 60
            }
        },{
            name: "pubCommonId",
            storage: {
                type: "cookie",
                name: "_pubcid",     // create a cookie with this name
                expires: 180
            }
        }],
        syncDelay: 5000       // 5 seconds after the first bidRequest()
    }
});
{% endhighlight %}

### Quantcast ID

Quantcast ID enables publishers that use Quantcast Measure tag to uniquely identify
their clients within Quantcast's extensive publisher network without relying on third party
cookies. The Quantcast User ID submodule makes the existing Quantcast first party
cookie available in the bid request. The first party cookie allows Quantcast to correlate
the bid request with Quantcast's Measure dataset.

Currently, Quantcast ID only works with the presence of Quantcast Measure tag. More information
about Measure can be found in https://www.quantcast.com/measure.

The Quantcast privacy policy is at https://www.quantcast.com/privacy/.

Add it to your Prebid.js package with:

{: .alert.alert-info :}
gulp build --modules=userId,quantcastIdSystem

#### Quantcast ID Configuration

Quantcast ID module does not require any configuration parameters at this time.

#### Quantcast ID Example

{% highlight javascript %}
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: "quantcastId",
        }]
    }
});
{% endhighlight %}


### Shared ID User ID Submodule

Shared ID User ID Module generates a UUID that can be utilized to improve user matching.This module enables timely synchronization which handles sharedId.org optout. This module does not require any registration.  

#### Building Prebid with Shared Id Support
Your Prebid build must include the modules for both **userId** and **sharedId** submodule. 
Add it to your Prebid.js package with:

ex: $ gulp build --modules=userId,sharedIdSystem

#### Prebid Params

Individual params may be set for the Shared ID User ID Submodule. 
```
pbjs.setConfig({
    usersync: {
        userIds: [{
            name: 'sharedId',
            params: {
                      syncTime: 60 // in seconds, default is 24 hours
             },
            storage: {
                name: 'sharedid',
                type: 'cookie',
                expires: 28
            },
        }]
    }
});
```

#### SharedId Configuration

{: .table .table-bordered .table-striped }
| Params under usersync.userIds[]| Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| name | Required | String | ID value for the Shared ID module - `"sharedId"` | `"sharedId"` |
| params | Optional | Object | Details for sharedId syncing. | |
| params.syncTime | Optional | Object | Configuration to define the frequency(in seconds) of id synchronization. By default id is synchronized every 24 hours | 60 |
| storage | Required | Object | The publisher must specify the local storage in which to store the results of the call to get the user ID. This can be either cookie or HTML5 storage. | |
| storage.type | Required | String | This is where the results of the user ID will be stored. The recommended method is `localStorage` by specifying `html5`. | `"html5"` |
| storage.name | Required | String | The name of the cookie or html5 local storage where the user ID will be stored. | `"sharedid"` |
| storage.expires | Optional | Integer | How long (in days) the user ID information will be stored. | `28` |

### Unified ID

The Unified ID solution is provided by adsrvr.org and the Trade Desk.

Add it to your Prebid.js package with:

{: .alert.alert-info :}
gulp build --modules=unifiedIdSystem

#### Unified ID Registration

You can set up Unified ID in one of these ways:

- Register with The Trade Desk from their [Unified ID page](https://www.thetradedesk.com/industry-initiatives/unified-id-solution).
- Utilize a [managed services](https://prebid.org/product-suite/managed-services/) company who can do this for you.

The Unified ID privacy is covered under the [TradeDesk Services Privacy Policy](https://www.thetradedesk.com/general/privacy).

#### Unified ID Configuration

{: .table .table-bordered .table-striped }
| Param under userSync.userIds[] | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| name | Required | String | `"unifiedId"` | `"unifiedId"` |
| params | Required for UnifiedId | Object | Details for UnifiedId initialization. | |
| params.partner | Either this or url required for UnifiedId | String | This is the partner ID value obtained from registering with The Trade Desk or working with a Prebid.js managed services provider. | `"myTtdPid"` |
| params.url | Required for UnifiedId if not using TradeDesk | String | If specified for UnifiedId, overrides the default Trade Desk URL. | "https://unifiedid.org/somepath?args" |
| value | Optional | Object | Used only if the page has a separate mechanism for storing the Unified ID. The value is an object containing the values to be sent to the adapters. In this scenario, no URL is called and nothing is added to local storage | `{"tdid": "D6885E90-2A7A-4E0F-87CB-7734ED1B99A3"}` |

#### Unified ID Examples

1) Publisher has a partner ID with The Trade Desk, and is using the default endpoint for Unified ID.

{: .alert.alert-warning :}
Bug: The default URL did not support HTTPS in Prebid.js 2.10-2.14. So instead of using
the 'partner' parameter, it's best to supply the Trade Desk URL as shown in this example.

{% highlight javascript %}
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: "unifiedId",
            params: {
                url: "//match.adsrvr.org/track/rid?ttd_pid=MyTtidPid&fmt=json"
            },
            storage: {
                type: "cookie",
                name: "pbjs-unifiedid",       // create a cookie with this name
                expires: 60                   // cookie can last for 60 days
            }
        }],
        syncDelay: 3000              // 3 seconds after the first auction
    }
});
{% endhighlight %}

2) Publisher supports UnifiedID with a vendor other than Trade Desk and HTML5 local storage.

{% highlight javascript %}
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: "unifiedId",
            params: {
                url: "URL_TO_UNIFIED_ID_SERVER"
            },
            storage: {
                type: "html5",
                name: "pbjs-unifiedid",    // set localstorage with this name
                expires: 60
            }
        }],
        syncDelay: 3000
    }
});
{% endhighlight %}

3) Publisher has integrated with UnifiedID on their own and wants to pass the UnifiedID directly through to Prebid.js.

{% highlight javascript %}
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: "unifiedId",
            value: {"tdid": "D6885E90-2A7A-4E0F-87CB-7734ED1B99A3"}
        }]
    }
});
{% endhighlight %}

### Audigent HaloId

Audigent is a next-generation data management platform and a first-of-a-kind "data agency" containing some of the most exclusive content-consuming audiences across desktop, mobile and social platforms. Our HaloId module allows for user id resolution and Audigent user data segmentation to be retrieved for users across the web.  For assistance setting up your module please contact us at [prebid@audigent.com](prebid@audigent.com).

#### HaloId Configuration
Add the Halo ID system to your Prebid.js package with:

{: .alert.alert-info :}
gulp build --modules=userId,haloIdSystem

Add HaloId to the userSync configuration.

```
pbjs.setConfig({
    userSync: {
        userIds: [{
            name: 'haloId',
            storage: {
                name: 'haloId',
                type: 'html5'
            }
        }]
    }
});
```

The `request.userId.haloId` will contain the Audigent HaloId and associated segments:
```
{
  "haloId": "user-halo-id",
  "auSeg": ["segment1", "segment2"]
}
```

## Adapters Supporting the User ID Sub-Modules

{% assign bidder_pages = site.pages | where: "layout", "bidder" %}

<table class="pbTable">
<tr class="pbTr"><th class="pbTh">Bidder</th><th class="pbTh">IDs Supported</th></tr>
{% for item in bidder_pages %}
{% if item.userIds != nil %}
<tr class="pbTr"><td class="pbTd">{{item.biddercode}}</td><td class="pbTd">{{item.userIds}}</td></tr>
{% endif %}
{% endfor %}
</table>

## Bidder Adapter Implementation

### Prebid.js Adapters

Bidders that want to support the User ID module in Prebid.js, need to update their bidder adapter to read the indicated bidRequest attributes and pass them to their endpoint.

{: .table .table-bordered .table-striped }
| ID System Name | ID System Host | Prebid.js Attr | Example Value |
| --- | --- | --- | --- | --- | --- |
| BritePool ID | BritePool | bidRequest.userId.britepoolid | `"1111"` |
| CriteoID | Criteo | bidRequest.userId.criteoId | `"1111"` |
| ID+ | Zeotap | bidRequest.userId.IDP | `"1111"` |
| ID5 ID | ID5 | bidRequest.userId.id5id | `"1111"` |
| IdentityLink | Trade Desk | bidRequest.userId.idl_env | `"1111"` |
| LiveIntent ID | Live Intent | bidRequest.userId.lipb.lipbid | `"1111"` |
| IntentIQ ID | IntentIQ | bidRequest.userId.intentiqid | `"1111"` |
| Lotame Panorama ID | Lotame | bidRequest.userId.lotamePanoramaId | `"e4b96a3d9a8e8761cef5656fb05f16d53938069f1684df4b2257e276e8b89a0e"` |
| Parrable ID | Parrable | bidRequest.userId.parrableId | `{"eid":"01.1594654046.cd0972d861e98ff3723a368a6efa69287a0df3f1cac9142afc2e7aed1caa8dd1b7fc0590b3baf67525f53e1228024c2805b6041206c7a23e34bb823b0659547d7d1d0dac2a11938e867f"}` |
| PubCommon ID | n/a | bidRequest.userId.pubcid | `"1111"` |
| Unified ID | Trade Desk | bidRequest.userId.tdid | `"1111"` |
| netID | netID | bidRequest.userId.netId | `"fH5A3n2O8_CZZyPoJVD-eabc6ECb7jhxCicsds7qSg"` |
| Shared ID | SharedId | bidRequest.userId.sharedid | `{"id":"01EAJWWNEPN3CYMM5N8M5VXY22","third":"01EAJWWNEPN3CYMM5N8M5VXY22"}` |

For example, the adapter code might do something like:

{% highlight javascript %}
   if (bidRequest.userId && bidRequest.userId.pubcid) {
    url+="&pubcid="+bidRequest.userId.pubcid;
   }
{% endhighlight %}

### Prebid Server Adapters

Bidders that want to support the User ID module in Prebid Server, need to update their server-side bid adapter to read the desired OpenRTB attributes noted in the example below and send them to their endpoint.

{% highlight bash %}
{
    "user": {
        "ext": {
            "eids": [{
                "source": "adserver.org",  // Unified ID
                "uids": [{
                    "id": "111111111111",
                    "ext": {
                        "rtiPartner": "TDID"
                    }
                }]
            },{
                "source": "pubcid.org",
                "uids": [{
                    "id":"11111111"
                }]
            },
            {
                "source": "id5-sync.com",
                "uids": [{
                    "id": "ID5-12345"
                }]
            },
            {
                source: "parrable.com",
                uids: [{
                    id: "01.1594654046.cd0972d861e98ff3723a368a6efa69287a0df3f1cac9142afc2e7aed1caa8dd1b7fc0590b3baf67525f53e1228024c2805b6041206c7a23e34bb823b0659547d7d1d0dac2a11938e867f"
                }]
            },{
                "source": "identityLink",
                "uids": [{
                    "id": "11111111"
                }]
            },{
                "source": "criteo",
                "uids": [{
                    "id": "11111111"
                }]
            },{
                "source": "britepool.com",
                "uids": [{
                    "id": "11111111"
                }]
            },{
                "source": "liveintent.com",
                "uids": [{
                    "id": "11111111"
                }]
            },{
                "source": "crwdcntrl.net",
                "uids": [{
                    "id": "e4b96a3d9a8e8761cef5656fb05f16d53938069f1684df4b2257e276e8b89a0e"
                }]
            },{
                "source": "netid.de",
                "uids": [{
                    "id": "11111111"
                }]
            },{
               "source": "sharedid.org",  // Shared ID
                "uids": [{
                    "id": "01EAJWWNEPN3CYMM5N8M5VXY22",
                    "ext": {
                        "third": "01EAJWWNEPN3CYMM5N8M5VXY22"
                    }
                }]
            }]
        }
    }
}
{% endhighlight %}

### ID Providers

If you're an ID provider that wants to get on this page:

- Fork Prebid.js and write a sub-module similar to one of the *IdSystem modules already in the [modules](https://github.com/prebid/Prebid.js/tree/master/modules) folder.
- Add your *IdSystem name into the modules/.submodules.json file
- Follow all the guidelines in the [contribution page](https://github.com/prebid/Prebid.js/blob/master/CONTRIBUTING.md).
- Submit a Pull Request against the [Prebid.js repository](https://github.com/prebid/Prebid.js).
- Fork the prebid.org [documentation repository](https://github.com/prebid/prebid.github.io), modify the /dev-docs/modules/userId.md, and submit a documentation Pull Request as well.

<a name="getUserIds"></a>

### Exporting User IDs

If you need to export the user IDs stored by Prebid User ID module, the `getUserIds()` function will return an object formatted the same as bidRequest.userId.

```
pbjs.getUserIds() // returns object like bidRequest.userId. e.g. {"pubcid":"1111", "tdid":"2222"}
```

You can use `getUserIdsAsEids()` to get the user IDs stored by Prebid User ID module in ORTB Eids format. Refer [eids.md](https://github.com/prebid/Prebid.js/blob/master/modules/userId/eids.md) for output format.
```
pbjs.getUserIdsAsEids() // returns userIds in ORTB Eids format. e.g. 
[
  {
      source: 'pubcid.org',
      uids: [{
          id: 'some-random-id-value',
          atype: 1
      }]
  },

  {
      source: 'adserver.org',
      uids: [{
          id: 'some-random-id-value',
          atype: 1,
          ext: {
              rtiPartner: 'TDID'
          }
      }]
  }
]
``` 

## Passing UserIds to Google Ad Manager for targeting

User IDs from Prebid User ID module can be passed to GAM for targeting in Google Ad Manager or to pass ahead in Google Exchange Bidding using ```userIdTargeting``` module. More details can be found [here](https://github.com/prebid/Prebid.js/blob/master/modules/userIdTargeting.md). In short, you just need to add the optional userIdTargeting sub-module into your `gulp build` command and the additional `userIdTargeting` config becomes available.

## Further Reading

* [Prebid.js Usersync](/dev-docs/publisher-api-reference.html#setConfig-Configure-User-Syncing)
* [GDPR ConsentManagement Module](/dev-docs/modules/consentManagement.html)
