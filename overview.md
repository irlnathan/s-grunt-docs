##What is Grunt?

Grunt is all about automating tasks like: 
- compressing your CSS and minimize your Javascript
- concatenating your CSS and Javascript files
- assuring links to your CSS and Javascript files are up to date.  

These are **tasks** that should be automated and that's exactly where Grunt excels.  

### No slight-of-hand
**Rule #1:** Sails integration with Grunt requires **no** _proprietary_ code.  All of the tasks provided by Sails can be run from the command line with Grunt "as-is".  To prove this and get you familiar with where things are located, let's do a quick example.

Create a sails project using:

```sh
 $ sails new gruntExample
 ```

 This generates a new project with the following folder structure:

```javascript
gruntExample
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

For this example, we'll be concentrating on the `tasks` folder:

```javascript
gruntExample
       ...
       |_tasks
         |_config
         |   |_clean.js
         |   |_coffee.js
         |   |_concat.js
         |   |_copy.js
         |   |_cssmin.js
         |   |_jst.js
         |   |_less.js
         |   |_sails-linker.js
         |   |_sync.js
         |   |_uglify.js
         |   |_watch.js
         |_register
             |_build.js
             |_buildProd.js
             |_compileAssets.js
             |_default.js
             |_linkAssets.js
             |_linkAssetsBuild.js
             |_linkAssetsBuildProd.js
             |_prod.js
             |_syncAssets.js
           pipeline.js
           README.md
           ...
```  
Typical Grunt usage involves three basic steps: 1) **Installing** the necessary _plugins_, 2) **configure** the **_task_** that utilizes the _plugin_, and finally 3) **execute** the task.  

<img src="http://i.imgur.com/dqZUcLM.jpg" />

### Installing the plugin

Sails provides a number of grunt plugins out-of-the-box including:

- grunt-contrib-clean
- grunt-contrib-coffee
- grunt-contrib-concat
- grunt-contrib-copy
- grunt-contrib-cssmin
- grunt-contrib-jst
- grunt-contrib-less
- grunt-contrib-uglify
- grunt-contrib-watch
- grunt-contrib-sails-linker
- grunt-contrib-sync

Each of these plugins does the heavy lifting for you and I'm convinced if you were to do a google search for "grunt 'solve world peace' plugin", you'd probably find several, each with different configuration options. 
>By the way, they install just like any other module via npm: `npm install [plugin name]`.

### Configuring a **task** for a _plugin_

So I'll open up `clean.js` from the `gruntExample/tasks/config` folder:

<img src="http://i.imgur.com/o87LDKw.jpg?1" />

The first part of this file **configures** the Grunt _plugin_.  

<img src="http://i.imgur.com/ywj3FOf.jpg" />

The second part of this file **loads** the tasks and the Grunt _plugin_ itself. 

<img src="http://i.imgur.com/NuvTxUF.jpg" />

### Executing a Grunt _task_

Although Sails runs some Grunt _tasks_ automatically, I can execute the `clean` _task_ from the command-line just like any other Grunt _task_ made without Sails.

So I'll start sails using sails lift just so we'll have something to "clean" but you could create your own files in the `gruntExample/.tmp/public` folder if you so desire.

```sh
 $ sails lift
```

`sails lift` is one of the "triggers" or events that starts some Grunt _tasks_.  In this case, the _tasks_ that were triggered actually copied files and folders into a `.tmp` folder, but more about those tasks in a minute. 

```javascript
gruntExample
	   |_.tmp
       |_public
	       |_js
	       | |_dependencies
	       |_styles
       ...
```  

I'll exit Sails by typing `control-c`.  Next I'll execute the following Grunt command:

```sh
 $ grunt clean:dev
