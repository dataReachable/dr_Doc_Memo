

* 错误截图
![add_of_undifined](./images/add_of_undifined.png)


* 网上方案[issue](https://github.com/babel/minify/issues/556)

* 核心解决方案
    - 将babel.config.json中的`env` 下的`presets`中的`minify`添加子项：`"mangle": false`即可
    
    - 如图 
      > ![add_of_undefined_resolved](./images/add_of_undefined_resolved.png)

