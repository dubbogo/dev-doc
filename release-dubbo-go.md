# 发布 Dubbo-go

## GPG

Ref: http://www.ruanyifeng.com/blog/2013/07/gpg.html

查看 GPG 版本。

```bash
$ gpg --version
```

安装 GPG（已安装 GPG 忽略）。

```bash
# for macOS
$ brew install gpg
```

查看密钥，假设公钥是 `8FDE893A8DE5184C7F76ECB9B9185984BF7D6735`。

```bash
$ gpg --list-keys
/home/nxw/.gnupg/pubring.kbx
----------------------------
pub   rsa3072 2022-12-04 [SC]
      8FDE893A8DE5184C7F76ECB9B9185984BF7D6735
uid           [ unknown] Xuewei Niu <justxuewei@apache.org>
sub   rsa3072 2022-12-04 [E]
# 设置公钥环境变量
$ PUB_KEY="8FDE893A8DE5184C7F76ECB9B9185984BF7D6735"
```

生成密钥，生成密钥之后需要设置 `PUB_KEY` 环境变量。

```bash
$ gpg --full-gen-key
# 上传公钥
$ gpg --keyserver hkp://pgpkeys.mit.edu --send-keys $PUB_KEY
# 生成公钥指纹
$ gpg --fingerprint $PUB_KEY
```

导出私钥和公钥。

```bash
$ gpg --output public.gpg --armor --export $PUB_KEY
$ gpg --output private.gpg --armor --export-secret-keys $PUB_KEY
```

导入私钥和公钥。

```bash
$ gpg --import public.gpg
$ gpg --import private.gpg
$ gpg --edit-key $PUB_KEY trust quit
```

吊销证书。

```bash
$ gpg --gen-revoke $PUB_KEY
```

## CHANGELOG

将近期合并的 pull requests 按照 features、bugfixes 和 enhancements 更新到对应分支的 CHANGELOG.md 中，示例 [dubbo-go#2360](https://github.com/apache/dubbo-go/pull/2360)。

各 release 分支 CHANGELOG：[main](https://github.com/apache/dubbo-go/blob/main/CHANGELOG.md) | [release-3.1](https://github.com/apache/dubbo-go/blob/release-3.1/CHANGELOG.md) | [release-3.0](https://github.com/apache/dubbo-go/blob/release-3.0/CHANGELOG.md)。

## 打包签名

在准备打包签名之前，检查以下内容是否符合要求：

- 每个文件的 LICENSE 头部是否正确, 包括 `*.java`, `*.go`, `*.xml`, `Makefile` 等；
- LICENSE 文件是否存在；
- NOTICE 文件是否存在；
- CHANGELOG.md 是否存在（变更内容格式符合规范）。

设置 release 环境变量。

```bash
$ GIT_TAG="v3.1.1-rc1"
$ GIT_TAG_MSG="$GIT_TAG release candidate 1"
```

打 tag。

```bash
$ cd path/to/dubbo-go
$ git tag -a $GIT_TAG -m $GIT_TAG_MSG
```

打包源代码并验证。

```bash
$ git archive --format=tar --prefix=dubbo-go-$GIT_TAG/ $GIT_TAG \
    | gzip > dubbo-go-$GIT_TAG-src.tar.gz
$ gpg --verify dubbo-go-$GIT_TAG-src.tar.gz.asc \
    dubbo-go-$GIT_TAG-src.tar.gz
$ shasum -a 512 dubbo-go-$GIT_TAG-src.tar.gz \
    > dubbo-go-$GIT_TAG-src.tar.gz.sha512
$ shasum --check dubbo-go-$GIT_TAG-src.tar.gz.sha512
```
