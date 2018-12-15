## 创建antd-design-pro项目

* 下载安装包

```
    wget -c -O ant-design-pro-2.1.1.tar.gz https://github.com/ant-design/ant-design-pro/archive/2.1.1.tar.gz
```
* 解压安装包
    
```
    tar -zxvf ant-design-pro-2.1.1.tar.gz
```
* 修改文件名

```
    mv ant-design-pro-2.1.1 skylla_test
```
* 进入目录git 初始化

```
    cd skylla_test
    git init
    git add .
    git commit -m "Init from skylla_test"

```
* 删除package.json中的 `husky,puppeteer,lint-staged`

* 配置docker环境
    * 先查看当前文件的位置（pwd） `/Users/thrive/Desktop/skylla_test`
    * 放在docker环境下
        `docker run -it --rm -v /Users/thrive/Desktop/skylla_test:/app node:8-alpine sh`
    * 修改Dockerfile.dev 文件，设置yarn的淘宝镜像源

    ```
        COPY package.json ./
        RUN npm config set registry https://registry.npm.taobao.org
        RUN npm install  --no-cache
    ```
    * 修改docker/docker-compose.dev.yml文件，把ant-design-pro修改为skylla_test
    * 在docker/docker-compose.dev.yml文件中添加server
    ```
        networks:
            default:
                external:
                    name: skylla_default
    ```
* 准备工作
    * `docker network create skylla_default`
    * `docker-compose -f ./docker/docker-compose.dev.yml build` 
* 启动服务

    * `docker-compose -f ./docker/docker-compose.dev.yml up`
