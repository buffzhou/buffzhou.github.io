---
description: >
  There are multiple ways of installing Hydejack and this document lays them out for you. The easiest way is with the Hydejack Starter Kit.
hide_description: true
---

# 安装
本文给出几种安装Hydejack的方法。
还有另一种方法是用gem [Ruby gem](#通过gem).
如果您不在乎杂乱的目录, 您可以 [the zip file](#通过zip).
最后您要是知道自己该怎么做，您可以直接用git[fork the git repository](#via-git).

买高级版 [follow these steps](#pro-version).

## 快速目录
{:.no_toc}
1. this unordered seed list will be replaced by toc as unordered list
{:toc}

## 使用 Starter Kit
使用 Starter Kit 的好处是不会扰乱您的blog库.
不仅如此，上传到 GitHub Pages只用按  `push`.

如果您有GitHub账户 [hy-starter-kit](https://github.com/qwtel/hy-starter-kit) . 或者 [download the source files][src] 再在您电脑上解压。

**小贴士**:  除了这个手册, 您也可以参考 Starter Kit 的快速向导。
{:.message}

`cd` 进入 `_config.yml` 安装所在的目录然后遵守步骤[Running locally](#running-locally).

要不然, 您可以用 [![Deploy to Netlify][dtn]{:data-ignore=""}][nfy]{:.no-hover.no-mark}.

[src]: https://github.com/qwtel/hy-starter-kit/archive/v8.1.1.zip
[nfy]: https://app.netlify.com/start/deploy?repository=https://github.com/qwtel/hydejack-starter-kit
[dtn]: https://www.netlify.com/img/deploy/button.svg

## 使用 gem
Jekyll 有 [built-in support](https://jekyllrb.com/docs/themes/) 以RubyGems为服务的网站.  

如果您还没有，请先创造一个新RudyGems网站:

~~~bash
$ jekyll new <PATH>
~~~

您的网站根目录看起来应该是这个样子。

~~~
├── _posts
│   └── 2017-04-07-welcome-to-jekyll.markdown
├── _config.yml
├── about.md
├── Gemfile
├── Gemfile.lock
└── index.md
~~~

**小贴士**: Hydejack使用Jekyll的默认`config.yml`, 但是建议您替换成
[Hydejack's default config file](https://github.com/qwtel/hydejack/blob/v8/_config.yml).
它包含所有Hydejack的设置选项和提供使用的设置 (像 minifying HTML 和 CSS 产生结构).
{:.message}

其次，如果您希望添加 `jekyll-theme-hydejack` 成为附属，输入以下语句在 `Gemfile`中.

~~~ruby
gem "jekyll-theme-hydejack"
~~~

(您也可以删除旧主题 `jekyll-theme-minima` 在Gemfile)

现在您希望更改 `_config.yml` 在您的Jekyll网站和设置Hydejack作为主题.
找到 `theme` 键然后设置这个值在 `jekyll-theme-hydejack`.

~~~yml
theme: jekyll-theme-hydejack
~~~

找更多信息在基于gem的网站,点击[Jekyll Documentation](http://jekyllrb.com/docs/themes/).

You can now continue with [running locally](#running-locally).

## Via zip
If you downloaded the [extended zip](https://github.com/qwtel/hydejack/releases),
extract the contents somewhere on your machine.
The high-level folder structure will look something like.

~~~
├── _data
├── _featured_categories
├── _featured_tags
├── _includes
├── _js
├── _layouts
├── _posts
├── _sass
├── assets
├── _config.yml
├── 404.md
├── about.md
├── index.html
└── posts.md
~~~

`cd` into the directory where `_config.yml` is located and follow the steps in [Running locally](#running-locally).

## Via git
If you are familiar with using git, you can add the [Hydejack repository](https://github.com/qwtel/hydejack)
as a remote, and merge its master branch into your working branch.

~~~bash
$ git remote add hydejack git@github.com:qwtel/hydejack.git
$ git pull hydejack master
~~~

You can also update Hydejack this way. The master branch will not contain work in progress,
but will contain major (breaking) changes.
This approach is recommended if you intend to customize Hydejack.

You can now continue with [running locally](#running-locally).

## PRO Version
If you bought the PRO version, you've received a zip archive with the following contents:

~~~
├── install
├── upgrade
├── CHANGELOG.pdf
├── Documentation.pdf
├── NOTICE.pdf
├── PRO License.pdf
├── PRO–hy-drawer License.pdf
├── PRO–hy-img License.pdf
├── PRO–hy-push-state License.pdf
└── .ssh
~~~

`install`
: Contains all files and folders needed to create a new blog.

`upgrade`
: Contains only the files and folders needed for upgrading form an earlier version of Hydejack (6.0.0 or above). See [Upgrade]{:.heading.flip-title} for more.

`.ssh`
: A hidden folder containing a SSH key for read-only access to the Hydejack PRO GitHub repository.
  You can use this to install Hydejack PRO as gem-based theme.
  See the [installation instructions](#pro-via-github-advanced) below.
  This is for advanced users.

For new installations only the `install` folder is relevant.
Unzip the archive somewhere on your machine, then `cd` *into* the `install` folder, e.g.

~~~bash
$ cd ~/Downloads/hydejack-pro-8.1.1/install/
~~~

You can now continue with [Running locally](#running-locally).

### PRO via GitHub (advanced)
If you know how to handle SSH keys, you can also install the PRO version as a gem-based theme via GitHub.
The advantage of this method is that you avoid cluttering your Jekyll repository with Hydejack's source files.

The downloaded zip contains a read-only key for a private GitHub repository.
It is located at `<dowloaded zip>/.ssh/hydejack_8_pro`.
You have to copy the key file to `~/.ssh` (or wherever your SSH keys are located), e.g.:

~~~bash
$ cp ~/Downloads/hydejack-pro-8.1.1/.ssh/hydejack_8_pro ~/.ssh/
~~~

It is required that your private key files are NOT accessible by others, e.g.:

~~~bash
$ chmod 600 ~/.ssh/hydejack_8_pro
~~~

Then add the following to `.ssh/config`:

~~~
Host hydejack
	HostName github.com
	IdentitiesOnly yes
	IdentityFile ~/.ssh/hydejack_8_pro
~~~

Next, open `Gemfile` in your Jekyll repository and add:

~~~ruby
gem "jekyll-theme-hydejack-pro", git: 'git@hydejack:qwtel/hydejack-8-pro.git'
~~~

In your `_config.yml`, add:

~~~yml
theme: jekyll-theme-hydejack-pro
~~~

You can now continue with [Running locally](#running-locally).

## Running locally
Make sure you've `cd`ed into the directory where `_config.yml` is located.
Before running for the first time, dependencies need to be fetched from [RubyGems](https://rubygems.org/):

~~~bash
$ bundle install
~~~

**NOTE**: If you are missing the `bundle` command, you can install Bundler by running `gem install bundler`.
{:.message}

Now you can run Jekyll on your local machine:

~~~bash
$ bundle exec jekyll serve
~~~

and point your browser to <http://localhost:4000> to see Hydejack in action.


Continue with [Config](config.md){:.heading.flip-title}
{:.read-more}


[upgrade]: upgrade.md
