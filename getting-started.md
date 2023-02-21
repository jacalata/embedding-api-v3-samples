This page used to be hosted in a standalone repo as the [Embedding v2->v3 Migration Guide](https://github.com/tableau/embedding-api-v3-guide)



## Introduction

With Tableau’s Embedding API v3, we have redesigned the experience for embedding to make your life easier, to modernize the experience, and to enable new functionality.
The most important new feature is the embedding web component, which allows you to configure embedded content entirely in HTML. Web components are a standard available in all modern browsers, that allow you to add custom, and 3rd party, functionality to your web page using html syntax as easily as adding an image, link, or div. Using the web component technology, we have created a tableau-viz component that you can use to add visualizations to your web pages.

## How to use this guide

This guide has a few goals:    
1) To help you understand key concepts of using the Embedding API v3     
2) to help you get started as a first-time developer on Tableau Embedding    
3) to help you migrate existing content to v3.

If you are looking for information on the previous API (v2), you can [read its documentation here](https://help.tableau.com/current/api/js_api/en-us/JavaScriptAPI/js_api.htm).

If you have comments or feedback, please submit an [Issue](https://github.com/tableau/embedding-api-v3-samples/issues) on this Github repository.

## Roadmap

For information about the next release of the Tableau Embedding API see the [Tableau Embedding API v3 Developer Preview
](https://embedding.tableauusercontent.com/preview/getting-started-v3.html). 



# Getting Started

In previous versions of Tableau Server, there existed a JavaScript API v1. You may still be using it if your embedding html looks like this: (note "v1" in the src attribute)

```html
<script type='text/javascript' src='https://myserver/javascripts/api/viz_v1.js'></script>
<div class='tableauPlaceholder' style='width: 1500px; height: 827px;'>
    <object class='tableauViz' width='1500' height='827' style='display:none;'>
    <param name='host_url' value='https%3A%2F%2Fmyserver' /> 
    <param name='embed_code_version' value='3' />
    <param name='site_root' value='&#47;t&#47;Superstore' />
    <param name='name' value='MyView' />
    <param name='tabs' value='no' /><param name='toolbar' value='yes' />
    <param name='showAppBanner' value='false' /></object>
</div>

```

After that was v2. In v2 you needed to use both html and javascript to initialize your embedded viz:

 
HTML (v2 - will also work in v3)

```html
<div id='tableauViz'></div>
```

JavaScript 

```javascript
let placeholderDiv = document.getElementById("tableauViz");
let src = "http://my-server/views/my-workbook/my-view";
let options = {}
let viz = new tableau.Viz(placeholderDiv, url, options);
```

To migrate from the JavaScript API v1 or v2 to the new JavaScript API v3, you can start by copying the snippet of embed code produced by the Share button on the viz, which will now give you something like:

```html
<script src = "https://myserver/javascripts/api/tableau.embedding.3.latest.js"></script>
<tableau-viz id="tableauViz" 
    src="http://my-server/views/my-workbook/my-view" 
    device="phone" toolbar="bottom" hide-tabs>
</tableau-viz>
```


You will notice that the setup of v3 is as simple as possible - a single element to add into your html, and simple attributes to customize it. Another key benefit to v3 is how easy it is to go from this simple copy-paste embed scenario to one where you create interactions with the viz via API.

Before we do that, let’s take a closer look at the key parts of initializing an embedded viz.

## Access the API

Accessing the API happens similarly in all versions: You simply include the correct .js file, this time including the tableau-3.js (or a variation of it) from your Server. As before, you can reference the .js from your on-premise Server, from Tableau Cloud, or from Tableau Public.

## Initialize the API

In the Embedding API v3, the embedded viz can be described entirely in html, and thanks to the new tableau-viz web component the initialization will be run automatically on page load. 

HTML:

```html
<tableau-viz id="tableauViz" 
      src='http://my-server/views/my-workbook/my-view'>
</tableau-viz>
```

### Configuration:

To specify options on how to initialize the Viz in the JSAPI v2, you would add those to the options object that is included in the javascript constructor’s arguments. In the Embedding API v3, you can simply add those as properties of the viz component:


```html
<tableau-viz id="tableauViz" 
    src="http://my-server/views/my-workbook/my-view" 
    device="phone" toolbar="bottom" hide-tabs>
</tableau-viz>
```

Note: for properties that accept a variety of values, such as src, device, and toolbar the syntax is `<property>=“<value>”`. For properties which are boolean, you either include the flag, to effectively set it to true (as shown with hideTabs), or omit it, to effectively set it to false.

Here is the list of properties you can add to your viz object:

|HTML Property	|JS Property	|Accepted Values	|Description	|
|---	|---	|---	|---	|
|hide-tabs	|Viz.hideTabs	|boolean	|Indicates whether tabs are hidden or shown.	|
|toolbar	|Viz.toolbar	|"top", "bottom", "hidden"	|Indicates the position of the toolbar or if it should be hidden	|
|height	|Viz.height	|\<string\> (e.g. "800px" or "800")	|Can be any valid CSS size specifier. If not specified, defaults to the published height of the view.	|
|width	|Viz.width	|\<string\> (e.g. "800px" or "800")	|Can be any valid CSS size specifier. If not specified, defaults to the published height of the view.	|
|device	|Viz.device	|\<string\>	|Specifies a device layout for a dashboard, if it exists. Values can be `default`, `desktop`, `tablet`, or `phone`. If not specified, defaults to loading a layout based on the smallest dimension of the hosting `iframe` element.	|
|instance-id-to-clone	|Viz.instanceIdToClone	|\<string\>	|Specifies the ID of an existing instance to make a copy (clone) of. This is useful if the user wants to continue analysis of an existing visualization without losing the state of the original. If the ID does not refer to an existing visualization, the cloned version is derived from the original visualization.	|
|disable-url-actions	|Viz.disableUrlActions	|boolean	|Indicates whether to suppress the execution of URL actions. This option does not prevent the URL action event from being raised. You can use this option to change what happens when a URL action occurs. If set to `true`and you create an event listener for the `URL_ACTION` event, you can use an event listener handler to customize the actions. For example, you can direct the URL to appear in an `iframe`on your web page. See [URL Action Example](https://help.tableau.com/current/api/js_api/en-us/JavaScriptAPI/js_api_ref.htm#urlaction).	|

*When adding properties to the tableau-viz component, use the HTML Property name. When [configuring via JavaScript (as explained below)](#alternative-approach-initialization-via-javascript), use the JS Property.*

### Filtering during initialization

In JSAPI v2 you can specify filtering to occur, by specifying field-value in the url (html) or options object (javascript). In the Embedding API v3, you can accomplish the same thing by add <viz-filter> elements as children to your <tableau-viz> objects. You can also set an initial parameter value.

```html
<tableau-viz id="tableauViz" 
    src="http://my-server/views/my-workbook/my-view" 
    device="phone" toolbar="bottom" hide-tabs>
    <viz-filter field="country" value="USA" />
    <viz-filter field="state" value="California,Texas,Washington"></viz-filter>
    <viz-parameter name="currency" value="dollars" />
    <viz-range-filter field="profit" min="0" max="10000"></viz-range-filter>
    <viz-relative-date-filter field="Date" anchor-date="March 17, 2001"
        period-type = "Year" range-type = "LastN" range-n = "2"></viz-relative-date-filter>
</tableau-viz>
```

*For viz-range-filter and viz-relative-date-filter, use the properties from* [*RangeFilterOptions*](https://help.tableau.com/current/api/js_api/en-us/JavaScriptAPI/js_api_ref.htm#rangefilteroptions_record) *and [RelativeDateFilterOptions](https://help.tableau.com/current/api/js_api/en-us/JavaScriptAPI/js_api_ref.htm#relativedatefilteroptions_record), but convert them to dash-casing.*

One important difference is that in the JSAPI v2 the filters that were loaded on initialization were added to the url of the viz. This limited the amount of filters that could be added, because url’s have a maximum length of 2048 characters. In the Embedding API v3, the filtering occurs separately from the viz url, so this constraint is gone. The initialization with filters still occurs on initial load.


### Event listeners

You can also add event listeners to the <tableau-viz> object. These let you run your own code, or call code from a library, when a specific event happens on the page.

```html
<tableau-viz id="tableauViz" 
    src="http://my-server/views/my-workbook/my-view"
    onMarksSelected="myFunctionToHandleMarksSelected">
</tableau-viz>
```

    (Where are the events listed?)
(In the above example, myFunctionToHandleMarksSelected() is a function which is specified in a JavaScript src file somewhere)

## Alternative Approach: Initialization via JavaScript

In v2, you always had to configure the viz in js code. In v3, you don't have to do that, but there are still scenarios where you may prefer to configure and initialize the viz via JavaScript instead of the above HTML <tableau-viz> component approach. Here is an example that demonstrates all of the above functionality using JavaScript and html in v3:

HTML:

```html
<div id='tableauViz'></div>
```


JavaScript:

```javascript
import {TableauViz} from '../tableau.embedding.3.0.0-alpha.23.js'

let viz = new TableauViz();

viz.src = 'http://my-server/views/my-workbook/my-view';
viz.toolbar = 'hidden';
viz.addFilter('Region', 'Central');
viz.addEventListener('marksSelected', handleMarksSelection);

document.getElementById('tableauViz').appendChild(viz); 
```

Important notes about the above approach:

* `new TableauViz()` does not render the viz. It creates a viz object so that you can configure the viz before it renders. It renders when you add it to the DOM, through something like document.body.appendChild(viz);
* You can use JavaScript’s built-in `document` object to choose where in your html you want to add the viz. The above example uses an empty div and getElementById similar to the JavaScript API v2 approach, but you may other approaches depending on your scenario. [Read more about JavaScript’s document object.](https://www.w3schools.com/js/js_htmldom_document.asp)
* All of the [configuration options listed above as properties of the viz object](#configuration), can be defined in the same manner as `viz.toolbar` above.
* Changing the properties, as shown in this example, will re-render the viz if it is already rendered. This is especially important to note for filtering. The addFilter method is not intended to be used after the viz has been initialized, instead use applyFilterAsync (and the other Filtering methods when appropriate).



## Interacting with the Viz

All of the above discusses how to initialize the Viz and to configure it during initial load. You will likely want to interact with the Viz — filter, add/remove event listeners, change parameters, query data, etc. — after the Viz loads. For this functionality, you do need to write javascript code. 

### Accessing the Viz object

If you [initialized the Viz in JavaScript](#alternative-approach-initialization-via-javascript), then you already have a Viz object which you can interact with. However if you configured it entirely in html, you now have to create a javascript object to represent it:

    
```javascript
    
import {TableauViz} from '../tableau.embedding.3.0.0-alpha.23.js'

let viz = document.getElementById('tableauViz');
````

After either variety of initialization code runs, you will have a Viz object to work with - however it will not be ‘interactive’ immediately after you initialize the Viz object, so you cannot immediately run any interactive methods like mark selection, querying etc. To run this as soon as possible after initialization you should use the onFirstInteractive event listener to ‘wait’ for the viz object to be ready.


### Accessing other Objects and Namespaces

Objects related to your current object are treated as properties (this is consistent with other modern Javascript, but is a change from v2). For example, to get the Worksheet object[s] for the current Sheet:

```javascript
viz.activeSheet.worksheets;
```

The only exception to this is when calling an asynchronous method, such as getUnderlyingDataAsync(). The reference will list the appropriate properties and methods for each Namespace.

Whether accessed with a property or getter method, related objects are returned as native JavaScript arrays and you can use standard library methods to manage them. For example, you can use JavaScript’s .find to select a specific object from the list:

```javascript
viz.activeSheet.worksheets
let sameSheet = workbook.publishedSheetsInfo()  # should that be a getter or a property??
    .find(sheet => sheet.name == "Sheet 1");
```

### Querying Data

 There are ??? ways to query data from the viz in v3*:
    - getSummaryDataAsync(options)
    - getUnderlyingTablesAsync() -> getUnderlyingTableDataAsync() ???? is this full data?
    - getSummaryColumnsInfoAsync(): returns only the Column info for when you just need to retrieve metadata about the columns.
    - ?? other?
    

    These are the options you can use to define the data that is returned:
    (is this missing some that also existed in v2?)
|Option	|Accepts	|Description	|
|---	|---	|---	|
|columnsToIncludeById	|int[] (columnID)	|Allows you to specify the list of columnIDs to return	|
|maxRows	|int	|Returns the first set of rows up to the number specified	|
|onlyFormattedValues	|bool	|Only return the formatted values (as specified by the viz author) and not the native values	|
|onlyNativeValues	|bool	|Only return the native values and not the formatted values	|

There are also new supporting properties on other objects that can help you set the values for the filter, such as the fieldID property of Column object, for use in columnsToInclude.

*In v2, getUnderlyingDataAsync() was deprecated and only available in Tableau 2020.1 or earlier. Therefore, it is not available in v3. Instead [use getUnderlyingTablesAsync() and getUnderlyingTableDataAsync()](https://help.tableau.com/current/api/js_api/en-us/JavaScriptAPI/js_api_ref.htm#gettabledata).

    
### Filtering, Selecting Marks, and Changing Parameters

Filtering, selecting marks, and changing parameters after initialization remain the same:

```javascript
worksheet.applyFilterAsync("Product Type", "Coffee", 
    tableau.FilterUpdateType.Replace``);

```
Note: the addFilter method is not intended to be used after the viz has been initialized, instead use applyFilterAsync (and the other Filtering methods when appropriate).    
    

```javascript
workbook.changeParameterValueAsync("Product Type", "Coffee");

```

*changeParameterValueAsync is available from Tableau Server 2022.3, Use tableau.embedding.3.3.0.js version for this functionality.*

```javascript
worksheet.selectMarksAsync("Product", "Caffe Latte", 
    tableau.SelectionUpdateType.Replace);`
```

### Event Listeners

To define your own actions for handling the events you listen to, you must write javascript methods. Event listeners can be added before initialization in html or js, and you can also add and remove event listeners using javascript while interacting with the viz- for example, perhaps you want to make a button visible only if the user has selected some data:

```javascript 
viz.addEventListener("marksSelection", function (marks) {
   changeMySelectionButton();
});
```
    
Or perhaps you already defined the event listener above, but for certain users you don't ever want to let the user see that button. When you identify one of those users, you just remove that event listener so the button will never be enabled:
    
```javascript
viz.removeEventListener("marksSelection", changeMySelectionUI);
```

 See the sample respondToEvents.html, resize.html, or selectMarks.html for some event listeners in action. 
    
