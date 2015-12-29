
# bendystraw [![NPM version](https://img.shields.io/npm/v/bendystraw.svg?style=flat-square)](https://www.npmjs.com/package/bendystraw)

<img src="http://i.imgur.com/Pdmetdq.png" alt="bendystraw" align="right" />
bendystraw is a set of [Gulp](https://github.com/gulpjs/gulp/) tasks for developing and deploying Angular apps.

Some features include [Browersync](https://www.browsersync.io/) development, multiple app environments and configuration, GitHub changelog and release creation, Slack integration, and more...

To see an example of an Angular app using bendystraw, check out the [example](https://github.com/brousalis/bendystraw-test) repo.

## Usage

    npm install --save-dev bendystraw

In order to use the gulp tasks, create a `gulpfile.js` with:

```
require('bendystraw')()
```

## Features
- **JS:**
  - Built for Angular
    - Dependency injection annotations
    - Compiles html files to the Angular template cache
    - Automatic file sort to avoid injection issues
  - Coffeescript support
  - Bower components injected through [wiredep](https://github.com/taptapship/wiredep)
  - Multiple script bundles through [useref](https://github.com/jonkemp/useref)
- **CSS:**
  - Sass support, indented or scss using node-sass
  - Autoprefixer
- **Images:**
  - Image compression
  - Grabs images out of Bower components
- **Development:**
  - Live reload with [Browsersync](https://www.browsersync.io/)
  - Support for multiple environments through [dotenv](https://github.com/motdotla/dotenv)
  - dotenv configuration output to Angular constants for injection
  - Environment specific logic in the views
- **Releasing:**
  - GitHub tag and release support
  - Changelog generation
- **Deployment:**
  - Amazon S3 bucket deploys
  - Asset revisioning (cache busting)
  - Cloudfront CDN support
  - Slack deployment messages
- **Testing:**
  - Karma/phantomjs support
  - `.spec.js` style and `/tests` folder

## Tasks

command | description
------- | ------------
`gulp` | defaults to server task
`gulp server` | builds the app to `/.dev` and runs the development server
`gulp build` | builds the app to `/build`
`gulp release` | bumps, tags, and creates a GitHub release based on `package.json` version
`gulp deploy` | deploys `/build` to an S3 bucket, posts to Slack if configured and successful
`gulp test` | runs tests using karma runner

## Config

To configure settings and paths, do this:
```javascript
require('bendystraw')({
  paths: {
    src: 'app', // override main javascript folder
    dest: 'public', // override the build folder
    styles: 'css' // override the stylesheet folder
  },
  port: 42 // port to launch the server on
})
```
Check out the default config values [here](https://github.com/brousalis/bendystraw/blob/master/gulpfile.js/config.js)

## Extra tasks

These tasks are used in the primary tasks, but you can run them manually.

command | description
------- | ------------
`gulp clean` | deletes the `/build` and `/.dev` folders
`gulp scaffold` | creates folders/files based on the config
`gulp ship` | chains together `build`, `release`, `deploy`
`gulp scripts` | compiles coffeescript files to javascript into the `./dev`
`gulp styles` | compiles sass files to css into the `/.dev`
`gulp templates` | compiles the html files then creates a templates.js file
`gulp fonts` | copies fonts from bower components into the build folder
`gulp images` | copy images from bower components into dev folder
`gulp images:build` | optimize images and put them in the build folder
`gulp images:optimize` | optimizes images from source folder and into dev folder
`gulp env` | creates a env.js file from a .env file
`gulp vendor` | copies third party libs from the `/vendor` folder into dev folder
`gulp other` | copies extra folders/files in the source folder into build folder

## Features

### Asset Injection

JS/CSS assets are handled a little old school with bendystraw, as it was built to support legacy applications. it uses [gulp-inject](https://github.com/klei/gulp-inject/releases) and [gulp-angular-filesort](https://github.com/klei/gulp-angular-filesort) to include the application's assets.

gulp-inject looks for this in your html file:

```html
<!-- inject:js -->
<!-- endinject -->
```

and injects all of your javascript files there, added in the correct order thanks to angular-filesort. when building the app (`gulp build`), [gulp-useref](https://github.com/jonkemp/gulp-useref) allows us to bundle multiple files together, like so:

```html
<!-- build:js app/app.js -->
  <!-- inject:templates -->
  <!-- endinject -->

  <!-- inject:js -->
  <!-- endinject -->
<!-- endbuild -->
```

Take a look at the bendystraw example [index.html](https://github.com/brousalis/bendystraw-test/blob/master/source/index.html) to see it set up correctly, and what the build creates. you can also use `gulp scaffold`.

We can even use [wiredep](https://github.com/taptapship/wiredep) to automatically include Bower components and third party libraries (configurable folder location):

```html
<!-- build:js(source) app/vendor.js -->
  <!-- bower:js -->
  <!-- endbower -->

  <!-- inject:vendor -->
  <!-- endinject -->
<!-- endbuild -->
```

### GitHub releases

Based on the application's `package.json` version, running `gulp release` will bump the version of the app, tag and create a release on the GitHub repo. first, you need to set

```bash
export GITHUB_TOKEN=
```

In your environment variables. get a personal access token from your [settings](https://github.com/settings/tokens) page.

> **WARNING!** Make sure your `package.json` is valid and has the correct `"repository"` and `"version"` values.

Then run `gulp release`. if successful, you should get a release simliar to the bendystraw-test  [releases](https://github.com/brousalis/bendystraw-test/releases/)

### S3 deployment

To utilize the `gulp deploy` task, you'll need the following environment variables set:

```bash
export AWS_BUCKET=
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=

// Cloudfront CDN - optional
export AWS_DISTRIBUTION_ID=
export AWS_CLOUDFRONT_DOMAIN=xxxxx.cloudfront.net
```

Then run `gulp deploy`. if you want to deploy a different environment, say `staging`, you need to configure your variables like this:

```bash
export STAGING_AWS_BUCKET=
export STAGING_AWS_ACCESS_KEY_ID=
export STAGING_AWS_SECRET_ACCESS_KEY=
```

and you would run: `gulp deploy --env staging`

### Slack integration

After a successful deploy, bendystraw can send a message to a Slack channel via an [incoming webhook](https://api.slack.com/incoming-webhooks). create a new incoming webhook [https://TEAMNAME.slack.com/apps/manage/custom-integrations](https://TEAMNAME.slack.com/apps/manage/custom-integrations), then set it's url as an environment variable:

```bash
export SLACK_WEBHOOK_URL=https://hooks.slack.com/services/...
```

### Angular $templateCache

The template files in the project get minified then output into a `templates.js` file, which looks something like this:

```javascript
angular.module("templates", [])
.run(["$templateCache", function($templateCache) {
  $templateCache.put("app/layouts/layout.html","<div ui-view=\"\"></div>");
}]);
```

Then include the `templates` module into your project. you can reference a template in Angular using `templateUrl` and the full path (including the .html)

The `templates.js` file gets bundled into your compiled app.js on build if you have your [index.html](https://github.com/brousalis/bendystraw-test/blob/master/source/index.html) set up correctly.

> **WARNING!** Make sure you include the `templates` module into your Angular project before building production! you will be missing your template code if you don't.

### Multiple Environments

bendystraw uses [dotenv](https://github.com/motdotla/dotenv) for app specific configuration. if you want to override variables per environment, create a `.env.environment`, then run any Gulp command with `--env environment` (the word environment can anything).

These variables will be dumped into an Angular module called `env` (name can be configured). load that into your app, then you have access to the `ENV` and `NODE_ENV` constants.

```javascript
angular.module('testApp', [
  'templates',
  'env'
]).config(function(ENV, NODE_ENV) {
  console.log('app config', ENV, NODE_ENV);
});
```

> **WARNING!** Do not put anything in this file you wouldn't want exposed to everyone! `.env` gets compiled and included in your source app.js.

## Thanks

bendystraw is inspired and based off of many Gulp projects. [gulp-starter](https://github.com/vigetlabs/gulp-starter/) by [vigetlabs](https://viget.com/extend) and [generator-gulp-angular](https://github.com/Swiip/generator-gulp-angular) for Yeoman by [Matthieu Lux](github.com/swiip). built at [Belly](http://github.com/bellycard)
