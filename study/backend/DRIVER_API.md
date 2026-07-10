# Driver API

> Status: Pending analysis
> 待分析源码：`filament/backend/include/backend/Driver.h`、`filament/backend/src/CommandStream.*`

## 计划内容

本文档将记录：

- Driver 公共接口类定义（文件:行号）
- 关键虚方法分组（资源创建、命令录制、同步、查询）
- DriverApi 与 Driver 的关系
- CommandStream 的命令录制与提交机制
- Engine 如何持有并调用 DriverApi
- Handle 体系（`backend/include/backend/Handle.h`）

## 待跟踪符号

- `Driver` 接口类
- `DriverApi`（CommandStream 派生或封装）
- `CommandStream`
- `Handle<T>`

> 以上为计划跟踪目标，尚未在源码中确认。确认后更新本文档并移除此标记。
