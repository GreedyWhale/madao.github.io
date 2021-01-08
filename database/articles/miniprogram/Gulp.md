今天总结下如何使用 Gulp 构建微信小程序项目。

微信小程序本来就自带压缩，热更新，ES6 语法支持这些，那为啥还要用 Gulp 呢，这是因为当你的项目分为三个环境的时候（一般真实的项目都分为开发，预发布，正式这三个环境），那就比较麻烦了，appid 肯定每个环境都不一样，请求的域名也不一样，总不能每次切换环境都手动改这些东西吧，而且不用构建工具的话，你就用不了 sass，scss 之类的各种"CSS 预处理器"。

开始吧：

- ### 下载 Gulp：

  ```
  npm install --save-dev gulp
  ```

- ### 需要的插件

  我觉着这才是构建工具最难的地方，太多插件了，根本不知道用哪个，所以这篇文章主要的是记录下这些插件，做个备忘。

  - gulp-newer

    gulp-newer 这个插件主要是用来增量编译的，就是不用每次改一个文件就重新编译所有的文件，只编译改动过的文件就好了

  - gulp-rename

    这个插件主要用来重命名文件，因为我会用到 scss 和 sass，我得把这两种文件在处理成 css 格式后重命名为`.wxss`结尾的文件

  - gulp-sass

    看名字就知道了，用来编译 sass 和 scss 文件

  - gulp-watch

    监听文件改动

  - jsonfile

    为了区分环境，要动态生成小程序 project.config.json 文件，这个插件就是用来读取 JSON 文件或者生成一个 JSON 文件的

  - replace-in-file

    这个插件用来替换所有文件中你想替换的内容，比如请求的基础域名写成`API_BASE`那么在编译的时候就可以根据环境替换这个`API_BASE`为不同的域名

  - del

    这个插件用来每次打包的时候清空文件目录。

  插件就这么多了，安装这些插件

  ```
  npm install --save-dev gulp-newer gulp-rename gulp-sass gulp-watch jsonfile replace-in-file del
  ```

