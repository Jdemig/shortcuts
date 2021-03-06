# Google Analytics Notes

See https://developers.google.com/analytics/devguides/collection/analyticsjs/
and https://support.google.com/analytics for details.


## JavaScript Tracking Snippet

There are two different tracking snippets available.  These need to be added
to each page you want to track.  It is recommended that it be added to the
&lt;head> tag before any other &lt;script> or &lt;style> tags.

```html
<script>
(function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
(i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
})(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

ga('create', 'UA-XXXXX-Y', 'auto');
ga('send', 'pageview');
</script>
```

Use the following snippet if your visitors primarily use modern browsers to
access your site.

```html
<script>
window.ga=window.ga||function(){(ga.q=ga.q||[]).push(arguments)};ga.l=+new Date;

ga('create', 'UA-XXXXX-Y', 'auto');
ga('send', 'pageview');
</script>
<script async src="https://www.google-analytics.com/analytics.js"></script>
```

These snippets end up creating a global function named `ga` which is known as
the command queue.  See the [command queue reference][ga-cmdq-ref] for details.

**Ready Callback**

The ready callback is called when the `analytics.js` library is fully loaded,
and all previous commands added to the queue have been executed.

```js
// @param {Tracker} [tracker] - The default tracker if one was created.
ga(function (tracker) {
  // Do something with the tracker or call methods on the ga object.
});
```

Note: `ga` object methods are only available when `analytics.js` has fully
loaded, so you should only reference them inside a ready callback.


## Trackers

Trackers collect and store data and then send that data to Google Analytics.
The data comes in field (i.e., name/value pairs) form.  See the
[field reference][ga-fld-ref] for applicable fields.

**Creating**

```js
// Syntax:
ga('create', [trackingId], [cookieDomain], [name], [fieldsObject]);

// Example:
ga('create', 'UA-XXXXX-Y', 'example.com', 'MyTracker', {
  allowAnchor: true, // The default
  allowLinker: false, // The default
  alwaysSendReferrer: false, // The default
  cookieDomain: 'example.com', // Synonymous with 3rd param
  cookieExpires: 63072000, // The default of 2 years
  cookieName: '_ga', // The default
  legacyHistoryImport: true, // The default
  sampleRate: 95, // 95% of users. 100% is the default
  siteSpeedSampleRate: 10, // 10% of users.  1% is the default
});
```

**Referencing**

Because the creation of trackers is asynchronous, we need a way to get a
reference to the tracker(s).  The trackers will be available when the ready
callback has been called.

```js
// @param {Tracker} [tracker] - The default tracker if one was created.
ga(function (tracker) {

  // If multiple trackers were created, the `ga` object provides two methods
  // to get access to the trackers.
  
  // Get tracker by name:
  let myTracker = ga.getByName('MyTracker');
  
  // Get all trackers:
  let trackers = ga.getAll();
});
```

**Getting Fields**

Once you have a reference to a tracker, you can use the `get` method to get field values.

```js
ga(function (tracker) {
  let value = tracker.get('fieldname');
});
```

**Setting/Updating Fields**

Tracker objects can be updated using the `set` method.  A tracker's `set`
method can be called on a tracker object itself or by adding a `set` command
to the `ga()` command queue.

The following examples use the command queue:

```js
ga('set', 'page', '/about');
ga('set', 'title', 'About Us');
// or
ga('set', {
  page: '/about',
  title: 'About Us'
});

// with a named tracker:
ga('myTracker.set', 'page', '/about');
```

The following examples use a tracker object:

```js
ga(function (tracker) {
  tracker.set('page', '/about');
  tracker.set('title', 'About Us');
  // or
  tracker.set({
    page: '/about',
    title: 'About Us'
  });
});
```


## Hits

Tracking of user interaction is done by sending "hits" to Google Analytics.
Here are the available types of hits: `event`, `exception`, `item`,
`pageview`, `screenview`, `social`, `timing`, and `transaction`.

Hits are made via the `send` method.

The following examples use the command queue:

```js
// Syntax:
ga('[trackerName.]send', [hitType], [...fields], [fieldsObject]);

// Example:
ga('send', 'pageview', {
  page: '/about',
  title: 'About Us'
});
// Or
ga('send', {
  hitType: 'pageview',
  page: '/about',
  title: 'About Us'
});
```

The following examples use a tracker object:

```js
ga(function (tracker) {
  // Syntax:
  tracker.send([hitType], [...fields], [fieldsObject]);
  
  // Example:
  tracker.send('pageview', {
    page: '/about',
    title: 'About Us',
  });
  // or
  tracker.send('pageview', '/about', { title: 'About Us' });
});
```

