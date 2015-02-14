_This README is optimistic. None of this exists yet._

# Basics

A CMS.

## Philosophy

Provide the basic building blocks to manage data, query and sort that data, and display it on a website.

The key components of the system are:

* Websites
* Collections
* Records
* Redirects
* Templates

### Websites

Yes, one can manage multiple websites. Each website has it’s own  collections, which in turn have their own records. A website has a hostname.

### Collections

A collection is a set of data elements (records) organized in into one of these variants:

* Table
* List
* Tree
* Calendar
* Map

All collection variants can be filtered by any of it’s record’s attributes. Every record in a collection automatically as an `id` attribute that is an auto-incrementing integer.

A table is the simplest variant which is most easily imagined to be an excel spreadsheet, each row being a record. A table’s schema can be locked or freeform. A table can be sorted by any of it’s record’s attributes. 

A list maintains it’s order by the record’s positions, like an array. The editor provides a UI to drag and drop records into the correct order. Every record in a list has a `position` attribute which is an integer.

A tree maintains it’s order along with a parent relationship. Every tree has only one root node which is parentless. The editor provides a UI to drag and drop records into the correct order and to nest them under a different parent. Moving a node with children also moves it’s children. Every record in a tree has `left`, `right`, and `parent_id` attributes.

A calendar maintains it’s order by the date or time associated with each record. A calendar can also optionally include an end date or time to create a time span for each record. The editor provides a calendar-like UI for editing and moving records to different dates and times. Every record in a calendar has either `start_at` and `end_at` or `start_on` and `end_on` attributes. `end_*` attributes might be empty.

A map maintain’s it’s positions by the global position associated with each record. A map-like UI is provided to manage the records. Every record in a map has `longitude`, `latitude`, and `address` attributes.

Every website has at least two collections: Pages and Global Variables.

#### Pages

Pages are a tree collection with a locked schema:

* Slug
* Title
* Template
* Variables
* Represents collection
* Format

The slug is the path part of the url. The full slug is the combination of this page’s slug with all of it’s ancestors slugs. An example could be a ‘company’ page who’s parent is an ‘about’ page. The company page’s slug would simply be ‘company’, but it’s fully slug would be `/about/company`.

The title is a string for the title tag and also is available in the template.

The template is chosen from the associated templates with the website. It is possible for a template to request variables, in which case they are automatically added to the page’s variables collection field.

The variables field is a table collection of free-form related records made available to the template. A variable record’s schema is simple: key and value. The key is a string and the value is any data-type known to the system.

A single page can represent a collection, in which case the page itself will have access to the collection as whole to possibly render an index and a “child page” will show up in the UI representing all the individual records in the collection. Enabling this would show these configurable options:

* Ordering
* Filtering
* Pagination

The child page is not a normal page, but instead has these configurable attributes:

* Title
* Template
* Variables

The title can be set using the attributes of the record itself if necessary. A single template is used for each record and will have access to that particular record’s attributes. And the variables collection is identical to the normal page’s one.

The format of a page defaults to html, but it can also be set to json. It is possible to have two pages with the same slug if one is html and the other is json.

#### Global variables

The global variables collection is a table collection with a schema identical to the variables collection mentioned above: key and value. The key is a string and the value is any data-type known to the system.

### Records

A record is like a row in an excel spreadsheet. Each column in the row is called a field. Record’s fields have types. Every record has an associated set of fields. It’s possible to have a pre-made field set that the record will adhere to, or to have a free-form set of fields.

All fields can be required or optional. Each field type has it’s own set of validations and configurable options. An example is the cropping size for image uploads.

The basic field types are:

* String
* Text
* Boolean
* Choose one
* Choose any
* Select
* Image
* Video
* Audio
* Date
* Time
* Location
* Matrix
* Any

An address might require a street string field, city string field, state string field, zip code string field, and have an optional location field. This would constitute an address field set.

The matrix field allows one to embed a field set into a field. This means that a collection of buildings might all have an address field, which itself is a collection of fields like tree, city, etc.

A collection locks it’s record’s schemas by choosing a field set to require it’s records to adhere to.

The global variables collection’s value field is set to ‘any’ which means the admin user can choose the type of field separately for each record.

Combining the any field with the matrix field means complex field types can be created at runtime with no programming. A matrix field can even contain other matrix fields. :mindblown:

### Redirects

Every website eventually needs to setup redirects. A handy UI is provided for setting redirects to pages or to external websites.

### Templates

A template is a .ejs file. EJS allows the template editor to write real code in javascript inside their html or json files. Templates are stored in the database. They can be edited as files, but must always be imported into the database before the changes will be reflected on the website. There are file editing synchronization hooks for github, dropbox, and S3 available.

Inside a template are many built in functions, but the most important are:

* `collection`
* `get`
* `partial`

The `collection` function gives access to any collection by it’s name. An example:

```ejs
<h1>Names</h1>
<ul>
	<% collection("names").each(function(name) { %>
    <li><%= name %>
  <% } %>
</ul>
```

When a variable is set globally or on a page, it can be gotten with `get`:

```ejs
<h1>About <%= get("company_name") %>
```

And `partial` is to include one template into another.

## Setup

Deploy to heroku with the [heroku button] below. There is also a docker container. Get instructions for [how to use dokku with basics] or use the docker container on your own infrastructure.

The requirements are:

* postgres 9.4
* ruby 2.2.0
* docker [some version]

Optional features can be enabled with:

* Caching: memcached & MEMCACHE_URL
* Background workers: redis & REDIS_URL
* Search: elastic search & ELASTIC_SEARCH_URL

When basics boots it will check for the optional features to be enabled and perform any necessary setup for them. The caching is robust and should prevent and fireballing you might receive. The background workers are for sending emails, making image previews, and other intense tasks that should be offloaded to a worker. And search is easy to setup and an admin UI is enabled to help guide the admin through the configuration.

