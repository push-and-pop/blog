# blog


### Hexo

#### 安装

```JavaScript
npm install -g hexo

npm install hexo-deployer-git --save

```

themes\redefine下更新git submodule

#### 发布

```YAML
1.修改_config.yml

deploy:
  type: git
  repository: https://github.com/push-and-pop/push-and-pop.github.io
  branch: main
  
2. 执行
hexo d

```

#### 本地预览

```Bash
执行
hexo g             # 生成页面
hexo s             # 本地预览
hexo g -d          # 生成页面并部署
hexo clean         # 清除缓存和已生成的静态文件

```

#### 常用命令

```Markdown
hexo clean         # 清除缓存和已生成的静态文件
hexo new "name"       # 新建文章
hexo new page "name"  # 新建页面
```

#### 主题
[redefine](https://redefine-docs.ohevan.com/zh/introduction)