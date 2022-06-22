---
title: Declaring Global variables in Google Apps Script
tags:  Google Apps Script, gas, javascript, clasp, google , global, variables
article_header:
  type: cover
  image:
    src: images/gas-banner001.png
---

## Declaring Global variables in Google Apps Script

If you are codding on `Google Apps Script` environment you may have noticed that, as the code executed needs to live inside a function there is no way to declare Global Variables that can be executed all across the Script.

In the "compilation" process you always choose which function to run, so passing variables from one function to other can be tricky.

Below you can see the cleanest way I have found.

### Properties Service Overview

For that we need to understand the [PropertiesService](https://developers.google.com/apps-script/reference/properties)
In a nutshell this service lets you store some data in the form of **Strings** , that can be retrieved in further executions of the script, and all across the code that is written for that Script, this will give us:
	- Global like data storage
	- Persistant data storage

In a nutshell we can find the signatures of the methods for this service by

```
w3m --dump https://developers.google.com/apps-script/reference/properties
``` 
My notes look something like this
I usually dump this signatures and add a `-------(X)` so when scrolling down my notes I can easily recall what methods I have already used

```
PropertiesService
PropertiesService.WHATEVERMETHOD

Methods

       Method           Return                 Brief description
                         type
PropertiesService.getDocumentProperties            Gets a property store (for this script only)  ------(X)
()                    Properties that all users can access within the open    FOR SOME REASON I CANNOT USE IT NULL
                                 document, spreadsheet, or form.
PropertiesService.getScriptProperties() Properties Gets a property store that all users can      ------(X)
                                 access, but only within this script.
PropertiesService.getUserProperties()   Properties Gets a property store that only the current   -----(X)
                                 user can access, and only within this script.

      Method          Return                  Brief description
                       type
propertiesRef.deleteAllProperties() Properties Deletes all properties in the current Properties   ---------(X)
                             store.
propertiesRef.deleteProperty(key) Properties Deletes the property with the given key in the    ---------(X)
                               current Properties store.
propertiesRef.getKeys()           String[]   Gets all keys in the current Properties store.
propertiesRef.getProperties()     Object     Gets a copy of all key-value pairs in the       ---------(X)
                               current Properties store.
                               Gets the value associated with the given key in
propertiesRef.getProperty(key)    String     the current Properties store, or null if no such  ---------(X)
                               key exists.
propertiesRef.setProperties(propertiesObject)Properties Sets all key-value pairs from the given object--------(X)
                                 in the current Properties store.
propertiesRef.setProperties                  Sets all key-value pairs from the given object
(propertiesObject,        Properties in the current Properties store, optionally
deleteAllOthers)               deleting all other properties in the store.
propertiesRef.setProperty(key, value) Properties Sets the given key-value pair in the current        -------(X)
                         Properties store.
```

### Initiating the Properties Service

I prepended the `propertiesRef.` to the children methods of the Properties.


So easily we iniciate the service like this

```
function main () {
	var propertiesRef = PropertiesService.getScriptProperties();
}
```

### Storing Properties

We can check then write a certain property, then Log it

```
function main () {
	var propertiesRef = PropertiesService.getScriptProperties();
	propertiesRef.setProperty('MY_VAR' , 'whateverValue');
	Logger.log(propertiesRef.getProperty('MY_VAR'));
}
```

### Problem with undeclared properties

The issue is if you are trying to Access a property that doesn't exists yet. It will throw out a `null` as its value. But I have found sometimes that we can get compilation errors as well when handling this.

```
function main () {
	var propertiesRef = PropertiesService.getScriptProperties();
	Logger.log(propertiesRef.getProperty('MY_VAR2'));
}
// Output : null
```


So I would make sure to initiate the property beforehand if it doesnt exists with a `false` value. Although remember that `false` would be parsed as a string.

```
function main(){
    var propertiesRef = PropertiesService.getScriptProperties();
    propertiesRef.getProperty('MY_VAR') ? 0 : propertiesRef.setProperty('MY_VAR' , 'false');
}
```

### Problem with string datatype

Remember that `false` is a string value, if you wanted to perform some logic with it you would have to had it into consideration.
For instance the next isolated line of code will show you how to store a true/false value , or any other.

```
propertiesRef.setProperty('MY_VAR' , sheetRef.getRange(2 , columnRef).getValue().toString());
```


### Using Properties as Global Variables

Finally you can then share some data between functions the following way.

```
function main(){
    var propertiesRef = PropertiesService.getScriptProperties();
    propertiesRef.getProperty('MY_VAR') ? 0 : propertiesRef.setProperty('MY_VAR' , 'false');

    functionOne();
    Logger.log(propertiesRef.getProperty('MY_VAR'));
}
function functionOne () {
    var propertiesRef = PropertiesService.getScriptProperties();
    propertiesRef.getProperty('MY_VAR') ? 0 : propertiesRef.setProperty('MY_VAR' , 'false');
    /*
        SOME CODE HERE
    */
    propertiesRef.setProperty('MY_VAR' , 'true');

}
```


### Conclusion

May not be the most orthodox solution to our scoping problems, but at least is a solution I have found, and just wanted to share it with you to gain some kudos in my life.
