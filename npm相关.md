**npm install**   本地安装

- 可以通过 `require()` 来引入本地安装的包

**npm install -g**  全局安装

- 将安装包放在 `/usr/local` 下或者你 `node` 的安装目录。

**npm install <module_name> -S 即 npm install <module_name> --save**

- 写入dependencies
- 之后运行`npm install --production`或者注明`NODE_ENV`变量值为`production`时，**会自动安装**`msbuild`到`node_modules`目录中

**npm install <module_name> -D 即 npm install <module_name> --save-dev**

- 写入devDependencies
- 之后运行`npm install --production`或者注明`NODE_ENV`变量值为`production`时，**不会自动安装**`msbuild`到`node_modules`目录中

### dependencies 与 devDependencies 的区别

- dependencies 是需要发布到生产环境的

- devDependencies 里面的插件只用于开发环境，不用于生产环境


