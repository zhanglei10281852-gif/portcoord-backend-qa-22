# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

真实额度不足时申报接口返回内部错误，而不是可重试的正常拒绝结果。请修复容量拒绝在服务链路中的表达。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/portcoord-backend-qa-22
- 仓库地址：https://github.com/zhanglei10281852-gif/portcoord-backend-qa-22.git
- parent SHA：e4304aa4606ed31ef814d8a5e00bba9d36cc1f43

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/portcoord-backend-qa-22.git bug-repro
cd bug-repro
git checkout --detach e4304aa4606ed31ef814d8a5e00bba9d36cc1f43
go test ./internal/declaration -run "^TestDeclaration_Submit_QuotaExceededRejects$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/declaration -run "^TestDeclaration_Submit_QuotaExceededRejects$" -count=1
--- FAIL: TestDeclaration_Submit_QuotaExceededRejects (0.00s)
    declaration_test.go:89: submit: internal: quota reservation failed: quota_exceeded: quota yard exceeded: limit 5000, requested 5001
FAIL
FAIL	portcoord/internal/declaration	0.007s
FAIL

```

stderr：

```text
warning: internal/declaration/declaration_test.go has type 100755, expected 100644
warning: internal/declaration/test_helpers_test.go has type 100755, expected 100644
warning: internal/declaration/declaration_test.go has type 100755, expected 100644
warning: internal/declaration/test_helpers_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/declaration -run "^TestDeclaration_Submit_QuotaExceededRejects$" -count=1
--- FAIL: TestDeclaration_Submit_QuotaExceededRejects (0.22s)
    declaration_test.go:89: submit: internal: quota reservation failed: quota_exceeded: quota yard exceeded: limit 5000, requested 5001
FAIL
FAIL	portcoord/internal/declaration	0.437s
FAIL

```

stderr：

```text
warning: internal/declaration/declaration_test.go has type 100755, expected 100644
warning: internal/declaration/test_helpers_test.go has type 100755, expected 100644
warning: internal/declaration/declaration_test.go has type 100755, expected 100644
warning: internal/declaration/test_helpers_test.go has type 100755, expected 100644

```

## 通过条件

在题面触发条件下，公开行为必须恢复且原始异常不再出现；定向命令 go test ./internal/declaration -run ^TestDeclaration_Submit_QuotaExceededRejects$ -count=1、相关包测试、全量测试、race、vet 和 build 必须通过；不得删除或跳过测试，也不得绕过目标逻辑。
