# Mockserver contributor API

While the mockserver provides out of the box support for most of the OData operation you are looking for, it is possible that you have certain scenario where you'd like to have a more granular control over what happens.

To that end, the mockdata API used internally is fully extensible and you can implement your own custom behavior for each entity set that may require it.

The starting point is that you can provide a javascript (or Typescript) file with the same name as your EntitySet or Singleton that will implement various feature.

This also goes hand to hand with providing the mockdata as JSON files, so for one entity file you can have one JSON file providing the initial dataset and one JS file providing the custom logic you want to define.

## Available functions

### getInitialDataset

This method allow to provide the initial dataset for the entity set dynamically
You just have to return an array of objects that will be used as the initial dataset.

- `getInitialDataSet?: (contextId: string) => object[];`
  - `contextId` represents the tenant id for the current request

### fetchEntries

This method allow to provide the GET request for entity set. It should be in the file named *<entitySetName>.js* where <entitySetName> is the name of the entity set.

If you want to return a specifi file as result of your GET request, here is a sample code:
```javascript
module.exports = {
  fetchEntries: function (keyValues, odataRequest) {
    const fs = require('fs');
    if (odataRequest.queryPath[0].path == '<entitySetName>') {
      return JSON.parse(fs.readFileSync(process.cwd() + '\\webapp\\localService\\data\\Expand.json', 'utf8'));
    }
  },
}
```

### addEntry

This method allow to provide POST request (e.g. create entity) for entity set. It should be in the file named *<entitySetName>.js* where <entitySetName> is the name of the entity set.

If you want to modify the request content of the create request:
```javascript
module.exports = {
    /**
     * Catch create Entity and add uri in the metadata.
     * 
     * @param {JSon} mockEntry oData from the request
     * @param {ODataRequest} odataRequest Instance of the current REST request
     */
  addEntry: function (mockEntry, odataRequest) {
    mockEntry.__metadata['uri'] = odataRequest.requestContent.url + "('" + odataRequest.requestContent.body.MessageUid + "')";
    this.base.addEntry(mockEntry);
  },
};
```
### executeAction

Provide a hook to handle the action calls, it's called whenever an action is executed.
It contains the action definition from the metadata alongside the current set of data from the action and the key of the object it's applying to

- `executeAction(actionDefinition: Action, actionData: any, keys: Record<string, any>): object;`

It should be in the generic file *EntityContainer.js*, the implementation looks like this:

```javascript
module.exports = {
  ...
  executeAction: function (actionDefinition, actionData, keys) {
    if (actionDefinition.name = 'Accept') {
      ...
    }
  },
  ...
};
```

### onBeforeAction / onAfterAction

Provides hook before and after an action is executed, this is mostly used to provide additional handling on top of the standard draft workflow.

- `onBeforeAction(actionDefinition: Action, actionData: any, keys: Record<string, any>): object;`
- `onAfterAction(actionDefinition: Action, actionData: any, keys: Record<string, any>, responseData: any): any;`

### onBeforeUpdateEntry / onAfterUpdateEntry

Provides hook before and after an entry is updated, this is mostly used to provide additional handling on top of the standard mechanism.

- `onBeforeUpdateEntry(keyValues: KeyDefinitions, updatedData: object): void;`
- `onAfterUpdateEntry(keyValues: KeyDefinitions, updatedData: object): void;`


### throwError

This method allows to throw an error, it's useful to provide a custom error message.

- `throwError(message: string, statusCode?: number, messageData?: object, isSAPMessage?: boolean, headers?: Record<string, string>): void;`
    - `message` contains the message you're sending
    - `statusCode` contains the status code you want to see on the request
    - `messageData` contains the data you want to send along with the message
    - `isSAPMessage` if set to true, the message will be sent as a SAP message
    - `headers` contains the headers you want to send along with the message

### base API

On top of providing ways to override default behavior, you also have access to a `base` API that comprises of the basic functionality offered by the mockdata API.

#### addEntry

Add a new entry to the data set.

`addEntry: (mockEntry: object, odataRequest: ODataRequest) => void;`

#### updateEntry

Update an existing entry, identified by it's keyValues into the data set.

`updateEntry: (keyValues: KeyDefinitions, newData: object, odataRequest: ODataRequest) => void;`

#### removeEntry

Remove an entry from the data set

`removeEntry: (keyValues: KeyDefinitions, odataRequest: ODataRequest) => void;`