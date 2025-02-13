首先安装`pip install jieba`。

最后在`mkdocs.yml`中配置：
```
plugins:
  - search:
      separator: '[\s\u200b\-]'
```

***
- [chinese-search-support](https://squidfunk.github.io/mkdocs-material/blog/2022/05/05/chinese-search-support/?h=search)
