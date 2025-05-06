cmd overlay -h
```
console:/ # cmd overlay -h
Overlay manager (overlay) commands:
  help
    Print this help text.
  dump [--verbose] [--user USER_ID] [[FIELD] PACKAGE[:NAME]]
    Print debugging information about the overlay manager.
    With optional parameters PACKAGE and NAME, limit output to the specified
    overlay or target. With optional parameter FIELD, limit output to
    the corresponding SettingsItem field. Field names are all lower case
    and omit the m prefix, i.e. 'userid' for SettingsItem.mUserId.
  list [--user USER_ID] [PACKAGE[:NAME]]
    Print information about target and overlay packages.
    Overlay packages are printed in priority order. With optional
    parameters PACKAGE and NAME, limit output to the specified overlay or
    target.
  enable [--user USER_ID] PACKAGE[:NAME]
    Enable overlay within or owned by PACKAGE with optional unique NAME.
  disable [--user USER_ID] PACKAGE[:NAME]
    Disable overlay within or owned by PACKAGE with optional unique NAME.
  enable-exclusive [--user USER_ID] [--category] PACKAGE
    Enable overlay within or owned by PACKAGE and disable all other overlays
    for its target package. If the --category option is given, only disables
    other overlays in the same category.
  set-priority [--user USER_ID] PACKAGE PARENT|lowest|highest
    Change the priority of the overlay to be just higher than
    the priority of PARENT If PARENT is the special keyword
    'lowest', change priority of PACKAGE to the lowest priority.
    If PARENT is the special keyword 'highest', change priority of
    PACKAGE to the highest priority.
  lookup [--user USER_ID] [--verbose] PACKAGE-TO-LOAD PACKAGE:TYPE/NAME
    Load a package and print the value of a given resource
    applying the current configuration and enabled overlays.
    For a more fine-grained alternative, use 'idmap2 lookup'.
  fabricate [--user USER_ID] [--target-name OVERLAYABLE] --target PACKAGE
            --name NAME [--file FILE] 
            PACKAGE:TYPE/NAME ENCODED-TYPE-ID/TYPE-NAME ENCODED-VALUE
    Create an overlay from a single resource. Caller must be root. Example:
      fabricate --target android --name LighterGray \
                android:color/lighter_gray 0x1c 0xffeeeeee
255|console:/ # 
```

以下是针对 Android 中 `cmd overlay` 命令的详细解析和使用指南，该命令用于管理 ​**运行时资源覆盖层（Runtime Resource Overlay, RRO）​**，常用于动态替换应用或系统资源（如主题、字符串、样式等）。

---

### ​**一、核心命令功能概览**

|命令|作用|
|---|---|
|`list`|列出所有覆盖层及其状态（按优先级排序）|
|`enable` / `disable`|启用或禁用指定覆盖层|
|`set-priority`|调整覆盖层的优先级（解决多个覆盖层冲突）|
|`lookup`|调试资源覆盖效果（查看资源实际应用值）|
|`fabricate`|临时创建覆盖层（需 root 权限）|

---

### ​**二、高频使用场景与示例**

#### ​**1. 查看当前覆盖层状态**
```bash
# 列出所有覆盖层（含目标包名、状态、优先级）
cmd overlay list

# 过滤特定覆盖层（如过滤 com.android.systemui 的覆盖层）
cmd overlay list com.android.systemui
```

**输出示例**：
```
com.example.theme:android
  [ ] com.example.darktheme # 未启用
  [x] com.example.bluetheme (priority 1) # 已启用，优先级1
```

---

#### ​**2. 启用/禁用覆盖层**
```bash
# 启用覆盖层（需包名和覆盖层名称）
cmd overlay enable com.example.theme:DarkModeOverlay

# 禁用覆盖层
cmd overlay disable com.example.theme:DarkModeOverlay

# 启用独占模式（禁用同一目标的其他覆盖层）
cmd overlay enable-exclusive --category com.example.theme
```

---

#### ​**3. 调整覆盖层优先级**

当多个覆盖层修改同一资源时，优先级高的覆盖层生效。
```bash
# 将覆盖层设为最高优先级
cmd overlay set-priority com.example.theme:DarkModeOverlay highest

# 将覆盖层设为最低优先级
cmd overlay set-priority com.example.theme:DarkModeOverlay lowest

# 调整覆盖层到指定父级上方（示例中将 com.example.A 移到 com.example.B 上方）
cmd overlay set-priority com.example.A com.example.B
```

---

#### ​**4. 调试资源覆盖（`lookup`）​**

验证资源是否被正确覆盖（需知道资源 ID 或名称）：
```bash
# 语法：cmd overlay lookup [--user USER_ID] 目标包名 资源类型/资源名
cmd overlay lookup --user 0 android:color/primary_dark
```

**输出示例**：
```
RESULT: 0x7f04001a # 覆盖后的资源 ID
```

---

#### ​**5. 临时创建覆盖层（`fabricate`）​**

**需 root 权限**，快速测试资源覆盖效果：
```bash
# 示例：覆盖系统颜色资源为浅灰色
cmd overlay fabricate --user 0 --target android --name TestOverlay \
    android:color/background_dark color 0xffeeeeee
```

**参数解析**：
- `--target android`：目标包名为系统资源
- `android:color/background_dark`：要覆盖的资源
- `color 0xffeeeeee`：资源类型和ARGB颜色值

---

### ​**三、关键参数说明**

|参数|说明|
|---|---|
|`--user USER_ID`|指定用户（0=主用户，10+为多用户）|
|`--category`|与 `enable-exclusive` 联用，仅禁用同分类覆盖层|
|`--verbose`|输出详细信息（如 `dump` 命令）|

---

### ​**四、典型应用场景**

1. ​**主题切换**  
    动态启用不同主题覆盖层（如深色模式）：
    ```bash
    cmd overlay enable com.android.theme.dark:DarkThemeOverlay
    ```
    
2. ​**CTS测试修复**  
    临时禁用冲突覆盖层以通过测试：
    ```bash
    cmd overlay disable com.vendor.invalidoverlay
    ```
    
3. ​**多设备适配**  
    通过优先级控制不同设备的资源覆盖顺序：
    ```bash
    cmd overlay set-priority com.device.specific.overlay highest
    ```
    
4. ​**资源调试**  
    使用 `lookup` 验证资源是否被正确覆盖。

---

### ​**五、注意事项**

1. ​**权限要求**：部分命令需要 `adb shell root` 权限（如 `fabricate`）。
2. ​**重启影响**：覆盖层状态默认不持久化，重启后恢复原配置。若需持久化，需修改 `/vendor/overlay` 目录下的配置。
3. ​**优先级冲突**：同一目标包的覆盖层优先级需明确，否则可能导致资源覆盖未生效。
4. ​**日志分析**：若覆盖层未生效，可通过 `logcat | grep OverlayManagerService` 查看错误日志。

通过灵活使用这些命令，开发者可以高效调试资源覆盖问题或实现动态主题切换等功能。