---
name: downloaddeb
description: Download deb packages from Kylinos archive with version control and multi-architecture support. Use this when users need to download specific deb packages with exact versions from Kylinos repository for arm64, amd64, loongarch64, and sw64 architectures.
license: MIT
---

# Kylinos Deb Package Downloader

This skill helps download deb packages from the Kylinos archive with precise version control and multi-architecture support.

## Repository URLs

The packages are hosted at:
- http://archive.kylinos.cn/kylin/KYLIN-ALL/pool/main/
- http://archive.kylinos.cn/kylin/KYLIN-ALL/pool/universe
- http://archive.kylinos.cn/kylin/KYLIN-ALL/pool/restricted/
- http://archive.kylinos.cn/kylin/KYLIN-ALL/pool/multiverse

## Path Convention

Packages are organized by first letter of package name:

- **Regular packages**: First letter determines subdirectory
  - Example: `ukui-notebook_3.2.0.1-0k2.16_arm64.deb` → `pool/main/u/ukui-notebook/`

- **Library packages** (lib*): First 4 characters determine subdirectory
  - Example: `liblqr-1-0_0.4.2-2.1_arm64.deb` → `pool/main/libl/liblqr-1-0/`

- **Fallback search**: If package not found in normal paths
  - Try `lib{first_letter}/{package}/` (e.g., `libx/xsltproc/`)
  - If still not found, **search all `libxxxxx/` directories** for the package
  - Example: `xsltproc` → found in `libx/libxslt/` (not `libx/xsltproc/`)

## Supported Architectures

Download packages for all 4 architectures:
1. **arm64** - ARM 64-bit
2. **amd64** - AMD/Intel 64-bit
3. **loongarch64** - LoongArch 64-bit
4. **sw64** - Sunway 64-bit

## Input Format

The user will provide a text file with packages to download. **Two formats are supported:**

### Format 1: Full deb filename (Recommended)
```
package_name_version_architecture.deb
```

Example:
```
ukui-notebook_3.2.0.1-0k2.16_arm64.deb
liblqr-1-0_0.4.2-2.1_amd64.deb
aisleriot_1%3a3.22.9-1_arm64.deb
```

**Format 1 解析规则**：
- 从文件名提取包名（如 `ukui-notebook`）和版本（如 `3.2.0.1-0k2.16`）
- **忽略文件名中的架构，下载所有 4 个架构**

### Format 2: Package name and version
```
package_name=version
```

Example:
```
aisleriot=1:3.22.9-1
gomoku.app=1.2.9-4build1
ukui-notebook=3.2.0.1-0k2.16
```

**两种格式都会下载所有 4 个架构**：arm64、amd64、loongarch64、sw64

## Download Process

For each package:
1. Parse package name and version from input file
2. For each architecture (arm64, amd64, loongarch64, sw64):
   - **Step 1**: Try normal path `{first_letter}/{package}/` (e.g., `x/xsltproc/`)
   - **Step 2**: Try `lib{first_letter}/{package}/` (e.g., `libx/xsltproc/`)
   - **Step 3**: If still not found, **search all `libxxxxx/` directories** for the package
     - Get directory listing of `{pool}/lib*/`
     - Search through each `libxxxxx/` subdirectory for the package
     - This finds packages like `xsltproc` in `libx/libxslt/`
   - Attempt to download from both `main` and `universe` pools
   - Save to output directory (default: `/root/deb/`)

## Usage

When this skill is invoked:
1. Ask for the input file path if not provided
2. Parse the package list
3. Download all packages for all architectures
4. Report success/failure for each package

## Error Handling

- Skip if package already exists (check file size > 0)
- Report if package not found for specific architecture
- Clean up failed/incomplete downloads
- Provide summary of all downloaded packages
