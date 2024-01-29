## 什么是Hugo
> Hugo是由Go语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。

## 快速开始
### 安装Hugo
1. 二进制安装（推荐：简单、快速）
  * 到 [github](https://github.com/gohugoio/hugo/releases) 下载对应的操作系统版本的Hugo二进制文件（hugo或者hugo.exe）
  * Mac下直接使用 Homebrew 安装：
```bash
brew install hugo
```
2. 源码安装  
略

### 生成站点
使用Hugo快速生成站点，比如希望生成到 /path/to/site 路径：
```bash
hugo new site /path/to/site
```
这样就在 /path/to/site 目录里生成了初始站点，进去目录：
```bash
cd /path/to/site
```
站点目录结构：
> ▸ archetypes/  
  ▸ content/  
  ▸ layouts/  
  ▸ static/  
    config.toml  

### 创建文章
创建一个 about 页面：
```bash
hugo new about.md
```

### 安装皮肤
到 [皮肤列表](https://www.gohugo.org/theme/) 挑选一个心仪的皮肤，比如你觉得 LoveIt 皮肤不错，找到相关的 GitHub 地址，创建目录 themes，在 themes 目录里把皮肤 git clone 下来：
```bash
# 创建 themes 目录
cd themes
git clone https://github.com/spf13/hyde.git
```

### 运行Hugo
在你的站点根目录执行 Hugo 命令进行调试：
```bash
hugo server --theme=LoveIt --buildDrafts
```
浏览器里打开： http://localhost:1313

### 部署
假设你需要部署在 GitHub Pages 上，首先在GitHub上创建一个Repository，命名为：xxx.github.io （xxx替换为你的github用户名）。
在站点根目录执行 Hugo 命令生成最终页面：
```bash
hugo --theme=LoveIt --baseURL="http://xxx.github.io/"
```
（注意，以上命令并不会生成草稿页面，如果未生成任何文章，请去掉文章头部的 draft=true 再重新生成。）

如果一切顺利，所有静态页面都会生成到 public 目录，将pubilc目录里所有文件 push 到刚创建的Repository的 master 分支。
```bash
cd public
git init
git remote add origin https://github.com/xxx/xxx.github.io.git
git add -A
git commit -m "first commit"
git push -u origin master
```
浏览器里访问：http://xxx.github.io/
