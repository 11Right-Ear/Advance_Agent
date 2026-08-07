# Code Mode Rules for Hybrid A* Path Planning

## When to Use Code Mode:

Use this mode when you need to write, modify, or refactor code. Ideal for:
- Implementing features (new GoalGuard states, smoothing algorithms)
- Fixing bugs (unsigned char trap, missing std::reverse)
- Creating new files (new Manager/Guard/Builder/Visualizer components)
- Making code improvements (adding curvature-aware speed, clearing collision cache)

## Pre-Implementation Checklist:

Before writing any code, ensure you have:

1. **Read the 4 mandatory skill files**:
   ```bash
   cat /home/kkcqh/workspace/endofraod_ws/poem/good/zh/Component-Patterns.zh.md
   cat /home/kkcqh/workspace/endofraod_ws/poem/good/zh/Coordinate-QuickRef.zh.md
   cat /home/kkcqh/workspace/endofraod_ws/poem/good/zh/Package-QuickRef.zh.md
   cat /home/kkcqh/workspace/endofraod_ws/poem/good/zh/Feature-Add-Workflow.zh.md
   ```

2. **Compared with ground truth**:
   ```bash
   diff -rq /home/kkcqh/workspace/endofraod_old_ws/src/path-plan \
           /home/kkcqh/workspace/endofraod_ws/src/path-plan | head -30
   ```

3. **Identified all files that need modification**:
   - List each file explicitly
   - Check for CMakeLists.txt changes if adding/removing files
   - Check for header includes in dependent files

4. **Created a TODO list**:
   ```markdown
   ## TODO
   - [ ] Modify src/collisiondetection.cpp (add unsigned char fix)
   - [ ] Modify src/smoother.cpp (add std::reverse to tracePath)
   - [ ] Modify src/planner.cpp (call smoothPath after tracePath)
   - [ ] Rebuild and verify
   ```

## Implementation Discipline:

### File Modification Order:
1. First modify headers (.h/.hpp)
2. Then modify implementation (.cpp)
3. Update CMakeLists.txt if adding/removing files
4. Rebuild before testing

### Critical Bug Patterns to Fix:

#### 1. unsigned char Trap (collisiondetection.cpp)
```cpp
// BEFORE (buggy):
auto val = grid_->getCell(cX, cY);
if (val >= OCCUPIED_THRESHOLD) return false;

// AFTER (fixed):
auto val = static_cast<int>(grid_->getCell(cX, cY));
if (val < 0 || val == 255) return true; // unknown = traversable
if (val >= OCCUPIED_THRESHOLD) return false;
```

#### 2. Missing std::reverse (smoother.cpp tracePath)
```cpp
// In the nullptr base case of recursive tracePath:
if (node == nullptr) {
    std::reverse(path.begin(), path.end()); // ADD THIS LINE
    return;
}
```

#### 3. smoothPath Not Called (planner.cpp)
```cpp
// In Planner::plan(), after getting path:
auto path = algorithm_->hybridAStar(start, goal);
// ADD: Apply smoothing
if (smoother_) {
    path = smoother_->smoothPath(voronoi_, path);
}
```

#### 4. collision_cache Not Cleared (goal_guard.cpp)
```cpp
// At start of tryAcceptGoal():
void GoalGuard::tryAcceptGoal(...) {
    clearCollisionCache_(); // ADD THIS at entry
    // ... rest of function
}
```

#### 5. Curvature-Aware Speed (path_segment_builder.cpp)
```cpp
// In buildNavigationGoal(), compute curvature per segment:
double kappa = computeCurvature(path, mid_idx);
double speed = speedLimitForCurvature(kappa);
path_msg.limit_forward_speed = speed;
```

### Rebuild and Verify:

```bash
# 1. Build
cd /home/kkcqh/workspace/endofraod_ws
catkin build path_plan --cmake-args -DCMAKE_BUILD_TYPE=Release

# 2. Check binary updated
md5sum devel/lib/hybrid_astar/hybrid_astar

# 3. Delete old logs (must do this!)
rm -f /home/kkcqh/workspace/logs/$(date +%Y-%m-%d)/endofload.log

# 4. Run test
roslaunch hybrid_astar manual.launch

# 5. Check new behavior
grep "[GoalGuard]" /home/kkcqh/workspace/logs/$(date +%Y-%m-%d)/endofload.log
```

## Absolute Prohibitions:

- ❌ **DO NOT** modify tracker code (`pcaf/.../steering_controller.cc`) — user said "不动跟踪器"
- ❌ **DO NOT** skip reading the 4 mandatory skill files
- ❌ **DO NOT** modify only one file when the fix requires multiple files
- ❌ **DO NOT** forget to rebuild after changes
- ❌ **DO NOT** let user run old logs — delete them before each test run
- ❌ **DO NOT** use `reinterpret_cast` or C-style casts — use `static_cast` with validation

## Mode Switching:

```
ask 模式 ─→ 概念解释 / 流程图 / 方案对比（不写代码）
architect 模式 ─→ 重大重构方案 / 约束清单 / 必问 follow_up_question
code 模式 ─→ search_and_replace 改代码 / 重编译验证
debug 模式 ─→ 日志分析 / grep 排查
```
