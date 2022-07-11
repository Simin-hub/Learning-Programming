1.安装nodejs和npm

​	sudo apt-get install nodejs

​	sudo apt-get install npm

查看版本：

​	node -v

​	npm -v

2.安装全局npm和n

​	sudo npm i -g npm

​	sudo npm install -g n

3.安装稳定版n

​	sudo n lts

4.安装全局webpack

​	sudo npm install webpack -g

​	sudo npm install webpack-cli -g

​	查看版本：

​	webpack --version

5.初始化

​		npm init

​		生成package.json

6.配置webpack配置文件

​	webpack.config.js

```
const path = require('path');
module.exports = {
    entry: "./src/index.js",//入口文件
    output: {
        path: path.join(__dirname, "/dist"),//输出文件夹
        filename: "[name].js"//输出文件名
    },
    module: {
        rules: [
            {
                //以css为结尾的文件
                test: /\.css$/,
                use: ['style-loder', 'css-loader'] //插件
            }, {
            //以less为结尾的文件
                test: /\.less$/,
                use: ['style-loder', 'css-loader', 'less-loader'] 
            }
        ]
    }
}
```

7.入口文件

​	主要通过入口文件引用其他文件资源，例如css文件、less文件、sass文件

```
require("./index.css");
require("./bootstrap-3.3.7/less/bootstrap.less")
```

8.出口文件

​	出口文件主要是把入口文件所引用的文件和资源家编译，然后在页面中引用出口文件

```
<script src="./dist/main.js"></script>
```







[配置文件参考自：](https://blog.csdn.net/p312011150/article/details/78019863?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task#t3)	

