# 📚 记忆管理逻辑说明

## 🧠 记忆管理概述

本系统的记忆管理包含两个主要阶段：**读取阶段**（生成前）和**保存阶段**（生成后）。

## 🔄 工作流程

### 📖 生成前 - 记忆读取阶段

当用户勾选 **"启用历史记录"** 时，系统会在生成小说前读取历史对话记忆并加入提示词。

#### 选项1: 读取压缩记忆 ❌
- **前端选项**: `读取压缩记忆` (未勾选)
- **技术实现**: `read_compressed: false`
- **读取源**: `memory/chunks/` 文件夹
- **文件格式**: `{novel_id}_chunk_001.json`
- **内容**: 完整的原始对话记录

#### 选项2: 读取压缩记忆 ✅
- **前端选项**: `读取压缩记忆` (已勾选)
- **技术实现**: `read_compressed: true`
- **读取源**: `memory/summaries/` 文件夹
- **文件格式**: `{novel_id}_summary_001.json`
- **内容**: AI压缩后的对话摘要

### 💾 生成后 - 记忆保存阶段

当用户勾选 **"生成后压缩"** 时，系统会在生成完成后自动压缩本次对话记录。

#### 选项1: 生成后压缩 ❌
- **前端选项**: `生成后压缩` (未勾选)
- **技术实现**: `use_compression: false`
- **操作**: 仅保存原始对话到 `chunks/` 文件夹
- **结果**: 不执行压缩操作

#### 选项2: 生成后压缩 ✅
- **前端选项**: `生成后压缩` (已勾选)
- **技术实现**: `use_compression: true`
- **操作**: 
  1. 保存原始对话到 `chunks/` 文件夹
  2. 自动压缩最新分片并保存到 `summaries/` 文件夹
- **结果**: 同时拥有原始记忆和压缩记忆

## 📁 文件结构

```
memory/
├── chunks/                          # 原始记忆分片
│   ├── {novel_id}_chunk_001.json   # 完整对话记录
│   └── {novel_id}_chunk_002.json
├── summaries/                       # 压缩记忆摘要
│   ├── {novel_id}_summary_001.json # AI压缩的摘要
│   └── {novel_id}_summary_002.json
└── {novel_id}_index.json           # 记忆索引文件
```

## 🎯 使用场景

### 场景1: 新手用户 - 简单模式
```
✅ 启用历史记录
❌ 读取压缩记忆     → 读取完整对话
❌ 生成后压缩       → 不压缩，保持原始记录
```
**特点**: 简单直接，记忆完整，但占用空间较大

### 场景2: 高级用户 - 效率模式  
```
✅ 启用历史记录
✅ 读取压缩记忆     → 读取压缩摘要
✅ 生成后压缩       → 自动压缩新记录
```
**特点**: 节省空间，提高效率，适合长期项目

### 场景3: 混合模式 - 平衡使用
```
✅ 启用历史记录
❌ 读取压缩记忆     → 读取完整对话
✅ 生成后压缩       → 压缩保存，为未来准备
```
**特点**: 当前使用完整记忆，为未来压缩读取做准备

## 🔧 技术细节

### 后端API参数
```json
{
  "use_memory": true,           // 是否启用历史记录
  "read_compressed": false,     // 是否读取压缩记忆
  "use_compression": true,      // 是否生成后压缩
  "recent_count": 20           // 读取最近N条记录
}
```

### 前端界面
```html
📚 记忆设置:
☐ 启用历史记录        <!-- use_memory -->
☐ 读取压缩记忆        <!-- read_compressed -->
☐ 生成后压缩          <!-- use_compression -->

💡 启用历史记录：生成前读取对话记忆到提示词
💡 读取压缩记忆：读取已压缩的记忆摘要(summaries)，否则读取原始记忆(chunks)
💡 生成后压缩：生成完成后自动压缩本次对话记录
```

## 📊 记忆统计信息

系统提供详细的记忆统计：
- **总消息数**: 所有保存的对话消息数量
- **分片数量**: chunks文件夹中的分片数量
- **压缩分片数**: summaries文件夹中的压缩摘要数量

## ⚠️ 注意事项

1. **首次使用**: 如果勾选"读取压缩记忆"但没有压缩文件，系统会返回空记忆
2. **压缩时机**: 压缩操作在生成完成后自动执行，不影响当前生成过程
3. **存储空间**: 压缩记忆可以大幅减少存储空间，建议长期项目启用
4. **记忆质量**: 压缩记忆由AI生成，可能丢失部分细节，但保留核心信息

## 🧪 测试验证

使用 `test_memory_logic.py` 脚本可以验证各种记忆管理场景：

```bash
python test_memory_logic.py
```

测试包括：
- 不启用记忆的生成
- 启用记忆但不压缩
- 启用记忆并自动压缩
- 读取压缩记忆的生成
- 文件结构验证 