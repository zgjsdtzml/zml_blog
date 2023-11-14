---
date: 2023-11-13 16:29:02
linktitle: 基于hugo搭建blog

title: 基于hugo搭建blog
weight: 10
tags : [
    "hugo",
]
---

## Step 1. 准备条件

- git下载
- hugo下载

## Step 2. 搭建过程

1. 环境准备

   - 下载hugo [window版本](https://gohugo.io/installation/windows/)
   - 配置环境变量到Path 

2. 创建hugo项目 （zml_blog）

   -  <font color=red>**hugo new site**</font> zml_blog
   - cd zml_blog
   - git init

3. git拉取某个选定的theme皮肤

   - 这里选定blackburn（参考[博客](https://pkolaczk.github.io/software/)和[UI](https://github.com/yoshiharuyamashita/blackburn)）
   - git submodule add git@github.com:yoshiharuyamashita/blackburn.git themes/blackburn

4. theme文件复制并替换

   ```
    除了git.ignore外
    把D:\2 work software\hugo\my_blog\zml_blog\themes\blackburn下
    文件复制到根目录 zml_blog下并替换
   ```

5. 复制theme的配置文件

   ```
   把config文件 
   D:\2 work software\hugo\my_blog\zml_blog\themes\blackburn\exampleSite内容
   复制到D:\2 work software\hugo\my_blog\zml_blog\content下
   ```

   注意最终生效的配置文件为hugo.toml

6. 复制theme配置页面

   ```
   把page文件
   D:\2 work software\hugo\my_blog\zml_blog\themes\blackburn\exampleSite\content内容
   复制到toml.config
   ```

   

7. 重新运行 

   ```
   hugo server
   ```

8. 编译生成public并上传到git

   ```
   D:\2 work software\hugo\my_blog\zml_blog\public下文件
   复制到D:\2 work software\hugo\my_blog\git_pages\zgjsdtzml.github.io下
   git add.
   git commit -m 'xxx'
   git push
   ```

   

9. add/update博客文章

   ```
   
   只需按照Markdown格式编写自己的文章
   然后将写好的文章放在zml_blog/content/posts
   hugo就会读取到这片文章，并将这片文章展示博客中
   ```



## CSS修改

- 本地调试

  ```
  D:\2 work software\hugo\my_blog\zml_blog\static\css中修改css
  ```

- 后续步骤即8

