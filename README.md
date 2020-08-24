# SASS(SCSS) color to filters mixin

A mixin to transform any given colors to a set of CSS filters.

## What It Does

This repo contains two things:

- An SCSS mixin which generates set of css filters to make any color look like as a target one,
- A sample worklow for caching mixin results (or any other data).

Since this mixin requires heavy calculations, it obviously can't be used in production as-is. Here is an example of Sass caching technique, which lets you calculate value just once and store it in cache. This way you can run watch task, and Sass won't attempt to run calculations every time - only if you use new color.

### Basic Usage

1. Install node modules

   ```bash
   npm install
   ```

2. Edit `main.css`. Use mixin as below (refer to `main.scss` for a fully working example):

   ```scss
   .button--red {
     background: green;
     @include color-to-filter(#ff0000) // color-to-filter(red) would also work.;;
   }
   ```

3. Compile: `npm run css:build`, or watch for changes: `npm run css:watch`
4. The `.button--red` will be visually red, and compiled css look will like that:

   ```css
   .button--red {
     background: green;
     /* filter values will always be different */
     filter: brightness(0) saturate(100%) invert(19%) sepia(70%) saturate(4218%)
       hue-rotate(352deg) brightness(94%) contrast(121%);
   }
   ```

## Color to Filter Mixin

Mixin code is based on this [algorithm](https://stackoverflow.com/a/43960991). Follow the link for the detailed explanation how it works.

### Allgorithm Results and Loss

Keep in mind, this algorithm is based on randomized values, so it produces different result every time. If you're not satisfied with the output, run it again.

Loss is the difference between actual target color and result (visible color emulated with filters). The less is the better. If loss is more then 20, you will get the warning, and the value won't be cached. Recompile to get better result.

### Productivity Issues

If you intend to use this code, please keep in mind that it's really, really slow. A calculation for one color can take from 30 seconds up to couple of minutes, depending on your machine. For a comparison, Javascript code runs in a couple of seconds. You might consider implementing JS algorithm in you project somehow.
If you still prefer a mixin, make use of caching technique, as described below, or create your own.

## Caching Technique

Although I call it caching here, it is not a real cache. It's not anyhow related to Sass caching. In fact this is just a quick and not super clean technique to avoid heavy calculation on each build.

### Workflow

Caching is performed by postcss plugin [postcss-extract](https://github.com/Nitive/postcss-extract), which extrats required @-rules declaration and saves to a separate file. The process is the following:

1. The first imported file in `main.scss` should be `abstracts/cached-vars` - loading the cache.
2. The latest part of the file is a custom `@export` rule, including declarations of the variabled to cache. Each line to be formatted like:
   `#{"$" + *variable_name*}: unquote(inspect(*variable_value*)) + " !global";`
   Global flag is not needed if you compile with dart-sass.
3. When you run the script, SCSS compiles to CSS first
4. PostCSS process extracts `@export` declaration from compiled css file and puts it to `cached-vars.scss` for further usage.

## Different Syntax for Libsass and Dart-sass

SASS/SCSS files currently can be compiled with 2 packages - [node-sass](https://www.npmjs.com/package/node-sass), which is based on Libsass and [sass](https://www.npmjs.com/package/sass), written in Dart.
Although dart-sass normally has no problem compiling files with outdated syntax, it yields deprecation warnings for `!global` flag, and requires slightly different approach if you want to use the new module import system.

This repo contains examples for both packagages, see below availble scripts for each of them:

- **Node-sass** - entry point `main.scss`
  - `npm run css:build`
  - `npm run css:watch`
- **Dart-sass** - entry point `main_dart.scss`
  - `npm run css:build:dartsass`
  - `npm run css:watch:dartsass`

_**Note:** `_colorfilters.scss` and `_colorfilters_dart.scss` are equal, except lines 28-29, which are import statements._

## Authors

- **[Irina Illustrova](https://github.com/Illustrova)**
- Original algorithm by **[MultiplyByZero at Stackoverflow](https://stackoverflow.com/a/43960991)**
