title: 自建一个node cli工具
author: Tommi_Max
tags:
  - 个人总结
categories:
  - 项目构建
  - ''
date: 2019-08-11 16:28:00
---
cli——**命令行界面**（英语：**command-line interface**），我们可以通过cli提供的特定命令执行、操作对应功能。

一个cli工具可以看作是一个工具包再外套上cli，通过cli上提供命令和参数在命令行界面上与人交互，然后完成命令对应的功能。所以在自建一个cli工具之前，首先应该明确它的作用是什么（解决什么问题），应该包含什么功能。还有一点最重要的是，**现在网上是否已经有满足需求的现成cli 工具可以直接使用**（有宝马还造什么自行车 🤪🤪🤪）。

### 目录结构 & 工作流程

首先在工程目录下新建一个文件夹，并新建`bin`目录存放命令执行的文件和`lib`目录存放命令调用的相应模块。

```
/bin  # ------ 命令执行文件
/lib  # ------ 工具模块
package.json
```

`package.json`包含了对cli的相关描述，与其它普通的项目类似，但有一个地方需要特别注意的是，`package.json`下的`bin`参数，**它定义了cli被调用的脚本命令名称及入口文件**，比如：

```
{
    "name":'tmc-cli'
    ...
    "bin": {
      "tmc": "index.js"
    }
    ...
}
```

上面的`bin`字段说明了这个`cli`的命令是`tmc`，当用户在输入`tmc`时就会去当前目录下的`bin`文件夹执行`index.js`文件。

### 实践步骤

1. 设置命令

   在`bin`目录下新建一个`index.js`作为入口文件。然后在`index.js`里写入以下代码：

```javascript
#!/usr/bin/env node

const program = require('commander') // npm i commander -D

program.version('1.0.0')
  .usage('<command> [项目名称]')
  .command('hello', 'hello')
  .command('init', '创建新项目')
  .parse(process.argv)
```

​		上面代码设置了`tmc`命令可以执行的`option`分别是`hello`和`init`，因为没有在`commander`里设置了`command`而没有调用`action`，所以当执行了相应的命令时，在调用了`parse`方法后，`commander`会自动去找当前文件目录下对应与命令同名的js文件并执行，如果文件不存在则会报错。对应的源码如下：

```javascript
// node_modules/commander/index.js

Command.prototype.parse = function(argv) {
  // implicit help
  if (this.executables) this.addImplicitHelpCommand();

  // store raw args
  this.rawArgs = argv;

  // guess name
  this._name = this._name || basename(argv[1], '.js');

  // github-style sub-commands with no sub-command
  if (this.executables && argv.length < 3 && !this.defaultExecutable) {
    // this user needs help
    argv.push('--help');
  }
 	... 
}
```

