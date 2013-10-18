grunt-templatize
================

Super simple grunt task to convert one or more handlebars-like template files into a Javascript module. Supports AMD, commonjs, and namespaced module formats.

# Simple Usage Example

First, make sure you have node.js and npm properly installed and working. You will also need to have `grunt-cli` installed globally:

```
npm install grunt-cli -g
```

Create a new project folder with `package.json` file and install dependencies from npm:

```
mkdir example
cd example
npm init
npm install grunt grunt-templatize --save-dev
mkdir templates
```

Create `Gruntfile.js` in the project folder root:

```javascript
module.exports = function (grunt) {

  'use strict';

  grunt.loadNpmTasks('grunt-templatize');

  grunt.initConfig({
    templatize: {
      app: {
        src: 'templates/*.tmplz',
        dest: 'dist/templates.js'
      }
    }
  });

  grunt.registerTask('default', ['templatize']);

};
```

Create the following template files:

`templates/header.tmplz`

```html
<header>
<h1>{{title}}</h1>
<div>{{body}}</div>
</header>
```

`templates/footer.tmplz`

```html
<footer>
<a src="{{url}}">{{text}}</a>
</footer>
```

Run the following command:

```
grunt templatize
```

This will generate `dist/templates.js` using the AMD module format with the following content:

```javascript
define({footer:function(model){return '<footer><a src="'+model.url+'">'+model.text+'</a></footer>';},
header:function(model){return '<header><h1>'+model.title+'</h1><div>'+model.body+'</div></header>';}});
```

Here is a beautified version of `dist/templates.js`:

```javascript
define({
  footer: function (model) {
    return '<footer><a src="' + model.url + '">' + model.text + '</a></footer>';
  },
  header: function (model) {
    return '<header><h1>' + model.title + '</h1><div>' + model.body + '</div></header>';
  }
});
```

This AMD module can be imported into your application and the template can be compiled with code similar to this:

```javascript
define(['templates'],function(templates) {
  'use strict';

  var headerModel = {
    title: 'Hello',
    body: 'World'
  };
  var header = templates.header(headerModel);

  var footerModel = {
    text: 'Privacy Policy',
    url: '/privacy.html'
  };
  var footer = templates.footer(footerModel);

});
```

The module format of the destination file can be changed using a `format` of `amd`, `commonjs`, or `namesapce`. This is configured like this:

```javascript
templatize: {
  app: {
    options: {
      format: 'commonjs'
    },
    src: 'templates/*.tmplz',
    dest: 'dist/templates.js'
  }
}
```

Results using `commonjs` format:

```javascript
module.exports={footer:function(model){return '<footer><a src="'+model.url+'">'+model.text+'</a></footer>';},
header:function(model){return '<header><h1>'+model.title+'</h1><div>'+model.body+'</div></header>';}};
```

```javascript
module.exports = {
  footer: function (model) {
    return '<footer><a src="' + model.url + '">' + model.text + '</a></footer>';
  },
  header: function (model) {
    return '<header><h1>' + model.title + '</h1><div>' + model.body + '</div></header>';
  }
};
```

Results using `namespace` format:

```javascript
!function(root){root.templatize.footer=function(model){return '<footer><a src="'+model.url+'">'+model.text+'</a></footer>';}
root.templatize.header=function(model){return '<header><h1>'+model.title+'</h1><div>'+model.body+'</div></header>';}}(this);
```

```javascript
!function (root) {
  root.templatize.footer = function (model) {
    return '<footer><a src="' + model.url + '">' + model.text + '</a></footer>';
  }
  root.templatize.header = function (model) {
    return '<header><h1>' + model.title + '</h1><div>' + model.body + '</div></header>';
  }
}(this);
```

Support for multiple output target destination files:

```javascript
grunt.initConfig({
  templatize: {
    app: {
      src: 'templates/app/*.tmplz',
      dest: 'dist/js/app-templates.js'
    },
    components: {
      src: 'templates/components/*.tmplz',
      dest: 'dist/js/components-templates.js'
    }
  }
});
```

Full configuration including default values:

```javascript
grunt.initConfig({
  templatize: {
    app: {
      // Glob of all source template files
      src: 'templates/app/*.tmplz',
      // Location of generated output file
      dest: 'dist/js/app-templates.js',
      // Options to pass to grunt-templatize
      options: {
        // Module format for output file. Possible values include
        // 'amd', 'commonjs', 'namespace'
        format: 'amd',
        // Prefix at top of each output file 
        prefix: 'define({',
        // Suffix at end of each output file
        suffix: '});',
        // Prefix before first source file 
        firstPrefix: '',
        // Prefix before each source file except the first
        eachPrefix: '',
        // Output between key (source filename) and function 
        eachMiddle: ':',
        // Suffix after each source file except the last
        eachSuffix: ',',
        // Suffix after last source file
        lastSuffix: '',
        // Options to pass to the templatize library
        templatize: {
          // Locates handlebars-like tags: {{ foo.bar }}
          regex: /\{{\s?(.*?)\s?\}}/g,
          // Wraps templatized output into a function
          prefix: "function(model){return '",
          suffix: "';}",
          // Specify model variable name (update this if the prefix
          // is changed with a variable name besides "model")
          model: "model",
          // Options to use with the HTML Minifier
          htmlmin: {
            removeComments: true,
            removeCommentsFromCDATA: true,
            collapseWhitespace: true,
            collapseBooleanAttributes: true,
            removeAttributeQuotes: false,
            removeRedundantAttributes: false,
            useShortDoctype: true,
            removeEmptyAttributes: false,
            removeOptionalTags: false    
          }
        }
      }
    }
  }
});
```

