# 数据库设计

## 核心表

### 音频 XML 表

```sql
CREATE TABLE audio_entries (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    entry_type TEXT NOT NULL,       -- EFFECT | RANDOM | GROUP | MUSIC | AMBIENT
    entry_id TEXT NOT NULL,         -- XML 中的 ID
    file TEXT DEFAULT '',           -- 音频文件路径
    attributes TEXT DEFAULT '{}',   -- JSON 键值对
    sort_order INTEGER DEFAULT 0   -- 排序序号
);
CREATE INDEX idx_audio_entries_type_eid ON audio_entries(entry_type, entry_id);

CREATE TABLE audio_items (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    entry_id INTEGER NOT NULL,      -- FK → audio_entries.id
    tag TEXT NOT NULL,              -- ITEM 的 TAG
    prob TEXT DEFAULT '',           -- 概率
    delay TEXT DEFAULT '',
    delay_range TEXT DEFAULT '',
    loop_delay TEXT DEFAULT '',
    loop_delay_range TEXT DEFAULT '',
    sort_order INTEGER DEFAULT 0,
    FOREIGN KEY (entry_id) REFERENCES audio_entries(id) ON DELETE CASCADE
);
```

### 音乐表

```sql
CREATE TABLE music (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    path TEXT NOT NULL UNIQUE,
    dir_path TEXT NOT NULL,
    size INTEGER DEFAULT 0,
    format TEXT,
    duration REAL DEFAULT 0,
    sample_rate INTEGER DEFAULT 0,
    channels INTEGER DEFAULT 0,
    bitrate INTEGER DEFAULT 0,
    sha256_hash TEXT DEFAULT '',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### 音效包表

```sql
CREATE TABLE soundpacks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    path TEXT NOT NULL UNIQUE,
    dir_path TEXT NOT NULL,
    size INTEGER DEFAULT 0,
    is_encrypted INTEGER DEFAULT 0,
    format_type INTEGER DEFAULT 0,
    ogg_count INTEGER DEFAULT 0,
    sha256_hash TEXT DEFAULT ''
);

CREATE TABLE soundpack_entries (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    soundpack_id INTEGER NOT NULL,
    entry_path TEXT NOT NULL,
    entry_name TEXT NOT NULL,
    entry_size INTEGER DEFAULT 0,
    entry_offset INTEGER DEFAULT 0,
    ogg_duration REAL DEFAULT 0,
    ogg_sample_rate INTEGER DEFAULT 0,
    ogg_channels INTEGER DEFAULT 0,
    data BLOB,
    UNIQUE(soundpack_id, entry_path),
    FOREIGN KEY (soundpack_id) REFERENCES soundpacks(id) ON DELETE CASCADE
);
```

## 关键查询优化

### N+1 修复 - 批量加载子条目
```sql
-- 之前：每个 entry 单独查 items（2000 次查询）
SELECT ... FROM audio_items WHERE entry_id = ?

-- 之后：一条 SQL 加载所有 items
SELECT ... FROM audio_items WHERE entry_id IN (?,?,...)
```

### 统计合并 - 5 次 COUNT → 1 次
```sql
SELECT
    COALESCE(SUM(CASE WHEN entry_type='EFFECT' THEN 1 ELSE 0 END), 0),
    COALESCE(SUM(CASE WHEN entry_type='RANDOM' THEN 1 ELSE 0 END), 0),
    COALESCE(SUM(CASE WHEN entry_type='GROUP'  THEN 1 ELSE 0 END), 0),
    COALESCE(SUM(CASE WHEN entry_type='MUSIC'  THEN 1 ELSE 0 END), 0),
    COALESCE(SUM(CASE WHEN entry_type='AMBIENT' THEN 1 ELSE 0 END), 0)
FROM audio_entries
```

### 缺失文件检查（SQL 批量）
```sql
-- 音乐库路径匹配
SELECT COUNT(*) FROM music WHERE path = ? OR REPLACE(path,'\','/') = ?
-- 音效包路径匹配
SELECT COUNT(*) FROM soundpack_entries WHERE REPLACE(entry_path,'\','/') = ?
```

## 通用工具

```go
// BuildInClause 构建 IN (?,?,?) 占位符 + 参数列表
func BuildInClause[T int64 | int](ids []T) (string, []interface{})
```

## 配置存储

`config.json` 文件，支持运行时读写：
```json
{
    "server_port": 8556,
    "npk_dirs": ["D:/DNF/ImagePacks2"],
    "music_dirs": ["D:/DNF/Music"],
    "soundpack_dirs": ["D:/DNF/SoundPacks"],
    "audio_xml_path": "D:/DNF/Audio.xml"
}
```

路径自动推导：设置 `npk_dirs` 后自动生成其他路径。