2. 编写对应命令的执行逻辑

   这个部分就是来实现命令相应的功能的，可以像普通Node程序那样对文件进行处理等等。比如在执行`init`操作后`tmc`会去我的一个仓库拉去模板代码，然后再根据我输入的一些选项来进行更细化的操作。

   Tmc-init.js代码如下：

   ```javascript
   #!/usr/bin/env node
   
   const program = require('commander')
   const path = require('path')
   const fs = require('fs')
   const glob = require('glob')
   const download = require('../lib/download')
   const inquirer = require('inquirer')
   const chalk = require('chalk')
   const logSymbols = require('log-symbols')
   const generator = require('../lib/generator')
   
   program.usage('<project-name>').parse(process.argv)
   
   // 获取新建项目名称
   let projectName = program.args[0]
   
   if (!projectName) { // project-name 必填
     // 相当于执行命令的--help选项，显示help信息，这是commander内置的一个命令选项
     program.help()
     return
   }
   
   const rootName = path.basename(process.cwd()) // 获取当前路径
   const list = glob.sync('*') // 遍历当前目录
   
   let next = undefined
   
   const inquirerList = [{
     name: 'projectVersion',
     message: '项目的版本',
     type: 'input',
     default: '0.1.0',
   }, {
     name: 'projectDescription',
     message: '项目的描述',
     type: 'input',
     default: 'test',
   }, {
     name: 'author',
     message: '项目的作者',
     type: 'input',
     default: 'kuntang',
   }]
   
   if (list.length) { // 如果当前目录不为空
     if (list.filter(name => {
         const fileName = path.resolve(process.cwd(), path.join('.', name))
         const isDir = fs.statSync(fileName).isDirectory()
         return name.indexOf(projectName) !== -1 && isDir
       }).length !== 0) {
       console.log(`项目${projectName}已经存在`)
       return
     }
     next = inquirer.prompt(inquirerList).then(answer => {
       return Promise.resolve({
         projectPath: projectName,
         ...answer
       })
     })
   } else if (rootName === projectName) {
     next = inquirer.prompt([{
       name: 'projectPath',
       message: '当前目录为空，且目录名称和项目名称相同，是否直接在当前目录下创建新项目？',
       type: 'confirm',
       default: true
     }, ...inquirerList]).then(answer => {
       return Promise.resolve({
         projectPath: answer.buildInCurrent ? '.' : projectName,
         ...answer
       })
     })
   } else {
     next = inquirer.prompt(inquirerList).then(answer => {
       return Promise.resolve({
         projectPath: projectName,
         ...answer
       })
     })
   }
   
   next && initialization()
   
   function initialization() {
     next.then(answer => {
       const {
         projectPath,
         projectVersion,
         projectDescription,
         author,
       } = answer
       if (projectPath !== '.') {
         fs.mkdirSync(projectPath)
       }
       return download(projectPath).then(downloadTempPath => {
         return {
           projectName,
           projectVersion,
           projectDescription,
           projectPath,
           author,
           downloadTemp: downloadTempPath,
         }
       })
     }).then(context => {
       const src = path.join(process.cwd(), context.downloadTemp)
       const dest = path.join(process.cwd(), context.projectPath)
       return generator(context, src, dest)
     }).then(context => {
       // 成功用绿色显示，给出积极的反馈
       console.log(logSymbols.success, chalk.green('创建成功:)'))
       console.log()
       console.log(chalk.green('cd ' + context.root + '\nnpm install\nnpm run dev'))
     }).catch(error => {
       // 失败了用红色，增强提示
       console.error(logSymbols.error, chalk.red(`创建失败：${error.message}`))
     })
   }
   ```

   Lib/download.js:

   ```javascript
   const download = require('download-git-repo')
   const ora = require('ora')
   const path = require('path')
   
   module.exports = function(dest) {
     dest = path.join(dest || '.', '.download-temp')
     return new Promise((resolve, reject) => {
       const url = 'https://github.com:kunkuntang/react-typescript-scaffold#template'
       const spinner = ora(`正在下载项目模板，源地址：${url}`)
       spinner.start()
         // 这里可以根据具体的模板地址设置下载的url，注意，如果是git，url后面的branch不能忽略
       download(url,
         dest, { clone: true }, (err) => {
           if (err) {
             spinner.fail() // wrong :(
             reject(err)
           } else {
             spinner.succeed() // ok :)
               // 下载的模板存放在一个临时路径中，下载完成后，可以向下通知这个临时路径，以便后续处理
             resolve(dest)
           }
         })
     })
   }
   ```

   lib/generator.js:

   ```javascript
   const Metalsmith = require('metalsmith')
   const Handlebars = require('handlebars')
   const rm = require('rimraf').sync
   
   module.exports = function(metadata = {}, src, dest = '.') {
     if (!src) {
       return Promise.reject(new Error(`无效的source：${src}`))
     }
   
     return new Promise((resolve, reject) => {
       Metalsmith(process.cwd())
         .metadata(metadata)
         .clean(false)
         .source(src)
         .destination(dest)
         .use((files, metalsmith, done) => {
           const meta = metalsmith.metadata()
           Object.keys(files).forEach(fileName => {
             if (fileName.includes('package.json')) {
               const t = files[fileName].contents.toString()
               files[fileName].contents = new Buffer.from(Handlebars.compile(t)(meta))
             }
           })
           done()
         }).build(err => {
           rm(src)
           err ? reject(err) : resolve({
             root: dest
           })
         })
     })
   }
   ```

   3. 发布到npm

      

### Commander 使用

commander是Node 命令行交互的一个工具，使用它可以创建自己想要的Node命令。

Commander有几个基本概念：

- version option

- option
- command & action

#### version

指定`cli`的版本号。

```javascript
const program = require('commander')
program
  .version('0.0.1', '-v, --version')
```

#### Option parsing

option是用来为commander命令添加一个选项的，用户可以不同的选项来能行特定的功能。

```javascript
#!/usr/bin/env node
 
/**
 * Module dependencies.
 */
 
var program = require('commander');
 
program
  .version('0.1.0')
  .option('-p, --peppers', 'Add peppers')
  .option('-P, --pineapple', 'Add pineapple')
  .option('-b, --bbq-sauce', 'Add bbq sauce')
  .option('-c, --cheese [type]', 'Add the specified type of cheese [marble]', 'marble')
  .parse(process.argv);
 
console.log('you ordered a pizza with:');
if (program.peppers) console.log('  - peppers');
if (program.pineapple) console.log('  - pineapple');
if (program.bbqSauce) console.log('  - bbq');
console.log('  - %s cheese', program.cheese);
```

