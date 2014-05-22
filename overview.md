##Introduction to Grunt and Sails integration

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

### Installing a plugin

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

###Grouping multiple _tasks_ together as a _task_

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

<!-- 

            _ _       _ _  __ _   
           (_) |     | (_)/ _| |  
  ___  __ _ _| |___  | |_| |_| |_ 
 / __|/ _` | | / __| | | |  _| __|
 \__ \ (_| | | \__ \ | | | | | |_ 
 |___/\__,_|_|_|___/ |_|_|_|  \__|
                                  
                                

 -->

##The _tasks_ executed when Sails starts via `$ sails lift`?

<img src="http://i.imgur.com/jrpKd2p.jpg" />

Sails starts by default in _development_ mode.   As we've already seen, `$ sails lift`, triggers the **default** _task_ which in turn executes many other tasks grouped by `compileAssets`, `linkAssets`, and `watcher` _tasks_.   These _tasks_ are responsible for getting javascript, styles, and mark-up files compiled, linking these files with appropropriate tags in `.html` and `.ejs` files in the `assets/` and `views/` folder, and copying the `assets/` folder to the working `.tmp/public` folder.  In _development_ mode, Grunt is also watching for changes to these files and updating them on an ongoing basis.

##The compileAssets _task_
=========

<img src="http://i.imgur.com/WBw9C62.jpg" />


As its name suggests the `compileAssets` group of tasks is responsible for compiling any `templates`, `.less`, `.coffee` files and copying the `assets` folder into `.tmp/public/`. The `compileAssets` _task_ executes five tasks:

###clean:dev _task_

Deletes the `public` folder (e.g. gruntExample/.tmp/public) and anything contained within it.

###jst:dev _task_

Precompiles Underscore templates to a .jst file. (i.e. it takes HTML template files and turns them into tiny javascript functions). 

>This can speed up template rendering on the client, and reduce bandwidth usage.

Which templates get pre-compiled is configured in the **pipeline.js** file.
>By default, any .html files contained in `gruntExample/assets/templates` will be pre-compiled.

###less:dev _task_

Compiles LESS files into CSS. 
>Only the `assets/styles/importer.less` is compiled. This allows you to control the ordering yourself, i.e. import your dependencies, mixins, variables, resets, etc. before other stylesheets.

###copy:dev _task_

Copies all directories and files, exept coffescript and less files, from the sails `assets/`folder into the `.tmp/public/` directory.

###coffee:dev _task_

Compiles coffeeScript files from assest/js/ into Javascript and places them into the `.tmp/public/js/` directory.

<!-- 
#####################################################################################################################
#####################################################################################################################
#####################################################################################################################
 -->

##The linkAssets _task_
=========

<img src="http://i.imgur.com/GYImd1Z.jpg" />

The `linkAssets` _task_ traverses through javascript files, files located in the `styles` folder, templates, and .jade files placing the appropriate tags  to link those files in `.html`, `.ejs`, and `.jade`.  The `linkAssets` _task_ is made up of six additional tasks that work heavily with `sails-linker` and `pipeline.js`.

**Sails-linker** automatically inserts tags (e.g. `<script>`, `<link>`, etc.) into mark-up files (e.g. .html, .ejs, .jade).  **Pipeline.js** allows you to configure the criteria and order in which your styles, javascript, and template files should be compiled and linked (via _Sails-linker_) into your views and static HTML files.

###sails-linker:devJS _task_

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

This task automatically injects javascript files via `<script>` tags into various `.html` and `.ejs` files.  
>As with all tasks sails-linker:devJS can be modified to meet your specific needs.

By default, this task effects all `.html` files located in:

- `.tmp/public/` or its sub-folders
- `views/` or its sub-folders

...as well as any `.ejs` files located in `views/` or its sub-folders.

`.html` and/or `.ejs` files with `<!--SCRIPTS-->` `<!--SCRIPTS END-->` tags will have javascript `src` links injected into them.

Sails-linker will use criteria and the order of the javascript files found in the `tasks/pipeline.js` file.

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

###sails-linker:devStyles _task_

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

This task automatically injects css files via `<link>` tags into various `.html` and `.ejs` files.  
>As with all tasks sails-linker:devStyles can be modified to meet your specific needs.

By default, this task effects all `.html` files located in:

- `.tmp/public/` or its sub-folders
- `views/` or its sub-folders

...as well as any `.ejs` files located in `views/` or its sub-folders.

`.html` and/or `.ejs` files with `<!--STYLES-->` `<!--STYLES END-->` tags will have css `href` links injected into them.

Sails-linker will use criteria and the order of the css files found in the `tasks/pipeline.js` file.

```javascript
...
var cssFilesToInject = [
  'styles/**/*.css'
];
...
```
> By default, css files found in `/assets/styles` and its sub-folders is injected first.  You can "hard-code" any styles you prefer and `sails-linker` will put any remaining css files it finds below.  Also recall, that `.less` files are only compiled into `.css` (and then subsequently linked here) if they are imported via `importer.less` found in the `assets\styles\` folder.

###sails-linker:devTpl _task_

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

>As with all tasks sails-linker:devTpl can be modified to meet your specific needs.

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

###sails-linker:devJsJade _task_

```javascript
...
devJsJade: {
      options: {
        startTag: '// SCRIPTS',
        endTag: '// SCRIPTS END',
        fileTmpl: 'script(src="%s")',
        appRoot: '.tmp/public'
      },
      files: {
        'views/**/*.jade': require('../pipeline').jsFilesToInject
      }
},
...
```

This task automatically injects javascript files via `script()` tags into `.jade` files found in the `views/` folder or sub-folders.  
>As with all tasks sails-linker:devJsJade can be modified to meet your specific needs.

`.jade` files with `// SCRIPTS` `// SCRIPTS END` tags will have javascript `src` links injected into them.

