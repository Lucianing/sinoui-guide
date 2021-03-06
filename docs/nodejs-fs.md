---
id: nodejs
title: node.js文件操作
sidebar_label: Node.js文件操作
---

> Node.js 的 [fs](https://nodejs.org/dist/latest-v10.x/docs/api/fs.html) 模块提供了一个 API，用于模仿以标准的 POSIX 函数的方式与文件系统进行交互。推荐使用[fs-extra](https://github.com/jprichardson/node-fs-extra)做日常文件操作。

## 常用 API

### ensureDir(dir,options,callback)

确认目录是否存在，如果不存在，则创建一个，类似于`mkdir -p`。

```javascript
import fs from 'fs-extra';

const path = 'E:\\temp\\';
async function checkDir() {
  try {
    await fs.ensureDir(path);
    console.log('success!');
  } catch (err) {
    console.error(err);
  }
}
```

### pathExists(file,callback)

通过检查文件系统来测试给定路径是否存在。类似于`fs.exists`

```js
async function checkExists() {
  const exists = await fs.pathExists('E:\\temp\test.js');
  if (exists) {
    console.log('文件存在');
  } else {
    console.log('文件不存在');
  }
}
```

### move(src, dest,options,callback)

移动文件或目录,甚至可以跨设备，类似于`mv`

- `src` 源路径
- `dest` 目标路径
- options
  - **overwrite** : 覆盖现有文件或目录，默认为 false。
- `callback` 回调方法

```js
const srcpath = '/tmp/file.txt';
const dstpath = '/tmp/this/path/does/not/exist/file.txt';

async function moveFile() {
  try {
    await fs.move(srcpath, dstpath);
    console.log('success!');
  } catch (err) {
    console.error(err);
  }
}
```

### ensureFile(file,callback)

确认文件是否存在。如果请求创建的文件位于不存在的目录中，则会创建这些目录，如果该文件已经存在，则不进行修改。

```js
const file = '/tmp/this/path/does/not/exist/file.txt';

async function ensureFileFn() {
  try {
    await fs.ensureFile(file);
    console.log('success!');
  } catch (err) {
    console.error(err);
  }
}
```

### 合并文件

将文件 A 的内容合并到文件 B 的末尾。可以使用[Stream](https://nodejs.org/api/stream.html)方式处理文件合并。代码如下所示：

```js
import { createReadStream, createWriteStream } from 'fs';

function mergeFile(from, to) {
  return new Promise((resolve, reject) => {
    const fromStream = createReadStream(from);
    const toStream = createWriteStream(to, {
      flags: 'a',
    });

    const stream = fromStream.pipe(toStream);

    stream.on('end', () => {
      resolve();
    });

    stream.on('error', (error) => {
      reject(error);
    });
  });
}
```

## 查找文件

可以使用[globby](https://github.com/sindresorhus/globby)在指定目录中查找文件。

安装:

```shell
yarn add globby
```

示例：

文件结构：

```
├── unicorn
├── cake
└── rainbow
```

查找除`cake`的文件：

```typescript
import globby from 'globby';

(async () => {
  const paths = await globby(['*', '!cake']);

  console.log(paths);
  //=> ['unicorn', 'rainbow']
})();
```

`globby(patterns, options?)`函数的第一个参数接收的是查找文件或者文件夹的模式表达式。它支持的模式包括：

- `*` - 它会匹配除了`/`之外的任意字符。所以可以用`*`匹配一级目录。
- `**` - 匹配所有任意字符，包括`/`。所以可以用`**`匹配任意级别的目录。
- `?` - 匹配任意一个字符。
- `(ab|cd)` - 正则分组模式，匹配`ab`或者`cd`。
- `[abc]` - 正则字符类模式，匹配`a`、`b`或者`c`字符。
- `{ab,cd}` - brace 模式，匹配`ab`或者`cd`。
- `!` - 否定模式。表示不能包含符合此模式的文件、文件夹。如上面的示例。

更多更详细的模式请参阅[micromatch](https://github.com/micromatch/micromatch)。

`globby()`会用`patterns`参数来判定文件路径是否匹配模式。如果匹配，则作为结果的一个数据项返回。

`globby()`也可以匹配文件夹，如下所示：

```
├── unicorn
  |__ README.md
├── cake
└── rainbow.md
```

```ts
import globby from 'globby';

(async () => {
  const paths = await globby(['*', '!cake'], { onlyFiles: false });

  console.log(paths);
  //=> ['unicorn', 'rainbow.md']
})();
```

## 总结

此篇文章主要列出了项目中用到的几个 Api,更多操作文件的 Api,我们后续会继续补充。

## 参考文章

- [fs-extra](https://github.com/jprichardson/node-fs-extra#nodejs-fs-extra)
- [fs|Node.js API 文档](http://nodejs.cn/api/fs.html)
- [将`fs-extra`代替`fs`使用](https://www.colabug.com/3668248.html)
- [globby](https://github.com/sindresorhus/globby)
- [micromatch](https://github.com/micromatch/micromatch)
