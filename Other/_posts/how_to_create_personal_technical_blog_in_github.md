使用github page搭建个人技术博客，主要是为了满足以下需求：
1. 写博客方便自由
2. 查找历史记录方便
3. 风格简洁

### 搭建步骤

以下就简单介绍一下搭建博客的步骤：
1. 申请github账号
```
	到github官网注册一个免费个人账户即可
```
2. 创建github仓库
```
	创建一个名称为${user_name}.github.io的仓库，${user_name}必须是登录的用户名
```
3. 设置github page
```
	在这个库的setting中设置如下：
	options -> GitHub Pages
	选择source为master branch
	再点击Choose a theme按钮，选择一个喜欢的主题，然后点击select theme按钮选中
	此时可以在浏览器中访问域名来检查是否成功：${user_name}.github.io
```
4. 配置博客
```
将库clone下来，clone的方式可以选择ssh或者https
clone下来后，修改_config.yml文件，内容如下：
theme: jekyll-theme-slate
title: Bruce
description: "专注MySQL运维"
```
5. 可以开始更新博客了

### 关于github page所使用的jekyll使用帮助
- github的帮助文档:[链接](https://help.github.com/en/articles/configuring-jekyll)
- 怎样使用github pages:[链接](https://help.github.com/en/articles/using-jekyll-as-a-static-site-generator-with-github-pages)
- jekyll的官方配置帮助文档:[链接](https://jekyllrb.com/docs/configuration/)
- jekyll的一个配置示例:[链接](https://github.com/daattali/beautiful-jekyll/blob/master/_config.yml)
- github的markdown语法:[链接](https://guides.github.com/features/mastering-markdown/)
