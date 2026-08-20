# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

推理计划已经成功落库，后台事件也能解码为运行对象，但其中的 `reference` 为空，`total_estimated_rows` 变成 0，和刚保存的运行详情对不上，下游因此无法核对计划。测试文件和现有断言请保持原样，验证过程不能跳过或弱化。请修复 outbox 载荷与持久化对象之间的契约。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/ai-09
- 仓库地址：https://github.com/zhanglei10281852-gif/ai-09.git
- parent SHA：1c14b27c464d1ee27973a742bff2b58d2aa7e478

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/ai-09.git bug-repro
cd bug-repro
git checkout --detach 1c14b27c464d1ee27973a742bff2b58d2aa7e478
go test ./internal/worker -run ^TestPlannedOutboxPayloadMatchesPersistedRun$ -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/worker -run ^TestPlannedOutboxPayloadMatchesPersistedRun$ -count=1
--- FAIL: TestPlannedOutboxPayloadMatchesPersistedRun (0.07s)
    annotation_core_behavior_test.go:83: planned payload = {ID:run_3cac48541389f2d16c7cf449 WorkspaceID:workspace_outbox SourceZoneID:origin_outbox TargetZoneID:destination_outbox ComputePoolID:pool_outbox Reference: State:queued ScheduledStartAt:2026-08-18 09:00:00 +0000 UTC ExpectedFinishAt:2026-08-18 10:00:00 +0000 UTC StartedAt:<nil> CompletedAt:<nil> ArchivedAt:<nil> TotalEstimatedRows:0 CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:1}, persisted run = {ID:run_3cac48541389f2d16c7cf449 WorkspaceID:workspace_outbox SourceZoneID:origin_outbox TargetZoneID:destination_outbox ComputePoolID:pool_outbox Reference:OUTBOX-RUN State:queued ScheduledStartAt:2026-08-18 09:00:00 +0000 UTC ExpectedFinishAt:2026-08-18 10:00:00 +0000 UTC StartedAt:<nil> CompletedAt:<nil> ArchivedAt:<nil> TotalEstimatedRows:240 CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:1}
FAIL
FAIL	github.com/zhanglei10281852-gif/ai/internal/worker	0.069s
FAIL

```

stderr：

```text
(empty)
```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/worker -run ^TestPlannedOutboxPayloadMatchesPersistedRun$ -count=1
--- FAIL: TestPlannedOutboxPayloadMatchesPersistedRun (0.66s)
    annotation_core_behavior_test.go:83: planned payload = {ID:run_f2fa4c58a79e9b2eeeef31f1 WorkspaceID:workspace_outbox SourceZoneID:origin_outbox TargetZoneID:destination_outbox ComputePoolID:pool_outbox Reference: State:queued ScheduledStartAt:2026-08-18 09:00:00 +0000 UTC ExpectedFinishAt:2026-08-18 10:00:00 +0000 UTC StartedAt:<nil> CompletedAt:<nil> ArchivedAt:<nil> TotalEstimatedRows:0 CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:1}, persisted run = {ID:run_f2fa4c58a79e9b2eeeef31f1 WorkspaceID:workspace_outbox SourceZoneID:origin_outbox TargetZoneID:destination_outbox ComputePoolID:pool_outbox Reference:OUTBOX-RUN State:queued ScheduledStartAt:2026-08-18 09:00:00 +0000 UTC ExpectedFinishAt:2026-08-18 10:00:00 +0000 UTC StartedAt:<nil> CompletedAt:<nil> ArchivedAt:<nil> TotalEstimatedRows:240 CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:1}
FAIL
FAIL	github.com/zhanglei10281852-gif/ai/internal/worker	1.001s
FAIL

```

stderr：

```text
(empty)
```

## 通过条件

推理计划生成的 outbox payload 解码后必须与已持久化运行对象保持契约一致，包括非空 reference、正确的 total_estimated_rows、运行 ID 和计划数据；下游能够据此核对同一计划。定向 worker 用例及相关 outbox 回归须通过，不得修改测试、跳过验证或削弱字段断言。
