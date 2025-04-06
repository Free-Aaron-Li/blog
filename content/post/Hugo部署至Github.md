+++
date = '2025-04-06T18:44:11+08:00'
draft = false
title = 'Hugo部署至Github'
+++

# 部署配置

Github Action 配置文件：

```yaml
name: deploy

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: "latest"
          extended: true

      - name: Build Web
        run: |
          hugo -F --cleanDestinationDir 
          mkdir -p public
          cp -r ./public/* ./

      - name: Deploy Web
        uses: peaceiris/actions-gh-pages@v4
        with:
          PERSONAL_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
          EXTERNAL_REPOSITORY: Free-Aaron-Li/blog
          PUBLISH_BRANCH: master
          PUBLISH_DIR: ./
          commit_message: auto deploy
```