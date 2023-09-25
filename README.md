# How to use patch-package

patch-package 允许在不修改原始依赖包源代码的情况下，对其进行补丁修复。

安装 patch-package

```bash
npm install patch-package --save-dev
```

在项目的根目录下创建一个名为patches的文件夹

```bash
mkdir patches
```

在patches文件夹下创建一个与text-svg包对应的补丁文件，文件名的格式通常是package-name.patch，其中package-name是第三方包的名称。 比如这里需要修改text-svg 这个包的 dependencies，所以创建一个text-svg.patch文件

```diff
--- a/node_modules/text-svg/package.json
+++ b/node_modules/text-svg/package.json
@@ -18,7 +18,7 @@
  "dependencies": {
    "canvas": "1.0.6"
  },
-  "optionalDependencies": {
+  "optionalDependencies": { }
```

在这个补丁文件中，修改了`textsvg`包的`package.json`文件，将其`canvas`的依赖版本从1.0.7改为1.0.6。另外，我们将`optionalDependencies`字段设置为空对象，以确保不会影响到其他依赖项。

运行以下命令来应用补丁：

```bash
npx patch-package text-svg
```

为了确保其他人克隆你的项目时也能自动应用这些补丁，将生成的补丁文件添加到你的版本控制系统中，并将其提交到代码库中。 在项目中创建postinstall脚本，以便在每次安装依赖时自动应用补丁。

```bash
#!/bin/sh

# Apply patch-package patches after npm install
npx patch-package
```

这段脚本会在每次运行`npm i`（或`npm install`）时自动调用`patch-package`命令来应用补丁。

在package.json文件中的scripts字段中添加一个postinstall脚本，指向你刚创建的postinstall文件：

```json
"scripts": {
  "postinstall": "./postinstall"
}
```

并且确保你的package.json文件中包含了patch-package作为开发依赖项：

```json
"devDependencies": {
  "patch-package": "^X.X.X"
}
```

这样，当其他人克隆你的项目并运行npm i时，postinstall脚本会自动调用patch-package命令，并应用你提交的补丁文件。
