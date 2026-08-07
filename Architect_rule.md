## Workflow

1. **Information Gathering**: Use provided tools to gather context about the task. Always read the 4 mandatory skill files first:
   ```bash
   ls /home/kkcqh/workspace/endofraod_ws/poem/good/         # 4 English .md
   ls /home/kkcqh/workspace/endofraod_ws/poem/good/zh/      # 4 Chinese .zh.md
   ```

2. **Clarifying Questions**: Ask the user clarifying questions to understand constraints, robot type, and specific requirements.

3. **Task Breakdown**: Once you have context, break down the task into clear, actionable steps and create a todo list using `update_todo_list`. Each todo item should be:
   - Specific and actionable
   - Listed in logical execution order
   - Focused on a single, well-defined outcome
   - Clear enough another mode could execute it independently

4. **Mermaid Diagrams**: Include Mermaid diagrams for complex workflows or system architecture. Avoid using double quotes ("") and parentheses () inside square brackets [] in Mermaid diagrams.

5. **User Confirmation**: Ask the user if they are pleased with the plan before proceeding.

6. **Mode Switching**: Use the `switch_mode` tool to request switching to another mode for implementation.

## Hybrid A* Specific Rules:

**必读文件顺序** (Always follow this order for path-plan issues):
1. `poem/good/{,zh/}Component-Patterns.md` — Component patterns
2. `poem/good/{,zh/}Coordinate-QuickRef.md` — Coordinate systems
3. `poem/good/{,zh/}Package-QuickRef.md` — Package structure
4. `poem/good/{,zh/}Feature-Add-Workflow.md` — Feature workflow
5. `include/constants.h` — All magic numbers
6. `src/planner.cpp` — Central dispatcher
7. `src/goal_guard.cpp` — State machine
8. `src/collisiondetection.cpp` — Collision detection (unsigned char trap)

**Key Debug Markers** (grep these first):
- `[GoalGuard][FRAME]` — Goal coordinates on receipt
- `[GoalGuard] entering WAITING/SEARCH/SAMPLING` — State transitions
- `[HEARTBEAT]` — MoveFeedback heartbeat
- `[NAV_FEEDBACK]` — Navigation feedback
- `[HybridAStar]` — Hybrid A* search events
- `Max iterations (40000) exhausted` — Iteration limit
- `前瞻角计算异常` — Tracker curvature issue (规划层没约束速度)

**Ground Truth Comparison** (Always compare with old workspace when in doubt):
```bash
OLD=/home/kkcqh/workspace/endofraod_old_ws/src/path-plan
NEW=/home/kkcqh/workspace/endofraod_ws/src/path-plan
diff -rq $OLD $NEW | head -30
```

## Critical Checks Before Proposing Solutions:

- [ ] Did you read the 4 mandatory skill files?
- [ ] Did you compare with `endofraod_old_ws`?
- [ ] Did you identify the coordinate system (world vs grid)?
- [ ] Did you grep for key markers (`[GoalGuard]`, `[HEARTBEAT]`, `[HybridAStar]`)?
- [ ] Did you draw a Mermaid state flow diagram?
- [ ] Did you provide multiple solution options (A/B/C) with risk comparison?
- [ ] Did you list TODO items (one per file to modify)?
- [ ] Did you remind user to delete old logs and rebuild?

## Output Format:

```markdown
# 方案分析报告

## 1. 系统拓扑
[mermaid diagram]

## 2. 问题定位
[symptom → root cause mapping]

## 3. 方案对比
| 方案 | 改动范围 | 风险 | 推荐 |
|---|---|---|---|
| A | ... | 🟢 | |
| B | ... | 🟡 | ★ |
| C | ... | 🔴 | |

## 4. 推荐实施步骤
1. ...
2. ...

## 5. 自检清单
- [ ] ...
```
