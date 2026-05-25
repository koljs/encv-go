# 修复 FilePreview.vue TS2339: Property 'segment_count' does not exist

## 问题分析

CI 报错：
```
Error: src/views/FilePreview.vue(83,59): error TS2339: Property 'segment_count' does not exist on type '{ version: number; container_id: string; container_type: string; is_seekable: boolean; original_duration?: number | undefined; segments: unknown[]; }'.
```

**根因**：`FilePreview.vue` 中的 `ContainerInfo` 接口（第 130-137 行）缺少 `segment_count` 属性，但模板第 83 行使用了 `containerInfo.segment_count`。

对比：
- `FileInfo.vue` 的 `ContainerData` 接口已正确定义了 `segment_count?: number | null` ✅
- `FilePreview.vue` 的 `ContainerInfo` 接口缺少 `segment_count` ❌
- 后端 `mobile_service.go:334` 确实返回了 `"segment_count": len(mf.Segments)` ✅

## 修复方案

在 `FilePreview.vue` 的 `ContainerInfo` 接口中添加 `segment_count` 属性，与 `FileInfo.vue` 的定义保持一致。

### 修改文件

**`app/encv-mobile/src/views/FilePreview.vue`** 第 130-137 行：

```typescript
// 修改前
interface ContainerInfo {
  version: number
  container_id: string
  container_type: string
  is_seekable: boolean
  original_duration?: number
  segments: unknown[]
}

// 修改后
interface ContainerInfo {
  version: number
  container_id: string
  container_type: string
  is_seekable: boolean
  original_duration?: number
  segment_count?: number
  segments: unknown[]
}
```

### 验证

运行 `vue-tsc --noEmit && vite build` 确认 TS2339 错误已消除。
