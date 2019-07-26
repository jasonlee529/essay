---
title: gulp default task
categories:
  - javascript
  - node
  - gulp
tags:
  - gulp
  - node
  - javascript
date: 2019-07-26 14:49:00
---
``` javascript
// 导入包
var gulp = require('gulp'),
    concat = require('gulp-concat'),
    merge = require('merge-stream'),
    preprocess = require('gulp-preprocess'),
    cssmin = require('gulp-clean-css'), // css压缩  
    sass = require('gulp-sass'),
    sassImport = require('gulp-sass-import'),
    rename = require('gulp-rename'), // 重命名
    del = require('del'), // 文件清理  
    runSequence = require('run-sequence'); // 多任务 
    
gulp.task('clean', function(cb) {
    return del("dist", cb);
})
gulp.task('bower', function(cb) {
    return gulp.src('bower_components/**')
        .pipe(gulp.dest('dist/bower_components'));
})
gulp.task('css', function(cb) {
    return gulp.src(['**/**/*.css', '!bower_components/**', '!node_modules/**', '!src/plugin/**'])
        .pipe(cssmin({
            advanced: true, //类型：Boolean 默认：true [是否开启高级优化（合并选择器等）]
            compatibility: '', //保留ie7及以下兼容写法 类型：String 默认：''or'*' [启用兼容模式； 'ie7'：IE7兼容模式，'ie8'：IE8兼容模式，'*'：IE9+兼容模式]
            keepBreaks: false, //类型：Boolean 默认：false [是否保留换行]
            keepSpecialComments: '*' //保留所有特殊前缀 当你用autoprefixer生成的浏览器前缀，如果不加这个参数，有可能将会删除你的部分前缀
        }))
        .pipe(gulp.dest('dist/'));
})
gulp.task('sass', function(cb) {
    return gulp.src(['**/**/*.scss', '!bower_components/**', '!node_modules/**', '!src/plugin/**'])
        .pipe(sassImport({
            filename: '_file',
        }))
        .pipe(sass().on('error', sass.logError))
        .pipe(cssmin({
            advanced: true, //类型：Boolean 默认：true [是否开启高级优化（合并选择器等）]
            compatibility: '', //保留ie7及以下兼容写法 类型：String 默认：''or'*' [启用兼容模式； 'ie7'：IE7兼容模式，'ie8'：IE8兼容模式，'*'：IE9+兼容模式]
            keepBreaks: false, //类型：Boolean 默认：false [是否保留换行]
            keepSpecialComments: '*' //保留所有特殊前缀 当你用autoprefixer生成的浏览器前缀，如果不加这个参数，有可能将会删除你的部分前缀
        }))
        .pipe(gulp.dest('dist/'));
})
gulp.task('html', function(cb) {
    return gulp.src(['**/**/*.html', '!src/plugin'])
        .pipe(gulp.dest('dist/'));
})
gulp.task('image', function(cb) {
    return gulp.src(['**/**/*.png', '**/**/*.jpg', '!bower_components/**', '!node_modules/**', '!src/plugin/**'])
        .pipe(gulp.dest('dist/'))
})
var folders=[];
gulp.task('js', function(cb) {
    var tasks = folders.map(function(element) {
        return gulp.src(element + '/**/*.js')
            .pipe(concat('all.js'))
            .pipe(gulp.dest("dist/" + element))
    });
  
    return merge(tasks);
});
gulp.task('env', function(cb) {
    var env1 = process.env.NODE_ENV || 'dev';
    console.log(process.argv);
    return gulp.src('env/env.' + env1 + '.js')
        .pipe(concat('env.js'))
        .pipe(gulp.dest('dist/env'));
});

const { series, parallel } = require('gulp');
exports.default = series('clean', parallel('bower', 'image', 'css', 'sass', 'html', 'env', 'js'));
var logger = function(msg) {
    console.log(msg);
};
```