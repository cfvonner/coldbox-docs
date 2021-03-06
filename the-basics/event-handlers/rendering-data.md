# Rendering Data

Every event handler controller can render data back to its caller in several formats: views/layouts, return data, and `event.renderdata()`.

## Handler Return Data

Handlers can return simple strings or complex objects. If they return simple strings, then the strings will be rendered out to the user:

```javascript
function index(event,rc,prc){
    return "<h1>Hello from my handler today at :#now()#</h1>";
}

function jsondata( event, rc, prc ){
    var mydata = myservice.getData();
    return serializeJSON( mydata );
}
```

Complex objects in normal MVC mode will not be rendered out. They only make sense when you are calling the action via a `runEvent()` call or from a ColdBox Proxy.

```javascript
function list( event, rc, prc ){
    prc.users = userService.list();
    if( event.isProxyRequest() ){
        return prc.users;
    }
    event.setView( "users/list" );
}
```

## event.renderData\(\)

Using the `renderdata()` method of the event object is the most flexible for RESTFul web services or pure data marshalling. Out of the box ColdBox can marshall data \(structs,queries,arrays,complex or even ORM entities\) into the following output formats:

* XML
* JSON
* JSONP
* HTML
* TEXT
* PDF
* WDDX
* CUSTOM

Here is the method signature:

```javascript
/**
* Use this method to tell the framework to render data for you. The framework will take care of marshalling the data for you
* @type.hint The type of data to render. Valid types are JSON, JSONP, JSONT, XML, WDDX, PLAIN/HTML, TEXT, PDF. The deafult is HTML or PLAIN. If an invalid type is sent in, this method will throw an error
* @data.hint The data you would like to marshall and return by the framework
* @contentType.hint The content type of the data. This will be used in the cfcontent tag: text/html, text/plain, text/xml, text/json, etc. The default value is text/html. However, if you choose JSON this method will choose application/json, if you choose WDDX or XML this method will choose text/xml for you.
* @encoding.hint The default character encoding to use.  The default encoding is utf-8
* @statusCode.hint The HTTP status code to send to the browser. Defaults to 200
* @statusText.hint Explains the HTTP status code sent to the browser.
* @location.hint Optional argument used to set the HTTP Location header
* @jsonCallback.hint Only needed when using JSONP, this is the callback to add to the JSON packet
* @jsonQueryFormat.hint JSON Only: query or array format for encoding. The default is CF query standard
* @jsonAsText.hint If set to false, defaults content mime-type to application/json, else will change encoding to plain/text
* @xmlColumnList.hint XML Only: Choose which columns to inspect, by default it uses all the columns in the query, if using a query
* @xmlUseCDATA.hint XML Only: Use CDATA content for ALL values. The default is false
* @xmlListDelimiter.hint XML Only: The delimiter in the list. Comma by default
* @xmlRootName.hint XML Only: The name of the initial root element of the XML packet
* @pdfArgs.hint All the PDF arguments to pass along to the CFDocument tag.
* @formats.hint The formats list or array that ColdBox should respond to using the passed in data argument. You can pass any of the valid types (JSON,JSONP,JSONT,XML,WDDX,PLAIN,HTML,TEXT,PDF). For PDF and HTML we will try to render the view by convention based on the incoming event
* @formatsView.hint The view that should be used for rendering HTML/PLAIN/PDF. By default ColdBox uses the name of the event as an implicit view
* @isBinary.hint Bit that determines if the data being set for rendering is binary or not.
*/
function renderData(
    type="HTML",
    required data,
    contentType="",
    encoding="utf-8",
    numeric statusCode=200,
    statusText="",
    location="",
    jsonCallback="",
     jsonQueryFormat="query",
    boolean jsonAsText=false,
    xmlColumnList="",
    boolean xmlUseCDATA=false,
    xmlListDelimiter=",",
    xmlRootName="",
    struct pdfArgs={},
    formats="",
    formatsView="",
    formatsRedirect={},
    boolean isBinary=false
)
```

Below are a few simple examples:

