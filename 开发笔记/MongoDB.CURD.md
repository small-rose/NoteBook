## 一、查询与映射运算符

- 查询选择器
- 映射运算符

### 1、查询选择器

有关不同BSON类型值的比较，请参见指定的BSON比较顺序。

名称 |  用法 示例 | 描述 | 
$eq  | {field:{$eq:value}} | 匹配字段field值等于指定值value的文档	|

$gt | {field:{$gt:value} |  匹配字段field值大于指定值value的文档 |

{field:{$gte:value}}	匹配字段field值≥指定值value的文档

{field:{$lt:value}}	匹配字段field值<指定值value的文档

{field:{$lte:value}}	匹配字段field值≤指定值value的文档

{field:{$ne:value}}	匹配字段field值≠指定值value的文档

{field:$in:[value1,value2,...]}	匹配字段field值 in 指定数组中的value值的文档

{field:$nin:[value1,value2,…]}	匹配字段field值 not in 指定数组中的value值的文档
 


### 2、映射运算符


## 二、查询与映射运算符