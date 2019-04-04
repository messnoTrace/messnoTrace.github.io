# Feather

基于[light](https://github.com/tommy351/hexo-theme-light)的个人博客主题

## Install

Execute the following command and modify `theme` in `_config.yml` to `feather`.

```
git@github.com:yunlzheng/hexo-theme-feather.git themes/feather
```

## Update

Execute the following command to update feather.

```
cd themes/feather
git pull
```

## Config

Default config:

``` yaml
menu:
  博客: /
  存档: /archives
  关于: /about

widgets:
- search
- recent_posts
- category
- tag


excerpt_link: Read More

twitter:
  username:
  show_replies: false
  tweet_count: 5

addthis:
  enable: true

fancybox: true

google_analytics:
rss:

comment_provider: duoshuo

# Facebook comment
facebook:
  appid: 123456789012345
  comment_count: 5
  comment_width: 840
  comment_colorscheme: light

# Duoshuo comment
duoshuo:
  short_name: moo123

society:
  enable: true
  github: https://github.com/yunlzheng
  weibo: http://weibo.com/503error
  google_plug: http://www.google.com

#About Page
about:
  name:
```

- **menu** - 导航菜单项
- **widget** - 侧边栏工具
- **excerpt_link** - "Read More" link text at the bottom of excerpted articles
- **addthis** - 加网分享按钮
  - **enable** - 是否启用加网分享
- **fancybox** - Enable [Fancybox]
- **google_analytics** - Google Analytics ID
- **rss** - RSS subscription link (change if using Feedburner)
-**duoshuo**: 国内还是用多说来的比较稳定
  -**short_name**: 在多说注册的short_name 

## Features


## Todo
  
   * 添加jsfiddle支持