```javascript
// html marshalling
function renderHTML(event,rc,prc){
    event.renderData( data="<h1>My HTML</h1>" );
}
// xml marshalling
function getUsersXML(event,rc,prc){
    var qUsers = getUserService().getUsers();
    event.renderData (type="XML", data=qUsers );
}
//json marshalling
function getUsersJSON(event,rc,prc){
    var qUsers = getUserService().getUsers();
    event.renderData( type="json", data=qUsers, statusCode=403 );
}
```

As you can see, it is very easy to render data back to the browser or caller. You can even choose plain and send HTML back if you wanted to.

### Render PDFs

You can also render out PDFs from ColdBox using the render data method. The data argument can be either the full binary of the PDF or simple values to be rendered out as a PDF; like views, layouts, strings, etc.

```javascript
// from binary
function pdf(event,rc,prc){
  var binary = fileReadAsBinary( file.path );
  event.renderData( data=binary, type="PDF" );
}

// from content
function pdf(event,rc,prc){
  event.renderData( data=renderView("views/page"), type="PDF" );
}
```

There is also a `pdfArgs` argument in the render data method that can take in a structure of name-value pairs that will be used in the `cfdocument` \([See docs](http://help.adobe.com/en_US/ColdFusion/9.0/CFMLRef/WSc3ff6d0ea77859461172e0811cbec22c24-7c21.html)\) tag when generating the PDF. This is a great way to pass in arguments to really control the way PDF's are generated uniformly.

```javascript
// from content and with pdfArgs
function pdf(event,rc,prc){
  var pdfArgs = { bookmark = "yes", backgroundVisible = "yes", orientation="landscape" };
  event.renderData(data=renderView("views/page"), type="PDF", pdfArgs=pdfArgs);
}
```

### Renderdata With Formats

The `renderData()` method also has two powerful arguments: `formats & formatsView`. If you currently have code like this:

```javascript
event.paramValue("format", "html");

switch( rc.format ){
    case "json" : case "jsonp" : case "xml" : {
        event.renderData(data=mydata, type=rc.format);
        break;
    } 
    case "pdf" : {
        event.renderData(data=renderView("even/action"), type="pdf");
        break;
    }
    case "html" : {
        event.setView( "event/action" );
        break;
      }
};
```

Where you need to param the incoming format extension, then do a switch and do some code for marshalling data into several formats. Well, no more, you can use our formats argument and ColdBox will marshall and code all that nasty stuff for you:

```javascript
event.renderData( data=MyData, formats="xml,json,html,pdf" );
```

That's it! ColdBox will figure out how to deal with all the passed in formats for you that `renderdata` can use. By convention it will use the name of the incoming event as the view that will be rendered for HTML and PDF; implicit views. If the event was users.list, then the view would be views/users/list.cfm. However, you can tell us which view you like if it is named different:

```javascript
event.renderData( data=MyData, formats="xml,json,html,pdf", formatsView="data/MyView" );
```

If you need to redirect for html events, you can pass any arguments you normally would pass to `setNextEvent` to `formatsRedirect`.

```javascript
event.renderData( data=MyData, formats="xml,json,html,pdf", formatsRedirect={event="Main.index"} );
```

### Custom Data Conversion

You can do custom data conversion by convention when marshalling CFCs. If you pass in a CFC as the `data` argument and that CFC has a method called `$renderdata()`, then the marshalling utility will call that function for you instead of using the internal marshalling utilities. You can pass in the custom content type for encoding as well:

```javascript
// get an instance of your custom converter
myConverter = getInstance("MyConverter")
// put some data in it
myConverter.setData( data );
// marshall it out according to your conversions and the content type it supports
event.renderData( data= myConverter, contentType=myConverter.getContentType() );
```

The CFC converter:

```javascript
component accessors="true"{

    property name="data" type="mytype";
    property name="contentType";

    function init(){ 
        setContentType("text");
        return this; 
    }

    // The magical rendering
    function $renderdata(){
        var d = {
            n = data.getName(),
            a = data.getAge(),
            c = data.getCoo(),
            today = now()
        };

        return d.toString();
    }

}
```

In this approach your `$renderdata()` function can be much more customizable than our internal serializers. Just remember to use the right contentType argument so the browser knows what to do with it.

