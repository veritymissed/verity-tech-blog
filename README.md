# 由Hexo架設的Blog


## Deploy前的設定

### 1.

`_config.yml`裡面，拉到最下方deploy的部分

```yml

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/veritymissed/verity-tech-blog
  branch: gh-pages
```

- `repo`改成這個專案的github repo url，
- `branch`要改成`gh-pages`，這樣`hexo deploy`後產生出來的靜態檔案（通常都是以年份開頭的資料夾如$PROJECT/2020），只會commit在remote repo的gh-pages分支。

### 2.

Github 自訂domain的`CNAME`檔，需要放在`/source/CNAME`。
`hexo generate`後就會產生在根目錄。


## Deploy的指令
```
hexo clean
hexo generate
hexo deploy
```
