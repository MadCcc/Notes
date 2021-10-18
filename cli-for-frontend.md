# 搭建前端CLI

## 前言

一个完善的项目模版可以减少我们对于项目基建的重复性工作，在使用React的时候更加如此。  
构建一个React项目常用的CLI(或者框架）有一些，我这里简单列举几个
1. **[create react app](https://create-react-app.dev/)**  
   React提供的 [create react app](https://create-react-app.dev/) 作为构建项目的起手未免有些过于简单了，只提供了Webpack这些使用React的最低限度配置，连Router都需要自己去配，有种我想造车但是炼铁技术都还没有成熟的感觉.
2. **[Next.js](https://nextjs.org/)**  
   [Next.js](https://nextjs.org/) 无疑是一个非常好用的基于React的SSR框架，优点有很多，比如甚至已经支持了ISG构建（Incremental Static Generation）。但引用Umi文档里的一句话，就是不够接地气。也就是说基于 [Next.js](https://nextjs.org/) 完成一个工程化的项目的话还是要做很多额外的配置
   > next.js 是个很好的选择，Umi 很多功能是参考 next.js 做的。要说有哪些地方不如 Umi，我觉得可能是不够贴近业务，不够接地气。比如 antd、dva 的深度整合，比如国际化、权限、数据流、配置式路由、补丁方案、自动化 external 方面等等一线开发者才会遇到的问题。
3. **[Umi](https://umijs.org/zh-CN)**  
   [Umi](https://umijs.org/zh-CN) 确实很接地气，从构建一个初始项目就考虑了很多工程问题，比如支持两种路由（但不兼容）：约定式路由和配置路由，支持项目内SSR直接配置以及SSR失败的降级方案。要说有什么不足的地方，可能就是使用人数还不够多，社区贡献会比较少，可能会有一些难以解决的小毛病。

上述三个只是一个构建项目"雏形"可以使用的技术，但是要融入一些我们自己的东西的话，比如集成Auth，初始化layout等，还是得在此之上做一些文章。

## 项目创建

所有伟大的项目都要从创建文件夹开始

```bash
# 创建文件夹
mkdir create-my-app && cd create-my-app

# 初始化npm仓库，一路回车即可
yarn init
```

我们会利用 `nodejs` 制作cli，所以我们需要一个安装这个npm包后可以被执行的command，去运行指定的js文件

```bash
# 创建bin文件夹
mkdir bin

# 创建入口文件
touch bin/create-my-app.js
```

```json
// package.json

{
   ...,
   "bin": {
      "create-my-app": "./bin/create-my-app.js"
   },
   ...
}
```

之后我们还需要一个项目模版，用来拷贝构建

```bash
# 创建项目模版
mkdir templates && mkdir templates/ssr-nextjs
```

再创建一个`src/`文件夹，主要开发代码都会放在这里

```bash
# 创建src目录
mkdir src
```

现在我们整个目录结构会是这样的：

```
├──bin/
   └──create-my-app.js
├──src/
├──templates/
   ├──ssr-nextjs/
      └──...(template files)
└──package.json
```

## 实现CLI

### 处理命令行指令

我们可以使用 [`commander`](https://github.com/tj/commander.js) 处理命令行的指令，并且作为CLI的入口

```bash
# 安装typescript依赖
yarn add typescript ts-node @types/node -D

# 创建入口文件
touch src/cli.ts
```

```ts
// src/cli.ts

const { program } = require('commander');

program.version('0.0.1');

const handler = () => {
  // TODO: 做一些CLI该做的事情
}

program.action(() => {
  handler();
});

program.parse(process.argv);
```

`commander`其实是可以接受并处理一些命令行参数的，此处为一个简单实现，具体参照`commander`的[文档](https://github.com/tj/commander.js)

### 实现用户交互

我们可以利用 [`chalk`](https://github.com/chalk/chalk) 和 [`Inquirer.js`](https://github.com/SBoudrias/Inquirer.js) 实现一些简单的用户交互。
[`chalk`](https://github.com/chalk/chalk) 可以美化输出，比如给输出的文字指定一些颜色。
[`Inquirer.js`](https://github.com/SBoudrias/Inquirer.js) 可以实现一些简单的用户输入，比如文字输入、选择等等。

我们先实现一个`Generator`类，用于项目构建

```bash
# 安装依赖，inquirer内含chalk依赖
yarn add inquirer
yarn add @types/inquirer -D

# 创建文件
touch src/generator.ts
```

```ts
// src/generator.ts

const fs = require('fs');
const chalk = require('chalk');
const inquirer = require('inquirer');

type AppOptions = {
  name: string;
  render: 'SSR' | 'CSR'
}

class AppGenerator {
  options: AppOptions;

  constructor() {
    this.options = {
      name: '',
      render: 'SSR',
    };
  }

  async init() {
    console.log(chalk.green('Starting create my app...'));
    console.log();
    await this.ask();
    this.write();
  }

  ask() {
    return inquirer.prompt([
      {
        type: 'input',
        name: 'name',
        message: 'Your project name:',
        default: 'new-my-app',
        validate(input: string) {
          if (fs.existsSync(input)) {
            return 'directory already exists';
          }
          return true;
        }
      },
      {
        type: 'list',
        name: 'render',
        message: 'Render Type:',
        choices: ['SSR', 'CSR'],
      }
    ]).then((answer: any) => {
      this.options.name = answer.name;
      this.options.render = answer.render;
    });
  }

  write() {
    // TODO: 这里做文件读写
  }

}

module.exports = AppGenerator;
```

`inquirer.prompt`接收一个数组作为参数，里面每一个元素都是提问用户的问题，可以设置这些问题的类型`type`、名称`name`(会作为答案的key)等等，并返回一个promise，第一个参数`answer`里就是所有用户输入/选择的内容，以一个`Object`的形式返回。  
这里我们设置了两个问题，并将这两个问题的答案存储到这个`AppGenerator` Class里面。第一个问题是新项目的名称，我们会把这个名称填入`package.json`的`name`字段中。第二个问题是项目的渲染方式，这将会决定我们给用户什么样的模版。这里还可以设置一些其他的问题，可以自行丰富。

这时我们可以用`ts-node`运行一下`src/cli.ts`，将会在命令行中展示给用户这两个问题，用户作答后程序就会退出

### 实现项目拷贝与模版填充

模版文件会放在`templates/ssr-nextjs`中，这里先不做展示了，我们先来看看如何将文件拷贝到用户的文件夹中去。这里我们用到的依赖如下：
- [ShellJs](https://github.com/shelljs/shelljs): 帮助我们模拟一些Linux指令，这里我们会用到`cp`指令拷贝文件
- [mem-fs-editor](https://github.com/SBoudrias/mem-fs-editor): 帮助我们实现文件读写，可以用于拷贝文件，也可以用于填充模版文件

```bash
# 安装依赖
yarn add shelljs mem-fs mem-fs-editor
yarn add @types/shelljs @types/mem-fs-editor -D
```

```ts
// src/generator.ts

// ...
const path = require('path');
const memFs = require('mem-fs');
const memFsEditor = require('mem-fs-editor');
const shell = require('shelljs');

// ...

class AppGenerator {
   options: AppOptions;
   tplDirPath: string;
   rootPath: string;
   fs: Editor;

   constructor() {
      const store = memFs.create();
      this.fs = memFsEditor.create(store);

      this.options = {
         name: '',
         render: 'SSR',
      };

      this.rootPath = path.resolve(__dirname, '../');
      this.tplDirPath = path.join(this.rootPath, 'templates/ssr-nextjs');
   }

   async init() {
      // ...
   }

   ask() {
      // ...
   }

   write() {
      this.buildTpl();
   }

   buildTpl() {
      const { name } = this.options;

      // 获取当前命令的执行目录，注意和项目目录区分
      const cwd = process.cwd();

      // 项目目录
      const projectPath = path.join(cwd, this.options.name);

      // 新建项目目录
      // 同步创建目录，以免文件目录不对齐
      fs.mkdirSync(projectPath);
      this.copyDir('*', projectPath);
      this.copyDir('.*', projectPath);

      // 使用存储的name答案填充模版中的空缺
      this.copyTpl('package.json', path.join(projectPath, 'package.json'), {
         name,
      });

      this.fs.commit(() => {
         console.log();
         console.log(`${chalk.grey(`创建项目: ${name}`)} ${chalk.green('✔ ')}`);
      });
   }

   getTplPath(file: string) {
      return path.join(this.tplDirPath, file);
   }

   copyTpl(file: string, to: string, data = {}) {
      const tplPath = this.getTplPath(file);
      this.fs.copyTpl(tplPath, to, data);
   }

   copy(file: string, to: string) {
      const tplPath = this.getTplPath(file);
      this.fs.copy(tplPath, to);
   }

   copyDir(dir: string, to: string) {
      const tplPath = this.getTplPath(dir);
      shell.cp('-r', tplPath, to);
   }

}

module.exports = AppGenerator;

```

其中`package.json`文件是一个`ejs`格式的模版文件，可以用以下方式留下空缺以待填充，这样最后生成的`package.json`中`name`字段就是用户输入的内容了。

```json
// templates/ssr-nextjs/package.json

{
   "name": "<%= name %>",
   ...
}
```

### 构建发布

到上一节为止一个CLI需要具备的功能我们都已经实现了，但细心的同学可能已经发现了，`bin/create-my-app.js`中还没有内容，那么这一步我们来做最后的工作。  
我们使用`father-build`进行构建，偷个懒，不自己配babel或者rollup了，有兴趣的同学可以自行配置。

```bash
# 安装依赖
yarn add father-build rimraf -D

# 创建配置文件
touch .fatherrc.js
```

```js
// .fatherrc.js

export default {
  entry: 'src/cli.ts',
  file: 'lib/cli.js',
  cjs: 'babel',
}
```

别忘了添加`.gitignore`，我们不应该把构建结果提交到git仓库上，`lib/`是我们的构建输出文件夹

```gitignore
node_modules/
.idea/

# build
lib/
```

然后让我们添加`npm scripts`

```json
// package.json

{
   ...,
   "scripts": {
      "clean": "rimraf lib",
      "build": "npm run clean && father-build build",
   },
   ...
}
```

然后我们运行 `yarn build` 就可以看到我们的构建结果了

最后我们填充一下 `bin/create-my-app.js`

```js
// bin/create-my-app.js

#!/usr/bin/env node

require('../lib/cli');
```

大功告成！代码已经构建完毕了，可以运行`node bin/create-my-app.js`庆祝一下了！   
但先别急，还有最最最后一步操作，**发布**

```bash
# 安装np用于发布控制
yarn add np -D
```

`np`同时可以执行一些npm script作为hook，比如我们可以不用手动执行`yarn build`构建，而是让`np`在发布之前自动构建

```json
// package.json

{
   ...,
   "scripts": {
      "clean": "rimraf lib",
      "build": "npm run clean && father-build build",
      "release": "np",
      "version": "npm run build"
   },
   ...
}
```

然后运行`yarn release`发布吧！前提是必须得有一个npm账号

## 使用CLI

在发布到npm仓库之后，我们就可以使用我们的CLI了，用法很简单，只需要运行一行短短的命令：
```bash
npx create-my-app
```

熟悉的场景再次出现在我们眼前！

## 总结

手撸CLI属实不易，还好找到一些很有参考价值的资料，并且参考`create-umi-app`的项目结构，这个CLI和这篇文章才由此诞生，感谢社区！  
参考资料：
- [手把手带你撸一个cli工具](https://segmentfault.com/a/1190000018660672)
- [create-umi-app](https://github.com/umijs/umi/tree/master/packages/create-umi-app)