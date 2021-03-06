## Use with Hosted V1

### IIS Deploy

To let IIS serve the files for you:

1. Clone this repository or download the files as a zip and place the contents of the `VersionOne.FeatureRequestor` 
folder into a directory, such as `C:\inetpub\wwwroot\v1requestor`.
2. In IIS, from the `Connetions` panel, open `Sites` and select or create a site.
3. Right click on the site and select `Add Application` or `Add Virtual Directory`.
4. Enter `v1requestor` for `Alias`, and for `Physical Path` put the directory you used in step 1.
5. Click `Ok`.
6. Browse to the new site. If you placed it directly into the default site, the address should be 
[`http://localhost/v1requestor`](http://localhost/v1requestor).
7. See the `How to configure for your VersionOne instance and projects` below.

### File Share Deploy

While it's probably better to install on a web server, you can actually run the Feature Requestor from a file share, but 
you have to enable a special flag in Google Chrome to do so. But, it appears this feature of Chrome was "rushed", so if 
you really want to do it, [read about the `--allow-file-access-from-files` Chrome option]
(http://stackoverflow.com/questions/4270999/google-chrome-allow-file-access-from-files-disabled-for-chrome-beta-8).

## Configure for V1 Projects

There are two configuration files:

1. config.js (or config.coffee) -- specifies the URL for VersionOne and a few other options
2. fields.js (or fields.coffee) -- specifies the projects and fields you want to display on the request form for each 
project

## config.js

Most importantly, change the `host`, `service`, and `versionOneAuth` variables to point to your own VersionOne 
instance. By default, they point to the VersionOne test intsance.

### host

*Url, default: http://eval.versionone.net*

The web server address where your VersionOne instance is locaated, most likely something like `http://www7.v1host.com` 
or `http://www11.v1host.com`.

### service

*Url, default: http://eval.versionone.net/platformtest/rest-1.v1/Data/*

The complete url for the Versionone REST API endpoint for your instance, ending with a `/`. If you log in to your 
instance at `http://www11.v1host.com/TeamAwesome`, then your REST API endpoint url is 
`http://www11.v1host.com/TeamAwesome/rest-1.v1/Data/`.

### versionOneAuth

*String, default: admin:admin*

Authentication credentials for a user that can submit a feature request into the projects you specify in `fields.js`. 
You should take care to give this user only the permissions you want, perhaps only to add requests for those projects. 

This must be in the form of `username:password`. This value gets [Base64-encoded]
(http://en.wikipedia.org/wiki/Base64) and sent as an HTTP `Authorization` header.

*Note:* we have some code for an alternative way of authenticating, but we're not finished with it. If you're interested 
in that, let us know.

### projectListClickTarget

*String, default: new*

This controls what happens when a user clicks a project name after searching

Valid values are:

* `new` -- open a new blank request form
* `list` -- open the list of existing requests to filter and select

### others

Modify the others to your heart's content.

## fields.js

The fields.js file is where specifies the fields that will be visiible for all projects or for specific projects when 
adding or editing a request.

### Settings

TODO: add info


### How to specify default fields

To specify which fields to show up for all projects by default, define the a setting named `default`, like this:

```javascript
"default": {
  RequestedBy: {
    title: 'Requested By',
    autofocus: true
  },
  Name: {
    title: 'Request Title'
  },
  Description: {
    title: 'Request Description (Project & Why needed)',
    type: 'TextArea',
    optional: true
  },
  Priority: {
    title: 'Priority',
    type: 'Select',
    assetName: 'RequestPriority'
  }
}
```

### How to specify fields for specific projects

For a sepcific project, you define fields with a key named after the project's Scope oid, like below. Note that this 
even lets you even use custom fields that are defined in your VersionOne instance. The `type` parameter refers to the 
field types available in [Backbone Forms](https://github.com/powmedia/backbone-forms).

```javascript
'Scope:173519': {
  RequestedBy: {
    title: 'Requested By',
    autofocus: true
  },
  Name: {
    title: 'Request Title'
  },
  Custom_RequestedETA: {
    title: 'Requested by (ETA)',
    type: 'Date'
  },
  Description: {
    title: 'Request Description (Project & Why needed)',
    type: 'TextArea',
    optional: true
  },
  Custom_ProductService: {
    title: 'Product/Service',
    type: 'Select',
    assetName: 'Custom_Product'
  },
  Custom_Team2: {
    title: 'Team',
    type: 'Select',
    assetName: 'Custom_Team'
  },
  Custom_HWRequestedlistandcostperunit: {
    title: 'Capacity or HW Requested',
    type: 'TextArea'
  },
  Custom_RequestImpact: {
    title: 'Request Impact',
    type: 'Select',
    assetName: 'Custom_Severity'
  }
}
```

# Configure with Service Gateway

TODO: below is outdated

Did you see those credentials embedded in JavaScript above? Yes, that could suck. 
We're looking at better ways to enable this to work from the web browser, but we also have a way to proxy the request 
through a "service gateway", but we don't have instructions for that yet. You can see a C# version and a Node.js 
version in the source code of the project, however. Contact us if you would like to use these features.

# Advanced: CoffeeScript

The main source for the app is actually CoffeeScript. It's been compiled to JavaScript, and those files are here in the 
repository, but if you'd prefer to customize the code in CoffeeScript rather than muck with JavaScript, then do this:

1. Install [Node.js](http://nodejs.org/) if you don't already have it.
2. Open a command prompt and change directory to where the `VersionOne.FeatureRequestor` folder is in your local repository clone.
3. Type `npm install coffee-script` (Or, [see alternatives for installing CoffeeScript](http://coffeescript.org/#installation)).
4. Type `./make.sh` to execute the CoffeeScript compiler. This is a simple script that regenerates a few `.js` files 
from the `.coffee` files in the project.
