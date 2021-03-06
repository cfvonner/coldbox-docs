# Get\/Set Request Values

We all need values in our applications. That is why we interact with the request context in order to place data from our model layer into it so our views can display it, or to retrieve data from a user's request. You will either interact with the event object to get/set values or put/read values directly via the received rc and prc references. We recommend using the references as structures which are much faster than method calls. The event object should not be discarded, however, as it has some pretty cool and funky methods of its own. Below are some examples of its coolness!

![](https://github.com/ortus/coldbox-platform-documentation/tree/24d3f3d16693b36ca41bf5ce0329c6ff33316ef0/images/RequestCollectionDataBus.jpg)

> **Note** We would recommend you use the private request collection for setting manual data and using the standard request collection for reading the user's request variables. This way a clear distinction can be made on what was sent from the user and what was set by your code.

```javascript
//set a value for views to use
event.setValue( "name", "Luis" );
event.setPrivateValue( "name", "Luis" );

// retrieve a value the user sent
event.getValue( "name" );
// retrieve a value the user sent or give me a default value
event.getValue( "isChecked", false );

// retrieve a private value
event.getPrivateValue( "name" );
// retrieve a private value or give me a default value
event.getPrivateValue( "isChecked", false );

// param a value
event.paramValue( "user_id", "" );
// param a private value
event.paramPrivateValue( "user_id", "" );

// remove a value
event.removeValue( "name" );
// remove a private value
event.removePrivateValue( "name" );

//check if value exists
if( event.valueExists( "name" ) ){

}
//check if private value exists
if( event.privateValueExists( "name" ) ){

}

// set a view for rendering
event.setView( 'blog/index' );

// set a layout for rendering
event.setLayout( 'main' );

// set a view and layout
event.setView( view="blog/userinfo", layout="ajax" );
```

> **Important** The most important paradigm shift from procedural to an MVC framework is that you NO LONGER will be talking to URL, FORM, REQUEST or any ColdFusion scope from within your handlers, layouts, and views. The request collection already has URL, FORM, and REQUEST scope capabilities, so leverage it.

