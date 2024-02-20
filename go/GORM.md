# GORM

## 声明模型

> ### 约定

1. **主键：**GORM 使用名为每个模型的默认主键的字段。`ID`
2. **表名：**默认情况下，GORM 将结构名称转换为表名并将其复数化。例如，一个结构变为数据库。`snake_case User -> users`
3. **列名：**GORM会自动将结构字段名称转化为数据库中的列名。`snake_Case`



`gorm.Model`

GORM 提供了一个名为 的预定义结构，其中包括常用字段：`gorm.Model`

```
// gorm.Model 的定义
type Model struct {
  ID        uint           `gorm:"primaryKey"`
  CreatedAt time.Time
  UpdatedAt time.Time
  DeletedAt gorm.DeletedAt `gorm:"index"`
}
```

- 嵌入到结构中：您可以直接**嵌入到结构中**以自动包含这些字段。这对于保持不同模型之间的一致性和利用 GORM 的内置约定非常有用，请参阅[嵌入式结构](https://gorm.io/zh_CN/docs/models.html#embedded_struct)体`gorm.Model`
- **字段包括**：
  - `ID`：每条记录的唯一标识符（主键）。
  - `CreatedAt`：自动设置为创建记录时的当前时间。
  - `UpdatedAt`：每当记录更新时自动更新到当前时间。
  - `DeletedAt`：用于软删除（将记录标记为已删除，而不实际将其从数据库中删除）。