- ### 实现

  1. 根据文档先在根目录下创建 gulpfile.js 文件

     现在的目录结构应该是这样的

     ![](/madao.github.io/database/images/articles/mini_program/gulp/image.png)

     src 文件 是用来放源代码的

  2. 引入插件

     ```

     const path = require('path');

     const gulp = require('gulp');

     const del = require('del');

     const sass = require('gulp-sass');

     const rename = require("gulp-rename");

     const jsonfile = require('jsonfile');

     const replace = require('replace-in-file');

     const watch = require('gulp-watch');

     const newer = require('gulp-newer');

     ```

  3. 定义环境变量 & 配置

     ```

     /*

       node.js中的环境变量，可以手动修改

     */
     // 故意加上空格的，其实没有，不加vite（1.0.0-rc.1）在打包的时候会报错
     const environment = process. env. NODE_ENV;

     const config = {};

     config.development = {

       appid: 'developmentAppid',

       buildPath: './build/development/',

       requestOrigin: 'https://developmentOrigin/',

       projectname: '开发环境'

     }

     config.staging = {

       appid: 'stagingAppid',

       buildPath: './build/staging/',

       requestOrigin: 'https://stagingOrigin/',

       projectname: '预发布环境'

     }

     config.production = {

       appid: 'productionAppid',

       buildPath: './build/production/',

       requestOrigin: 'https://productionOrigin/',

       projectname: '正式环境'

     }

     /*

       buildPath 打包后文件存放路径

       sourcePath 源代码路径

       configPath 小程序工具配置文件打包后存放路径

     */

     const buildPath = path.join(__dirname, `${config[environment].buildPath}`);

     const sourcePath = path.join(__dirname, './src');

     const configPath = path.join(buildPath, './project.config.json');

     ```

  4. 定义具体的任务

     我这里使用的是 gulp 4.0，这篇文章我觉得很好，想要快速了解 Gulp 可以看下：
     [Gulp 4 入门指南](https://github.com/baixing/FE-Blog/issues/7)

     具体的文件处理流程是这样的：

     1. 构建前清理旧的 build 文件夹
     2. 将 src 中的代码移入 build 目录下，并使用 gulp-newer 插件对 build 目录下的文件进行处理
     3. 将 scss 文件编译成 wxss 文件
     4. 清除 build 文件下，发布小程序后不用的文件（比如：scss 文件，README.md 文件等）
     5. 替换所有文件中的域名为当前环境的域名，并生成 project.config.json 文件

     流程清楚了就开始写代码：

     ```

     gulp.task('default', gulp.series(

         'cleanBuildFolder',

         'moveSourceToBuildFolder',

         'compileScss',

         'cleanUselessFile',

         'build'

     ));

     先定义一个默认的任务，然后定好每个任务的名字，然后开始一个一个实现

     ```

     ```

     gulp.task('cleanBuildFolder', () =>

         del([

             `${buildPath}**/*`

         ])

     );

     gulp.task('moveSourceToBuildFolder', () =>

         gulp

             .src(`${sourcePath}/**/*.*`)

             .pipe(newer(buildPath))

             .pipe(gulp.dest(buildPath))

     );

     /*

       这里要注意，要用src 的 base参数，将文件目录指定到当前目录，

       因为每个通过scss文件编译成的wxss文件位置要和scss文件位置保持一致

     */

     gulp.task('compileScss', () =>

         gulp

             .src(path.join(buildPath, '**/*.{sass,scss}'), { base: './' })

             .pipe(sass().on('error', sass.logError))

             .pipe(rename((file) => { file.extname = '.wxss' }))

             .pipe(gulp.dest('./'))

     );

     gulp.task('cleanUselessFile', () =>

         del([

             path.join(buildPath, './**/*.{sass,scss}'),

             path.join(buildPath, './**/*.md')

         ])

     )

     function createConfigFile() {

         const { requestOrigin, appid, projectname } = config[environment];

         return replace({

             files: path.join(buildPath, '**/*'),

             from: [ /'API_BASE'/g ],

             to: [`'${requestOrigin}'`]

         })

             .then(() => {

                 jsonfile.readFile(configPath, (error, oldFile) => {

                 if (error) { oldFile = null };

                     const fileContent = {

                         appid,

                         projectname,

                         description: '项目配置文件',

                         packOptions: { ignore: [] },

                         setting: {

                             urlCheck: true,

                             es6: true,

                             postcss: true,

                             minified: true,

                             newFeature: true

                         },

                         compileType: 'miniprogram',

                         libVersion: '2.3.0',

                         debugOptions: { hidedInDevtools: []},

                         isGameTourist: false,

                         condition: oldFile ? oldFile.condition : { search: { current: -1, list: [] },

                         conversation: oldFile ? oldFile.conversation : { current: -1, list: []}, plugin: { current: -1, list: [] }, game: { currentL: -1,list: []},

                         miniprogram: oldFile ? oldFile.miniprogram : {current: -1,list: [] } }

                     }

                 jsonfile.writeFile(configPath, fileContent, { spaces: 2 });

             })

         })

     }

     /*

       https://blog.csdn.net/weixin_40817115/article/details/81079507

       为什么使用done

     */

     gulp.task('build', done => {

         createConfigFile()

             .then(done)

         });

     }

     // 修改最初定义的default任务

     gulp.task('default', gulp.series(

         'cleanBuildFolder',

         'moveSourceToBuildFolder',

         'compileScss',

         'cleanUselessFile',

         'build',

         done => {

             done();

             console.log('编译完成');

             console.log(`当前环境为${environment}`);

         }

     ));

     // 最后还要实现一个watch任务

     gulp.task('watch', gulp.series('default', () =>

         watch(`${sourcePath}/**/*.*`, gulp.series(

             'cleanBuildFolder',

             'moveSourceToBuildFolder',

             'compileScss',

             'cleanUselessFile'

         )))

     )

     ```

  5. 给 package.json 中添加编译命令

     ```
     "scripts": {

         "dev": "NODE_ENV='development' gulp watch",

         "stag": "NODE_ENV='staging' gulp",

         "prod": "NODE_ENV='production' gulp"

     }
     ```

到这里就结束了，完整的代码可以看我的[github](https://github.com/GreedyWhale/gulp-demo)
