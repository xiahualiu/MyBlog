---
title: "使用 VS Code 编辑并预览 LaTeX 文件"
date: 2020-02-11T13:17:03-05:00
draft: false
---

今天这篇博文会介绍如何使用 Visual Studio Code 来编辑和预览 LaTeX 论文。

<!--more-->

## 前置需求

在进入教程之前，首先要确认你安装了如下的程序：

* TeXlive 2019
* Visual Studio Code

可以通过以下 CMD 命令确认以上程序已经安装，（按 Win+R 组合键唤起运行窗口输入 cmd 即可打开命令行界面）

```cmd
xelatex -v
```

如果显示 `not recognized` 或者找不到，就代表你没有安装好 TeXlive，怎么安装 TeXlive 不在这里介绍了。

## 第一步

确保了你安装好了这两个软件后就可以进入下一步了。

第一步我们要打开安装后的 VS Code，在左侧的侧边栏选择插件选项，就是小爬虫下面哪一个选项，四个小方块，然后在插件搜索栏里面输入 latex。

安装第一个名字为 "LaTeX Workshop" 的插件，安装完成后按照指示 "Reload" 就可以了。

## 第二步

确认了第一步完成后，在插件栏里面能够找到新安装的 "LaTeX Workshop" 插件就可以了。

直到现在我们就可以利用这个插件来进行英文论文的书写了，但是还不能编译中文 tex 文件。为了能够顺利的编译中文，我们需要修改默认编译链使用 XeLaTeX 编译器。

下一步比较麻烦，首先打开任意一个文件夹，然后点击文件路径栏，在栏里面输入：

```cmd
%APPDATA%\Code\User\
```

然后回车，跳转到设置文件夹，找到 `settings.json` 文件 (如果没有就自己创建一个)，使用我们安装的 VS Code 打开，然后用以下的内容覆盖：

```json
{
    "latex-workshop.latex.tools": [
        {
            "name": "xelatex-no-pdf",
            "command": "xelatex",
            "args": [
                "-no-pdf",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOC%"
            ]
        },
        {
            "name": "xelatex-pdf",
            "command": "xelatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOC%"
            ]

        },
        {
            "name": "latexmk",
            "command": "latexmk",
            "args": [
              "-synctex=1",
              "-interaction=nonstopmode",
              "-file-line-error",
              "-pdf",
              "-outdir=%OUTDIR%",
              "%DOC%"
            ],
            "env": {}
          },
          {
            "name": "pdflatex",
            "command": "pdflatex",
            "args": [
              "-synctex=1",
              "-interaction=nonstopmode",
              "-file-line-error",
              "%DOC%"
            ],
            "env": {}
          },
          {
            "name": "bibtex",
            "command": "bibtex",
            "args": [
              "%DOCFILE%"
            ],
            "env": {}
          }
    ],
    "latex-workshop.latex.recipes":[
        {
            "name": "xelatex-single-run",
            "tools": [
                "xelatex-pdf",
            ]
        },
        {
            "name": "xelatex ➞ bibtex ➞ xelatex × 2",
            "tools": [
                "xelatex-no-pdf",
                "bibtex",
                "xelatex-no-pdf",
                "xelatex-pdf"
            ]
        },
        {
            "name": "latexmk 🔃",
            "tools": [
                "latexmk"
            ]
        },
        {
            "name": "latexmk (latexmkrc)",
            "tools": [
                "latexmk_rconly"
            ]
        },
        {
            "name": "latexmk (lualatex)",
            "tools": [
                "lualatexmk"
            ]
        },
        {
            "name": "pdflatex ➞ bibtex ➞ pdflatex × 2",
            "tools": [
                "pdflatex",
                "bibtex",
                "pdflatex",
                "pdflatex"
            ]
        },
        {
            "name": "Compile Rnw files",
            "tools": [
                "rnw2pdf"
            ]
        }
    ]
}
```

有点长哈，是我自己写的，一定要把这个原封不动的写入 `settings.json` 文件。拷贝覆盖了之后，保存，重启 VS Code 就可以了。

至此我们已经完成了所有的设置，下一步就是使用啦！

## 第三步

Visual Studio Code 是一款十分强大的通用编辑器，基本上所有的的文件类型都可以编辑，而且本身由微软出品，更新十分快，非常稳定，是 Windows 平台首选的编辑器。

我们首先要选择打开文件夹，然后选择论文的根目录，这样我们就可以在左边的文件浏览器看到所有的文件啦，选择 `*.tex` 结尾的文件打开，左边的侧边栏就会多出一个 TeX 选项，这就是我们新安装的插件啦。进入这个选项可以看到很多相关的命令，其中默认的编译链可以适用于中文论文的编译。选择 "View LaTeX PDF - View in VSCode tab" 就可在侧边栏预览 PDF 啦，注意只有编译一遍才有预览界面哦。

### 如何编译

选择 "Build LaTeX project" 进入，再点击下面含有 "xelatex" 的配方就可以啦，一共有两个，第一个是当你保存时自动进行的 "xelatex-single-run" 只编译一遍，但是不会更新 bibtex 的内容，另一个是完整的编译过程"xelatex-\>bibtex-\>xelatex x 2"，我们需要**在第一次打开**项目的时候选择这个进行一遍完整编译，它会更新 bib 的内容，但是太慢。另外当你更新了 bib 文件的时候也要用这个完整的编译链完成引用的更新哦，不然会出错的。





