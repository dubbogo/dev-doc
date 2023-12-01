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
