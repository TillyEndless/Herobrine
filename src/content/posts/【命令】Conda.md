---
title: 【命令】Conda
published: 2025-02-22T00:00:00.000Z
description: ''
image: ''
tags:
  - journal
category: 命令
draft: false
lang: ''
---
***目录***

<!-- toc -->

- [**1. 环境管理基础**](#1-%E7%8E%AF%E5%A2%83%E7%AE%A1%E7%90%86%E5%9F%BA%E7%A1%80)
- [**2. 创建与删除环境**](#2-%E5%88%9B%E5%BB%BA%E4%B8%8E%E5%88%A0%E9%99%A4%E7%8E%AF%E5%A2%83)
- [**3. 激活与退出环境**](#3-%E6%BF%80%E6%B4%BB%E4%B8%8E%E9%80%80%E5%87%BA%E7%8E%AF%E5%A2%83)
- [**4. 包管理**](#4-%E5%8C%85%E7%AE%A1%E7%90%86)
- [**5. 环境配置与导出**](#5-%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE%E4%B8%8E%E5%AF%BC%E5%87%BA)
- [**6. 高级操作**](#6-%E9%AB%98%E7%BA%A7%E6%93%8D%E4%BD%9C)
- [**7. 环境路径与信息**](#7-%E7%8E%AF%E5%A2%83%E8%B7%AF%E5%BE%84%E4%B8%8E%E4%BF%A1%E6%81%AF)
- [**常用场景示例**](#%E5%B8%B8%E7%94%A8%E5%9C%BA%E6%99%AF%E7%A4%BA%E4%BE%8B)
  * [**从零创建并管理环境**](#%E4%BB%8E%E9%9B%B6%E5%88%9B%E5%BB%BA%E5%B9%B6%E7%AE%A1%E7%90%86%E7%8E%AF%E5%A2%83)
  * [**克隆并清理环境**](#%E5%85%8B%E9%9A%86%E5%B9%B6%E6%B8%85%E7%90%86%E7%8E%AF%E5%A2%83)
- [**注意事项**](#%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)

<!-- tocstop -->

以下是 Conda 环境管理的常用命令，按功能分类整理：

---

### **1. 环境管理基础**
| 命令 | 说明 |
|------|------|
| `conda --version` | 检查 Conda 版本 |
| `conda update conda` | 更新 Conda 自身 |
| `conda env list` 或 `conda info --envs` | 列出所有已创建的环境（`*` 表示当前环境） |

---

### **2. 创建与删除环境**
| 命令 | 说明 |
|------|------|
| `conda create --name myenv` | 创建名为 `myenv` 的空白环境 |
| `conda create --name myenv python=3.9` | 创建指定 Python 版本的环境 |
| `conda create --name myenv --clone base` | 克隆 `base` 环境到 `myenv` |
| `conda remove --name myenv --all` | 彻底删除环境 `myenv` |

---

### **3. 激活与退出环境**
| 命令 | 说明 |
|------|------|
| `conda activate myenv` | 激活环境（Windows/macOS/Linux 通用） |
| `conda deactivate` | 退出当前环境（返回上一级或系统环境） |
| `source activate myenv` | 旧版 Conda（4.4 之前）激活方式（仅 Linux/macOS） |

---

### **4. 包管理**
| 命令 | 说明 |
|------|------|
| `conda list` | 查看当前环境已安装的包 |
| `conda search numpy` | 搜索包 `numpy` 的可用版本 |
| `conda install numpy` | 安装包（默认最新版本） |
| `conda install numpy=1.21` | 安装指定版本的包 |
| `conda install -c conda-forge pandas` | 从特定频道（如 `conda-forge`）安装包 |
| `conda remove numpy` | 卸载包 |
| `conda update numpy` | 更新指定包 |
| `conda update --all` | 更新当前环境所有包 |

---

### **5. 环境配置与导出**
| 命令 | 说明 |
|------|------|
| `conda env export > environment.yml` | 导出当前环境的包列表到 `environment.yml`（包含精确版本） |
| `conda env create -f environment.yml` | 根据 `environment.yml` 文件创建环境 |
| `conda env update -f environment.yml` | 更新当前环境以匹配 `environment.yml` |

---

### **6. 高级操作**
| 命令 | 说明 |
|------|------|
| `conda clean --all` | 清理未使用的包和缓存 |
| `conda config --show` | 查看 Conda 配置（如默认频道、环境路径） |
| `conda config --set env_prompt "({name}) "` | 自定义终端提示符的环境名称显示格式 |

---

### **7. 环境路径与信息**
| 命令 | 说明 |
|------|------|
| `conda info` | 查看 Conda 系统信息（包括环境存储路径） |
| `which python`（Linux/macOS）或 `where python`（Windows） | 查看当前 Python 解释器路径 |

---

### **常用场景示例**

#### **从零创建并管理环境**
```bash
# 创建 Python 3.10 环境
conda create --name myproject python=3.10
conda activate myproject

# 安装必要包
conda install numpy pandas matplotlib
conda install -c conda-forge scikit-learn

# 导出环境配置
conda env export > environment.yml

# 退出环境
conda deactivate
```

#### **克隆并清理环境**
```bash
# 克隆现有环境
conda create --name myproject_backup --clone myproject

# 删除旧环境
conda remove --name myproject --all
```

---

### **注意事项**
1. **频道优先级**：使用 `-c` 指定频道时，优先级高于默认频道。
2. **跨平台兼容性**：导出的 `environment.yml` 可能因操作系统不同导致依赖冲突（需手动调整）。
3. **混合使用 Conda 和 Pip**：在 Conda 环境中使用 `pip install` 可能导致依赖冲突，建议优先使用 Conda 包。

掌握这些命令后，你可以高效管理 Python 项目依赖和环境！如果有其他问题，随时告诉我具体场景，我会提供更针对性的建议。
