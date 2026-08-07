# Orchestrator Mode Rules for Hybrid A* Path Planning

## Core Principles:

1. **Break Down Complex Tasks**: When given a complex task, decompose it into logical subtasks delegatable to specialized modes.

2. **Delegate with Full Context**: For each subtask, use `new_task` tool with comprehensive instructions including:
   - All necessary context from parent task or previous subtasks
   - Clearly defined scope (what to accomplish)
   - Explicit statement of what NOT to do
   - Instructions for signaling completion via `attempt_completion`
   - Statement that these instructions supersede general mode instructions

3. **Track Progress**: Monitor subtask completion and analyze results before proceeding.

4. **Explain Delegation**: Help user understand why specific tasks go to specific modes.

5. **Synthesize Results**: When all subtasks complete, provide comprehensive overview.

6. **Ask Clarifying Questions**: When necessary to understand task breakdown.

7. **Suggest Improvements**: Based on completed subtask results.

## Mode Selection Guidelines:

### Ask Mode (Concept/Explanation)
Use when user asks:
- "为什么..." (Why does X happen?)
- "是什么..." (What is X?)
- "有什么区别..." (What's the difference between X and Y?)
- "能解释一下..." (Can you explain...)

### Architect Mode (Design/Refactoring)
Use when task involves:
- Multi-file architectural changes
- New component design (Manager/Guard/Builder/Visualizer)
- Algorithm selection (A* vs RRT* vs Hybrid A*)
- Performance optimization planning
- Major refactoring (>3 files)

### Code Mode (Implementation)
Use when:
- User explicitly says "切换到 code 模式" or "实施方案 X"
- Bug location is known and fix is straightforward
- Single-file or well-understood multi-file changes
- Feature implementation following existing patterns

### Debug Mode (Diagnosis)
Use when:
- "为什么还是不行？" (Why is it still not working?)
- Log analysis required
- Root cause unknown
- Multiple possible causes to narrow down

## Hybrid A* Specific Delegation Patterns:

### Pattern 1: P1 - "编译了新代码但跑的是旧代码"
1. **Debug**: Check `rospack find hybrid_astar` and `md5sum` of binary
2. **Ask**: Explain ROS_PACKAGE_PATH workspace precedence
3. **Code**: (only if fix needed) Modify ~/.zshrc

### Pattern 2: P2 - GoalGuard 目标被阻塞
1. **Debug**: grep `[GoalGuard]`, analyze state machine path
2. **Ask**: Explain V3→V5 state machine differences
3. **Code**: Fix collision_cache_, unsigned char trap, or projection search

### Pattern 3: P3 - 路径不平滑
1. **Debug**: Check if smoothPath() is called, if std::reverse exists
2. **Ask**: Explain smoothing algorithm (gradient descent + B-Spline)
3. **Code**: Add smoothPath() call, fix std::reverse in tracePath

### Pattern 4: P4 - LSP 报错雪崩
1. **Debug**: Identify all references to deleted component
2. **Code**: Remove references from CMakeLists.txt, headers, cpp files

### Pattern 5: P5 - 前瞻角计算异常
1. **Debug**: Check pcaf.log for `max_curvature: 0.499`
2. **Ask**: Explain Ackermann kinematics constraint (DO NOT modify tracker)
3. **Code**: Add curvature-aware speed in path_segment_builder.cpp

## Subtask Delegation Template:

```markdown
请切换到 [MODE] 模式，完成以下任务：

## 任务描述
[清晰描述要完成什么]

## 上下文
- 用户原始问题：[原始用户消息]
- 相关文件：[文件路径]
- 日志位置：[日志路径]
- 关键 marker：[相关日志 marker]

## 范围边界
- ✅ 要做：[明确任务范围]
- ❌ 不要做：[明确排除范围]

## 成功标准
- [可验证的完成条件]

## 完成后
请使用 `attempt_completion` 工具，简要总结：
- 发现了什么
- 建议什么方案（或已实现什么修改）
```

## Absolute Rules:

- ❌ **DO NOT** delegate without providing full context
- ❌ **DO NOT** delegate without clear scope boundaries
- ❌ **DO NOT** skip Debug mode for "why still not working" questions
- ❌ **DO NOT** delegate Ask questions to Code mode
- ❌ **DO NOT** proceed without user confirmation for major decisions

## Workspace-Specific Reminders:

- **Workspace**: `/home/kkcqh/workspace/endofraod_ws/src/path-plan`
- **Ground Truth**: `/home/kkcqh/workspace/endofraod_old_ws/src/path-plan`
- **Logs**: `/home/kkcqh/workspace/logs/$(date +%Y-%m-%d)/endofload.log`
- **Tracker**: `/home/kkcqh/workspace/pcaf/log/$(date +%Y-%m-%d)/pcaf.log`
- **Skills**: `/home/kkcqh/workspace/endofraod_ws/poem/good/zh/*.zh.md`
