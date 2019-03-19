---
title: 搭建个人网站(github+hexo)
date: 2019-02-05 20:18:40
tags: [github, hexo]
id: l2
categories: 教程
---

> 从零开始搭建一个属于自己的网站

##  主任务

###  准备内容

> github 账号,nodejs

### 创建项目

在 github 创建一个 project，命名格式:用户名.github.io(例如：liaoyf.github.io)

### 安装 hexo

```bash
mkdir l_website #创建文件夹随便命名 例 l_website
npm install -g hexo #全局安装hexo
cd l_website
hexo init
hexo server #启动服务 默认端口是4000
```

访问 localhost:4000 就可以看到默认的页面

### 部署网站

修改 l_website 下的 \_config.yml 最下面的代码如下

```yaml

---
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:用户名/用户名.github.io.git #例：git@github.com:liaoyf/liaoyf.github.io.git
  branch: master
  message: blog
```

安装 hexo-deployer-git

```bash
npm install hexo-deployer-git --save #安装 hexo-deployer-git
hexo delopy #自动提交 简写 hexo d
```

**打开 https://用户名.github.io 我的是https://liaoyf.github.io 完美！**

### 解析自己域名至 github

如果想用自己的域名访问：

用户名.github.io 项目下新增 CNAME 文件并提交。直接在 l_website/source 目录下，添加 CHAME 文件，写入自己的域名并提交。命令行操作如下：

```bash
cd l_website/source #在source目录下直接添加CNAME文件
echo 自己的域名>CHAME #例如 liaoyf.com>CHAME
hexo g
hexo d
```

查询自己的 github 地址 ip,我这边是 185.199.111.153

```bash
ping 用户名.github.io
```

配置自己的域名以阿里云为例

1. 按照流程申请自己的域名(xxx.com)
2. 添加域名解析如图

![图1](/images/domain1.png)
![图2](/images/domain2.png)
![图3](/images/domain3.png)

3. 等十分钟刷新 xxx.com 即可

## 支线任务

### 写 HelloWord

```bash
hexo new "HelloWord" #创建新博客
hexo generate --watch #实时监听文件的变化，访问localhost:4000 刷新即可看见
```

 打开 l_website/source/\_posts/HelloWord.md，编辑内容后执行

```bash
hexo g -d #编译并提交
```

### 更换网页主题（next）

> hexo 默认安装的 landscape，个人比较喜欢 next 的主题，下面就以 next 为例

```bash
cd l_website
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

打开\_config.yml 修改 theme 为 next 后编译并提交

```yaml

---
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```

### 添加主题/标签

> 在 l_website/source 下创建\_data 文件夹，在\_data 创建 next.yml 这个文件会覆盖 next 下的配置。按照自己的需求修改即可。下面只按照 添加主题标签为例

修改配置文件，菜单中增加 tags 和 categories 菜单

```yaml

---
menu:
  home: / || home #主页
  tags: /tags/ || tags #标签
  categories: /categories/ || th #主题
  archives: /archives/ || archive
```

#### 创建标签/主题

```bash
hexo new page tags #创建标签
```

source 下会多个 tags 的目录，打开 source/tags/index.md 修改内容如下：

```yaml
---
title: 标签
date: 2019-02-02 15:47:29
type: 'tags'
---

```

再打开 l_website/scaffolds/post.md 增加 tags，之后每次新增文章会自动写入 tags

```yaml

---
tags: { { tags } }
```

打开之前写的 HelloWord.md,在顶上加入标签

```yaml

---
tags: [github, hexo]
```

创建主题只需重复上述过程把 tags 改为 categories 即可

最后编译并发布完成
