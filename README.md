# Mingcong Liu 的个人主页

线上地址：<https://onion-liu.github.io>（自定义域名 `www.aha-time.com`，见 `CNAME`）

本站基于 [al-folio](https://github.com/alshedivat/al-folio) v1 构建。v1 的布局、样式和脚本都由版本化的 Ruby gem（`al_folio_core` 等）提供，
这个仓库只保存配置和个人内容，因此以后升级主题不再需要合并上游的大量文件。

## 内容在哪里改

| 内容                         | 文件                                                       |
| ---------------------------- | ---------------------------------------------------------- |
| 姓名、站点设置               | `_config.yml`                                              |
| 首页简介、头像、单位         | `_pages/about.md`（头像在 `assets/img/selfie.jpg`）        |
| 论文列表                     | `_bibliography/papers.bib`                                 |
| 论文配图                     | `assets/img/publication_preview/`（bib 中的 `preview` 字段） |
| 新闻                         | `_news/*.md`                                               |
| 邮箱 / GitHub / Scholar / X  | `_data/socials.yml`                                        |
| 合作者主页链接               | `_data/coauthors.yml`                                      |

bib 条目支持的按钮字段：`pdf`、`html`、`code`、`website`、`dataset`、`arxiv`、`slides`、`poster` 等；
设置 `selected={true}` 的论文会显示在首页。

## 本站的定制

- `_includes/hook/bib.liquid`：在期刊/会议名后加粗显示缩写，例如 “… (**NeurIPS**), 2021”。这是 al-folio 官方提供的扩展点。
- `_layouts/bib.liquid`：在 `al_folio_core` 的同名文件基础上只增加了 **Dataset** 按钮。
  该覆盖记录在 `.al-folio-overrides.yml` 中，升级后可用 `bundle exec al-folio upgrade overrides audit` 检查上游是否有变化。

## 部署

推送到 `master` 后，GitHub Actions（`.github/workflows/deploy.yml`）会自动构建并发布到 `gh-pages` 分支。

## 本地预览

```bash
bundle install
bundle exec jekyll serve   # http://localhost:4000
```

或者使用 Docker：`docker compose up`（http://localhost:8080）。

## 升级主题

1. 参考上游最新的 [`Gemfile`](https://github.com/alshedivat/al-folio/blob/main/Gemfile) 修改本仓库 `Gemfile` 中 `al_*` gem 的版本号，然后运行 `bundle update`。
2. 运行 `bundle exec al-folio upgrade audit` 和 `bundle exec al-folio upgrade overrides audit` 检查兼容性。
3. 本地构建确认无误后提交。
