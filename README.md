# qiita-article-best-before

Set header to old articles.

## Configure

Export two envs.

```
export QIITA_TOKEN='xxxxxxxxxxxxxxxxxx'
export QIITA_USER='your-id'
```

## Usage


```
rake all          # put warning header to all items
rake open[id]     # Open article with browser
rake protect[id]  # Protect from expiring timeless article
rake show         # show all items
```
