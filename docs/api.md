# API Reference

## JSONEditor

### Constructor

#### `JSONEditor(container [, options [, json]])`

Constructs a new JSONEditor.

*Parameters:*

- `{Element} container`

  An HTML DIV element. The JSONEditor will be created inside this container element.

- `{Object} options`

  Optional object with options. The available options are described under
  [Configuration options](#configuration-options).

- `{JSON} json`

  Initial JSON data to be loaded into the JSONEditor. Alternatively, the method `JSONEditor.set(json)` can be used to load JSON data into the editor.

*Returns:*

- `{JSONEditor} editor`

  New instance of a JSONEditor.

### Configuration options

- `{Object} ace`

  Provide a custom version of the [Ace editor](http://ace.c9.io/) and use this instead of the version that comes embedded with JSONEditor. Only applicable when `mode` is `code`.

  Note that when using the minimalist version of JSONEditor (which has Ace excluded), JSONEditor will try to load the Ace plugins `ace/mode/json` and `ace/ext/searchbox`. These plugins must be loaded beforehand or be available in the folder of the Ace editor.

- `{Object} ajv`

  Provide a custom instance of [ajv](https://github.com/epoberezkin/ajv), the
  library used for JSON schema validation. Example:

  ```js
  var options = {
    ajv: Ajv({ allErrors: true, verbose: true })
  }
  ```

- `{function} onChange()`

  Set a callback function triggered when the contents of the JSONEditor change.
  This callback does not pass the changed contents, use `get()` or `getText()` for that.
  Note that `get()` can throw an exception in mode `text`, `code`, or `preview`, when the editor contains invalid JSON.
  Will only be triggered on changes made by the user, not in case of programmatic changes via the functions `set`, `setText`, `update`, or `updateText`.
  See also callback functions `onChangeJSON(json)` and `onChangeText(jsonString)`.

- `{function} onChangeJSON(json)`

  Set a callback function triggered when the contents of the JSONEditor change.
  Passes the changed JSON document.
  Only applicable when option `mode` is `tree`, `form`, or `view`.
  The callback will only be triggered on changes made by the user, not in case of programmatic changes via the functions `set`, `setText`, `update`, or `updateText`.
  Also see the callback function `onChangeText(jsonString)`.

- `{function} onChangeText(jsonString)`

  Set a callback function triggered when the contents of the JSONEditor change.
  Passes the changed JSON document inside a string (stringified).
  The callback will only be triggered on changes made by the user, not in case of programmatic changes via the functions `set`, `setText`, `update`, or `updateText`.
  Also see the callback function `onChangeJSON(json)`.

- `{function} onClassName({ path, field, value })`

  Set a callback function to add custom CSS classes to the rendered nodes. Only applicable when option `mode` is `tree`, `form`, or `view`.

  The callback is invoked with an object containing `path`, `field` and `value`:

  ```
  {
    path: string[],
    field: string,
    value: string
  }
  ```
  The function must either return a string containing CSS class names, or return `undefined` in order to do nothing for a specific node.

  In order to update css classes when they depend on external state, you can call `editor.refresh()`.

- `{function} onEditable({ path, field, value })`

  Set a callback function to determine whether individual nodes are editable or read-only. Only applicable when option `mode` is `tree`, `text`, or `code`.

  In case of mode `tree`, the callback is invoked as `editable(node)`, where the first parameter is an object:

  ```
  {
    field: string,
    value: string,
    path: string[]
  }
  ```

  The function must either return a boolean value to set both the nodes field and value editable or read-only, or return an object `{field: boolean, value: boolean}` to set set the read-only attribute for field and value individually.

  In modes `text` and `code`, the callback is invoked as `editable(node)` where `node` is an empty object (no field, value, or path). In that case the function can return false to make the text or code editor completely read-only.

- `{function} onError(error)`

  Set a callback function triggered when an error occurs. Invoked with the error as first argument. The callback is only invoked
  for errors triggered by a users action, like switching from code mode to tree mode or clicking the Format button whilst the editor doesn't contain valid JSON.

- `{function} onModeChange(newMode, oldMode)`

  Set a callback function triggered right after the mode is changed by the user. Only applicable when
  the mode can be changed by the user (i.e. when option `modes` is set).

- `{function} onNodeName({ path, type, size })`

  Customize the name of object and array nodes. By default the names are brackets with the number of childs inside,
  like `{5}` and `[32]`. The number inside can be customized. using `onNodeName`.

  The first parameter is an object containing the following properties:

  ```
  {
    path: string[],
    type: 'object' | 'array',
    size: number
  }
  ```

  The `onNodeName` function should return a string containing the name for the node. If nothing is returned,
  the size (number of childs) will be displayed.

- `{function} onValidate(json)`

  Set a callback function for custom validation. Available in all modes.

  On a change of the JSON, the callback function is invoked with the changed data. The function should return
  an array with errors or null if there are no errors. The function can also return a `Promise` resolving with
  the errors retrieved via an asynchronous validation (like sending a request to a server for validation).
  The returned errors must have the following structure: `{path: Array.<string | number>, message: string}`.
  Example:

  ```js
  var options = {
    onValidate: function (json) {
      var errors = [];

      if (json && json.customer && !json.customer.address) {
        errors.push({
          path: ['customer'],
          message: 'Required property "address" missing.'
        });
      }

      return errors;
    }
  }
  ```
  
  Also see the option `schema` for JSON schema validation.

- `{function} onValidationError(errors)`

  Set a callback function for validation and parse errors. Available in all modes.

  On validation of the json, if errors of any kind were found this callback is invoked with the errors data.

  On change, the callback will be invoked only if errors were changed.

  Example:

  ```js
  var options = {
    /**
    * @param {Array} errors validation errors
    */
    onValidationError: function (errors) {
      errors.forEach((error) => {
        switch (error.type) {
          case 'validation': // schema validation error
            ...
            break;
          case 'customValidation': // custom validation error
            ...
            break;
          case 'error':  // json parse error
            ...
            break;
          ...
        }
      });
      ...
    }
  }
  ```
  
  
- `{function} onCreateMenu(items, node)`
  
  Customize context menus in tree mode.

  Sets a callback function to customize the context menu in tree mode. Each time the user clicks on the context menu button, an array of menu items is created. If this callback is configured, the array with menu items is passed to this function. The menu items can be customized in this function in any aspect of these menu items, including deleting them and/or adding new items. The function should return the final array of menu items to be displayed to the user.

  Each menu item is represented by an object, which may also contain a submenu array of items. See the source code of example 21 in the examples folder for more info on the format of the items and submenu objects.

  The second argument `node` is an object containing the following properties:

  ```
  {
    type: 'single' | 'multiple' | 'append'
    path: Array,
    paths: Array with paths
  }
  ```

  The property `path` containing the path of the node, and `paths` contains the same path or in case there are multiple selected nodes it contains the paths of all selected nodes.
  When the user opens the context menu of an append node (in an empty object or array), the `type` will be `'append'` and the `path` will contain the path of the parent node.

- `{boolean} escapeUnicode`

  If true, unicode characters are escaped and displayed as their hexadecimal code (like `\u260E`) instead of of the character itself (like `☎`). `false` by default.

- `{boolean} sortObjectKeys`

  If true, object keys in 'tree', 'view' or 'form' mode list be listed alphabetically instead by their insertion order. Sorting is performed using a natural sort algorithm, which makes it easier to see objects that have string numbers as keys. `false` by default.

- `{boolean} history`

  Enables history, adds a button Undo and Redo to the menu of the JSONEditor. `true` by default. Only applicable when `mode` is 'tree' or 'form'.

- `{String} mode`

  Set the editor mode. Available values: 'tree' (default), 'view', 'form', 'code', 'text', 'preview'. In 'view' mode, the data and datastructure is read-only. In 'form' mode, only the value can be changed, the data structure is read-only. Mode 'code' requires the Ace editor to be loaded on the page. Mode 'text' shows the data as plain text.
  The 'preview' mode can handle large JSON documents up to 500 MiB. It shows a preview of the data, and allows to
  transform, sort, filter, format, or compact the data.

- `{String[]} modes`

  Create a box in the editor menu where the user can switch between the specified modes. Available values: see option `mode`.

- `{String} name`

  Initial field name for the root node, is `undefined` by default. Can also be set using `JSONEditor.setName(name)`. Only applicable when `mode` is 'tree', 'view', or 'form'.

- `{Object} schema`

  Validate the JSON object against a JSON schema. A JSON schema describes the
  structure that a JSON object must have, like required properties or the type
  that a value must have.

  See [http://json-schema.org/](http://json-schema.org/) for more information.

  Also see the option `onValidate` for custom validation.

- `{Object} schemaRefs`

  Schemas that are referenced using the `$ref` property from the JSON schema that are set in the `schema` option,
  the object structure in the form of `{reference_key: schemaObject}`

- `{boolean} search`

  Enables a search box in the upper right corner of the JSONEditor. `true` by default. Only applicable when `mode` is 'tree', 'view', or 'form'.

- `{Number} indentation`

  Number of indentation spaces. `2` by default. Only applicable when `mode` is 'code', 'text', or 'preview'.

- `{String} theme`

  Set the Ace editor theme, uses included 'ace/theme/jsoneditor' by default. Please note that only the default theme is included with JSONEditor, so if you specify another one you need to make sure it is loaded.

- `{Object} templates`

  Array of templates that will appear in the context menu, Each template is a json object precreated that can be added as a object value to any node in your document. 

  The following example allow you can create a "Person" node and a "Address" node, each one will appear in your context menu, once you selected the whole json object will be created.

  ```js
  var options = {
    templates: [
          {
              text: 'Person',
              title: 'Insert a Person Node',
              className: 'jsoneditor-type-object',
              field: 'PersonTemplate',
              value: {
                  'firstName': 'John',
                  'lastName': 'Do',
                  'age': 28
              }
          },
          {
              text: 'Address',
              title: 'Insert a Address Node',
              field: 'AddressTemplate',
              value: {
                  'street': "",
                  'city': "",
                  'state': "",
                  'ZIP code': ""
              }
          }
      ]
  }
  ```

- `{Object} autocomplete`

  *autocomplete* will enable this feature in your editor in tree mode, the object have the following **subelements**:

  - `{string} filter`
  - `{Function} filter`

     Indicate the filter method of the autocomplete. Default to `start`.

     - `start`         : Match your input from the start, e.g. `ap` match `apple` but `pl` does not.
     - `contain`       : Contain your input or not, e.g. `pl` match `apple` too.
     - Custom Function : Define custom filter rule, return `true` will match you input.

  - `{string} trigger`

     Indicate the way to trigger autocomplete menu. Default to `keydown`

     - `keydown` : When you type something in the field or value, it will trigger autocomplete.
     - `focus`   : When you focus in the field or value, it will trigger the autocomplete.

  - `{number[]} confirmKeys`

     Indicate the KeyCodes for trigger confirm completion, by default those keys are:  `[39, 35, 9]` which are the code for [right, end, tab]

  - `{boolean} caseSensitive`

     Indicate if the autocomplete is going to be strict case-sensitive to match the options.

  - `{Function} getOptions (text: string, path: string[], input: string, editor: JSONEditor)`

     This function will return your possible options for create the autocomplete selection, you can control dynamically which options you want to display according to the current active editing node.
     
     *Parameters:*

     - `text`   : The text in the current node part. (basically the text that the user is editing)
     - `path`   : The path of the node that is being edited as an array with strings.
     - `input`  : Can be "field" or "value" depending if the user is editing a field name or a value of a node.
     - `editor` : The editor instance object that is being edited.

     *Returns:*

     - Can return an array with autocomplete options (strings), for example `['apple','cranberry','raspberry','pie']`
     - Can return `null` when there are no autocomplete options.
     - Can return an object `{startFrom: number, options: string[]}`. Here `startFrom` determines the start character from where the existing text will be replaced. `startFrom` is `0` by default, replacing the whole text.
     - Can return a `Promise` resolving one of the return types above to support asynchronously retrieving a list with options.

- `{boolean} mainMenuBar`

  Adds main menu bar - Contains format, sort, transform, search etc. functionality. `true` by default. Applicable in all types of `mode`.

- `{boolean} navigationBar`

  Adds navigation bar to the menu - the navigation bar visualize the current position on the tree structure as well as allows breadcrumbs navigation. `true by default. Only applicable when `mode` is 'tree', 'form' or 'view'.

- `{boolean} statusBar`

  Adds status bar to the bottom of the editor - the status bar shows the cursor position and a count of the selected characters. `true` by default. Only applicable when `mode` is 'code', 'text', or 'preview'.

- `{function} onTextSelectionChange(start, end, text)`

  Set a callback function triggered when a text is selected in the JSONEditor.

  callback signature should be:
  ```js
  /**
  * @param {{row:Number, column:Number}} start Selection start position
  * @param {{row:Number, column:Number}} end   Selected end position
  * @param {String} text selected text
  */
  function onTextSelectionChange(start, end, text) {
    ...
  }
  ```
  Only applicable when `mode` is 'code' or 'text'.

- `{function} onSelectionChange(start, end)`

  Set a callback function triggered when Nodes are selected in the JSONEditor.

  callback signature should be:
  ```js
  /**
  * @typedef {{value: String|Object|Number|Boolean, path: Array.<String|Number>}} SerializableNode
  * 
  * @param {SerializableNode=} start
  * @param {SerializableNode=} end
  */
  function onSelectionChange(start, end) {
    ...
  }
  ```
  Only applicable when `mode` is 'tree'.
  
- `{function} onEvent({ field, path, value? }, event)`

  Set a callback function that will be triggered when an event will occur in 
  a JSON field or value.
  
  In case of field event, node information will be 

  ```
  {
    field: string,
    path: {string|number}[]
  }
  ```

  In case of value event, node information will be

  ```
  {
    field: string,
    path: {string|number}[],
    value: string
  }
  ```

  signature should be:
  ```js
  /**
  * @param {Node} the Node where event has been triggered 
  identified by {field: string, path: {string|number}[] [, value: string]}`
  * @param {event} the event fired
  */
  function onEvent(node, event) {
    ...
  }
  ```
  Only applicable when `mode` is 'form', 'tree' or 'view'.  

- `{boolean} colorPicker`

  If `true` (default), values containing a color name or color code will have a color picker rendered on their left side.

- `{function} onColorPicker(parent, color, onChange)`

  Callback function triggered when the user clicks a color.
  Can be used to implement a custom color picker.
  The callback is invoked with three arguments:
  `parent` is an HTML element where the color picker can be attached,
  `color` is the current color,
  `onChange(newColor)` is a callback which has to be invoked with the new color selected in the color picker.
  JSONEditor comes with a built-in color picker, powered by [vanilla-picker](https://github.com/Sphinxxxx/vanilla-picker).

  A simple example of `onColorPicker` using `vanilla-picker`:

  ```js
  var options = {
    onColorPicker: function (parent, color, onChange) {
      new VanillaPicker({
        parent: parent,
        color: color,
        onDone: function (color) {
          onChange(color.hex)
        }
      }).show();
    }
  }
  ```

- `{boolean | function({field, value, path}) -> boolean} timestampTag`

  If `true` (default), a tag with the date/time of a timestamp is displayed
  right from values containing a timestamp. By default, a value is 
  considered a timestamp when it is (a) an integer number with a value larger 
  than Jan 1th 2000, `946684800000`, or (b) it's field name contains any of the
  following strings (case insensitive): `'date'`, `'time'`, `'created'`,
  `'updated'`, `'deleted'`.
  
  When `timestampTag` a is a function, a timestamp tag will be displayed when 
  this function returns `true`. The function is invoked with an object as first 
  parameter:
  
  ```
  {
    field: string,
    value: string,
    path: string[]
  }
  ```

  Whether a value is a timestamp can be determined implicitly based on 
  the `value`, or explicitly based on `field` or `path`.

  Example:
  
  ```js
  var options = {
    timestampTag: function ({ field, value, path }) {
      if (field === 'dateCreated') {
        return true
      }

      return false
    }
  }
  ```
  
  Only applicable for modes `tree`, `form`, and `view`.

- `{string} language`

  The default language comes from the browser navigator, but you can force a specific language. So use here string as 'en' or 'pt-BR'. Built-in languages: `en`, `pt-BR`, `zh-CN`, `tr`, `ja`, `fr-FR`. Other translations can be specified via the option `languages`.

- `{Object} languages`

  You can override existing translations or provide a new translation for a specific language. To do it provide an object at languages with language and the keys/values to be inserted. For example:

  ```
  'languages': {
    'pt-BR': {
      'auto': 'Automático testing'
    },
    'en': {
      'auto': 'Auto testing'
    }
  }
  ```

  All available fields for translation can be found in the source file `src/js/i18n.js`.

- `{HTMLElement} modalAnchor`

  The container element where modals (like for sorting and filtering) are attached: an overlay will be created on top
  of this container, and the modal will be created in the center of this container.

- `{boolean} enableSort`

  Enable sorting of arrays and object properties. Only applicable for mode 'tree'. `true` by default.

- `{boolean} enableTransform`

  Enable filtering, sorting, and transforming JSON using a [JMESPath](http://jmespath.org/) query. Only applicable for mode 'tree'. `true` by default.

- `{Number} maxVisibleChilds`

  Number of children allowed for a given node before the "show more / show all" message appears (in 'tree', 'view', or 'form' modes). `100` by default.

### Methods

#### `JSONEditor.collapseAll()`

Collapse all fields. Only applicable for mode 'tree', 'view', and 'form'.

#### `JSONEditor.destroy()`

Destroy the editor. Clean up DOM, event listeners, and web workers.

#### `JSONEditor.expandAll()`

Expand all fields. Only applicable for mode 'tree', 'view', and 'form'.

#### `JSONEditor.focus()`

Set focus to the JSONEditor.

#### `JSONEditor.get()`

Get JSON data.

This method throws an exception when the editor does not contain valid JSON,
which can be the case when the editor is in mode `code`, `text`, or `preview`.

*Returns:*

- `{JSON} json`

  JSON data from the JSONEditor.

#### `JSONEditor.getMode()`

Retrieve the current mode of the editor.

*Returns:*

- `{String} mode`

  Current mode of the editor, for example `tree` or `code`.

#### `JSONEditor.getName()`

Retrieve the current field name of the root node.

*Returns:*

- `{String | undefined} name`

  Current field name of the root node, or undefined if not set.

#### `JSONEditor.getNodesByRange(start, end)`

A utility function for getting a list of `SerializableNode` under certain range.

This function can be used as complementary to `getSelection` and `onSelectionChange` if a list of __all__ the selected nodes is required.

*Parameters:*

- `{path: Array.<String>} start`

  Path for the first node in range

- `{path: Array.<String>} end`

  Path for the last node in range

#### `JSONEditor.getSelection()`

Get the current selected nodes, Only applicable for mode 'tree'.

*Returns:*

- `{start:SerializableNode, end: SerializableNode}`

#### `JSONEditor.getText()`

Get JSON data as string.

*Returns:*

- `{String} jsonString`

  Contents of the editor as string. When the editor is in code `text`, `code` or `preview`,
  the returned text is returned as-is. For the other modes, the returned text
  is a compacted string. In order to get the JSON formatted with a certain
  number of spaces, use `JSON.stringify(JSONEditor.get(), null, 2)`.

#### `JSONEditor.getTextSelection()`

Get the current selected text with the selection range, Only applicable for mode 'text' and 'code'.

*Returns:*

- `{start:{row:Number, column:Number},end:{row:Number, column:Number},text:String} selection`


#### `JSONEditor.refresh()`

Force the editor to refresh the user interface and update all rendered HTML. This can be useful for example when using `onClassName` and the returned class name depends on external factors.


#### `JSONEditor.set(json)`

Set JSON data.
Resets the state of the editor (expanded nodes, search, selection).
See also `JSONEditor.update(json)`.

*Parameters:*

- `{JSON} json`

  JSON data to be displayed in the JSONEditor.

#### `JSONEditor.setMode(mode)`

Switch mode. Mode `code` requires the [Ace editor](http://ace.ajax.org/).

*Parameters:*

- `{String} mode`

  Available values: `tree`, `view`, `form`, `code`, `text`, `preview`.

#### `JSONEditor.setName(name)`

Set a field name for the root node.

*Parameters:*

- `{String | undefined} name`

  Field name of the root node. If undefined, the current name will be removed.

#### `JSONEditor.setSchema(schema [,schemaRefs])`

Set a JSON schema for validation of the JSON object. See also option `schema`.
See [http://json-schema.org/](http://json-schema.org/) for more information on the JSON schema definition.

*Parameters:*

- `{Object} schema`

  A JSON schema.

- `{Object} schemaRefs`

  Optional, Schemas that are referenced using the `$ref` property from the JSON schema, the object structure in the form of `{reference_key: schemaObject}`

#### `JSONEditor.setSelection(start, end)`

Set selection for a range of nodes, Only applicable for mode 'tree'.

- If no parameters sent - the current selection will be removed, if exists.
- For single node selecion send only the `start` parameter.
- If the nodes are not from the same level the first common parent will be selected

*Parameters:*

- `{path: Array.<String>} start`

  Path for the start node

- `{path: Array.<String>} end`

  Path for the end node

#### `JSONEditor.setText(jsonString)`

Set text data in the editor.

This method throws an exception when the provided jsonString does not contain
valid JSON and the editor is in mode `tree`, `view`, or `form`.

*Parameters:*

- `{String} jsonString`

  Contents of the editor as string.

#### `JSONEditor.setTextSelection(startPos, endPos)`

Set text selection for a range, Only applicable for mode 'text' and 'code'.

*Parameters:*

- `{row:Number, column:Number} startPos`

  Position for selection start

- `{row:Number, column:Number} endPos`

  Position for selection end

#### `JSONEditor.update(json)`

Replace JSON data when the new data contains changes.
In modes `tree`, `form`, and `view`, the state of the editor will be maintained (expanded nodes, search, selection).
See also `JSONEditor.set(json)`.

*Parameters:*

- `{JSON} json`

  JSON data to be displayed in the JSONEditor.

#### `JSONEditor.updateText (json)`

Replace text data when the new data contains changes.
In modes `tree`, `form`, and `view`, the state of the editor will be maintained (expanded nodes, search, selection).
Also see `JSONEditor.setText(jsonString)`.

This method throws an exception when the provided jsonString does not contain
valid JSON and the editor is in mode `tree`, `view`, or `form`.

*Parameters:*

- `{String} jsonString`

  Contents of the editor as string.

### Static properties

- `{string[]} JSONEditor.VALID_OPTIONS`

  An array with the names of all known options.

- `{object} ace`

  Access to the bundled Ace editor, via the [`brace` library](https://github.com/thlorenz/brace).
  Ace is used in code mode.
  Same as `var ace = require('brace');`.

- `{function} Ajv`

  Access to the bundled [`ajv` library](https://github.com/epoberezkin/ajv), used for JSON schema validation.
  Same as `var Ajv = require('ajv');`.

- `{function} VanillaPicker`

  Access to the bundled [`vanilla-picker` library](https://github.com/Sphinxxxx/vanilla-picker), used as color picker.
  Same as `var VanillaPicker = require('vanilla-picker');`.


### Examples

A tree editor:

```js
var options = {
    "mode": "tree",
    "search": true
};
var editor = new JSONEditor(container, options);
var json = {
    "Array": [1, 2, 3],
    "Boolean": true,
    "Null": null,
    "Number": 123,
    "Object": {"a": "b", "c": "d"},
    "String": "Hello World"
};
editor.set(json);
editor.expandAll();

var json = editor.get(json);
```

A text editor:

```js
var options = {
    "mode": "text",
    "indentation": 2
};
var editor = new JSONEditor(container, options);
var json = {
    "Array": [1, 2, 3],
    "Boolean": true,
    "Null": null,
    "Number": 123,
    "Object": {"a": "b", "c": "d"},
    "String": "Hello World"
};
editor.set(json);

var json = editor.get();
```

## JSON parsing and stringification

In general, to parse or stringify JSON data, the browsers built in JSON parser can be used. To create a formatted string from a JSON object, use:

```js
var formattedString = JSON.stringify(json, null, 2);
```

to create a compacted string from a JSON object, use:

```js
var compactString = JSON.stringify(json);
```

To parse a String to a JSON object, use:

```js
var json = JSON.parse(string);
```