Sails-linker will use criteria and the order of the javascript files found in the `tasks/pipeline.js` file.

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

###sails-linker:devStylesJade _task_

```javascript
...
devStylesJade: {
      options: {
        startTag: '// STYLES',
        endTag: '// STYLES END',
        fileTmpl: 'link(rel="stylesheet", href="%s")',
        appRoot: '.tmp/public'
      },

      files: {
        'views/**/*.jade': require('../pipeline').cssFilesToInject
      }
},
...
```

This task automatically injects css files via `link()` tags into `.jade` files found in the `views/` folder or sub-folders.  
>As with all tasks sails-linker:devStylesJade can be modified to meet your specific needs.

`.jade` files with `// STYLES` `<// STYLES END` tags will have css `href` links injected into them.

Sails-linker will use criteria and the order of the css files found in the `tasks/pipeline.js` file.

```javascript
...
var cssFilesToInject = [
  'styles/**/*.css'
];
...
```
> By default, css files found in `/assets/styles` and its sub-folders is injected first.  You can "hard-code" any styles you prefer and `sails-linker` will put any remaining css files it finds below.  Also recall, that `.less` files are only compiled into `.css` (and then subsequently linked here) if they are imported via `importer.less` found in the `assets\styles\` folder.


###sails-linker:devTplJade _task_

```javascript
...
devTplJade: {
      options: {
        startTag: '// TEMPLATES',
        endTag: '// TEMPLATES END',
        fileTmpl: 'script(type="text/javascript", src="%s")',
        appRoot: '.tmp/public'
      },
      files: {
        'views/**/*.jade': ['.tmp/public/jst.js']
      }
}
...
```

This task automatically injects compiled JST template files via `script()` tags into `.jade` files found in the `views/` folder or sub-folders.
>**Remember:** the templates were compiled earlier into functions in a new `jst.js` file.

>As with all tasks sails-linker:devTpl can be modified to meet your specific needs.

By default, this task effects all `.jade` files located in the `views/` or its sub-folders.

`.jade` files with `// TEMPLATES` `<// TEMPLATES END` tags will have javascript `src` links injected into them. 

<!-- 
#####################################################################################################################
#####################################################################################################################
#####################################################################################################################
 -->

##The watch _task_
=========

<img src="http://i.imgur.com/l4BthaG.jpg" />

Unlike the `compileAssets` and `linkAssets` _tasks_, the **watch** _task_ consists of the configuration of a _plugin_ (e.g. the **watch** _plugin_).  

The **watch** _task_ looks for changes in files in a given folder or set of folders. If a change is detected, the _task_ triggers two other tasks-- **syncAssets** (discussed below) and **linkAssets** (discussed above).

###The syncAssets _task_

<img src="http://i.imgur.com/nG1UomJ.jpg" />

The `syncAssets` _task_ is similar to the `compileAssets` _task_ in that it is responsible for compiling any `templates`, `.less`, and `.coffee` files.  `syncAssets` uses a slightly different `sync plugin` that instead of wholesale copying the `assets` folder into `.tmp/public/`, it instead only copies files that have changed.  The syncAssets task executes four tasks:

###jst:dev _task_

Precompiles Underscore templates to a .jst file. (i.e. it takes HTML template files and turns them into tiny javascript functions). 

>This can speed up template rendering on the client, and reduce bandwidth usage.

Which templates get pre-compiled is configured in the **pipeline.js** file.
>By default, any .html files contained in `gruntExample/assets/templates` will be pre-compiled.

