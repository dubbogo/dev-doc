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

## 创建 tag 信息

设置 release 环境变量。

```bash
$ MAIN_VERSION="3.1.1"

# 发布 rc 版本
$ RC_VERSION="1"
$ GIT_TAG="v$MAIN_VERSION-rc$RC_VERSION"
$ GIT_TAG_MSG="v$MAIN_VERSION release candidate $RC_VERSION"

# 发布生产版本
$ GIT_TAG="v$MAIN_VERSION"
$ GIT_TAG_MSG="v$MAIN_VERSION production ready"

# 查看 tag/tag 信息
$ echo "Tag: $GIT_TAG, msg: $GIT_TAG_MSG"
```

打 tag 并推送至远端，请确保 **CHANGELOG 的 pull request 已经被合并**。

```bash
$ cd path/to/dubbo-go
# checkout 至 release-x.y 分支或者 main
$ git checkout <RELEASE_BRANCH>
$ git pull
# 操作 tag
$ git tag -a $GIT_TAG -m $GIT_TAG_MSG
$ git tag -n9 | grep $GIT_TAG
$ git push origin $GIT_TAG
```

## 打包签名

在准备打包签名之前，检查以下内容是否符合要求：

- 每个文件的 LICENSE 头部是否正确, 包括 `*.java`, `*.go`, `*.xml`, `Makefile` 等；
- LICENSE 文件是否存在；
- NOTICE 文件是否存在；
- CHANGELOG.md 是否存在（变更内容格式符合规范）。

设置 GPG 环境变量。

```bash
$ gpg --list-keys
/home/nxw/.gnupg/pubring.kbx
----------------------------
pub   rsa3072 2022-12-04 [SC]
      8FDE893A8DE5184C7F76ECB9B9185984BF7D6735
uid           [ unknown] Xuewei Niu <justxuewei@apache.org>
sub   rsa3072 2022-12-04 [E]
$ GPG_EMAIL="justxuewei@apache.org"
$ GPG_NAME="Xuewei Niu"
```

打包源代码并验证。

```bash
$ git archive --format=tar --prefix=dubbo-go-$GIT_TAG/ $GIT_TAG \
    | gzip > dubbo-go-$GIT_TAG-src.tar.gz
# 签名
$ gpg -u $GPG_EMAIL --armor --output dubbo-go-$GIT_TAG-src.tar.gz.asc \
    --detach-sign dubbo-go-$GIT_TAG-src.tar.gz
$ gpg --verify dubbo-go-$GIT_TAG-src.tar.gz.asc \
    dubbo-go-$GIT_TAG-src.tar.gz
# 哈希验证
$ shasum -a 512 dubbo-go-$GIT_TAG-src.tar.gz \
    > dubbo-go-$GIT_TAG-src.tar.gz.sha512
$ shasum --check dubbo-go-$GIT_TAG-src.tar.gz.sha512
```

## 上传包至 Apache SVN 仓库

```bash
$ svn checkout https://dist.apache.org/repos/dist/dev/dubbo
$ cd dubbo
$ svn update
$ (gpg --list-sigs $GPG_NAME && gpg --armor --export $GPG_NAME) >> KEYS
$ mkdir -p dubbo-go/$GIT_TAG
# 复制 dubbo-go-$GIT_TAG-src.tar.gz、dubbo-go-$GIT_TAG-src.tar.gz.asc 和
# dubbo-go-$GIT_TAG-src.tar.gz.sha512 到 dubbo-go/$GIT_TAG
$ tree dubbo-go/$GIT_TAG
dubbo-go/v1.5.6-rc1/
├── dubbo-go-v1.5.6-rc1-src.tar.gz
├── dubbo-go-v1.5.6-rc1-src.tar.gz.asc
└── dubbo-go-v1.5.6-rc1-src.tar.gz.sha512
$ svn add dubbo-go/$GIT_TAG
$ svn commit --username <APACHE USERNAME> -m "Release dubbo-go $GIT_TAG"
```

检查 SVN 内容

- KEYS: https://dist.apache.org/repos/dist/dev/dubbo/KEYS
- Release 包: https://dist.apache.org/repos/dist/dev/dubbo/dubbo-go

## 发送发版邮件

发送 VOTE 邮件至 dev@dubbo.apache.org，主题是 "[VOTE] Release Apache Dubbo-go v3.1.1-rc1"。

```
Hello Dubbo/Dubbogo community

This is a vote for the release of Apache Dubbo-go v3.1.1-rc1

Release candidates:
https://dist.apache.org/repos/dist/dev/dubbo/dubbo-go/v3.1.1-rc1

Git tags published:
https://github.com/apache/dubbo-go/tree/v3.1.1-rc1

Publish tag hash:
d204c4ef491dd22c58c4fd2e0a30e0f9d4e57715

The artifact has been signed with a key:
8FDE893A8DE5184C7F76ECB9B9185984BF7D6735, it can be found in the KEYS file https://dist.apache.org/repos/dist/dev/dubbo/KEYS

Polls will remain open for at least 72 hours or until the necessary number of votes is reached.

Please vote accordingly:
[] +1 Yes
[] +0 no opinion
[] -1 Do not approve, cause

Thank you, Apache Dubbo/Dubbogo team
```

在获得 3 位 committers/PMCs 的确认邮件后，发送 RESULT 邮件至 dev@dubbo.apache.org，主题是 "[RESULT][VOTE] Release Apache Dubbo-go v3.1.1-rc1"。Vote and result thread 地址可以在 [dev@dubbo.apache.org](https://lists.apache.org/list.html?dev@dubbo.apache.org) 中找到。

```
Hello Dubbo/Dubbogo Community,

The release Dubbo-go v3.1.1-rc1 vote finished, We’ve received 3 +1 (binding) votes.

+1 binding, {Member0}
+1 binding, {Member1}
+1 binding, {Member2}

The vote and result thread:
https://lists.apache.org/thread/6hybm3jd88v7phxoy178mxbtwxw8yrzv

The vote passed. Thanks all.
I will proceed with the formal release later.

Best regards,
The Apache Dubbogo Team
```
