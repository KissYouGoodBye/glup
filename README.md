# Gulp

## 配置

类似于grunt，都是基于Node.js的前端构建工具。不过gulp压缩效率更高。

工具/原料

[nodejs](http://nodejs.cn/) / [npm](https://www.npmjs.com/)

方法/步骤

首先要确保pc上装有node，然后在global环境和项目文件中都install gulp
```bash
npm install gulp -g   (global环境)
npm install gulp --save-dev (项目环境)
```
在项目中install需要的gulp插件，一般只压缩的话需要
```
npm install gulp-minify-css gulp-concat gulp-uglify gulp-rename del --save-dev
```
更多插件可以在这个链接中找到 http://gratimax.net/search-gulp-plugins/

在项目的根目录新建gulpfile.js，require需要的module

```js
var gulp = require('gulp'),
    minifycss = require('gulp-minify-css'),
    concat = require('gulp-concat'),
    uglify = require('gulp-uglify'),
    rename = require('gulp-rename'),
    del = require('del');
```

压缩css
```js
gulp.task('minifycss', function() {
    return gulp.src('src/*.css')      //压缩的文件
        .pipe(gulp.dest('minified/css'))   //输出文件夹
        .pipe(minifycss());   //执行压缩
});
```

压缩js
```js
gulp.task('minifyjs', function() {
    //gulp.src([])可以用数组的形式加载不同格式，不同位置的文件
    return gulp.src('src/*.js')
        .pipe(concat('main.js'))    //合并所有js到main.js
        .pipe(gulp.dest('minified/js'))    //输出main.js到文件夹
        .pipe(rename({suffix: '.min'}))   //rename压缩后的文件名
        .pipe(uglify())    //压缩
        .pipe(gulp.dest('minified/js'));  //输出
});
```

执行压缩前，先删除文件夹里的内容
```js
gulp.task('clean', function(cb) {
    del(['minified/css', 'minified/js'], cb)
});
默认命令，在cmd中输入gulp后，执行的就是这个命令
gulp.task('default', ['clean'], function() {
    gulp.start('minifycss', 'minifyjs');
});
```

然后只要cmd中执行，gulp即可

## 插件开发

借助`through2`模块处理流，封装一个函数去处理：
```js
var { dest, src } = require('gulp');
var through = require('through2');
src('./input.txt').pipe(((prefix) => {
    console.log(prefix)
    if (!prefix) {
        prefix = "";
    }
    var prefix = Buffer.from(prefix);
    var stream = through.obj(function (file, encoding, callback) {
        // 如果file类型不是buffer 退出不做处理
        if (!file.isBuffer()) {
            return callback();
        }
        // 将字符串加到文件数据开头
        file.contents = Buffer.concat([prefix, file.contents]);
        // 确保文件会传给下一个插件
        this.push(file);
        // 告诉stream引擎，已经处理完成
        callback();
    });
    return stream;
})('')).pipe(dest('./output'));
```
