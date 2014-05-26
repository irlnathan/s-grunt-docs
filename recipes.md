##How do I integrate Twitter Bootstrap and jQuery in the Asset pipeline?

First, create a sails project using:

```sh
 $ sails new jQueryBootstrap
 ```

Sails generates a new project with the following folder structure:

```javascript
jQueryBootstrap
       |_api
       |_assets
       |_config
       |_node_modules
       |_tasks
       |_views                
.gitignore
.sailsrc
app.js
Gruntfile.js
package.json
README.md
                
```

Place the `bootstrap.css` file in the `styles` folder (e.g. `twitterBootstrap/assets/styles`).  Place `jquery.js` and any necessary bootstrap javascript files in the `js` folder (e.g. `twitterBootstrap/assets/js/dependencies`).
>**Note:**  Sails-linker will check all sub-folders under the `styles/` folder for `.css` files and the `js/` folder for javascript files.

Next, open up `homepage.ejs` located in the `views` folder (e.g. `twitterBootstrap/views).  Replace all of the contents of the file with the following mark-up:

```html
<div class="container">
  <div class="jumbotron" style="text-align: center">
    <h1>yourApp</h1>

    <h2>yourApp's will change the face of web applications!</h2>
    <a href="/user/new" class="btn btn-lg btn-success">Sign up now!</a>
  </div>
</div>
```

Now, start sails using `sails lift`:

```sh
 $ sails lift
 ```

Open up a browser and go to: `http://localhost:1337`.

Any view you create in Sails will have access to both jQuery and Twitter Bootstrap through Sails the `layout.ejs` file located in the `views` folder (e.g. `twitterBootstrap/views`).  The `layout.ejs` file contains a couple of special tags that Sails recognizes and injects links to your javascript and styles.

> For a more detailed description of what's going on see [Everything you ever wanted to know about Sails Grunt Integration]().

##What if I have additional css files that I want to load after bootstrap.css is loaded?

Using the previous example as a start, open up `pipeline.js` located in the `/jQueryBoostrap/tasks` folder.  `pipeline.js` configures the order in which javascript and css files are injected into Sails views.

So in this example, I want boostrap.css to load before any other javascript files are loaded.  Here's what `pipeline.js` looks like by default:

```javascript
...
var cssFilesToInject = [
	'styles/**/*.css'
];

var jsFilesToInject = [

	// Dependencies like sails.io.js, jQuery, or Angular
	// are brought in here
	'js/dependencies/**/*.js',

	// All of the rest of your client-side js files
	// will be injected here in no particular order.
	'js/**/*.js'
];


var templateFilesToInject = [
	'templates/**/*.html'
];
...
```

I'm going to add to the array in `pipeline.js` to include `bootstrap.css` first.  

I'll change the file from this...

```javascript
...
var cssFilesToInject = [
	'styles/**/*.css'
];
...
```

...to this...

```javascript
...
var cssFilesToInject = [
	'styles/boostrap.css',
	'styles/**/*.css'
];
...
```
All `.css` files will now be loaded after `boostrap.css`.


##I want to use AngularJS with Sails, how do I set-up file dependencies and the intial "bootstrap" page?

First, create a sails project using:

```sh
 $ sails new angularExample
 ```

Sails generates a new project with the following folder structure:

```javascript
angularExample
       |_api
       |_assets
       |_config
       |_node_modules
       |_tasks
       |_views                
.gitignore
.sailsrc
app.js
Gruntfile.js
package.json
README.md
                
```

Place `angular.js` in the `depdendencies` folder (e.g. `angularExample/assets/js/dependencies`).  You have several options for your initial SPA "bootstrap" page.  

###Initial SPA "boostrap" page: as an index.html file in the `assets/` folder

Create a new `index.html` file in the `assets` folder of the project (e.g. `angularExample/assets`).  For example:

```html
<html ng-app>
	<head>
		<title></title>
	</head>
	<body>
		<h1>My Angular App</h1>
		<!--SCRIPTS-->
		<!--SCRIPTS END-->
	</body>
</html>
```

Open up the `routes.js` file located in the `config` folder (e.g. `angularExample/config`).  Currently, the default route points to the default `homepage.ejs` file in the `views` folder.  Comment out the route so `config.js` looks like this:

```javascript
...
// '/': {
//   view: 'homepage'
// },
...
```

Now, start sails using `sails lift`:

```sh
 $ sails lift
 ```

Open up a browser and go to: `http://localhost:1337`.

Sails first looks to the `routes.js` file for guidance on what to load in the default route.  Next, Sails looks to the `aasets/` folder for an `index.html` file.  Therefore, the new `index.html` file is loaded with Angular, ready for you to develop your SPA.

###Initial SPA "boostrap" page: as a server-side view in the `views/` folder

Make sure the default route in `routes.js` file located in the `config` folder (e.g. `angularExample/config`)is uncommented and looks like this: 

```javascript
...
 '/': {
   view: 'homepage'
 },
...
```

You can put the `ng-app` directory in your `layout.ejs` file located in the `views` folder (e.g. `angularExample/views`) to be applied to all server-rendered views or place it in a specific view (e.g. homepage.ejs) if you want to limit the app to a particular view.

