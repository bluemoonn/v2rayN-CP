# 版本号格式说明

## 概述

不同平台对版本号格式有不同要求。本项目自动转换版本号以适配各平台。

## 版本号转换规则

### 输入格式（Git Tag）
支持以下格式：
- `v7.16.3` - 标准版本
- `v7.16.3-no-ad` - 带后缀版本
- `7.16.3-beta-1` - 预发布版本

### 各平台输出格式

| 平台 | 转换规则 | 示例输入 | 输出结果 |
|------|---------|---------|---------|
| **Windows** | 保持原样 | `v7.16.3-no-ad` | `7.16.3-no-ad` |
| **macOS** | 保持原样 | `v7.16.3-no-ad` | `7.16.3-no-ad` |
| **Linux (deb)** | 第一个 `-` 转为 `~`，其余 `-` 转为 `.` | `v7.16.3-no-ad` | `7.16.3~no.ad` |
| **Linux (rpm)** | 第一个 `-` 转为 `_`，其余 `-` 转为 `.` | `v7.16.3-no-ad` | `7.16.3_no.ad` |

### 详细说明

#### Debian/Ubuntu (.deb)
- Debian 包版本格式：`<version>~<suffix>`
- 波浪号 `~` 表示预发布版本，排序时会低于正式版
- 不允许使用连字符 `-`（除了 revision 部分）
- 示例：
  - `v7.16.3-no-ad` → `7.16.3~no.ad`
  - `v7.16.3-beta-1` → `7.16.3~beta.1`

#### Red Hat/RPM (.rpm)
- RPM 包版本格式：`<version>_<suffix>`
- 下划线 `_` 用于分隔主版本和附加信息
- 连字符 `-` 保留用于 Release 字段
- 示例：
  - `v7.16.3-no-ad` → `7.16.3_no.ad`
  - `v7.16.3-beta-1` → `7.16.3_beta.1`

#### Windows/macOS
- 无严格限制
- 移除前导 `v`
- 保持其他字符不变
- 示例：
  - `v7.16.3-no-ad` → `7.16.3-no-ad`

## 使用建议

### 推荐的版本号格式

1. **标准版本**：`v7.16.3`
   - 所有平台：`7.16.3`

2. **定制版本**：`v7.16.3-no-ad`
   - Windows/macOS: `7.16.3-no-ad`
   - Debian: `7.16.3~no.ad`
   - RPM: `7.16.3_no.ad`

3. **预发布版本**：`v7.16.3-beta-1`
   - Windows/macOS: `7.16.3-beta-1`
   - Debian: `7.16.3~beta.1`
   - RPM: `7.16.3_beta.1`

### 避免的格式

❌ 不要使用：
- 多个连续的连字符：`v7.16.3--custom`
- 特殊字符：`v7.16.3+custom` (在某些系统可能有问题)
- 空格：`v7.16.3 custom`

## 技术实现

### 自动转换脚本

#### package-debian.sh
```bash
# Sanitize version for Debian package
Version=$(echo "$Version" | sed 's/^[vV]//' | sed 's/-/~/1' | sed 's/-/./g')
```

#### package-rhel.sh
```bash
sanitize_rpm_version() {
  local ver="$1"
  ver="${ver#[vV]}"
  ver=$(echo "$ver" | sed 's/-/_/1' | sed 's/-/./g')
  echo "$ver"
}
```

## 常见问题

**Q: 为什么 Linux 包的版本号和 Windows 不一样？**  
A: Linux 包管理系统对版本号格式有严格要求，必须遵守各发行版的规范。

**Q: 我可以使用 `v7.16.3-custom-build-1` 吗？**  
A: 可以，会自动转换为：
- Debian: `7.16.3~custom.build.1`
- RPM: `7.16.3_custom.build.1`

**Q: 版本号中的波浪号 `~` 是什么意思？**  
A: 在 Debian 系统中，波浪号表示预发布版本，版本比较时会低于同数字的正式版本。例如 `7.16.3~beta` < `7.16.3`。
