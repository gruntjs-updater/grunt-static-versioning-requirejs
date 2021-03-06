# [Grunt-Static-Versioning-Requirejs](https://npmjs.org/package/grunt-static-versioning-requirejs)

GSVR is a [Grunt plugin](http://gruntjs.com/plugins) that allows you to version all your static content so that you can set far out content expiration on Javascript, CSS, HTML Templates and images and ensure that current builds of software are always serving the right versions.

GSRV is meant to work with [James Burke's](http://jrburke.com/) awesome [RequireJS](http://requirejs.org/) library for scripts and optionally with [LESS](http://lesscss.org/) for stylesheets.
If you don't use a module loader with your Javascript - **YOU SHOULD!**

Using a CommonJS or AMD architecture is almost a must in this day and for such script heavy web applications.  And, Require forces you into a non-blocking, modular architecuture which subsequently allows for out of the box script packaging and minification.

This plugin is meant to be used during the continuous integration process since it effectively 'productionalizes' your content.  That doesn't have to be the case if you have a custom build process outside of any CI tool.

## Plugin Features

1. RequireJS Optimizer Integration to version just created production level Require AMD packages
2. Optional Content Delivery Network integration to FTP your content to your favorite CDN
3. Regex replacement within source code so that you can tweak your static paths to point to the new production content (locally or on CDN - ex: /js-build instead of /js or [CDNPATH]/js-build)
4. Client side template support to serve your Angular, Knockout, etc. templates from your CDN instead of your webserver (**requires CORS support from your CDN**)
5. **Coming Soon** LESS/CSS/StyleSheet Images Support so that you can version all your CSS and your stylesheet images and have them sent to your desired location

## Getting Started
This plugin requires Grunt `~0.4.1`

If you haven't used [Grunt](http://gruntjs.com/) before, be sure to check out the [Getting Started](http://gruntjs.com/getting-started) guide, as it explains how to create a [Gruntfile](http://gruntjs.com/sample-gruntfile) as well as install and use Grunt plugins. Once you're familiar with that process, you may install these plugins with these commands:

	npm install grunt-contrib-requirejs --save-dev
	npm install grunt-static-versioning-requirejs --save-dev

Once the plugins have been installed, enable them inside your Gruntfile with these line of JavaScript:

	grunt.loadNpmTasks('grunt-contrib-requirejs');
	grunt.loadNpmTasks('grunt-static-versioning-requirejs');


### Overview
In your project's Gruntfile, add a section named `requirejs` to the data object passed into `grunt.initConfig()`.

Refer to the [requirejs grunt plugin sample configuration](https://github.com/gruntjs/grunt-contrib-requirejs) for all the configuration options.

The configuration below assumes that the require configuration file will be in /static/js and be named site.js.
The output for the plugin goes in /static/js-build.

#### RequireJS Configuration

	requirejs: {
		compile: {
			options: {
				dir: 'static/js-build',
				baseUrl: 'static/js',					// Directory to look for the require configuration file
				mainConfigFile: 'static/js/site.js',	// This is relative to the grunt file
				paths: {
					jquery: 'empty:',	// exclude from build because we use the google cdn
					ko: 'empty:',		// exclude from build because we use the microsoft cdn
					angular: 'empty:'	// exclude from build because we use the google cdn
				},
				modules: [
					{ name: 'app/global' },				// Create a global bundle
					{
						name: 'app/account/account',	// Creaete an account bundle and exclude global
						exclude: ['app/global']
					}
				],
				preserveLicenseComments: false,		// remove all comments
				removeCombined: true,				// remove files which aren't in bundles
				optimize: 'uglify2'					// minify bundles with uglify 2
			}
		}
	}

##### Options

For a complete list of options see the [requirejs grunt plugin sample configuration](https://github.com/gruntjs/grunt-contrib-requirejs).

#### Setup Require in your web application

Require JS should be served in your layout page, master page, etc just before the closing body tag.  Serve require from the cloud fare CDN if you don't have a CDN of your own:

		<script data-main="/js/site.js" src="//cdnjs.cloudflare.com/ajax/libs/require.js/2.1.8/require.js"></script>
		</body>
	</html>


I also serve JQuery, Knockout or Angular from direct script tags after require because ideally they'd already be cached in a user's browser.

I don't think Angular as of yet is AMD compliant so you'd need to add a shim entry to your app.js or site.js file to make it AMD aware.
Here's mine:

	(function(){

		'use strict';

		requirejs.config({
			paths: {
				lib: 'lib',
				app: 'app',
				ko: 'https://ajax.aspnetcdn.com/ajax/knockout/knockout-2.2.1',
				angular: 'https://ajax.googleapis.com/ajax/libs/angularjs/1.1.5/angular.min'
			},
			shim: {
				'lib/underscore': {
					exports: '_'
				},
				angular: {
					exports: 'angular'
				},
				'lib/datepicker/bootstrap-datepicker.all': { deps: ['lib/bootstrap-datepicker'] }
			}
		});

		define('angular', [], function () { return angular; });
		require(['app/ngBootstrap']);

	})();


Then, I put actual script src tags in my javascript for JQuery, Knockout and Angular.  - Suggestion: use conditional comments to fork JQuery and serve the 2+ version to all browsers other than IE8 and below.

This is a Microsoft MVC website, so notice how I'm requiring the global bundle then letting the local pages add their own scripts only after global has loaded.  

		<script data-main="/js/site.js" src="//cdnjs.cloudflare.com/ajax/libs/require.js/2.1.8/require.js"></script>
		<script src="//ajax.googleapis.com/ajax/libs/jquery/2.0.3/jquery.min.js"></script>
		<!--[if lt IE 9]-->	<script src="//ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script><!--[endif]-->
		<script src="//ajax.googleapis.com/ajax/libs/angularjs/1.0.7/angular.min.js"></script>
		<script src="//netdna.bootstrapcdn.com/twitter-bootstrap/2.3.2/js/bootstrap.min.js"></script>
	
		<script>
			require(['app/global'], function ($, global) {
				@RenderSection("Scripts", required: false)
			});
		</script>
	</body>
	</html>

Then, in my views I end up adding script like this:

	@section Scripts{
		require(['app/account/account']);
	}


That can be abstracted to work for any language or server side MVC platform - Node/Express, Ruby/Rails, etc.  The benefit here is that we're using require everywhere to attach scripts and the only path in the entire application that needs to change to switch to production content is the data-main attribute of the require script!


##### Options

For a complete write up of RequireJS [study the site in it's entirety](http://requirejs.org/)!

#### Static Versioning Plugin Configuration
In your project's Gruntfile, add a section named `static_versioning` to the data object passed into `grunt.initConfig()`.

	static_versioning: {
      jsversioning: {
       src:'static/js-build',
       removeAfterUpload:false,
       cdn:{
           target:'/',
           type:'ftp',
           host: 'localhost',
           username:'username',
           pass: 'Password1',
           port: 21
       },
       replace:{
           path:'http://dev-rev-src.dev.activenetwork.com:90',
           src: [
                'Views/',
                'Content/js/site.js'
            ],
           replacements:[{
               from: '/js',
               to: '/$FOLDERNAME'
           }]
       }
      }
    

### Options

#### src

This is the directory to look for that requirejs generates.  It's relative to the grunt file.

#### removeAfterUpload

Boolean value to specify that the generated directory should be deleted after the plugin runs.  Defaults to false.

#### cdn

Object with the following properties:

	{
		target: '',		// Directory to create or look for on the FTP server under which all content will go
		type: 'ftp',	// Only ftp supported at the moment
		host: '',		// FTP or SFTP endpoint
		username: ,''	// user name for ftp/sftp
		pass: '',		// password for ftp/sftp
		port: '', 		// port for ftp/sftp
	}

If CDN is omitted, it will just leave the versioned folders in the file system.

#### replace

See the example above.

##### replace.path

This is a string which can be used in your regular expressions to edit paths.

##### replace.src

This is a string array of locations where the regular expression will search.

##### replace.overwrite

This forces the files to be overwritten with the regex matches.  Leave true for now.

##### replace.replacements

This is an object array that takes a from (regex) and a to (function) and lets you operate on matches inside of the function.  Add more replacements here to inject new paths around your site.

<%= v.path %> - this is the path specified in the options and can be used in your replacements.

<%= v.folderName %> - this is the final folder name of the js folder which would be something like js-build-2013.2.1 if you passed in 2013.2.1 on the command line.

## Grunt Task

Run both tasks by configuring a task:

	grunt.registerTask('build', ['requirejs', 'static_versioning']);

Then you can pass a custom version into the command line call to setup the exactly resulting folder name.
If your output folder js-build and you call grunt build with the version switch like this:

	grunt build --version-number=2013.1.2

Then the resulting Javascript folder will be:

	/js-build-2013.1.2

Ideally, you'd be calling grunt build from a continuous integration build and it would let you pass in a build version number to associate your static content with a specific build.

### Usage Examples

The example above in the configuration section searches all files in a Views folder and subfolders looking for the data-main attribute on the requirejs script.  The regular expression and replacement injects the CDN path in front of the js file being requested in data-main attribute.  

RequireJS does support omitting http and https in the data-main attribute to have it use the same protocol as the page being requested.


## Contributing
Written by [Carlos Martin](https://github.com/pirumpi) and [Steven Tate](https://github.com/tateman66).

## Release History

* 11/08/2013   0.1.0   Initial Release
* 11/09/2013   0.1.1   Code cleanup
* 11/09/2013   0.1.2 - 0.1.6   ReadMe changes
* 02/02/2014     0.3.3      Adding SFTP support
* 02/05/2014     0.3.4      Fixing SFTP issues
* 02/10/2014     0.3.6      Code cleanup
* 02/18/2014     0.3.7      Fixing minor errors
* 02/18/2014     0.4.0      Removing unused options

#####Coming soon

- LESS support with the Grunt Less plugin
- Image support
- Client side templates