```

Recall that `dev` is an argument I can pass via the _task_ (e.g. _clean:dev_).  If I go back and look at the `.tmp` folder, `public` and its sub folders no longer exist, they've been "cleaned".  What I'm trying to show here is that even though Sails provided the Grant _task_ you don't need Sails to execute them.  They execute from command-line like any other plain-old grunt task.

###Grouping multiple _tasks_ together

Grunt also provides a way to group multiple tasks together as part of another **named** _task_.  Let's take a look at `default.js` located in the `tasks/register/default.js` folder:

<img src="http://i.imgur.com/Sqcqh25.jpg" />

So `default.js` creates another _task_, the **default** _task_, which triggers other tasks located in an array.  So when the `default` _task_ is executed three other tasks will be executed -- `compileAssets`, `linkAssets`, and `watch`.  

<img src="http://i.imgur.com/WBw9C62.jpg" />

<img src="http://i.imgur.com/GYImd1Z.jpg" />

<img src="http://i.imgur.com/l4BthaG.jpg" />

> **Note:** because the _watch_ task as no dependent tasks, it links directly to the task configuration file (e.g. watch.js) and not a separate register file.  

Sails provides the means for you to link specific Grunt _tasks_ with different Sails triggers. 

###How Sail's _triggers_ execute Grunt _tasks_

There are four Sail's _triggers_ each of which executes a specific Grunt _task_:

<img src="http://i.imgur.com/93N4KJw.jpg" />

These **registered** _tasks_ can be found in the `.\tasks\register` folder.

###Which _tasks_ are executed when Sails starts via `$ sails lift`?

<img src="http://i.imgur.com/jrpKd2p.jpg" />

As we've already seen, `$ sails lift`, triggers the **default** _task_.  The `default` _task_ in turn executes many other tasks grouped by `compileAssets`, `linkAssets`, and `watcher` tasks. 

###The compileAssets _task_

<img src="http://i.imgur.com/WBw9C62.jpg" />


The `compileAssets` _task_ executes five additional tasks:

####clean:dev _task_

Deletes the `public` folder (e.g. gruntExample/.tmp/public) and anything contained within it.

####jst:dev _task_

Precompiles Underscore templates to a .jst file. (i.e. it takes HTML template files and turns them into tiny javascript functions). 

>This can speed up template rendering on the client, and reduce bandwidth usage.

Which templates get pre-compiled is configured in the **pipeline.js** file.
>By default, any .html files contained in `gruntExample/assets/templates` will be pre-compiled.

####less:dev _task_

Compiles LESS files into CSS. Only the `assets/styles/importer.less` is compiled. This allows you to control the ordering yourself, i.e. import your dependencies, mixins, variables, resets, etc. before other stylesheets.

####copy:dev _task_

Copies all directories and files, exept coffescript and less files, from the sails assets folder into the .tmp/public/ directory.

####coffee:dev _task_

Compiles coffeeScript files from assest/js/ into Javascript and places them into .tmp/public/js/ directory.

###The linkAssets _task_

<img src="http://i.imgur.com/GYImd1Z.jpg" />

The `linkAssets` _task_ executes six additional tasks that work heavily with `sails-linker` and `pipeline.js`

**Sails-linker** automatically inserts tags (e.g. `<script>`, `<link>`, etc.) into mark-up files (e.g. .html, .ejs, .jade).  **Pipeline.js** allows you to configure the order in which your css, javascript, and template files should be compiled and linked (via _Sails-linker_) from your views and static HTML files.

####sails-linker:devJS _task_

```javascript
...
devJs: {
      options: {
        startTag: '<!--SCRIPTS-->',
        endTag: '<!--SCRIPTS END-->',
        fileTmpl: '<script src="%s"></script>',
        appRoot: '.tmp/public'
      },
      files: {
        '.tmp/public/**/*.html': require('../pipeline').jsFilesToInject,
        'views/**/*.html': require('../pipeline').jsFilesToInject,
        'views/**/*.ejs': require('../pipeline').jsFilesToInject
      }
},
...
```

This task automatically injects javascript files via `<script>` tags into various `.html` and `.ejs` files.  As with all tasks sails-linker:devJS can be modified to meet your specific needs.

By default, this task effects all `.html` files located in:

- `.tmp/public/` or its sub-folders
- `views/` or its sub-folders

...as well as any `.ejs` files located in `views/` or its sub-folders.

`.html` and/or `.ejs` files with `<!--SCRIPTS-->` `<!--SCRIPTS END-->` tags will have javascript `src` links injected into them.

Sails-linker will use the order of the javascript files found in the `tasks/pipeline.js` file:

```javascript
...
var jsFilesToInject = [

  // Dependencies like sails.io.js, jQuery, or Angular
  // are brought in here
  'js/dependencies/**/*.js',

  // All of the rest of your client-side js files
  // will be injected here in no particular order.
  'js/**/*.js'
];
...
```
> By default, javascript files found in `/assets/js/dependencies/` and its sub-folders is injected first thereafter javascript files found in `/assets/js` and its sub-folders is injected.  You can also "hard-code" any `script` tags you prefer and `sails-linker` will put any remaining javascript files below those tags.

####sails-linker:devStyles _task_

```javascript
...
devStyles: {
      options: {
        startTag: '<!--STYLES-->',
        endTag: '<!--STYLES END-->',
        fileTmpl: '<link rel="stylesheet" href="%s">',
        appRoot: '.tmp/public'
      },

      files: {
        '.tmp/public/**/*.html': require('../pipeline').cssFilesToInject,
        'views/**/*.html': require('../pipeline').cssFilesToInject,
        'views/**/*.ejs': require('../pipeline').cssFilesToInject
      }
},
...
```

This task automatically injects css files via `<link>` tags into various `.html` and `.ejs` files.  As with all tasks sails-linker:devStyles can be modified to meet your specific needs.

By default, this task effects all `.html` files located in:

- `.tmp/public/` or its sub-folders
- `views/` or its sub-folders

...as well as any `.ejs` files located in `views/` or its sub-folders.

`.html` and/or `.ejs` files with `<!--STYLES-->` `<!--STYLES END-->` tags will have css `href` links injected into them.

Sails-linker will use the order of the css files found in the `tasks/pipeline.js` file.

```javascript
...
var cssFilesToInject = [
  'styles/**/*.css'
];
...
```
> By default, css files found in `/assets/styles` and its sub-folders is injected first.  You can "hard-code" any styles you prefer and `sails-linker` will put any remaining css files it finds below.

####sails-linker:devTpl _task_

```javascript
...
devTpl: {
      options: {
        startTag: '<!--TEMPLATES-->',
        endTag: '<!--TEMPLATES END-->',
        fileTmpl: '<script type="text/javascript" src="%s"></script>',
        appRoot: '.tmp/public'
      },
      files: {
        '.tmp/public/index.html': ['.tmp/public/jst.js'],
        'views/**/*.html': ['.tmp/public/jst.js'],
        'views/**/*.ejs': ['.tmp/public/jst.js']
      }
},
...
```

This task automatically injects compiled JST template files via `<script>` tags into various `.html` and `.ejs` files.
>**Remember:** the templates were compiled earlier into functions in a new `jst.js` file.

As with all tasks sails-linker:devTpl can be modified to meet your specific needs.

By default, this task effects all `.html` files located in:

- `.tmp/public/` or its sub-folders
- `views/` or its sub-folders

...as well as any `.ejs` files located in `views/` or its sub-folders.

`.html` and/or `.ejs` files with `<!--TEMPLATES-->` `<!--TEMPLATES END-->` tags will have javascript `src` links injected into them.  For example:

```html
<!--TEMPLATES-->
    <script type="text/javascript" src="/jst.js"></script>
