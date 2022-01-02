---
title: "dev.toの記事もGitで管理する"
emoji: "🔖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["dev.to"]
published: true
---

## TL;DR

- [Manage your dev\.to blog posts from a GIT repo and use continuous deployment to auto publish/update them \- DEV Community 👩‍💻👨‍💻](https://dev.to/maxime1992/manage-your-dev-to-blog-posts-from-a-git-repo-and-use-continuous-deployment-to-auto-publish-update-them-143j?signin=true)
- [.github/workflows/build\.yml](https://dev.to/beeman/automate-your-dev-posts-using-github-actions-4hp3)
- `yarn upgrade --latest`

## 仕様について

- [dev.toの仕様](https://developers.forem.com/api/#operation/updateArticle)が変わっているため、`yarn upgrade --latest` でテンプレートを更新する必要がある
- [dev.to](https://dev.to)側で発行されるIDが記事の更新に必要な関係上、記事の作成はdev.to上で行う必要がある
- 記事のURLは `canonical_url` で指定できる

## リポジトリ

[xhiroga/dev\.to](https://github.com/xhiroga/dev.to)
