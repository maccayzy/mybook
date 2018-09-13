# 开发环境配置

#### 前端开发环境
> + **NodeJs**
>> + 下载[nodejs](https://nodejs.org/en/download/)
>> + 官方仓库网速太慢，要切换到阿里的源
>>> `$ npm config set registry https://registry.npm.taobao.org/`
>> + 查看是否切换成功
>>> `$ npm config get registry`
>> + 配置全局安装模块的位置，防止安装到Ｃ盘
>>> `$ npm config set prefix "D:\\nodejs\node_global"`
>> + 设置环境变量
>>> 系统变量下新建【NODE_PATH】，输入D:\nodejs\node_global\node_modules
>>> 用户变量 下的【Path】修改为 D:\nodejs\node_global

> + **Vue**
>> 安装vue脚手架
>>> ```
>>> $ npm install vue --save
>>> $ npm install --global vue-cli
>>> $ npm init webpack myproject
>>> ```

#### 后端开发环境
> + **Java**
> + **Python**
> + **Maven** 

#### 项目管理
> + **Git**
> + **Svn**