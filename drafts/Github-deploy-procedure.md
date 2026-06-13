# Github Deploy Procedures

``` bash
# 第一步：先將你的 Markdown 原始碼與設定檔備份到 GitHub 的 main 分支
git add .
git commit -m "Update index and tps40200 part2"
git push origin main

# 第二步：再把編譯好的網頁推送到 gh-pages 分支，正式上線！
mkdocs gh-deploy
```

# reference
- [Git入門指南_公開版_v1](https://docs.google.com/document/u/0/d/1D6T35rzn-vbdkLtfKkoqHIHYgYAaCCl7Ew5Fs5kisuo/mobilebasic)
