# Debug Mode Rules for Hybrid A* Path Planning

## Debugging Workflow:

### Phase 1: Reproduce (取证)
1. Read the latest log segment from `/home/kkcqh/workspace/logs/$(date +%Y-%m-%d)/endofload.log`
2. Read tracker log from `/home/kkcqh/workspace/pcaf/log/$(date +%Y-%m-%d)/pcaf.log` (if tracking issues)
3. Record user's exact symptom description in Chinese

### Phase 2: Correlate (关联)
Grep for key markers to anchor phenomenon to specific code location:

```bash
# GoalGuard state machine path
grep -E "entering WAITING|entering SEARCH|entering SAMPLING|WAITING timed out|SEARCH found no alternative" \
    /home/kkcqh/workspace/logs/$(date +%Y-%m-%d)/endofload.log | tail -30

# Hybrid A* iteration status
grep -E "Max iterations|goal reached|expanded" \
    /home/kkcqh/workspace/logs/$(date +%Y-%m-%d)/endofload.log | tail -20

# Coordinate check
grep "FRAME" /home/kkcqh/workspace/logs/$(date +%Y-%m-%d)/endofload.log | tail -10

# Heartbeat check
grep "HEARTBEAT" /home/kkcqh/workspace/logs/$(date +%Y-%m-%d)/endofload.log | tail -10

# Error check
grep -E "ERR|WARN" /home/kkcqh/workspace/logs/$(date +%Y-%m-%d)/endofload.log | tail -20
```

### Phase 3: Diagnose (根因)
Build "current state vs expected state" comparison table. Consider 5-7 possible sources:

1. **P1: Binary confusion** — `rospack find hybrid_astar` returns old workspace
2. **P2: unsigned char trap** — unknown cell (-1) treated as occupied (255)
3. **P3: collision_cache pollution** — stale cache from previous goal
4. **P4: coordinate mismatch** — world vs grid mixed in arithmetic
5. **P5: smoothPath not called** — path never smoothed
6. **P6: V2/V5 state machine drift** — different log strings in old vs new
7. **P7: launch param mismatch** — param name in launch differs from code

Distill down to 1-2 most likely sources, then add logs to validate assumptions.

### Phase 4: Propose (方案)
Ask user to confirm diagnosis before fixing:
- List 1-2 most likely root causes with evidence
- Ask user to verify with specific command output
- Wait for user confirmation before switching to code mode

### Phase 5: Verify (验证)
After fix:
1. Rebuild: `cd /home/kkcqh/workspace/endofraod_ws && catkin build path_plan`
2. Check binary: `md5sum devel/lib/hybrid_astar/hybrid_astar`
3. Delete old logs: `rm -f /home/kkcqh/workspace/logs/$(date +%Y-%m-%d)/endofload.log`
4. User runs test
5. Check new behavior in fresh logs

## Absolute Prohibitions:

- ❌ **DO NOT** modify tracker code (`steering_controller.cc`) — user said "不动跟踪器"
- ❌ **DO NOT** skip log analysis — always start with grep
- ❌ **DO NOT** propose fixes without confirming root cause with user
- ❌ **DO NOT** let user run old logs — insist on deleting them
- ❌ **DO NOT** suggest modifying multiple files without listing each one

## Common Bug Patterns (Quick Reference):

| Symptom | Root Cause | Fix Location |
|---------|------------|--------------|
| Goal always blocked | unsigned char trap | `collisiondetection.cpp:20` |
| Path not smooth | smoothPath() not called | `planner.cpp:380` |
| Path direction wrong | tracePath missing std::reverse | `smoother.cpp:145` |
| SEARCH always fails | collision_cache not cleared | `goal_guard.cpp:130` |
| dist_to_goal wrong | world vs grid confusion | `goal_guard.cpp:150` |
| 前瞻角异常 | No curvature-aware speed | `path_segment_builder.cpp:65` |
| Binary old | ROS_PACKAGE_PATH order | ~/.zshrc |

## Log Marker Reference:

| Marker | Meaning | Source File |
|--------|---------|-------------|
| `[GoalGuard][FRAME]` | Goal coordinates printed | `goal_guard.cpp` |
| `[GoalGuard] entering WAITING` | State → WAITING_OBSERVE | `goal_guard.cpp` |
| `[GoalGuard] WAITING timed out after ...ms` | observe timeout | `goal_guard.cpp` |
| `[GoalGuard] SEARCH found no alternative, REJECTED` | SEARCH failed | `goal_guard.cpp` |
| `[HEARTBEAT]` | MoveFeedback timer | `move_feedback_manager.cpp` |
| `[NAV_FEEDBACK]` | Navigation feedback | `navigation_feedback_manager.cpp` |
| `[HybridAStar]` | A* search events | `algorithm.cpp` |
| `Max iterations (40000) exhausted` | Iteration limit | `algorithm.cpp` |
| `前瞻角计算异常` | Tracker curvature error | `steering_controller.cc:103` |

## Ground Truth Comparison:

Always compare with old workspace when uncertain:
```bash
OLD=/home/kkcqh/workspace/endofraod_old_ws/src/path-plan
NEW=/home/kkcqh/workspace/endofraod_ws/src/path-plan

diff $OLD/src/goal_guard.cpp $NEW/src/goal_guard.cpp
diff $OLD/src/collisiondetection.cpp $NEW/src/collisiondetection.cpp
diff $OLD/src/smoother.cpp $NEW/src/smoother.cpp
```

## Exit Criteria:

- User confirms diagnosis with "OK" or "是的"
- User provides additional log evidence
- User explicitly says "切换到 code 模式" to implement fix
