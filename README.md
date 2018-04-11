# blog

npm install

建立themes 进入themes文件夹 将hexo-theme-next删掉 重新再themes下clone这个地址https://github.com/iissnan/hexo-theme-next.git

### 主题配置

hexo-theme-next 下的config.yml文件改动

* scheme: Mist
* 这里要注意的是菜单栏的位置改动以及名称改动
```yml
  menu:
  home: / || home
  archives: /archives/ || archive
  resume: /resume/ || user
  #tags: /tags/ || tags
  #categories: /categories/ || th
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```

about-->resume
关于-->简历 在主题下languages的配置文件zh-Hans.yml