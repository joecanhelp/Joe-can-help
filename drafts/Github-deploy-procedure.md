# Github Deploy Procedures

```
# 第一步：先將你的 Markdown 原始碼與設定檔備份到 GitHub 的 main 分支
git add .
git commit -m "Update index and tps40200 part2"
git push origin main

# 第二步：再把編譯好的網頁推送到 gh-pages 分支，正式上線！
mkdocs gh-deploy```
