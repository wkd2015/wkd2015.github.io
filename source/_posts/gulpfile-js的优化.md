---
title: 项目gulp中js编译任务的优化
date: 2016-08-10 23:25:26
tags: [JavaScript,Gulp]
---

##### 存在的问题

1. 原来使用的gulp-require不支持gulp.src()传入,因此只能通过forEach遍历PAGES调用,每次新增页面都要修改PAGES,在git合并文件时,还会因为位置的问题导致数组内多个重复的字符串.
2. 时间上显示的并不准确,运行此任务时显示的是毫秒级的时间,但任务真正结束大概需要近两分钟.
3. 每次编译都是所有文件都被编译一遍,未被修改过的组件和模块也在编译合成压缩之列
<!--more-->
``` javascript
gulp.task('js-build', function () {
    PAGES.forEach(function (v) {//PAGES为路径中间部分的字符串数组
        rjs({
            baseUrl: 'src',
            name: 'p/' + v + '/index',
            out: 'p/' + v + '/index.js',
            removeCombined: true,
            paths: {
                zepto: 'http://zepto.js',
                underscore: 'http://underscore.js',
                juicer: 'http://underscore.js'
            },
            shim: {
                underscore: {
                    exports: '_'
                },
                zepto: {
                    exports: '$'
                }
            }
        })
            .pipe(uglify())
            .pipe(gulp.dest('./build/'));
    });
});
```



##### 新方案
``` javascript
gulp.task('rjs-build',function () {
    return gulp.src(["./src/p/**/index.js"])
        .pipe(changed('./build/p'))
        .pipe(requirejsOptimize(function (file) {
            console.log(file.relative.substring(0,file.relative.length - 3));
            return {
                name: 'p/' + file.relative.substring(0,file.relative.length - 3),
                //removeCombined: true,
                baseUrl: 'src',
                paths: {
                    // 带上http的 避免打包libjs
                    zepto: 'http://zepto.js',
                    underscore: 'htp://underscore.js',
                    juicer: 'htp://underscore.js'
                },
                shim: {
                    underscore: {
                        exports: '_'
                    },
                    zepto: {
                        exports: '$'
                    }
                }
            }
        }))
        .pipe(uglify())
        .pipe(gulp.dest('./build/p'));
});
```

1. 使用gulp-requirejs-optimize,支持gulp.src()传入,不需要再每次修改数组文件
2. 通过gulp-changed来检测开发目录下文件是否变更,筛选保证只有修改过的文件通过


> 最终修改后的时间上显示,即使gulp.src,对效率并没有太多的影响,但是运算的时间可以准确显示,在经过gulp-changed处理后,只编译修改过的文件的情况下,单个页面编译的时间仅需1-2秒