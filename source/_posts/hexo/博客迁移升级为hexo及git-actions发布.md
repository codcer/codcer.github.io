---
layout: hexo
title: 博客迁移升级为hexo及git actions发布
tags: [博客,hexo]
slug: ea799fdc
keywords: hexo,博客
date: 2024-10-25 23:07:34
---

好长时间没有写博文了，现在忙于找工作中，突然想把之前自己的博客升级的几个步骤，大概描述下：

> 功能
    
1. 迁移至hexo，简单易用，支持pc和便携设备; ☑︎
2. 皮肤定制;☑︎
3. 匿名评论;☑︎
4. 首页名言打字机效果；☑︎
5. github actions自动化；☑︎
6. 访问数统计并关联至`github`；☐

## 博客 迁移至hexo
   
   按照[hexo安装](https://hexo.io/zh-cn/docs/#%E5%AE%89%E8%A3%85-Hexo)

   ```shell
    npm install -g hexo-cli 
    # npm install hexo
   ```

   常用的命令

   ```shell
    hexo new [layout] <title>

    # -p, --path	文章的路径。 自定义文章的路径。
    # -r, --replace	如果存在的话，替换当前的文章。
    # -s, --slug	文章别名。 自定义文章的 URL。
   ```

## 皮肤定制;
   
    1、进入hexo初始化的仓库，切换到theme目录下，执行如下操作

    ```shell
        git clone https://github.com/probberechts/hexo-theme-cactus
    ```

    2、按照[cactus主题](https://github.com/probberechts/hexo-theme-cactus)的说明操作

## 匿名评论;

   因hexo默认没有内置匿名评论，因此这里，结合网上的资料，给`cactus`主题集成匿名评论功能；

   ```ts
    // themes/cactus-dark/layout/_partial/comments.ejs
    <% if(page.comments){ %>
        <div class="blog-post-comments">
            // disqus 是hexo自带须登录的评论功能
            <% if(theme.disqus_shortname){ %>
            <div id="disqus_thread">
                <noscript><%= __('comments.no_js') %></noscript>
            </div>
            <% } %>
            // valine 匿名评论
            <% if(theme.valine.enable){ %>
                <div class="vcomment"></div>
            <% } %>
        </div>
    <% } %>
   ```

   ```ts
    // themes/cactus-dark/layout/_partial/scripts.ejs
    <!-- Valine Comments -->
    <% if (theme.valine.enable){ %>
        // 用到的这些库，都存到js目录下
        <%- js('js/av-min.js') %>
        <%- js('js/Valine.min.js') %>
        <script type="text/javascript">
            var notify = '<%= theme.valine.notify %>' == true ? true : false;
            var verify = '<%= theme.valine.verify %>' == true ? true : false;
            var GUEST_INFO = ['nick','mail','link'];
            var guest_info = '<%= theme.valine.guest_info %>'.split(',').filter(function(item){
                return GUEST_INFO.indexOf(item) > -1
            });
            guest_info = guest_info.length == 0 ? GUEST_INFO :guest_info;
            new Valine({
                el: '.vcomment',
                notify: notify,
                verify: verify,
                appId: "<%= theme.valine.appId %>",
                appKey: "<%= theme.valine.appKey %>",
                avatar:"<%= theme.valine.avatar %>",
                placeholder: "<%= theme.valine.placeholder %>",
                guest_info:guest_info,
                pageSize:"<%= theme.valine.pageSize %>"
            })
        </script>
    <% } %>
   ```

   ```yml
   # themes/cactus-dark/_config.yml 加入这段配置
   disqus:
    enabled: false
    shortname: cactus-1

    # Valine comment system. https://valine.js.org
    valine:
    enable: true # if you want use valine, please set this value to true
    appId:  nRach5vXbSulMeiRHgqCbUCT-gzGzoHsz # your leancloud appId
    appKey: CK2vLdlylRq40EiH2lu4lU2H # your leancloud appKey
    notify: false # Mail notify
    verify: false # Verify code
    avatar: mm # Gravatar style : mm/identicon/monsterid/wavatar/retro/hide
    placeholder: 说点什么 # Comment Box placeholder
    guest_info: nick,mail,link # Comment header info
    pageSize: 10 # comment list page size

  ```

## 首页名言打字机效果；
    ```ts
    $(function () {
        if (location.pathname !== "/") return;
        // 这个文件是node任务自动生成的json文件，生产每天的名言警句【收录中】，
        $.get("/data/today.json", function (data) {
            var str = (data.content || "") + "\n" + (data.translation || "");

            var options = {
                strings: [
                    str + "\nWelcome to my blog. ^1000",
                    str + "\nHave a good day. ^1000",
                    str + "\nYou can follow me on the Github. ^2000",
                    str + `\n---- ${data.author}. ^1000`,
                ],
                typeSpeed: 20,
                startDelay: 300,
                // loop: true,
            };
            var typed = new Typed(".description .typed", options);
        });
    });
    ```

## github actions自动化

    ```yml
    # _config.yml
    # 常用的配置列出
    # Site
    title: Liu
    subtitle: Hello Liu | 前端全栈! - WEB FULL STACK! BANG!! BANG!!!
    description: '前端全栈. 持续学习,持续写博客. 此生理想、近期计划、今日功课。'
    keywords: [前端全栈,WEB FULL STACK,前端,前端博客,web前端博客,前端模块化,前端工程化,前端数据监控,性能优化,网页制作,前端,js,html5,css,踩坑小报告,微前端,树莓派,前端开发,区块链,网络,Mongodb,Vue.js,Angular.js,node.js]
    author: Codcer
    language: zh-CN
    timezone: Asia/Shanghai

    # URL
    ## Set your site url here. For example, if you use GitHub Page, set url as 'https://username.github.io/project'
    url: https://codcer.github.io
    permalink: :year/:month/:day/:title/


    ## Themes: https://hexo.io/themes/
    theme: cactus-dark

    # Deployment
    ## Docs: https://hexo.io/docs/one-command-deployment
    deploy:
    type: git
    repo: 
        github:
        url: https://github.com/codcer/codcer.github.io.git
        # 关联的分支
        branch: main 
        # github tokens
        token: $DEPLOY_BLOG
    ```

    ```yml
        # 在对应的github仓库下点击`actions`按钮后，再`new workflow`
        # .github/workflows/hexo.yml 
        name: Deploy Hexo Blog

        on:
        push:
            branches: [ "main" ]

        jobs:
        build:

            runs-on: ubuntu-latest

            strategy:
            matrix:
                node-version: [20.10.0]
                # See supported Node.js release schedule at https://nodejs.org/en/about/releases/

            steps:
            - name: Checkout code
            uses: actions/checkout@v4
            
            - name: Use Node.js ${{ matrix.node-version }}
            uses: actions/setup-node@v4
            with:
                node-version: ${{ matrix.node-version }}
                # cache: 'npm'

            - name: Cache Node Modules
            uses: actions/cache@v4
            with:
                path: node_modules
                key: ${{ runner.OS }}-node-${{ hashFiles('**/package-lock.json') }}
                restore-keys: |
                ${{ runner.OS }}-node-
                
            - name: Install dependencies and build
            env: 
                DEPLOY_BLOG: ${{secrets.DEPLOY_BLOG}}
            run: |
                git config --global user.name "codcer"
                git config --global user.email "codcer@gmail.com"
                rm -rf public
                npm install
                npm run build
                echo 'Build done'
                
            - name: Deploy to GitHub Pages
            uses: peaceiris/actions-gh-pages@v4
            with:
                github_token: ${{ secrets.DEPLOY_BLOG }}
                publish_dir: ./public
    ```


## 访问数统计并关联至`github`；