
# Developer mode with Chrome Browser 

* Open the developer console on the side of conkolla window
* Reload the page; you will now have the useful js functions loaded.

In the Editor window, an extra library is loaded containing some useful JS function to effectively work with multiple entities and multiple clusters (connections). Basically you use the editor to:

1. GET the records you need to work on or for which you need the data from the UI. Then you use the JS console to work with the data and create
2. update/delete records on same, other or multiple controllers.

But hey, you could also do all the work in pure JavaScript.

## Available on editor UI window
Based on 7.2.0 (unchanged since 5.3.3) `answer_rw_console.js`

| name      |      type    | details  |returns|
| ------------- |:-------------:|--------|-------|
|`dataRecords`|Array| Contains the records of the last GET request from the editor window.||
|`editor.getSession().getValue()`|`returns` str| Returns the content if the editor window|str|
|`postEntity( conID, entity, data)`|`conID`: str or int, `entity`: str: sdp object name (ex: entitlements), `data` obj:JSON|Post (usually create) an `entity` *type* for connection `conID` with `data`. _async_:promise|promise|
|`putEntity( conID, entity, data)`|`conID`: str or int, `entity`: str: sdp object name (ex: entitlements), `data` obj:JSON|Put (usually update) an `entity` *type* for connection `conID` with `data`. _async_|promise|
|`deleteEntity( conID, entity, entityId)`|`conID`: str or int, `entity`: str: sdp object name (ex: entitlements), `entityID` str|Delete an `entity` for connection `conID` with identifier `conID` (usually an UUIDv4 or a DN). _async_|promise|
|`putEntity( conID, entity, data)`|`conID`: str or int, `entity`: str: sdp object name (ex: entitlements), `data` obj:JSON|Put (usually update) an `entity` *type* for connection `conID` with `data`. _async_|promise|
|`getEntity(conID, entity)`|`conID`: str or int, `entity`: str: sdp object name (ex: entitlements)| Gets all entities of name `entity`. Stores the result in variable `consoleDataRecords`. _async_|promise|
|`getEntityById(conID, entity, entityID)`|`conID`: str or int, `entity`: str: sdp object name (ex: entitlements), `entityID`: str: uuid of the entity|gets entity by id, sotres it in variable `consoleDataRecords`._async_|promise|
|`getAndUpdateEntity(conID, entity, entityID, modFunction)`|`conID`: str or int, `entity`: sdp object (ex: entitlements), `modFunction`:function(jsonObject):jsonObject|Gets and updates an entity. The `modFunction` should have one argument, for the json object (from the get entity part) and return the modified json object (for the updateEntity part). _async_|promise|
|`entitiesFromEditor(conID, entity, xEntityFunction)`|`conID`: str or int, `entity`: str: sdp object name (ex: entitlements), `xEntityFunction`: function: `postEntity` or `putEntity`|Parses the text in the editor to JSON, then do the `xEntityFunction` on all records in the JSON. Text must represent the sdp entities `entity`._async_|promise|
| `findEntities(conID, path, filterAttribute, filterValue, key, filterTransformFunc`|`conID`: str or int, `path`: str, `filterAttribute`:str, `filterValue`: str, `key`: str, `filterTransformFunc`: function |Find entities on exact match: Search with query=`filterValue`, then for every record found, check if the `record.filterAttribute` exact matches the `filterValue`. If a transform function was given, apply the function on the retrieved record's field before doing the match. Use the `record.key` value when added to the results as an identifier. Matched records are stored in results[]. | promise, containting results[]|
|`downloadAsFile(data, mimeType, filename)`|`data`, `mimeType`, `filename`: str|Download arbitrary data in form of a file.||
|` getCSV(data)`|`data` json object| Gets as CSV preresentation of data; first level deep, values all stringified.|str|
|`dataRecordsGetCSV()`||Wrapper for `getCSV(dataRsession().getValue())`|str|
|`editorGetCSV()`||Wrapper for `getCSV(editor.get)`|str|

## Best practices

* `dataRecords` is populated when the request has finished, and will be unchanged until a new request (from the Conkolla UI).
* `consoleDataRecords` is not safe to use unless its just a occular check for the programmer, or you know how it behaves. 
* If you depend in your code on the result of a previous call (a `getEntity` for example), make sure you wait until the promise is resolved by using `await`:

```javascript
var result = await getEntity(1,"entitlements");
for(record of result.data){
    // do somehting with the record
}
```
Hence it is advised to work in the editor window if you are unsure.

* Check the [examples page](/examples.md).

