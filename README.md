# Reactive Table
A reactive table designed for Meteor.

Demo and Feature Comparison: http://reactive-table.meteor.com/

Another Demo: http://reactive-table-leaderboard.meteor.com/

### Note on Versions
The latest version of reactive-table only supports Meteor version 0.9.0 or higher.
For Meteor 0.8, use reactive-table version 0.3.21.
For older versions of Meteor, you can use reactive-table v0.2.5 ([documentation](https://github.com/ecohealthalliance/reactive-table/tree/v0.2.5)).
If you're updating to Meteor 0.8.0, note that reactiveTable is now a template with keyword arguments rather than a helper. The functionality should be the same, but please report bugs in the issues.

### Table of Contents
- [Quick Start](#quick-start)
- [Customization](#customization)
  - [Settings](#settings)
    - [rowClass Examples](#rowclass-examples)
    - [Settings Object](#settings-object)
  - [Styling](#styling)
  - [Setting columns](#setting-columns)
    - [Setting column headers](#setting-column-headers)
    - [Templates](#templates)
    - [Virtual columns](#virtual-columns)
      - [HTML](#html)
    - [Nested objects and arrays](#nested-objects-and-arrays)
    - [Hidden columns](#hidden-columns)
- [Using events](#using-events)
- [Multiple tables](#multiple-tables)
- [Internationalization](#internationalization)

## Quick Start

Install reactive table:

    meteor add aslagle:reactive-table


This package adds a template called reactiveTable. Create and subscribe to a collection, and pass it to the template:

    {{> reactiveTable collection=myCollection}}

When the whole collection should be in the table, it's best to pass in the Meteor collection object (returned by new Meteor.Collection()). You can also pass in the cursor returned by collection.find() to show a subset of the collection, or a plain array to show data that's not in a Meteor collection.

If you're new to Meteor, note that global variables aren't available from templates. You can add template helpers to access them:

    Template.myTemplate.helpers({
        myCollection: function () {
            return myCollection;
        }
    });


## Customization

The reactiveTable helper accepts additional arguments that can be used to configure the table.

    {{> reactiveTable collection=collection showNavigation='never' rowsPerPage=5}}

### Settings

* `showFilter`: Boolean. Whether to display the filter box above the table. Default `true`.
* `rowsPerPage`: Number.  The desired number of rows per page. Defaults to 10.
* `showNavigation`: 'always', 'never' or 'auto'.  The latter shows the navigation footer only if the collection has more rows than `rowsPerPage`.
* `showNavigationRowsPerPage`: Boolean. If the navigation footer is visible, display rows per page control. Default 'true'.
* `fields`: Object. Controls the columns; see below.
* `showColumnToggles`: Boolean. Adds a 'Columns' button to the top right that allows the user to toggle which columns are displayed. (Note: there aren't translations for this button yet - please [add one](#internationalization) if you're using it.) Add `hidden` to fields to hide them unless toggled on, see below. Default `false`.
* `useFontAwesome`: Boolean. Whether to use [Font Awesome](http://fortawesome.github.io/Font-Awesome/) for icons. Requires the `fortawesome:fontawesome` package to be installed. Default `true` if `fortawesome:fontawesome` is installed, else `false`.
* `class`: String. Classes to add to the table element in addition to 'reactive-table'. Default: 'table table-striped table-hover col-sm-12'.
* `id`: String. Unique id to add to the table element. Default: generated with [_.uniqueId](http://underscorejs.org/#uniqueId).
* `rowClass`: String or function returning a class name. The row element will be passed as first parameter.

#### rowClass examples

As a function

```js
rowClass: function(item) {
  var qnt = item.qnt;
  //
  switch (qnt) {
    case 0:
      return 'danger';
    case 1:
    case 2:
      return 'warning';
    default:
      return ''
  }
},
```

as a string

```js
rowClass: 'danger',
```

#### Settings Object

Settings can also be grouped into a single object to pass to the table:

    {{> reactiveTable settings=settings}}

Define the settings in a helper for the template that calls reactiveTable:

    Template.myTemplate.helpers({
        settings: function () {
            return {
                collection: collection,
                rowsPerPage: 10,
                showFilter: true,
                fields: ['name', 'location', 'year']
            };
        }
    });

You can continue to pass some settings as named arguments while grouping the others into the settings object:

    {{> reactiveTable collection=collection fields=fields settings=settings}}

### Styling

Add bootstrap or bootstrap-3 to style the table, or add your own css. The generated table will have the class 'reactive-table'. To use [Font Awesome](http://fortawesome.github.io/Font-Awesome/) for icons, also add the `fortawesome:fontawesome` package.
You can also use the argument `class` to define table styling:

    {{> reactiveTable class="table table-bordered table-hover" collection=myCollection}}


### Setting columns

To specify columns, add a fields key to the settings object.


Fields can simply be an array of field names (attributes in the collection).

    { fields: ['name', 'location', 'year'] }


#### Setting column headers

To set labels for the column headers, use an array of field elements, each with a key (the attribute in the collection) and a label (to display in the table header).

    { fields: [
        { key: 'name', label: 'Name' },
        { key: 'location', label: 'Location' },
        { key: 'year', label: 'Year' }
    ] }

The label can be a string or a function:

    { key: 'name', label: function () { return new Spacebars.SafeString('<i>Name</i>'); } }

All columns are sortable by default, but sorting can be disabled by setting `sortable` to false:

    { key: 'year', label: 'Year', sortable: false }


#### Templates

You can specify a template to use to render cells in a column, by adding `tmpl` to the field options.

    { fields: [
        { key: 'name', label: 'Name', tmpl: Template.nameTmpl },
        { key: 'location', label: 'Location', tmpl: Template.locationTmpl }
    ] }

The template's context will be the full object, so it will have access to all fields.

#### Virtual columns

You can also compute a function on the attribute's value to display in the table, by adding `fn` to the field.

    { fields: [
        {
            key: 'resources',
            label: 'Number of Resources',
            fn: function (value, object) { return value.length; }
        }
    ] }

If the key exists in the record, it will be passed to `fn` in `value`. Otherwise, `value` will be `null`.

The `object` argument contains the full object, so you can compute a value using multiple fields.

By default, fields that use `fn` will be sorted by the result of this function. If you want to sort by the field's orignial value instead (for example, if you are making a date human-readable), set `sortByValue` to `true` on the field object.

##### HTML

You can use HTML in a virtual column by creating a Spacebars SafeString:

    fn: function (value) {
        return new Spacebars.SafeString('<a href="+Routes.route['view'].path({_id:value})+">View</a>');
    }

When adding user-generated fields to the HTML, ensure that they have been properly escaped to prevent cross-site scripting vulnerabilities.

#### Default sorting

You can use a column as the default sort order by adding `sort` to the field:

    { fields: [
        { key: 'year', label: 'Year', sort: 'descending' }
    ] }

It will accept any truthy value for ascending order, and `'desc'`, `'descending'` or `-1` for descending order.

#### Nested objects and arrays

For elements of nested objects and arrays, use mongo's syntax in the key:

    {'key': 'emails.0.address', label: 'Email Address'}

#### Hidden columns

To hide a column, add `hidden` to the field. It can be a boolean or a function.

    { key: 'location', label: 'Location', hidden: true }
    { key: 'location', label: 'Location', hidden: function () { return true; } }

If the `showColumnToggles` setting is `true`, hidden columns will be available in a dropdown and can be enabled by the user.

## Using events

Make the event selector be `tr`, and you'll have your row object in `this`:

```JavaScript
Template.posts.events({
  'click .reactive-table tr': function (event) {
    // set the blog post we'll display details and news for
    var post = this;
    Session.set('post', post);
  }
});
```

If you want single elements inside a row to become clickable, you still have to target `tr`. Otherwise `this` won't refer to the corresponding object of your targeted row. With this in mind, you have to specify a `target` inside your `'click .reactive-table tr'` eventlistener:

```JavaScript
Template.posts.events({
  'click .reactive-table tr': function (event) {
    event.preventDefault();
    var post = this;
    // checks if the actual clicked element has the class `delete`
    if (e.target.className == "delete") {
      Posts.remove(post._id)
    }
  }
});
```

## Internationalization

Internationalization support is provided using [anti:i18n](https://github.com/anticoders/meteor-i18n).

Add anti:i18n to your project:

    meteor add anti:i18n

To set your language to French:

    i18n.setLanguage('fr');

We currently have translations (except the 'Columns' button) for:

- Brazilian Portuguese (pt-br)
- Czech (cs)
- Dutch (nl)
- Finnish (fi)
- French (fr)
- German (de)
- Hebrew (he)
- Italian (it)
- Norwegian (no)
- Russian (ru)
- Slovak (sk)
- Spanish (es)
- Swedish (sv)
- Turkish (tr)
- Ukrainian (ua)

For other languages or the 'Columns' button, contribute a translation to [reactive_table_i18n.js](https://github.com/ecohealthalliance/reactive-table/blob/master/lib/reactive_table_i18n.js).
