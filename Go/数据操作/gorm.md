# 注意事项

### 软删除与唯一复合索引

##### 软删除字段类型

GORM 中的软删除字段（如 `gorm.DeletedAt` 或包含在 `gorm.Model` 中）默认是 `*time.Time` 类型，对应数据库的 `DATETIME` 或 `TIMESTAMP` 类型，允许存储 `NULL` 值。未删除的记录，此字段即为 `NULL`。

不过软删除插件 `gorm.io/plugin/soft_delete`同时也提供其他的数据格式支持。

##### 核心问题与冲突

大多数 SQL 数据库（如 MySQL、PostgreSQL、SQLite 等）在处理**唯一索引**时，会将多个 **NULL** 值视为**不重复**的值。

当尝试创建一个包含业务字段和默认 `DeletedAt` 字段的唯一复合索引时，会产生冲突。

例如：`UNIQUE (Name, DeletedAt)`，可以创建多条 `(Name='A', DeletedAt=NULL)` 的记录，因为 `NULL` 不被认为是重复的。这违背了“未删除的记录中，`Name` 字段必须是唯一的”的业务逻辑。

##### 解决方案

**提示：** 当使用 `DeletedAt` 创建唯一复合索引时，**必须**使用其他非 `NULL` 的数据类型来表示未删除状态，例如通过 `gorm.io/plugin/soft_delete` 插件将字段类型定义为 Unix 时间戳。

```go
import "gorm.io/plugin/soft_delete"

type User struct {
  ID        uint
  Name      string                `gorm:"uniqueIndex:udx_name"`
  DeletedAt soft_delete.DeletedAt `gorm:"uniqueIndex:udx_name"`
}
```

将 `DeletedAt` 定义为 `soft_delete.DeletedAt`（Unix 时间戳），未删除记录的 `DeletedAt` 值为 `0`。
