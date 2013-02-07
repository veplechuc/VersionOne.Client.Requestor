# TODO: refactor everything below!!! or remove

# RequireJS: loading modules

RequireJS provides support for AMD (Asynchronous Module Definition) loading. [Learn details here](http://requirejs.org/docs/whyamd.html).

## index.html: `data-main='scripts/main'` specifies the "entry point" for the application

This is how we bootstrap the modules with RequireJS:

* Include `scripts/require.js`
* Specify the entry point using HTML5 `data-` attribute `data-main='scripts/main'`

RequireJS automatically creates `main.js` from `main`.

```html
<html>
<head>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="Content/jquery.mobile-1.2.0.min.css" type="text/css" />
    <link rel="stylesheet" href="Content/jquery.mobile.v1.css" type="text/css" />   
    <link rel="stylesheet" href="Content/v1assetEditor.css" />
    <link rel="stylesheet" href="Content/toastr.css" />
    <link href="scripts/templates/bootstrap.css" rel="stylesheet" />
    <script src='scripts/require.js' data-main='scripts/main' type='text/javascript'></script>
</head>
```

## scripts/main.js: `require(...)` and `shim:` loading AMD friendly and AMD-challenged modules as one happy family

Inside this script:

* `requirejs.config` and `shim` coerces ordinary scripts to be good modular AMD citizens -- [See docs](http://requirejs.org/docs/api.html#config-shim).
* `require([...], function() )` defines the required modules and calls back when loaded
* Finally, wire up a jQuery document ready handler with `$()` and instantiate an instance of `v1AssetEditor`

### Notes
RequireJS will load the list of modules passed to `require()`, and then pass each one into the function parameter as a single object conforming to the [AMD pattern](https://github.com/amdjs/amdjs-api/wiki/AMD). Note that I've only declared formal parameters for the first three, because I only need to reference them. However, and I'm not sure why, I did have to load these modules via main, rather than as requirements for `v1AssetEditor`, where they are really needed. I don't understand why.

### `scripts/main.js` full text:

```javascript
requirejs.config({
  // The shim allows these non-AMD scripts to participate
  // in the AMDified loading for other modules
    shim: {
    	'underscore': {
    		exports: '_'
    	}
        ,'backbone': {
            deps: ['underscore', 'jquery'],
            exports: 'Backbone'
        }
        ,'jsrender' : {
        	deps: ['jquery']
        }
    }
});

require([
        '../config',
        'v1assetEditor',
        'jquery',
        'backbone',
        'backbone-forms',
        'editors/list',
        'templates/bootstrap', 
    	'toastr',
        'jsrender'
    ],
    function(
        v1config,
        v1assetEditor,
        $)
    {
    	$(document).ready(function () {
    	    var editor = new v1assetEditor(v1config);
    	});
    }
);
```

# `config.coffee`: simplified Backbone Forms configuration

The first module passed to the initialization function in `scripts/main` was `../config`. 

This module sets some host-specific URLs and also takes in the `fields` module that we discussed above. It does some clean up of this so that when Backbone Forms processes it has all the attributes it needs. For example, `optional: true` will *prevent* the additional of `field.validators = ['required']`

### `config.coffee` full text:

```coffeescript
define ['./fields'], (fields) ->
  showDebugMessages = true
   
  host = 'http://localhost';
  service = 'http://localhost/VersionOne.Web/rest-1.v1/Data/';
  versionOneAuth = 'admin:admin';

  serviceGateway = false
  
  #var serviceGateway = 'http://localhost/v1requestor/Setup';

  projectListClickTarget = 'new'
  #projectListClickTarget = 'list'

  configureFields = (obj) ->
    for fieldGroupName, fieldGroup of obj
      for fieldName, field of fieldGroup
        if field.type == 'Select'
          field.options = [] # Ajax will fill 'em in
          field.editorAttrs =
            'data-class': 'sel'
            'data-assetName': field.assetName
            'data-rel': fieldName
        else
          if field.optional == true          
          else
            field.validators = ['required']
        if field.type == 'TextArea'
          field.editorAttrs =
            style: 'height:200px'
        if field.autofocus == true
          if not field.editorAttrs
            field.editorAttrs = {}
          field.editorAttrs.autofocus = 'autofocus'
        # Delete properties, if they exist, from field
        delete field.autofocus
        delete field.optional

  configureFields fields

  assetName = "Request"
  
  options =
    showDebug: showDebugMessages
    host: host
    service: service
    serviceGateway: serviceGateway
    versionOneAuth: versionOneAuth
    assetName: assetName
    formFields: fields
    projectListClickTarget: projectListClickTarget

  return options
```

# Highlights from `v1AssetEditor`

I'm not going to claim that `v1AssetEditor.coffee` is a perfect, modern example of MV* style development. Far from it!

As I listed above, there are a number of areas I think this can be improved, especially to take advantage of Backbone's features.

That being said, I'll highlight how jQuery Mobile, the fields config, Backbone Forms, and the VersionOne JSON support work together to simplify creating and editing a VersionOne Request.

And, leaving it at that, I'll ask for input and advice on how to evolve this into something more reusable for us as a team, and by extension, to external developers building on top of the VersionOne Platform and API.

## `index.html`: detail div (page) HTML

While it's not necessary to really understand jQuery Mobile in depth to grok the rest of this article, 
[you can view jQuery Mobile docs here](http://www.jquerymobile.com) to get a better understanding of how flexible and powerful it is.

Conceptually, jQuery Mobile allows you to define a desktop-friendly, mobile-optimized HTML5 ready page using standard HTML elements plus additional HTMl5 `data-` attributes that, on load, it scans for and use to configure and `enhance` the page.

You can create multiple "pages" by using `data-role='page'` on a simple `<div>` tag. And, more sophisticatedly, you can [dynamically creates pages and elements at run-time](http://jquerymobile.com/demos/1.2.0/docs/pages/page-dynamic.html).

Key highlights from below:

* `data-role="page"` sets up a div as a "page" that can be navigated to via hash tag, like `index.html#detail`
* Other `data-` attributes configure various other aspects of jQueryMobile to enhance the standard HTML elements into desktop & mobile friendly presentations
* The `<form>` and its contained `<div id="fields"></div>` element are where the form fields will get injected by Backbone Forms
* Finally, the two buttonns in the footer get wired up to event handlers inside of the `v1AssetEditor` class

```html
<div data-role="page" id="detail" data-theme="b"> 
    <div data-role="header" style="text-align:center;padding-top:5px" data-theme="b">
        <div>&nbsp;
            <img style="background:white;width:120px;padding:2px;margin-left:6px;" src="images/logo.png" border="0" align="left" /><span class="ui-block-b" style="font-size:120%;">&nbsp;&nbsp;Requestor App <span class='title'></span></span>
        </div>
        <div>&nbsp;
            <fieldset class="ui-grid-a">
                <div class="ui-block-a"><a href="#list" data-role="button" data-icon="arrow-u">List</a></div>
                <div class="ui-block-b"><a href="#" data-role="button" class="new" data-icon="plus">New</a></div>
            </fieldset>
        </div>
    </div>

    <div data-role="content" class="sideGradient">      
        <div id="assetFormDiv">
            <form id="assetForm">
                <div id="fields"></div>
            </form> 
        </div>
        <br />            
    </div>

    <div data-role="footer" data-theme="b" data-position="fixed">
        <fieldset class="ui-grid-a">
            <div class="ui-block-a">
                <a href="#" data-role="button" id="saveAndNew" data-icon="plus" data-mini="true">Save and New</a>
            </div>            
            <div class="ui-block-b">
                <a href="#" data-role="button" id="save" data-icon="check" data-mini="true">Save</a>
            </div>
        </fieldset>
    </div>    
</div>​
```

### Additional jQuery Mobile references

* [Backbone.js, RequireJS and jQuery Mobile](http://jquerymobile.com/test/docs/pages/backbone-require.html) -- not part of the 1.2.0 release, part of the upcoming release
* [PhoneGap apps with jQuery Mobile](http://jquerymobile.com/demos/1.2.0/docs/pages/phonegap.html)

## `v1AssetEditor.coffee`: constructor and initialize

### Summary
The `constructor` and `initialize` methods take care of setting up initial event handlers, sourcing config data from a service gateway (if configured), and subscribing to various jQuery Mobile events and a couple of custom events that are produced by Backbone.Events.

### Details
* `constructor` creates some property bags and also "mixes in" the options supplied from the caller. [See the MixIn pattern](http://addyosmani.com/resources/essentialjsdesignpatterns/book/#mixinpatternjavascript)
* If the `options` hash contained a `serviceGateway` property with a value != false, then it will attempt to source the credentials from that URL, rather than rely on hard-coded credentials. This is by no means a sophisticated or highly secure way to do authentication, but it allows the externally hosted code to use the HTTP API without needing to be served from the same web-server.
* `initialize` does some basic jQuery click handler wireup for button handlers (TODO: implement a simple convention based wireup like Caliburn.Micro, and probably Durandal)
* The `pageinit` and `pagebeforeshow` events are [jQuery Mobile Events](http://jquerymobile.com/demos/1.2.0/docs/api/events.html) that respond to those virtual events for the given pages.
* The `@on "assetCreated", (that, asset) -> ...` and other one utilize [Backbone.Events](http://backbonejs.org/#Events) for simple, custom "publish/subcribe" type notification. In this case, the subscribing code will modify the List page when a Request object is created or modified. We'll see later how these events are published in the ajax callback handlers.
* **Note**: at the very end of the script is ` _.extend VersionOneAssetEditor::, Backbone.Events`. This uses underscore.js's extension function to mixin Backbone.Events into the VersionOneAssetEditor class. I suppose I could have used that for my hand-rolled mixin above.

```coffeescript
define ["backbone", "underscore", "toastr", "jquery", "jquery.mobile", "jsrender"], (Backbone, _, toastr, $) ->
  
  # logging, etc omitted
  
  class VersionOneAssetEditor
    constructor: (options) ->          
      continueSettingOptions = =>
        options.whereParamsForProjectScope =
          acceptFormat: contentType
          sel: ""
        options.queryOpts = acceptFormat: contentType
        options.contentType = contentType      
        for key of options
          @[key] = options[key]      
        @initialize()
      
      contentType = "haljson"
      debugEnabled = options.showDebug
      
      if options.serviceGateway
        $.ajax(options.serviceGateway).done((data) ->
          options.headers = data
          continueSettingOptions()
        ).fail (ex) ->
          error "Failed to get setup data. The URL used was: " + options.serviceGateway
          log ex
      else
        options.headers = Authorization: "Basic " + btoa(options.versionOneAuth)
        continueSettingOptions()
        
    initialize: ->
      @requestorName = ""
      @refreshFieldSet "default"
      
      $(".new").click =>
        @newAsset()

      selectFields = []
      @enumFields (key, field) ->        
        # TODO: hard-coded
        selectFields.push key if key isnt "Description" and field.sel isnt false

      @selectFields = selectFields
      $("#list").live "pageinit", @configureListPage

      $("#list").live "pagebeforeshow", =>
        @listFetchIfNotLoaded()
      
      $("#projectsPage").live "pagebeforeshow", =>
        @setTitle "Projects"

      @on "assetCreated", (that, asset) ->
        success "New item created"
        that._normalizeIdWithoutMoment asset
        that._normalizeHrefWithoutMoment asset
        that.listItemPrepend asset

      @on "assetUpdated", (that, asset) ->
        success "Save successful"
        that._normalizeIdWithoutMoment asset
        that._normalizeHrefWithoutMoment asset
        that.listItemReplace asset

      @configureProjectSearch()
      @toggleNewOrEdit "new"
```


##
```coffeescript
    refreshFormModel: ->
      @assetFormModel = Backbone.Model.extend({schema: @getFormFields()})

    listFetchIfNotLoaded: ->
      @loadRequests() unless @_listLoaded

    configureProjectSearch: ->
      searchTerm = null
      ajaxRequest = null
      projectSearch = $("#projectSearch")
      projectSearch.pressEnter (e) =>
        target = $(e.currentTarget)
        return if searchTerm is target.val()
        searchTerm = target.val()
        return if searchTerm.length < 4
        $.mobile.loading('show')
        ajaxRequest.abort() if ajaxRequest
        assetName = "Scope"
        url = @getAssetUrl(assetName) + "&" + $.param(
          sel: "Name"
          page: "100,0"
          find: "'" + searchTerm + "'"
          findin: "Name"
        )
        request = @createRequest(url: url)
        projects = $("#projects")
        ajaxRequest = $.ajax(request).done((data) =>
          ajaxRequest = null
          projects = $("#projects").empty()
          for val in data
            @projectItemAppend val
          projects.listview "refresh"
          $.mobile.loading('hide')
        ).fail(@_ajaxFail)

    configureListPage: ->
      assets = $("#assets")
      assets.empty()
      assets.listview()
      @_listLoaded = false

    setProject: (projectIdref) ->
      @projectIdref = projectIdref
      @refreshFieldSet projectIdref
      #this.configureListPage();
      @_listLoaded = false
```
# jQuery Promises for deferred loading
# VersionOne API with JSON examples

``` 

    loadRequests: (projectIdref) ->
      @_listLoaded = true
      if not projectIdref?
        projectIdref = @projectIdref
      else
        @setProject projectIdref
      url = @getAssetUrl(@assetName) + "&" + $.param(
        where: "Scope='" + projectIdref + "'"
        sel: "Name,RequestedBy"
        page: "75,0" # TODO: temporary... until real paging support
        sort: "-ChangeDate"
      )
      request = @createRequest(url: url)
      assets = $("#assets")
      assets.empty()
      $.ajax(request).done((data) =>
        info "Found " + data.length + " requests"
        for item, i in data
          @listAppend item
        assets.listview "refresh"
      ).fail @_ajaxFail
      @changePage "#list"

    refreshFieldSet: (fieldSetName) ->
      if @formFields[fieldSetName]?
        @fieldSetName = fieldSetName
      else
        @fieldSetName = "default"
      @refreshFormModel()

    getFormFields: ->
      return @formFields[@fieldSetName]

    getFormFieldsForSelectQuery: ->
      fields = []
      @enumFields (key) ->
        fields.push key
      fields = fields.join(",")
      fieldsClause = $.param(sel: fields)
      return fieldsClause

    listAppend: (item) ->
      assets = $("#assets")
      templ = @listItemFormat(item)
      assets.append templ

    listItemFormat: (item) ->
      templ = $("<li></li>")
      templ.html $("#assetItemTemplate").render(item)
      templ.children(".assetItem").bind "click", (e) =>        
        target = $(e.currentTarget)
        href = target.attr("data-href")
        @editAsset target.attr("data-href")
      return templ

    projectItemAppend: (item) ->
      projects = $("#projects")
      templ = @projectItemFormat(item)
      projects.append templ

    projectItemFormat: (item) ->
      templ = $("<li></li>")
      templ.html $("#projectItemTemplate").render(item)
      templ.children(".projectItem").bind "click", (e) =>
        target = $(e.currentTarget)        
        idref = target.attr("data-idref")
        name = target.attr("data-name")
        @setTitle name
        @setProject idref
        if @projectListClickTarget is "new"
          @newAsset()
        else
          @loadRequests()
      return templ

    setTitle: (title) ->
      $(".title").text ": " + title

    listItemPrepend: (item) ->
      templ = @listItemFormat(item)
      assets = $("#assets")
      assets.prepend templ
      assets.listview "refresh"

    _normalizeIdWithoutMoment: (item) ->
      id = item._links.self.id
      id = id.split(":")
      id.pop() if id.length is 3
      id = id.join(":")
      item._links.self.id = id

    _normalizeHrefWithoutMoment: (item) ->
      href = item._links.self.href
      if href.match(/\D\/\d*?\/\d*$/)
        href = href.split("/")
        href.pop()
        href = href.join("/")
        item._links.self.href = href

    listItemReplace: (item) ->    
      # Thanks to Moments:
      id = item._links.self.id
      templ = @listItemFormat(item)
      assets = $("#assets")
      assets.find("a[data-assetid='" + id + "']").each ->
        listItem = $(this)      
        #var newItem = @listItemFormat(item);
        listItem.closest("li").replaceWith templ
      assets.listview "refresh"
```
newAsset

```coffeescript
    newAsset: (modelData, href) ->
      @configSelectLists().done =>
        asset = null
        if modelData?        
          asset = new @assetFormModel(modelData)
        else
          asset = new @assetFormModel()
        @asset = asset
        form = new Backbone.Form(model: asset).render()
        @form = form
        $("#fields").html form.el
        if modelData?
          @toggleNewOrEdit "edit", href
        else
          @toggleNewOrEdit "new"
        @changePage "#detail"
        @resetForm() unless modelData
        $("#detail").trigger "create"
```
# How Backbone Forms and its Schema / Model approach works
# Relationships fetching

```coffeescript
    configSelectLists: ->    
      # Setup the data within select lists
      # TODO: this should not happen on EVERY new click.
      promise = new $.Deferred()
      ajaxRequests = []
      model = new @assetFormModel().schema
      selectLists = []
      for key, value of model
        selectLists.push value if value.options.length < 1 if value.type is "Select"

      for field in selectLists
        assetName = field.editorAttrs["data-assetName"]
        fields = field.formFields
        fields = ["Name"] if not fields? or fields.length is 0
        url = @service + assetName + "?" + $.param(@queryOpts) + "&" + $.param(
          sel: fields.join(",")
          sort: "Order"
        )
        request = @createRequest(
          url: url
          type: "GET"
        )
        ajaxRequest = $.ajax(request).done((data) =>
          if data.length > 0
            for option in data
              field.options.push
                val: option._links.self.id
                label: option.Name
          else
            @debug "No results for query: " + url
        ).fail(@_ajaxFail)
        ajaxRequests.push ajaxRequest

      return promise if @resolveIfEmpty(promise, ajaxRequests)
      
      @resolveWhenAllDone promise, ajaxRequests
      
      return promise

    resolveIfEmpty: (promise, ajaxRequests) ->
      if ajaxRequests.length < 1
        promise.resolve()
        return true
      return false

    resolveWhenAllDone: (promise, ajaxRequests) ->
      $.when.apply($, ajaxRequests).then ->
        promise.resolve()
```

Console-play with the forms dynamically
editAsset

```coffeescript
    editAsset: (href) ->
      fields = @getFormFields()
      
      url = @host + href + "?" + $.param(@queryOpts)
      fieldsClause = @getFormFieldsForSelectQuery()
      url += "&" + fieldsClause

      asset = @createFetchModel url

      asset.exec().done(=>
        modelData = {}
        model = new @assetFormModel().schema
        links = asset.get('_links')
        for key of model
          value = asset.get(key)
          if value?            
            if fields[key].type == 'Date'
              value = new Date(Date.parse(value))
            modelData[key] = value
          else
            rel = links[key]
            continue if not rel?
            val = links[key]
            if val? and val.length > 0
              val = val[0]
              id = val.idref
              modelData[key] = id
        @newAsset modelData, href
      ).fail @_ajaxFail

    toggleNewOrEdit: (type, href) ->
      save = $("#save")
      saveAndNew = $("#saveAndNew")
      if type is "new"
        save.unbind "click"
        save.bind "click", (evt) =>
          evt.preventDefault()
          @createAsset @assetName, (asset) =>          
            # refresh
            @editAsset asset._links.self.href
        saveAndNew.unbind "click"
        saveAndNew.bind "click", (evt) =>
          evt.preventDefault()
          @createAsset @assetName, =>
            @newAsset()          
            # Hardcoded:
            $("#Name").focus()
      else if type is "edit"
        save.unbind "click"
        save.bind "click", (e) =>
          e.preventDefault()
          @updateAsset href
        saveAndNew.unbind "click"
        saveAndNew.bind "click", (e) =>
          e.preventDefault()
          @updateAsset href, =>
            @newAsset()

    createRequest: (options) ->
      options.headers = @headers  unless @serviceGateway
      return options

    createFetchModel: (url) ->
      options = {}
      options.headers = @headers unless @serviceGateway
      props = 
        url: url
        exec: ->
          return @fetch(options)

      fetchModel = Backbone.Model.extend(props)

      return new fetchModel()

    createAsset: (assetName, callback) ->
      url = @getAssetUrl(assetName)
      @saveAsset url, "assetCreated", callback

    updateAsset: (href, callback) ->
      url = @host + href + "?" + $.param(@queryOpts)
      @saveAsset url, "assetUpdated", callback

    saveAsset: (url, eventType, callback) ->
      validations = @form.validate()
      if validations?
        error "Please review the form for errors", null,
          positionClass: "toast-bottom-right"
        return
      dto = @createDto()
      debug "Dto:"
      debug dto
      request = @createRequest(
        url: url
        type: "POST"
        data: JSON.stringify(dto)
        contentType: @contentType
      )
      $.ajax(request).done((data) =>
        @trigger eventType, @, data
        callback data if callback
      ).fail @_ajaxFail

    createDto: ->
      dto = @form.getValue()

      # TODO: hard-coded for test
      dto._links = Scope:
        idref: @projectIdref

      $("#fields select").each ->
        el = $(this)
        id = el.attr("name")
        val = el.val()
        relationAssetName = el.attr("data-rel")
        if relationAssetName
          dto._links[relationAssetName] = idref: val
          delete dto[id]
      return dto

    getAssetUrl: (assetName) ->
      url = @service + assetName + "?" + $.param(@queryOpts)
      return url

    changePage: (page) ->
      $.mobile.changePage page

    resetForm: ->
      @enumFields (key, field) ->
        $("[name='" + key + "']").each ->
          unless field.type is "select"
            $(this).val ""
            $(this).textinput()    
      '''
      sel = $("[name='Priority']")
      sel.selectmenu()
      sel.val "RequestPriority:167"
      sel.selectmenu "refresh"
      '''

    enumFields: (callback) ->
      for key, field of @getFormFields()
        callback key, field

    findField: (fieldName) ->
      fields = [null]
      index = 0
      addField = (key, field) ->
        fields[index++] = field if key is fieldName
      @enumFields addField
      return fields[0]

  debugEnabled = false
  _.extend VersionOneAssetEditor::, Backbone.Events

  return VersionOneAssetEditor
```
