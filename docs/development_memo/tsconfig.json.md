# tsconfig.json

## tsc 编译后不保留目录结构

### rootDir

[rootDir](https://www.typescriptlang.org/tsconfig/#rootDir)：当 `outDir: "build"` 选项被配置了，输出的文件夹结构**本来**会和项目源码结构一样，但是假设你根目录下只有一个 `src` 文件夹（.dockerignore 一般会忽略 `test` 文件夹），那么 tsc 编译输出的 `build` 文件夹中不会有 `src` 文件夹，而是直接将 `src` 下的文件放在 `build` 下。
 
此时如果 `npm run start` 的指令是 `node ./dist/src/bin/www.js`，就会找不到该文件。
 
**修复办法**：增加 `rootDir: "."` 选项