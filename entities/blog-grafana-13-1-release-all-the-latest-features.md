---

title: "Grafana 13.1 release: observability as code updates, extending Grafana Assistant across more data sources, and more"
created: 2026-06-26
updated: 2026-08-01
type: entity
tags: [article]
source: "[[raw/articles/blog-grafana-13-1-release-all-the-latest-features]]"
sources:
  - raw/articles/blog-grafana-13-1-release-all-the-latest-features
review_value: 7
review_confidence: 8
review_stars: 4
review_recommendation: worth-reading
---

# Grafana 13.1 release: observability as code updates, extending Grafana Assistant across more data sources, and more

> **来源**: [Grafana 13.1 release: observability as code updates, extending Grafana Assistant across more data sources, and more](https://grafana.com/blog/grafana-13-1-release-all-the-latest-features/)


Published Time: 2026-06-24^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]


Markdown Content:
![Image 1: Grafana 13.1 release: observability as code updates, extending Grafana Assistant across more data sources, and more](https://grafana.com/mw/_next/image/?url=https%3A%2F%2Fa-us.storyblok.com%2Ff%2F1022730%2F1200x628%2Fe0de6fdd6d%2Fgrafana-13-1-meta-image.png&w=3840&q=75)^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]


•

2026-06-24•9 min

[![Image 2: Twitter](https://a-us.storyblok.com/f/1022730/18x14/455626a417/icon-twitter-gray.svg)](https://twitter.com/intent/tweet?text=Grafana%2013.1%20release%3A%20observability%20as%20code%20updates%2C%20extending%20Grafana%20Assistant%20across%20more%20data%20sources%2C%20and%20more&url=https%3A%2F%2Fgrafana.com%2Fblog%2Fgrafana-13-1-release-all-the-latest-features%2F)[![Image 3: Facebook](https://a-us.storyblok.com/f/1022730/8x14/08ec63c702/icon-facebook-gray-sm.svg)](https://www.facebook.com/sharer/sharer.php?u=https%3A%2F%2Fgrafana.com%2Fblog%2Fgrafana-13-1-release-all-the-latest-features%2F)[![Image 4: LinkedIn](https://a-us.storyblok.com/f/1022730/16x16/6788867fe2/icon-linkedin-grey.svg)](https://www.linkedin.com/sharing/share-offsite/?url=https%3A%2F%2Fgrafana.com%2Fblog%2Fgrafana-13-1-release-all-the-latest-features%2F) ^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]

Earlier this year, [Grafana 13 laid the groundwork](https://grafana.com/blog/grafana-13-release-all-the-latest-features/?pg=grafana-13-1-release-all-the-latest-features&plcmt=in-text) for making it easier and faster than ever to turn your data into actionable insights. ^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]

With our latest minor release, Grafana 13.1, we're building on that foundation, expanding observability as code, bringing Grafana Assistant to more data sources, and streamlining the everyday workflows teams rely on to visualize, analyze, and act on their data. ^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]

Below are just some of the highlights from Grafana 13.1. If you want to explore _all_ the latest updates, please refer to the [changelog](https://github.com/grafana/grafana/blob/main/CHANGELOG.md) or our [What’s New documentation](https://grafana.com/docs/grafana/latest/whatsnew/whats-new-in-v13-1/?pg=grafana-13-1-release-all-the-latest-features&plcmt=in-text). ^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]

## [](http://grafana.com/blog/grafana-13-1-release-all-the-latest-features/#managing-dashboards-as-code-whats-new-in-git-sync)Managing dashboards as code: what's new in Git Sync

[Git Sync](https://grafana.com/docs/grafana/latest/as-code/observability-as-code/git-sync/?pg=grafana-13-1-release-all-the-latest-features&plcmt=in-text), a feature that brings native GitOps workflows into your Grafana instance, [reached general availability](https://grafana.com/blog/git-sync-grafana/?pg=grafana-13-1-release-all-the-latest-features&plcmt=in-text) with the release of Grafana 13. We added features to give you more flexibility and control when managing your dashboards as code, including GitHub App authentication and support for GitLab, BitBucket, and pure Git. ^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]

But we didn’t stop there. Grafana 13.1 brings four more enhancements to Git Sync that make it even easier to incorporate observability as code into your day-to-day workflows. ^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]

### [](http://grafana.com/blog/grafana-13-1-release-all-the-latest-features/#import-dashboards-straight-into-a-provisioned-folder)Import dashboards straight into a provisioned folder

_Generally available in all editions of Grafana_^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]


You can now [import dashboard JSON](https://grafana.com/whats-new/2026-06-23-git-sync--import-dashboards-from-the-ui-to-simplify-adding-them-in-a-synced-folder/?pg=grafana-13-1-release-all-the-latest-features&plcmt=in-text) straight into a Git Sync-provisioned folder, picking the file path, branch, commit message, and workflow as part of the import. ^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]

From a folder, hit **Import** and Grafana walks you through a provisioned import flow: pick the file path, branch, commit message, and workflow, and the dashboard is committed back to your repository as part of the import. ^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]

Uniqueness is path-based, so two dashboards can share a title as long as they live at different paths in the repo, and a conflicting path stops the import before anything is overwritten. ^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]

### [](http://grafana.com/blog/grafana-13-1-release-all-the-latest-features/#sync-dashboards-at-the-root-level)Sync dashboards at the root level

_Generally available in all editions of Grafana_^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]


You can now [sync dashboards at the root level](https://grafana.com/whats-new/2026-06-23-git-sync--dashboard-synchronisation-now-available-at-root-level/?pg=grafana-13-1-release-all-the-latest-features&plcmt=in-text), without a containing folder, so provisioned dashboards can live alongside your non-provisioned ones. This is useful when a repo represents your whole Grafana setup, or when forcing everything under one folder doesn’t align with how your team organizes dashboards. ^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]

Pick **Sync external storage directly at root level without a containing folder** in the setup wizard and your provisioned dashboards land at the root, alongside everything else, instead of being scoped under a single folder. ^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]

### [](http://grafana.com/blog/grafana-13-1-release-all-the-latest-features/#make-dashboard-context-visible-by-default)Make dashboard context visible by default

_Available in public preview in all editions of Grafana_ ^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]

Git Sync-provisioned folders [now render their](https://grafana.com/whats-new/2026-06-23-git-sync--readmemd-files-added-to-a-folder-in-git-are-displayed-in-the-ui/?pg=grafana-13-1-release-all-the-latest-features&plcmt=in-text)`README.md` inline by default, so the context for a folder travels with it. ^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]

Just drop a `README.md` next to your dashboards in the repo and it shows up in Grafana, including links, ownership notes, runbooks, or whatever your team wants to see sitting alongside their dashboards. ^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]

### [](http://grafana.com/blog/grafana-13-1-release-all-the-latest-features/#sign-commits-automatically)Sign commits automatically

_Generally available in in all editions of Grafana_ ^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]

Git Sync can now [sign commits with GPG, SSH, or S/MIME keys](https://grafana.com/whats-new/2026-06-23-git-sync--verified-commits/?pg=grafana-13-1-rele ^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]

→ [[raw/articles/blog-grafana-13-1-release-all-the-latest-features|原文存档]] ^[raw/articles/blog-grafana-13-1-release-all-the-latest-features.md]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

