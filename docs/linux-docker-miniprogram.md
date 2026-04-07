# Linux Docker + 小程序运行说明

## 1. 直接运行这个仓库，实际需要什么

如果你只是想把项目跑起来，不重新编译源码，最少需要下面这些条件：

1. Linux 服务器一台
2. Docker Engine 和 `docker compose` 插件
3. 数据库初始化脚本 `xzs-mysql.sql`
4. 仓库里自带的发布产物：
   `release/java/xzs-3.9.0.jar`

这个仓库已经自带了后端 jar，jar 内也已经包含了 `student` 和 `admin` 两套静态页面，所以单纯跑 Web 端时，不需要再额外安装 Node.js、npm、Maven。

数据库脚本不在仓库里，仓库只给了下载说明：

- `sql/README.md`
- `docker/README.md`
- `release/README.md`

导入前需要把 SQL 文件开头改成：

```sql
CREATE DATABASE `xzs` CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
USE xzs;
```

然后把文件放进仓库根目录的 `sql/` 下面，比如：

```text
sql/xzs-mysql.sql
```

## 2. Linux Docker 启动方式

仓库里原来的 `docker/docker-compose.yml` 依赖作者提供的镜像，并且 Java 容器使用 `host` 网络。这里补了一个更适合你自己在 Linux 上直接跑的编排文件：

- `docker-compose.linux.yml`

启动命令：

```bash
cd /path/to/xzs-mysql
docker compose -f docker-compose.linux.yml up -d
docker compose -f docker-compose.linux.yml logs -f xzs
```

启动成功后：

```text
学生端: http://你的服务器IP:8000/student
管理端: http://你的服务器IP:8000/admin
后端接口: http://你的服务器IP:8000/api
```

停止命令：

```bash
docker compose -f docker-compose.linux.yml down
```

## 3. 小程序需要额外做什么

小程序代码在：

- `source/wx/xzs-student`

注意，小程序本身不是用 Docker 跑的。Docker 只负责跑 MySQL 和 Java 后端。小程序要用微信开发者工具单独打开这个目录。

你需要改 3 个地方：

1. 小程序接口地址
   文件：`source/wx/xzs-student/config.js`
   把 `baseAPI` 改成你的 Linux 服务器地址，例如：

```js
module.exports = {
  baseAPI: "http://192.168.1.20:8000"
}
```

2. 小程序 `appid`
   文件：`source/wx/xzs-student/project.config.json`
   把里面的 `appid` 改成你自己的微信小程序 `appid`

3. 后端微信配置
   文件：`source/xzs/src/main/resources/application.yml`
   修改这两个值：

```yml
system:
  wx:
    appid: 你自己的appid
    secret: 你自己的secret
```

## 4. 小程序联调时的限制

如果只是开发者工具里调试：

- 仓库当前 `project.config.json` 里已经是 `urlCheck: false`
- 你可以先用 `http://局域网IP:8000`

如果要真机访问或正式发布：

1. 后端必须换成公网可访问的 HTTPS 域名
2. 小程序 `baseAPI` 必须改成 HTTPS 地址
3. 还要把这个域名配置到微信公众平台的小程序“服务器域名”里

也就是说：

- 本地调试可以用 `http://IP:8000`
- 真机和上线必须用 `https://你的域名`

## 5. 如果你还想从源码重编译

只有在你打算重新打包后端或前端时，才需要这些环境：

1. JDK 8
2. Maven 3.6+
3. Node.js 14 或 16
4. npm

源码目录：

- 后端：`source/xzs`
- 学生端 Vue：`source/vue/xzs-student`
- 管理端 Vue：`source/vue/xzs-admin`
- 微信小程序：`source/wx/xzs-student`
