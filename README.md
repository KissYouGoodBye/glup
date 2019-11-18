# Gulp

## 配置

类似于`grunt`，都是基于`Node.js`的前端构建工具。不过gulp压缩效率更高

工具和原料

- [nodejs](http://nodejs.cn/)
- [npm](https://www.npmjs.com/)

方法和步骤

首先要确保系统上装有`node`，然后在全局环境和项目文件中安装`gulp`
```bash
npm install gulp -g # global环境
npm install gulp --save-dev # 项目环境
```
在项目中安装需要的`gulp`插件，一般只压缩的话需要
```bash
npm install gulp-minify-css gulp-concat gulp-uglify gulp-rename del --save-dev
```
更多插件可以在这个链接中找到 http://gratimax.net/search-gulp-plugins

在项目的根目录新建`gulpfile.js`，引入需要的模块

```js
var gulp = require('gulp'),
    minifycss = require('gulp-minify-css'),
    concat = require('gulp-concat'),
    uglify = require('gulp-uglify'),
    rename = require('gulp-rename'),
    del = require('del');
```
压缩`css`文件
```js
gulp.task('minifycss', function() {
    return gulp.src('src/*.css') //压缩的文件
        .pipe(gulp.dest('minified/css')) //输出文件夹
        .pipe(minifycss()); //执行压缩
});
```
压缩`js`文件
```js
gulp.task('minifyjs', function() {
    // gulp.src([])可以用数组的形式加载不同格式，不同位置的文件
    return gulp.src('src/*.js')
        .pipe(concat('main.js')) //合并所有js到main.js
        .pipe(gulp.dest('minified/js'))  //输出main.js到文件夹
        .pipe(rename({suffix: '.min'}))  //rename压缩后的文件名
        .pipe(uglify()) //压缩
        .pipe(gulp.dest('minified/js')); //输出
});
```
执行压缩前，先删除文件夹里的内容
```js
gulp.task('clean', function(cb) {
    del(['minified/css', 'minified/js'], cb)
});
```
默认命令，在`cmd`中输入`gulp`后，执行的就是这个命令
```js
gulp.task('default', ['clean'], function() {
    gulp.start('minifycss', 'minifyjs');
});
```

然后只要`cmd`中执行`gulp`即可

## 插件开发

借助`through2`模块处理流，封装一个函数去处理
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
开发时候注意要理解流的概念，`through`处理后的`file`对象是一个`vinyl`类型，`vinyl`可以理解为是`buffer`加了路径的类型，如下面这个例子
```js
var Vinyl = require('vinyl');
var file = new Vinyl({
    cwd: '/',
    base: '/test/',
    path: '/test/file.js',
    contents: Buffer.from('Yao')
}); // <File "file.js" <Buffer 59 61 6f>>
var prefix = Buffer.from('Eno'); // <Buffer 45 6e 6f>
// bufferData经过through处理为gulp能识别的流形式，再用pipe处理
var bufferData = Buffer.concat([prefix, file.contents]); // <Buffer 45 6e 6f 59 61 6f>
```
我们可以使用`file.contents`转化为`Buffer`类型，结合`Buffer.from(string)`和`Buffer.concat()`制作一个新的`Buffer`数据，然后通过`through`处理为`gulp`能识别的流，注意`through`和`stream`的流形式是不兼容的，虽然他们都有`pipe`方法
