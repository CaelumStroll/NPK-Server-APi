# API 文档

Base URL: `http://localhost:8556/api`

## 通用响应格式

```json
{ "code": 0, "message": "success", "data": {} }
{ "code": 1, "message": "error message" }
```

## NPK 管理

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/npk/list` | NPK 文件列表 |
| GET | `/npk/list/paginated?q=&page=1&size=50` | 分页列表 |
| POST | `/npk/scan` | 扫描目录 `{path}` |
| GET | `/npk/:name/info` | NPK 详细信息 |
| GET | `/npk/:name/assets` | Album 列表 |
| GET | `/npk/frames?npk=&album=` | 帧详情列表 |
| POST | `/npk/:name/save` | 保存 NPK |
| DELETE | `/npk/:name` | 删除 NPK |

## IMG 帧操作

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/npk/:name/assets/:index?album=` | 获取帧图像 |
| PUT | `/npk/:name/assets/:index/frame/:fid?album=` | 更新帧属性 `{x,y,width,height,frame_width,frame_height}` |
| DELETE | `/npk/:name/assets/:index?album=` | 删除帧 |
| POST | `/npk/:name/assets/add-link?album=` | 添加引用帧 `{targetIndex, position}` |
| POST | `/npk/:name/assets/modify-link?album=` | 修改引用目标 `{spriteIndex, newTargetIndex}` |

## IMG 编辑辅助

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/npk/:name/assets/batch-replace` | 批量替换帧（FormData: files, album, color_format, start_index） |
| POST | `/npk/:name/assets/batch-import` | 批量导入帧（FormData: files, album, position, current_index, color_format） |
| GET | `/npk/:name/album/export-coords?album=` | 导出基准坐标（x,y 每行） |
| POST | `/npk/:name/album/import-coords` | 导入基准坐标（FormData: file, album） |
| GET | `/npk/preview?npk=&album=&mode=combined` | Album 预览画布 |
| GET | `/npk/:name/assets/:index/ani/frame/:fid?album=` | 动画帧 |

## 外部文件对比

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/external-npk/albums?npkPath=` | 外部 NPK Album 列表（含 Sprite 元数据） |
| GET | `/external-npk/frame?npkPath=&albumPath=&frameIndex=` | 外部 NPK 帧图像 |
| POST | `/external-img/upload` | 上传外部 IMG（FormData: file） |
| GET | `/external-img/info?imgPath=` | 外部 IMG 信息 |
| GET | `/external-img/frame?imgPath=&frameIndex=` | 外部 IMG 帧图像 |

## 音乐管理

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/music/list?query=&format=&page=&size=` | 音乐列表 |
| POST | `/music/scan` | 扫描目录 |
| GET | `/music/:id/stream` | 流式播放（支持 Range） |
| POST | `/music/:id/convert` | 格式转换 `{format}` |

## 音效管理

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/soundpacks/list?query=` | 音效包列表（支持搜索文件名/OGG名/路径） |
| GET | `/soundpacks/:id` | 音效包详情 |
| GET | `/soundpacks/:id/entries` | OGG 条目列表 |
| GET | `/soundpacks/:id/entries/:eid/stream` | 流式播放 OGG |
| POST | `/soundpacks/:id/entries/:eid/download` | 下载 OGG |
| POST | `/soundpacks/:id/entries/import` | 导入 OGG |
| POST | `/soundpacks/:id/entries/batch-export` | 批量导出 |
| POST | `/soundpacks/:id/entries/batch-import` | 批量导入 |
| POST | `/soundpacks/:id/entries/batch-delete` | 批量删除 |

## XML 音频编辑器

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/xml-audio/list?query=&type=` | 条目列表（按 type 筛选） |
| GET | `/xml-audio/entry/:type/:id` | 条目详情 |
| POST | `/xml-audio/entry` | 保存条目（新增/编辑） |
| DELETE | `/xml-audio/entry/:type/:id` | 删除条目 |
| POST | `/xml-audio/open` | 打开 XML 文件 `{path}` |
| GET | `/xml-audio/list-files?dir=` | 浏览目录中的 XML 文件 |
| GET | `/xml-audio/stream?file=` | 流式播放音频（自动从音乐库/音效包查找） |
| POST | `/xml-audio/export` | 导出到 XML 文件 |
| GET | `/xml-audio/check-missing` | 检查缺失文件 |
| POST | `/xml-audio/reload` | 从 XML 重新加载 |

## 视频管理

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/videos/list?query=&format=&page=&size=` | 视频列表 |
| GET | `/videos/search?query=&format=&encrypted_only=` | 视频搜索 |
| POST | `/videos/scan` | 扫描目录 |

## 数据库管理

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/db/stats` | 数据库统计 |
| POST | `/db/compact` | 压缩数据库（VACUUM） |
| POST | `/db/music/rebuild` | 重建音乐索引 |
| POST | `/db/soundpacks/rebuild` | 重建音效索引 |

## 设置

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/settings` | 获取设置 |
| POST | `/settings` | 更新设置 |
| POST | `/settings/npk-dir` | 添加 NPK 目录 |
| DELETE | `/settings/npk-dir` | 移除 NPK 目录 |
