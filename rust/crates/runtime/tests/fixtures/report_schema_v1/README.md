# 报告 Schema v1 Fixture 集

通过 `cargo test -p runtime report_schema -- --nocapture` 验证。

`runtime::report_schema::tests::fixture_report` 中的代码内 fixture 涵盖：
- fact / hypothesis / confidence 标签
- 带有已检查层面和查询窗口的负面证据
- 字段级 delta 归因
- 规范报告 id 加上内容哈希
- 确定性投影/编辑溯源
- 消费者能力协商和降级投影
