---
title: "Gitで個人用と会社用プロジェクトの設定を分ける方法"
date: 2022-10-28
lastmod: 2024-11-15
draft: false
slug: how-to-config-git-to-use-different-config-files-by-condition
---

## 問題

個人と会社の Git プロジェクトがあります。どうやって会社と個人プロジェクトの設定を分けられますか？

例えば会社のプロジェクトではユーザ名は会社のメールアドレスですが、個人のプロジェクトでは個人のメールアドレスになります。これを個々のプロジェクトで設定する必要がなく、一括で設定する方法があります。

## 解決方法

例えば、以下のフォルダ構造とします。

- ~/workspace/personal_projects/
  - personal_project1
  - personal_project2
- ~/workspace/work_projects/
  - work_project1
  - work_project2

`.gitconfig` の `includeIf` を使うと、フォルダによって、違う設定ファイルを指定できます。

例として、以下の三つの設定ファイルでフォルダによって違う設定ファイルをロードすることが実現できます。

`~/.gitignore`:

```.gitignore
[includeIf "gitdir:~/workspace/work_projects/"]
    path = ~/.gitconfig_work
[includeIf "gitdir:~/workspace/personal_projects/"]
    path = ~/.gitconfig_personal
```

`~/.gitconfig_work`:

```.gitignore
[user]
    name = myname
    email = myname@work.example.com
```

`~/.gitconfig_personal`:

```.gitignore
[user]
    name = myanothername
    email = myanothername@personal.example.com
```

## 参考

- [Is it possible to have different Git configuration for different projects?](https://stackoverflow.com/a/43884702/4910876)
- [git-config includes](https://git-scm.com/docs/git-config#_includes)
