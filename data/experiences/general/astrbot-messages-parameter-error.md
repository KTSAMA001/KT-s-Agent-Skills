## AstrBot "messages 参数非法" 错误
2026-02-02 | astrbot|bug|待调查 | ⚠️ | 未知

### 问题现象
在执行某些操作或生成回复后，AstrBot 返回错误：
```
错误类型: BadRequestError
错误信息: Error code: 400 - {'error': {'code': '1214', 'message': 'messages 参数非法。请检查文档。'}}
```

### 触发场景
1. 尝试使用 `/找` 命令搜索文件时
2. 执行简单的对话回复时（如回复 "/？"、"你又g了"）
3. 错误不是在命令执行时触发，而是在尝试将回复发送到群聊时

### 可能原因
1. **消息格式问题**: 回复内容被包装成 `messages` 格式传递给某个 API 时，格式不符合预期
2. **输出过长**: 搜索命令的输出可能包含过多内容，导致参数超限
3. **特殊字符**: 输出中可能包含特殊字符、空格或中文，导致解析失败
4. **参数缺失**: 某些必需的消息字段可能缺失或格式错误

### 排查方向
1. 检查 AstrBot 日志文件 (`platform.log` 或相关日志)
2. 简化命令输出，限制长度（如 `| head -n 10`）
3. 检查 AstrBot 配置中关于消息长度、字符编码的限制
4. 查看 GitHub Issues: https://github.com/AstrBotDevs/AstrBot/issues

### 参考链接
- AstrBot官网: https://astrbot.app
- 官方文档: https://docs.astrbot.app
- GitHub仓库: https://github.com/AstrBotDevs/AstrBot

### 验证记录
- [ ] 查看后台日志确认根本原因
- [ ] 检查是否与特定字符或内容相关
- [ ] 测试不同长度和复杂度的消息
- [ ] 联系官方或提交 issue

### 相关经验
暂无