在Node.js中，用户输入的参数可以通过`process.argv`得到。`commander`的`option`只定义不处理用户输入的参数，需要显式将`process.argv`传入到`parse`方法调用才能处理。

连续输入参数`-abc`等同于`-a -b -c`，在`cli`输入`--template-engine`，在`commander`可以通过`program.templateEngine`驼峰命名取得。

通过给选项加`--no`前缀可以给它设置为`false`，如设置`--no-sauce`后从`program.sauce`取到的是`false`。

要想拿到选项后输入的值，可以设置中括号（选填）或者尖括号（必填），如`.option('-m --myarg [myVar]', 'my option var')`或`.option('-m --myarg <myVar>', 'my require var')`，然后可以这样来取得值`var myInput = program.myarg`。

#### Command 子命令

**`Command`是一个特殊的`option`**，通过设置子命令可以让工具根据入参来实现的特定的小任务，并且对于子命令也可以像主命令一样设置它的`option`。

```javascript
#!/usr/bin/env node
 
var program = require('commander');
 
program
  .command('rm <dir>')
  .option('-r, --recursive', 'Remove recursively')
  .action(function (dir, cmd) {
    console.log('remove ' + dir + (cmd.recursive ? ' recursively' : ''))
  })
 
program.parse(process.argv)
```



#### Option & Command Option

对于主命令的`option`和子命令有时候比较难区分，我们以一个例子来说明一下它们的使用：

比如对于主命令`tmc`来说，它具有初始化项目的功能，那么可以给它定义一个名叫`init`的`option`：

```javascript
#!/usr/bin/env node

const program = require('commander')

program
  .version('0.1.0')
  .option('-i, --init', 'Init the project')
  .parse(process.argv);

if(program.init) {
  // ... do something
}
```

更多的使用方法参照：

[Commander]: https://www.npmjs.com/package/commander	"commander npm官网"

### inquirer

inquirer是一个可以和命令行交互的一个工具包，它提供了多种交互方式：input、radio、select等等，简单的用法如下：

```javascript
var inquirer = require('inquirer');
inquirer
  .prompt([
    /* Pass your questions in here */
  ])
  .then(answers => {
    // Use user feedback for... whatever!!
  });
```

[inquirer]: https://www.npmjs.com/package/inquirer



### download-git-repo

这个工具可以把远程仓库上的代码下载到本地然后作下一步的操作，它的使用方法比较简单，

例如下载`github`上的代码时可以这样写：

```javascript
const download = require('download-git-repo');

// download(url, desc, option, callback);
// url: 仓库地址；
// desc: 下载到本地目录的路径；
// option: 下载的一些额外选项；
// callback: 下载完成后的回调函数，可以处理错误信息；
download('https://mygitlab.com:flipxfx/download-git-repo-fixture#my-branch', 'test/tmp', { clone: true }, function (err) {
  console.log(err ? 'Error' : 'Success')
})
```

更多的使用方法参照：

[download-git-repo]: https://www.npmjs.com/package/download-git-repo



### Metalsmith

Metalsmith是一个静态网站生成器，说抽象了，简单点我个人理解就是它可以把指定目录下的所有文件都遍历出来，然后再把遍历出来的文件每一个都通过定义好的拆件处理一遍，最后再放回去指定的目标路径下。整个过程有点像项目流程构建工具`Grunt`或者`Gulp`。

通过他我们可以很轻松地实现模板插值填充功能。当然如果只有几个文件需要模板插值的话，也可以通过Node API把文件的内容读出来，然后调用模板工具API里对应的编译接口，最后把编译后的内容再覆盖到文件中，这样就不会有种牛刀用到杀鸡上的感觉了。

```javascript
// Metalsmith(src).use(plugin).build(callback)
Metalsmith(src)
  .use(layouts('handlebars'))
  .build(function(err) {
    if (err) throw err;
    console.log('Build finished!');
  });
```

更多的使用方法参照：

[Metalsmith]: https://www.npmjs.com/package/metalsmith	"metalsmith npm官网"

### update-notifier

这个工具可以在用户使用你的工具包的时候检查是够有新的版本可以更新，从而觉得下一步的操作。比如如果我发布了某个工具包的新版本，而某位用户还在使用旧版本的时候就会提醒他有新版本可以更新。

```javascript
const updateNotifier = require('update-notifier');
const pkg = require('./package.json');
 
updateNotifier({pkg}).notify();
```

更多的使用方法参照：

[update-notifier]: https://www.npmjs.com/package/update-notifier	"update-notifier npm官网"


