**DEPRECATED** in favor [webpack-cluster](https://github.com/mdreizin/webpack-cluster).

[![NPM version](http://img.shields.io/npm/v/webpack-config-stream.svg?style=flat)](https://www.npmjs.org/package/webpack-config-stream) [![Travis build status](http://img.shields.io/travis/mdreizin/webpack-config-stream/master.svg?style=flat)](https://travis-ci.org/mdreizin/webpack-config-stream) [![AppVeyor build status](https://ci.appveyor.com/api/projects/status/github/mdreizin/webpack-config-stream?svg=true&branch=master)](https://ci.appveyor.com/project/mdreizin/webpack-config-stream) [![Code Climate](https://codeclimate.com/github/mdreizin/webpack-config-stream/badges/gpa.svg)](https://codeclimate.com/github/mdreizin/webpack-config-stream) [![Code Climate](https://codeclimate.com/github/mdreizin/webpack-config-stream/badges/coverage.svg)](https://codeclimate.com/github/mdreizin/webpack-config-stream) [![Dependency Status](https://david-dm.org/mdreizin/webpack-config-stream.svg?style=flat)](https://david-dm.org/mdreizin/webpack-config-stream) [![Dependency Status](https://david-dm.org/mdreizin/webpack-config-stream/dev-status.svg?style=flat)](https://david-dm.org/mdreizin/webpack-config-stream#info=devDependencies)

[webpack](https://github.com/webpack/webpack)-[config](https://github.com/mdreizin/webpack-config)-stream
=========================================================================================================

Helps to build bundles based on webpack configs

<h2 id="documentation">Documentation</h2>

For API docs please see the [documentation page](https://github.com/mdreizin/webpack-config-stream/blob/master/docs/API.md)!

<h2 id="sample">Sample</h2>

Here is a quick sample of what `webpack-config-stream` does:

`gulpfile.js`

``` javascript
'use strict';

var path = require('path'),
    gulp = require('gulp'),
    webpack = require('webpack-config-stream'),
    DefaultCacheStrategy = require('webpack-config-stream/lib/defaultCacheStrategy');

var src = './src',
    dest = './dist',
    configOptions = {
        debug: true,
        devtool: '#source-map',
        watchDelay: 200
    },
    compilerOptions = {
        useMemoryFs: true,
        progress: true
    },
    CONFIG_FILENAME = webpack.Config.FILENAME,
    CACHE_DEPENDS_ON = [
        './package.json'
    ],
    RUN_CACHE_STRATEGY = new DefaultCacheStrategy({
        dependsOn: [
            '[dirname]/**/*.*'
        ].concat(CACHE_DEPENDS_ON)
    }),
    WATCH_CACHE_STRATEGY = new DefaultCacheStrategy({
        dependsOn: [
            '[filename]'
        ].concat(CACHE_DEPENDS_ON)
    });

gulp.task('webpack', [], function() {
    return gulp.src(path.join(src, '**', CONFIG_FILENAME), { base: path.resolve(src) })
        .pipe(webpack.cache(RUN_CACHE_STRATEGY))
        .pipe(webpack.init(compilerOptions))
        .pipe(webpack.props(configOptions))
        .pipe(webpack.run())
        .pipe(webpack.format({
            version: false,
            timings: true
        }))
        .pipe(webpack.failAfter({
            errors: true,
            warnings: true
        }))
        .pipe(webpack.ignore())
        .pipe(gulp.dest(dest));
});

gulp.task('watch', function() {
    gulp.watch(path.join(src, '**/*.*')).on('change', function(event) {
        if (event.type === 'changed') {
            gulp.src(event.path, { base: path.resolve(src) })
                .pipe(webpack.closest())
                .pipe(webpack.cache(WATCH_CACHE_STRATEGY))
                .pipe(webpack.init(compilerOptions))
                .pipe(webpack.props(configOptions))
                .pipe(webpack.watch(function(err, stats) {
                    gulp.src(this.path, { base: this.base })
                        .pipe(webpack.proxy(err, stats))
                        .pipe(webpack.format({
                            verbose: true,
                            version: false
                        }))
                        .pipe(webpack.ignore())
                        .pipe(gulp.dest(dest));
                }));
        }
    });
});

```