<!--TEMPLATES END-->
```

####sails-linker:devJsJade _task_

This task automatically injects ASK MIKE ABOUT THIS

####sails-linker:devStylesJade _task_

This task automatically injects ASK MIKE ABOUT THIS

####sails-linker:devTplJade _task_

This task automatically injects ASK MIKE ABOUT THIS

###The watch _task_

<img src="http://i.imgur.com/l4BthaG.jpg" />

Unlike the `compileAssets` and `linkAssets` _tasks_, the **watch** _task_ consists of the configuration of a _plugin_ (e.g. the **watch** _plugin_).  

The **watch** _task_ looks for changes in files in a given directory or set of directories. If a change is detected, the _task_ triggers two other tasks-- **syncAssets** (discussed below) and **linkAssets** (discussed above).

###The syncAssets _task_

<img src="http://i.imgur.com/nG1UomJ.jpg" />

The compileAssets task executes four additional tasks:

####jst:dev _task_

Precompiles Underscore templates to a .jst file. (i.e. it takes HTML template files and turns them into tiny javascript functions). 

>This can speed up template rendering on the client, and reduce bandwidth usage.

Which templates get pre-compiled is configured in the **pipeline.js** file.
>By default, any .html files contained in `gruntExample/assets/templates` will be pre-compiled.

####less:dev _task_

```javascript
...
grunt.config.set('less', {
    dev: {
      files: [{
        expand: true,
        cwd: 'assets/styles/',
        src: ['importer.less'],
        dest: '.tmp/public/styles/',
        ext: '.css'
      }]
    }
});
...
```

Compiles LESS files into CSS. Only the `assets/styles/importer.less` is compiled. This allows you to control the ordering yourself, i.e. import your dependencies, mixins, variables, resets, etc. before other stylesheets.

####sync:dev _task_

```javascript
...
grunt.config.set('sync', {
    dev: {
      files: [{
        cwd: './assets',   
        src: ['**/*.!(coffee)'],
        dest: '.tmp/public'
      }]
    }
});
...
```

####coffee:dev _task_

```javascript
...
grunt.config.set('coffee', {
    dev: {
      options: {
        bare: true,
        sourceMap: true,
        sourceRoot: './'
      },
      files: [{
        expand: true,
        cwd: 'assets/js/',
        src: ['**/*.coffee'],
        dest: '.tmp/public/js/',
        ext: '.js'
      }, {
        expand: true,
        cwd: 'assets/js/',
        src: ['**/*.coffee'],
        dest: '.tmp/public/js/',
        ext: '.js'
      }]
    }
  });
  ...
  ```

Compiles coffeeScript files from `assest/js/` into Javascript and places them into `.tmp/public/js/` directory.

A grunt task to keep directories in sync. It is very similar to grunt-contrib-copy but tries to copy only those files that have actually changed. It specifically synchronizes files from the `assets/` folder to `.tmp/public/`, overwriting anything that's already there.