###less:dev _task_

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

Compiles LESS files into CSS. 
>Only the `assets/styles/importer.less` is compiled. This allows you to control the ordering yourself, i.e. import your dependencies, mixins, variables, resets, etc. before other stylesheets.

###sync:dev _task_

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

A Grunt _task_ to keep directories in sync. It is very similar to grunt-contrib-copy but tries to copy only those files that have actually changed. It specifically synchronizes files from the `assets/` folder to `.tmp/public/`, overwriting anything that's already there.

###coffee:dev _task_

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

<!-- 

            _ _       _ _  __ _                                _ 
           (_) |     | (_)/ _| |                              | |
  ___  __ _ _| |___  | |_| |_| |_   ______ _ __  _ __ ___   __| |
 / __|/ _` | | / __| | | |  _| __| |______| '_ \| '__/ _ \ / _` |
 \__ \ (_| | | \__ \ | | | | | |_         | |_) | | | (_) | (_| |
 |___/\__,_|_|_|___/ |_|_|_|  \__|        | .__/|_|  \___/ \__,_|
                                          | |                    
                                          |_|                    
                                  

 -->

##The _Tasks_ executed when Sails starts via $ sails lift --prod

<img src="http://i.imgur.com/vSEIdkN.jpg" />

`$ sails lift --prod`, triggers the **prod** _task_. Similar to _development_ mode, Sails executes Grunt _tasks_ that are responsible for getting javascript, styles, and mark-up files compiled.  In _production_ mode, however, Sails adds Grunt _tasks_ that concatonates javascript and style files as well as minifying and compressing them as `production.js` and `production.css`.  The remaining Grunt _tasks_ link these files with appropriate tags in `.html` and `.ejs` files in the `assets/` and `views/` folder and copying the `assets/` folder to the working `.tmp/public` folder.  Unlike, `development` mode, the Grunt _tasks_ associated with watching for file changes is omitted.

##The compileAssets _task_

<img src="http://i.imgur.com/WBw9C62.jpg" />

As its name suggests the `compileAssets` group of tasks is responsible for compiling any `templates`, `.less`, `.coffee` files and copying the `assets` folder into `.tmp/public/`. The `compileAssets` _task_ executes five tasks:

###clean:dev _task_

Deletes the `public` folder (e.g. gruntExample/.tmp/public) and anything contained within it.

###jst:dev _task_

Precompiles Underscore templates to a .jst file. (i.e. it takes HTML template files and turns them into tiny javascript functions). 

>This can speed up template rendering on the client, and reduce bandwidth usage.

Which templates get pre-compiled is configured in the **pipeline.js** file.
>By default, any .html files contained in `gruntExample/assets/templates` will be pre-compiled.

###less:dev _task_

Compiles LESS files into CSS. 
>Only the `assets/styles/importer.less` is compiled. This allows you to control the ordering yourself, i.e. import your dependencies, mixins, variables, resets, etc. before other stylesheets.

###copy:dev _task_

Copies all directories and files, exept coffescript and less files, from the sails `assets/`folder into the `.tmp/public/`directory.

###coffee:dev _task_

Compiles coffeeScript files from assest/js/ into Javascript and places them into the `.tmp/public/js/` directory.

_____

###concat _task_

```javascript
...
grunt.config.set('concat', {
    js: {
      src: require('../pipeline').jsFilesToInject,
      dest: '.tmp/public/concat/production.js'
    },
    css: {
      src: require('../pipeline').cssFilesToInject,
      dest: '.tmp/public/concat/production.css'
    }
});
...
```

Concatenates javascript and css files, and saves concatenated files in `.tmp/public/concat/ directory`.

`concat` will use criteria and the order of the `css` and `js` files found in the `tasks/pipeline.js` file:

```javascript
...
var cssFilesToInject = [
  'styles/**/*.css'
];

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

###uglify _task_

```javascript
...
module.exports = function(grunt) {

  grunt.config.set('uglify', {
    dist: {
      src: ['.tmp/public/concat/production.js'],
      dest: '.tmp/public/min/production.js'
    }
  });

  grunt.loadNpmTasks('grunt-contrib-uglify');
};
...
```

Minifies client-side javascript assets in `.tmp/public/concat/production.js`.  **Note:** The `concat` _task_ must be executed before this _task_.

###cssmin _task_

```javascript
...
grunt.config.set('cssmin', {
    dist: {
      src: ['.tmp/public/concat/production.css'],
      dest: '.tmp/public/min/production.css'
    }
});
...
```

Minifies css files and places them into the `.tmp/public/min/` directory. **Note:** The `concat` _task_ must be executed before this _task_.

###sails-linker:prodJS _task_

```javascript
...
prodJs: {
      options: {
        startTag: '<!--SCRIPTS-->',
        endTag: '<!--SCRIPTS END-->',
        fileTmpl: '<script src="%s"></script>',
        appRoot: '.tmp/public'
      },
      files: {
        '.tmp/public/**/*.html': ['.tmp/public/min/production.js'],
        'views/**/*.html': ['.tmp/public/min/production.js'],
        'views/**/*.ejs': ['.tmp/public/min/production.js']
      }
},
...
```

This task automatically injects the concatonated javascript file found in `.tmp/public/min/production.js` via `<script>` tags into various `.html` and `.ejs` files.  
>As with all tasks sails-linker:prodJS can be modified to meet your specific needs.

By default, this task effects all `.html` files located in:

- `.tmp/public/` or its sub-folders
- `views/` or its sub-folders

...as well as any `.ejs` files located in `views/` or its sub-folders.

`.html` and/or `.ejs` files with `<!--SCRIPTS-->` `<!--SCRIPTS END-->` tags will have javascript `src` links injected into them. 

###sails-linker:prodStyles _task_

```javascript
...
prodStyles: {
      options: {
        startTag: '<!--STYLES-->',
        endTag: '<!--STYLES END-->',
        fileTmpl: '<link rel="stylesheet" href="%s">',
        appRoot: '.tmp/public'
      },
      files: {
        '.tmp/public/index.html': ['.tmp/public/min/production.css'],
        'views/**/*.html': ['.tmp/public/min/production.css'],
        'views/**/*.ejs': ['.tmp/public/min/production.css']
      }
},
...
```

This task automatically injects the concatonated `.css` file found in '.tmp/public/min/production.css' via `<link>` tags into various `.html` and `.ejs` files.  
>As with all tasks sails-linker:prodStyles can be modified to meet your specific needs.

By default, this task effects all `.html` files located in:

- `.tmp/public/` or its sub-folders
- `views/` or its sub-folders

...as well as any `.ejs` files located in `views/` or its sub-folders.

`.html` and/or `.ejs` files with `<!--STYLES-->` `<!--STYLES END-->` tags will have css `href` links injected into them.

###sails-linker:devTpl _task_

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

>As with all tasks sails-linker:devTpl can be modified to meet your specific needs.

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

###sails-linker:prodJsJade _task_

```javascript
...
prodJsJade: {
      options: {
        startTag: '// SCRIPTS',
        endTag: '// SCRIPTS END',
        fileTmpl: 'script(src="%s")',
        appRoot: '.tmp/public'
      },
      files: {
        'views/**/*.jade': ['.tmp/public/min/production.js']
      }
},
...
```

This task automatically injects the concatonated javascript file found in `.tmp/min/production.js` via `script()` tags into `.jade` files found in the `views/` folder or sub-folders.  
>As with all tasks sails-linker:prodJsJade can be modified to meet your specific needs.

`.jade` files with `// SCRIPTS` `// SCRIPTS END` tags will have javascript `src` links injected into them.

###sails-linker:prodStylesJade _task_

```javascript
...
prodStylesJade: {
      options: {
        startTag: '// STYLES',
        endTag: '// STYLES END',
        fileTmpl: 'link(rel="stylesheet", href="%s")',
        appRoot: '.tmp/public'
      },
      files: {
        'views/**/*.jade': ['.tmp/public/min/production.css']
      }
},
...
```

This task automatically injects concatonated css files found in `.tmp/min/production.css` via `link()` tags into `.jade` files found in the `views/` folder or sub-folders.  
>As with all tasks sails-linker:devStylesJade can be modified to meet your specific needs.

`.jade` files with `// STYLES` `<// STYLES END` tags will have css `href` links injected into them.

###sails-linker:devTplJade _task_

```javascript
...
devTplJade: {
      options: {
        startTag: '// TEMPLATES',
        endTag: '// TEMPLATES END',
        fileTmpl: 'script(type="text/javascript", src="%s")',
        appRoot: '.tmp/public'
      },
      files: {
        'views/**/*.jade': ['.tmp/public/jst.js']
      }
}
...
```

This task automatically injects compiled JST template files via `script()` tags into `.jade` files found in the `views/` folder or sub-folders.
>**Remember:** the templates were compiled earlier into functions in a new `jst.js` file.

>As with all tasks sails-linker:devTpl can be modified to meet your specific needs.

By default, this task effects all `.jade` files located in the `views/` or its sub-folders.

`.jade` files with `// TEMPLATES` `<// TEMPLATES END` tags will have javascript `src` links injected into them. 