If any of the fields passed with the `send` command are already set on the
tracker object, the values passed in the command will be used rather than the
values stored on the tracker.

**Hit Callback**

To be notified when a hit is done sending, you set the `hitCallback` field.
`hitCallback` is a function that gets called with no arguments as soon as the
hit has been successfully sent.

Whenever you put critical site functionality inside the `hitCallback`
function, you should always use a timeout function to handle cases where the
`analytics.js` library fails to load.

```js
var form = document.getElementById('signup-form');

form.addEventListener('submit', function (evt) {
  var formSubmitted = false;

  function submitForm() {
    if (!formSubmitted) {
      formSubmitted = true;
      form.submit();
    }
  }

  evt.preventDefault();

  setTimeout(submitForm, 1000);

  ga('send', 'event', 'Signup Form', 'submit', {
    hitCallback: submitForm
  });
});
```

**Transport Mechanism**

By default, `analytics.js` picks the HTTP method and transport mechanism with
which to optimally send hits.  The three options are `'image'` (using an
`Image` object), `'xhr'` (using an `XMLHttpRequest` object), or 'beacon' using
the new `navigator.sendBeacon` method.

The former two methods have a problem where hits are often not sent if the
page is being unloaded.  The `navigator.sendBeacon` method, by contrast, is a
new HTML feature created to solve this problem.

If your user's browser supports the `navigator.sendBeacon`, you can specify
`'beacon'` as the transport mechanism and not have to worry about setting a
hit callback.

The following code sets the transport mechanism to 'beacon' in browsers that support it.

```js
// Updates the default tracker to use `navigator.sendBeacon` if available.
ga('set', 'transport', 'beacon');
```


## Plugins

Plugins are scripts that enhance the functionality of `analytics.js` to aid in
measuring user interaction.

**Requiring**

The `require` command takes the name of a plugin and registers it for use with
the command queue.

```js
// Syntax:
ga('[trackerName.]require', pluginName, [pluginOptions]);

// Example:
ga('myTracker.require', 'displayfeatures', {
  cookieName: 'display_features_cookie'
});
```

**Loading**

The `require` command initializes the plugin methods for use with the command
queue, but it does not load the plugin script itself.  If you're using a
third-party plugin, or writing a plugin yourself, you'll need to manually add
the plugin code to the page.

The recommended method for adding plugin code to the page is via a `<script>`
tag with the `async` attribute set to ensure it doesn't block the loading of
other features on your site.

```html
<script>
// Tracking snippet ...
// ...
ga('require', 'myplugin');
</script>

<!-- ... -->

<!-- Note: plugin scripts must be included after the tracking snippet. -->
<script async src="/path/to/my-plugin.js"></script>
```

Note: Official Google Analytics plugins for `analytics.js` get loaded
automatically when they're required; you do not need to add script tags for
them.  The complete list of official `analytics.js` plugins can be found under
the Offical Plugins section in the left-side navigation of the
[guide][ga-guide].

Because both the `analytics.js` library and any plugins are loaded
asynchronously, it can be a challenge to know when plugins are fully loaded
and ready to be used.

The `analytics.js` library solves this problem by halting the execution of the
command queue when it encounters a `require` command for a plugin that isn't
yet loaded.  Once the plugin is loaded, queue execution continues as normal.

**Calling Plugin Methods**

After requiring a plugin, it's methods become available for use with the
command queue.  Here is the command signature for calling plugin methods:

```js
// Syntax:
ga('[trackerName.][pluginName:]methodName', ...args);
```

For example, the Enhanced Ecommerce plugin's `addProduct` method can be called like this:

```js
ga('ec:addProduct', {
  id: 'P12345',
  quantity: 1
});
```

Or on a named tracker by adding the tracker name to the command string:

```js
ga('myTracker.ec:addProduct', {
  id: 'P12345',
  quantity: 1
});
```


## Debugging

See https://developers.google.com/analytics/devguides/collection/analyticsjs/debugging
for details.

**The Debug Version of the analytics.js Library**

Google Analytics provides a debug version of the `analytics.js` library that
logs detailed messages to the Javascript console as its running.  These
messages include successfully executed commands as well as warnings and error
messages that can tell you when your tracking code is set up incorrectly.  It
also provides a breakdown of each hit sent to Google Analytics, so you can see
exactly what data is being tracked.

You can enable the debug version of `analytics.js` by changing the URL in the
JavaScript tracking snippet from
`https://www.google-analytics.com/analytics.js` to
<code><span>https</span>://www.google-analytics.com/analytics_<strong>debug</strong>.js</code>.

