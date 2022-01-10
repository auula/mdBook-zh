# 在持续集成中运行 `mdbook`

虽然以下示例中使用在 Travis CI,但原则上应该，也可以直接转移到其他持续集成提供商.

## 确保 mdbook 的构建与测试成功

以下是 Travis CI 的`.travis.yml`示例，确保配置了`mdbook build`和`mdbook test`运行成功。加快 CI 运转时间的关键是缓存`mdbook`的安装,以便您可以不用每次 CI 运行就编译一次`mdbook`。

```yaml
language: rust
sudo: false

cache:
  - cargo

rust:
  - stable

before_script:
  - (test -x $HOME/.cargo/bin/cargo-install-update || cargo install cargo-update)
  - (test -x $HOME/.cargo/bin/mdbook || cargo install --vers "^0.3" mdbook)
  - cargo install-update -a

script:
  - mdbook build path/to/mybook && mdbook test path/to/mybook
```

## 发布你的图书到 GitHub Pages

遵循这些命令的结果是，在您的存储库运行 CI 成功后,您的图书将发布到 GitHub 页面的`master`分支.

首先,创建一个新的 GitHub 的"Personal Access Token"，保证"public_repo"权限(或私有存储库的"repo")。转到 Travis CI 网站的该库设置页面，并添加一个名为`GITHUB_TOKEN`的环境变量，该变量能保证安全和*不*显示在日志中.

然后,将此代码段附加到您的`.travis.yml`，并更新为`book`目录路径:

> `book`目录是`mdbook build`的默认输出目录，请根据你的构建目录填写

```yaml
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GITHUB_TOKEN
  local-dir: path/to/mybook/book
  keep-history: false
  on:
    branch: master
```

就这样!

### 手动发布你的图书到 Github Pages

如果你的 CI 并不支持 GitHub pages, 或是与其他平台进行整合:
_注意: 不要求一定是 tmp 目录_:

```console
$> git worktree add /tmp/book gh-pages
$> mdbook build
$> rm -rf /tmp/book/* # this won't delete the .git directory
$> cp -rp book/* /tmp/book/
$> cd /tmp/book
$> git add -A
$> git commit 'new book message'
$> git push origin gh-pages
$> cd -
```

或是放入 Makefile 文件:

```makefile
.PHONY: deploy
deploy: book
	@echo "====> deploying to github"
	git worktree add /tmp/book gh-pages
	rm -rf /tmp/book/*
	cp -rp book/* /tmp/book/
	cd /tmp/book && \
		git add -A && \
		git commit -m "deployed on $(shell date) by ${USER}" && \
		git push origin gh-pages
```
