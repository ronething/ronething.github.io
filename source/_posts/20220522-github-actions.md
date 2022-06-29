---
title: blog github actions config
date: 2022-05-22 10:44:26
tags:
categories:
top_img:
---

## 0x00

blog 配置一下 github actions

<!--more-->

## 0x01

前阵子发现 travis 已经失效，考虑配置一下 github actions

```yaml
name: blog deploy

on:
  push:
    branches:
      - blog # blog 分支 push 的时候才进行作业
    paths:
      - 'source/**'
      - 'themes/**'
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [ 12 ]
    name: A job to deploy blog.
    steps:
      - name: Checkout
        uses: actions/checkout@v1
        with:
          submodules: true # Checkout private submodules(themes or something else).

      - name: 2. set node version
        uses: actions/setup-node@master
        with:
          node-version: ${{ matrix.node-version }}
    
      # Caching dependencies to speed up workflows. (GitHub will remove any cache entries that have not been accessed in over 7 days.)
      - name: Cache node modules
        uses: actions/cache@v1
        id: cache
        with:
          path: node_modules
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-
      - name: Install Dependencies
        if: steps.cache.outputs.cache-hit != 'true'
        run: npm ci # 类似 npm install, 比 npm install 速度更快
    
      # Deploy hexo blog website.
      - name: update blog
        env:
          GITHUB_LOGIN: ronething
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          ./node_modules/hexo/bin/hexo clean && ./node_modules/hexo/bin/hexo g
          cd ./public
          git init
          git config user.name "xxxx"
          git config user.email "xxxx"
          git add .
          git commit -m "update blog"
          git push --force --quiet https://$GITHUB_LOGIN:$GITHUB_TOKEN@github.com/$GITHUB_REPOSITORY.git master:master

      - name: echo
        run: |
          echo "update blog ok"

```

## 0x02

那么希望可以持续更新下 blog
