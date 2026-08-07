# Ask Mode Rules for Hybrid A* Path Planning

## Core Principles:

1. **Answer Thoroughly**: Analyze code, explain concepts, and access external resources. Always answer user's questions completely.

2. **No Implementation Unless Asked**: Do not switch to implementing code unless explicitly requested by the user.

3. **Include Mermaid Diagrams**: When they clarify the response, include Mermaid diagrams for state machines, data flow, or coordinate transformations.

4. **Speak in 中文 (Chinese)**: Unless user specifies otherwise, respond in Chinese for conceptual explanations.

## Mandatory Pre-Read Files (for path-plan issues):

Always read these 4 files before answering questions about the path-plan repository:

```bash
# English versions
cat /home/kkcqh/workspace/endofraod_ws/poem/good/Component-Patterns.md
cat /home/kkcqh/workspace/endofraod_ws/poem/good/Coordinate-QuickRef.md
cat /home/kkcqh/workspace/endofraod_ws/poem/good/Package-QuickRef.md
cat /home/kkcqh/workspace/endofraod_ws/poem/good/Feature-Add-Workflow.md

# Chinese versions
cat /home/kkcqh/workspace/endofraod_ws/poem/good/zh/Component-Patterns.zh.md
cat /home/kkcqh/workspace/endofraod_ws/poem/good/zh/Coordinate-QuickRef.zh.md
cat /home/kkcqh/workspace/endofraod_ws/poem/good/zh/Package-QuickRef.zh.md
cat /home/kkcqh/workspace/endofraod_ws/poem/good/zh/Feature-Add-Workflow.zh.md
```

## Key Log Locations:

- **Planning logs**: `/home/kkcqh/workspace/logs/$(date +%Y-%m-%d)/endofload.log`
- **Tracker logs**: `/home/kkcqh/workspace/pcaf/log/$(date +%Y-%m-%d)/pcaf.log`

## Critical Markers to Search:

| Marker | Meaning | Source |
|--------|---------|--------|
| `[GoalGuard][FRAME]` | Goal coordinates on receipt | `goal_guard.cpp` |
| `entering WAITING/SEARCH/SAMPLING` | State machine transitions | `goal_guard.cpp` |
| `WAITING timed out after ...ms` | Blocking timeout | `goal_guard.cpp` |
| `SEARCH found no alternative, REJECTED` | Search failure | `goal_guard.cpp` |
| `[HEARTBEAT]` | MoveFeedback heartbeat | `move_feedback_manager.cpp` |
| `[NAV_FEEDBACK]` | Navigation feedback | `navigation_feedback_manager.cpp` |
| `[HybridAStar]` | Hybrid A* search events | `algorithm.cpp` |
| `Max iterations (40000) exhausted` | Iteration limit hit | `algorithm.cpp` |
| `前瞻角计算异常` | Tracker curvature rejection | `steering_controller.cc:103` |
| `max_curvature: 0.499` | Path curvature too high | `path_follow_adapter.cc:116` |

## Common Question Templates:

### Q: "为什么 goal 被阻塞？"
→ Check: unsigned char trap, collision_cache_, coordinate mismatch, V2/V5 binary confusion

### Q: "为什么路径不平滑？"
→ Check: smoothPath() 未调用, tracePath() 缺 std::reverse, wCurvature 权重不一致

### Q: "为什么前瞻角计算异常？"
→ This is a tracker-layer symptom with planning-layer root cause:
- Tracker correctly rejects curvature > kappaMax
- PathSegmentBuilder sets speed=0.5 regardless of curvature
- Fix: Add curvature-aware speed in planning layer

### Q: "为什么 dist_to_goal 和 RViz 显示不一致？"
→ Coordinate system confusion: world vs grid
- Fix: Use PoseBridge::worldToGrid() consistently
- Log both values for debugging

## Response Format:

```markdown
🎯 Direct Verdict
[一句话结论 + 风险等级]

🔬 Under the Hood
[数据链 / 失败根因流程 / mermaid 图]

📊 解决方案对比
| 方案 | 改动范围 | 风险 | 推荐 |
|---|---|---|---|
| A | ... | 🟢 | |
| B | ... | 🟡 | ★ |

🚩 Watchouts
[3-5 个易踩的坑]
```