**Testing Your Implementation without Sending Hits**

You can disable sending of hits by disabling the `sendHitTask` task.  Here's an example:

```js
// Tracking snippet ...
...
ga('create', 'UA-XXXXX-Y', 'auto');

if (location.hostname == 'localhost') {
  ga('set', 'sendHitTask', null);
}

ga('send', 'pageview');
```

**Trace Debugging**

Enabling trace debugging will output more verbose information to the console.

To enable trace debugging, load the debug version of `analytics.js` as
described above and add the following line of JavaScript to the tracking
snippet prior to any calls to the command queue.

```js
window.ga_debug = { trace: true };
```

**The Google Analytics Debugger Chrome Extension**

Google Analytics also provides a Chrome extension that can enable the debug
version of `analytics.js` without requiring you to change your tracking code.
This allows you to debug your own sites and also see how other sites have
implemented Google Analytics tracking with `analytics.js`.

**Google Tag Assistant**

Google Tag Assistant is a Chrome Extension that helps you validate the
tracking code on your website and troubleshoot common problems.  It's an ideal
tool for debugging and testing your `analytics.js` implementations locally and
ensuring everything is correct before deploying your code to production.

Tag Assistant works by letting you record a typical user flow.  It keeps track
of all the hits you send, checks them for any problems, and gives you a full
report of the interactions.  If it detects any issues or potential
improvements, it will let you know.



## Campaign

**Fields**

There are many campaign related fields that may be set, which include the ID,
name, source, medium, keyword, and content.  See [field reference][ga-fld-cn]
for details.


## Sessions

**Session Control**

Sessions can be controlled with the `sessionControl` field.  See
[reference][ga-fld-sc] for details.

```js
// Starts a new session.
ga('send', 'pageview', { sessionControl: 'start' });
```


## Content Grouping

A content grouping is a set of content groups.  A content group is a
collection of content.  This content could be pages in a certain section of
your website or screens from a certain part of your app.

**Google's Description**

Content Grouping lets you group content into a logical structure that reflects
how you think about your site or app, and then view and compare aggregated
metrics by group name in addition to being able to drill down to the
individual URL, page title, or screen name.  For example, you can see the
aggregated number of pageviews for all pages in a group like Men/Shirts, and
then drill in to see each URL or page title.

See https://support.google.com/analytics/answer/2853423?hl=en for more
details.

**Creating Content Groups**

To create a content group, you first need to create a content grouping.  Then
you can create content groups for each of your content groupings.  In each of
the content groups will be items, each representing different pages, screens,
or sections of your website/web app.

In Google Analytics (GA), you do the following:

* From GA, go to the Admin section.
* Choose an Account, Property, and View.
* In the View, select Content Grouping.
* Here you can create a new content grouping or edit an existing grouping.
* From the Content Grouping Settings section you can set groups by three means:
  by tracking code, using extraction, and using rule definition.
  + By Tracking Code requires you to add a snippet of JS to each page to define
    the group.
  + Using Extraction lets you define groups by matching on reg exp capture groups
    on the page URI, page title, or screen name.
  + Using Rule Definitions lets you define groups by defining rule sets that match
    on the page URI, page title, or screen name on various rules.  The rules
    include matching exactly, contains, starts with, ends with, regex, is one of,
    and the negation of the previous six for a total of twelve.  Each rule set can
    contain multiple rules which can be OR'd or AND'd together.  (It appears you
    can always define a rule set that is equivalent to an Extraction.  I have not
    yet confirmed this.)

> Currently you can create up to five content groupings.  Within each of those,
> there is no limit to the number of content groups you can define.

In Google Tag Manager (GTM), you do the following:

* See this [YouTube video](https://www.youtube.com/watch?v=OSKEEy2xb9E&time_continue=474).


## Misc

GA tracks the document title, so be sure to pick the title you want to see in
the GA web interface.  Also, it appears it may be limited to 1500 bytes.


## Useful Links

[Glossary][ga-glossary]



[ga-glossary]: https://support.google.com/analytics/topic/6083659
[ga-guide]: https://developers.google.com/analytics/devguides/collection/analyticsjs/
[ga-cmdq-ref]: https://developers.google.com/analytics/devguides/collection/analyticsjs/command-queue-reference
[ga-fld-ref]: https://developers.google.com/analytics/devguides/collection/analyticsjs/field-reference
[ga-fld-cn]: https://developers.google.com/analytics/devguides/collection/analyticsjs/field-reference#campaignName
[ga-fld-sc]: https://developers.google.com/analytics/devguides/collection/analyticsjs/field-reference#sessionControl
