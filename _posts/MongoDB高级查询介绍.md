---
title: MongoDB高级查询介绍
date: 2017-12-3 22:14:19
tags:
- MongoDB
- MongoDB查询
categories:
- 数据库
---

> 在几乎所有的项目中对数据库的操作是必要且高频的，对数据库的操作可以大致可分为读和写，其中读的概率要远远大于写。我们选用非关系性数据库一个很重要的原因在于它的查询相对于关系性数据库来讲是非常高效的，它极大地减少了表之间联合查询，只要数据结构设计合理，很多时候一条查询语句即可得到你想要的结果。

# 基本查询

基本查询指的是通过 `find` 语句进行的查询，也是使用比较多的一种查询，类似于关系型数据库的 `select`。

## 准备测试数据

为了方便进行语法示例，我们在数据库中装填一些数据。
<!-- more -->
```json
{
    "_id" : ObjectId("5a23dff29369d959a2ec17a0"),
    "age" : 30.0,
    "label" : [
        "钓鱼",
        "旅行"
    ],
    "name" : "ZhanShan",
    "nickname" : "张三"
},
{
    "_id" : ObjectId("5a23e0239369d959a2ec17a1"),
    "age" : 28.0,
    "label" : [
        "喝茶"
    ],
    "name" : "LiShi",
    "nickname" : "李四"
},
{
    "_id" : ObjectId("5a23e07b9369d959a2ec17a2"),
    "age" : 17.0,
    "label" : [
        "锻炼",
        "泡吧",
        "直播"
    ],
    "name" : "WangWu",
    "nickname" : "王五"
}
```

## 查询示例

* `findOne()` 执行查询后仅返回一条数据，不管查询条件如何都只会返回一条结果。
    ```json
    # 查询年龄大于18岁的用户
    > db.user.findOne({ age: { $gt: 18 } })
    {
        "_id" : ObjectId("5a23dff29369d959a2ec17a0"),
        "name" : "ZhanShan",
        "nickname" : "张三",
        "age" : 30,
        "label" : [
            "钓鱼",
            "旅行"
        ]
    }
    ```
* `find()` 执行查询后返回多条数据，具体根据传入的查询来决定。
    ```json
    > db.user.find()
    { "_id" : ObjectId("5a23dff29369d959a2ec17a0"), "name" : "ZhanShan", "nickname" : "张三", "age" : 30, "label" : [ "钓鱼", "旅行" ] }
    { "_id" : ObjectId("5a23e0239369d959a2ec17a1"), "name" : "LiShi", "nickname" : "李四", "age" : 28, "label" : [ "喝茶" ] }
    { "_id" : ObjectId("5a23e07b9369d959a2ec17a2"), "name" : "WangWu", "nickname" : "王五", "age" : 17, "label" : [ "锻炼", "泡吧", "直播" ] }

    # 查询参数用类JSON对象来表示，例如 `{name: 'ZhanShan'}`，其中name可以省略引号。
    > db.user.find({name: 'ZhanShan'})
    { "_id" : ObjectId("5a23dff29369d959a2ec17a0"), "name" : "ZhanShan", "nickname" : "张三", "age" : 30, "label" : [ "钓鱼", "旅行" ] }

    # 指定结果返回的字段投影，这和sql中指定返回字段作用相同
    > db.user.find({ age: { $gt: 18 } }, {name:1})
    { "_id" : ObjectId("5a23dff29369d959a2ec17a0"), "name" : "ZhanShan" }
    # 默认主键编号将被投影，如果不希望看到可以明确指定期可见性为0
    > db.user.find({ age: { $gt: 18 } }, {name:1, _id:0})
    { "name" : "ZhanShan" }

    # 比较操作 $lt $lte $gt $gte $ne，用来对字段值进行比较匹配，例如查询未满18周岁的用户。
    > db.user.find({ age: { $lt: 18 } }, {name: 1, age: 1})
    { "_id" : ObjectId("5a23e07b9369d959a2ec17a2"), "name" : "WangWu", "age" : 17 }
    # 对应上面比较操作的含义是小于、小于等于、大于、大于等于和不等于

    # $and 包含多个条件，之间是并的关系，例如查询成年人且爱好都是钓鱼的用户
    > db.user.find({$and:[{age:{$gte:18}}, {label:'钓鱼'}]})
    { "_id" : ObjectId("5a23dff29369d959a2ec17a0"), "name" : "ZhanShan", "nickname" : "张三", "age" : 30, "label" : [ "钓鱼", "旅行" ] }

    # $or 包含多个条件，之间是或的关系，用法和上面示例的类似。
    # $nor 相当于$or取反

    # $not 用作其他条件之上取反，例如查询不姓张的用户
    > db.user.find({nickname: {$not: /张/}})
    { "_id" : ObjectId("5a23e0239369d959a2ec17a1"), "name" : "LiShi", "nickname" : "李四", "age" : 28, "label" : [ "喝茶" ] }
    { "_id" : ObjectId("5a23e07b9369d959a2ec17a2"), "name" : "WangWu", "nickname" : "王五", "age" : 17, "label" : [ "锻炼", "泡吧", "直播" ] }

    # $mod 将查询的值除以第一个给定的值，如果余数等于第二个值则匹配成功，例如查询用户年龄是偶数的有哪些？
    > db.user.find({age: {$mod: [2, 0]}})
    { "_id" : ObjectId("5a23dff29369d959a2ec17a0"), "name" : "ZhanShan", "nickname" : "张三", "age" : 30, "label" : [ "钓鱼", "旅行" ] }
    { "_id" : ObjectId("5a23e0239369d959a2ec17a1"), "name" : "LiShi", "nickname" : "李四", "age" : 28, "label" : [ "喝茶" ] }

    # $in 查询一个键的多个值，只要键匹配其中的一个即可，这类似于sql中的in查询。例如查询爱好是钓鱼或锻炼的用户
    > db.user.find({label: {$in: ['钓鱼', '锻炼']}})
    { "_id" : ObjectId("5a23dff29369d959a2ec17a0"), "name" : "ZhanShan", "nickname" : "张三", "age" : 30, "label" : [ "钓鱼", "旅行" ] }
    { "_id" : ObjectId("5a23e07b9369d959a2ec17a2"), "name" : "WangWu", "nickname" : "王五", "age" : 17, "label" : [ "锻炼", "泡吧", "直播" ] }
    # $nin为不包含，就是上面的条件取反。

    # $all 同时匹配多个值，也就是说要全部满足条件。例如同时有钓鱼和锻炼爱好的用户在目前的数据库中没有
    > db.user.find({label: {$all: ['钓鱼', '锻炼']}})
    >

    # $exists 过滤是否出现指定的字段，1取存在0取不存在，例如判断有填写手机号码的用户是哪些，示例数据中没有添加过phone字段，所以下面的查询肯定没有数据返回。
    > db.user.find({phone: {$exists: 1}})
    >
    # 如果我们为李四增加phone字段，则李四用户会被查询出来
    > db.user.update({name: 'LiShi'}, {$set: {phone: 13723458987}})
    WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
    > db.user.find({phone: {$exists: 1}})
    { "_id" : ObjectId("5a23e0239369d959a2ec17a1"), "name" : "LiShi", "nickname" : "李四", "age" : 28, "label" : [ "喝茶" ], "phone" : 13723458987 }

    # 对字段的模糊匹配，在sql中使用like关键字，在这里是使用正则匹配语法来操作的。例如查询姓王的用户有哪些？
    > db.user.find({nickname: /^王/})
    { "_id" : ObjectId("5a23e07b9369d959a2ec17a2"), "name" : "WangWu", "nickname" : "王五", "age" : 17, "label" : [ "锻炼", "泡吧", "直播" ] }

    # 如果字段值是数组形式，该怎样查询呢？
    # 单个元素匹配，直接指定值就可以了，例如看看爱好喜欢直播的用户有哪些？
    > db.user.find({label: '直播'})
    { "_id" : ObjectId("5a23e07b9369d959a2ec17a2"), "name" : "WangWu", "nickname" : "王五", "age" : 17, "label" : [ "锻炼", "泡吧", "直播" ] }

    # 多个元素匹配，同时都匹配。
    > db.user.find({label: {$all: ['直播', '喝茶']}})
    >

    # 多个元素匹配，只满足一个就可以了
    > db.user.find({label: {$in: ['直播', '喝茶']}})
    { "_id" : ObjectId("5a23e0239369d959a2ec17a1"), "name" : "LiShi", "nickname" : "李四", "age" : 28, "label" : [ "喝茶" ], "phone" : 13723458987 }
    { "_id" : ObjectId("5a23e07b9369d959a2ec17a2"), "name" : "WangWu", "nickname" : "王五", "age" : 17, "label" : [ "锻炼", "泡吧", "直播" ] }

    # 匹配数组中某个索引的值，记住此时字段名称+索引需要用引号引起来。
    > db.user.find({'label.1': '泡吧'})
    { "_id" : ObjectId("5a23e07b9369d959a2ec17a2"), "name" : "WangWu", "nickname" : "王五", "age" : 17, "label" : [ "锻炼", "泡吧", "直播" ] }

    # 按数组长度匹配，例如查询只有一个爱好的用户
    > db.user.find({label: {$size: 1}})
    { "_id" : ObjectId("5a23e0239369d959a2ec17a1"), "name" : "LiShi", "nickname" : "李四", "age" : 28, "label" : [ "喝茶" ], "phone" : 13723458987 }

    # 使用$slice对数组切片，正数是前面多少条，负数是尾部多少条。这非常类似于Python中的列表切片操作，从数组中返回你指定的元素，这些你要的元素通过索引标识。仔细看下面的示例就比较容易理解了。
    > db.user.find({_id: ObjectId("5a23e07b9369d959a2ec17a2")}, {label: {$slice: 2}})
    { "_id" : ObjectId("5a23e07b9369d959a2ec17a2"), "name" : "WangWu", "nickname" : "王五", "age" : 17, "label" : [ "锻炼", "泡吧" ] }
    > db.user.find({_id: ObjectId("5a23e07b9369d959a2ec17a2")}, {label: {$slice: 1}})
    { "_id" : ObjectId("5a23e07b9369d959a2ec17a2"), "name" : "WangWu", "nickname" : "王五", "age" : 17, "label" : [ "锻炼" ] }
    > db.user.find({_id: ObjectId("5a23e07b9369d959a2ec17a2")}, {label: {$slice: 1}})

    # 使用$来指定符合条件的任意一个数组元素，即匹配的数组中仅返回查询到的值
    > db.user.find({label: '锻炼'}, {'label.$': 1})
    { "_id" : ObjectId("5a23e07b9369d959a2ec17a2"), "label" : [ "锻炼" ] }

    # $elemMatch 要求同时使用多个条件语句来对一个数组元素进行判断，我们添加一些测试数据以演示这个修饰符的使用方法。
    > db.user.update({name:'WangWu'}, {$set: {heartbeat: [60,67,59,69,71,73,80,90,91,55,101]}})
    WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
    # 找出心跳值在80-90之间的用户
    > db.user.find({heartbeat:{$elemMatch:{$gte:80, $lte:90}}})
    { "_id" : ObjectId("5a23e07b9369d959a2ec17a2"), "name" : "WangWu", "nickname" : "王五", "age" : 17, "label" : [ "锻炼", "泡吧", "直播" ], "heartbeat" : [ 60, 67, 59, 69, 71, 73, 80, 90, 91, 55, 101 ] }
    > db.user.find({heartbeat:{$elemMatch:{$gte:180, $lte:190}}})
    >

    # 查询内嵌文档，只需要在指定查询字段时用分隔符.进行区分即可，我们添加一些测试数据以方便。
    > db.user.update({name:'ZhanShan'}, {$set:{score:{'java':78.5, 'c++': 90}}})
    WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
    # 注意像这种多字段需要用引号引起来，下面查找用户c++课程考试分数在90分以上的用户有哪些？
    > db.user.find({'score.c++': {$gte: 90}})
    { "_id" : ObjectId("5a23dff29369d959a2ec17a0"), "name" : "ZhanShan", "nickname" : "张三", "age" : 30, "label" : [ "钓鱼", "旅行" ], "score" : { "java" : 78.5, "c++" : 90 } }
    ```
* `count()` 统计数据行
    ```json
    > db.user.find().count()
    3
    ```
* `skip limit` 分页返回数据，在项目中做查询的时候势必要进行分页的操作，这是个好习惯，不管程序员有没有指定分页参数，在DAO层中都必须有默认的分页值。
    ```json
    # 例如每页只有2条数据，下面的参数表示第一页
    > db.user.find().skip(0).limit(2)
    { "_id" : ObjectId("5a23dff29369d959a2ec17a0"), "name" : "ZhanShan", "nickname" : "张三", "age" : 30, "label" : [ "钓鱼", "旅行" ], "score" : { "java" : 78.5, "c++" : 90 } }
    { "_id" : ObjectId("5a23e0239369d959a2ec17a1"), "name" : "LiShi", "nickname" : "李四", "age" : 28, "label" : [ "喝茶" ], "phone" : 13723458987 }
    # 当查询第2页时，应该使用下面的参数
    > db.user.find().skip(2).limit(2)
    { "_id" : ObjectId("5a23e07b9369d959a2ec17a2"), "name" : "WangWu", "nickname" : "王五", "age" : 17, "label" : [ "锻炼", "泡吧", "直播" ], "heartbeat" : [ 60, 67, 59, 69, 71, 73, 80, 90, 91, 55, 101 ] }
    ```
* `sort()` 排序，-1是降序，1是升序
    ```json
    # 依据年龄字段升序
    > db.user.find().skip(0).limit(2).sort({age: 1})
    { "_id" : ObjectId("5a23e07b9369d959a2ec17a2"), "name" : "WangWu", "nickname" : "王五", "age" : 17, "label" : [ "锻炼", "泡吧", "直播" ], "heartbeat" : [ 60, 67, 59, 69, 71, 73, 80, 90, 91, 55, 101 ] }
    { "_id" : ObjectId("5a23e0239369d959a2ec17a1"), "name" : "LiShi", "nickname" : "李四", "age" : 28, "label" : [ "喝茶" ], "phone" : 13723458987 }
    > db.user.find().skip(0).limit(2).sort({age: -1})
    { "_id" : ObjectId("5a23dff29369d959a2ec17a0"), "name" : "ZhanShan", "nickname" : "张三", "age" : 30, "label" : [ "钓鱼", "旅行" ], "score" : { "java" : 78.5, "c++" : 90 } }
    { "_id" : ObjectId("5a23e0239369d959a2ec17a1"), "name" : "LiShi", "nickname" : "李四", "age" : 28, "label" : [ "喝茶" ], "phone" : 13723458987 }
    ```
* `distinct` 取出不重复的数据 `db.runCommand({distinct:'user', key: 'age'})`

# 聚合查询

主要用来对集合中的文档进行变换和组合，从而对数据进行分析加以利用。

## 语法结构

`db.集合名称.aggregate(构件1, 构件...)` 由于聚合的结果要返回客户端，因此聚合的结果必须限制在**16M**以内。

## 准备测试数据
    ```json
    > for (var i = 0; i &lt; 100; i++) {
    ...     for (var j = 0; j &lt; 4; j++) {
    ...         db.scores.insert({userId:"s" + i, course:"课程" + j, score:Math.random()*100})
    ...     }
    ... }
    WriteResult({ "nInserted" : 1 })
    > db.scores.find().count()
    400
    ```

## 查询方法示例

由于本次准备的测试数据过多，在返回数据较多行时会省略显示，并附加 Type it for more 表示数据有多行。
    ```json
    # 查找所有80分以上的学生，仅使用一个构件match
    > db.scores.aggregate( {$match: {score:{$gte:80}}} )
    Type "it" for more

    # 将每个学生的名字投影出来，使用两个构件match和project
    > db.scores.aggregate({$match:{score:{$gte:80}}}, {$project:{userId:1}})
    Type "it" for more
    
    # 对学生的名字分组，某个学生的名字出现一次就加1
    > db.scores.aggregate({$match:{score:{$gte:80}}}, {$project:{userId:1}}, {$group:{_id:"$userId", count:{$sum:1}}})
    { "_id" : "s99", "count" : 1 }
    { "_id" : "s96", "count" : 1 }
    { "_id" : "s94", "count" : 1 }
    Type "it" for more
    # 对结果按照count进行降序
    > db.scores.aggregate({$match:{score:{$gte:80}}}, {$project:{userId:1}}, {$group:{_id:"$userId", count:{$sum:1}}}, {$sort:{count:-1}})
    Type "it" for more
    # 只取前面三个
    db.scores.aggregate({$match:{score:{$gte:80}}}, {$project:{userId:1}}, {$group:{_id:"$userId", count:{$sum:1}}}, {$sort:{count:-1}}, {$limit:3})
    ```

## 管道操作符

构件也即管理操作符，下面对其进行一个说明。
    1. `$match` 对文档进行筛选，通常放到管道最前面的位置，在实际应用中要特别注意这一点，它可以显著优化查询性能。
    1. `$project` 字段投影
    1. 指定包含和排除字段 `db.scores.aggregate({$project:{userId:1}})`
    1. 也可以重命名字段 `db.scores.aggregate({$project:{userId2:"$userId"}})`
    1. `$add` `$subtract` `$multiply` `$divice` `$mod` 数学表达式

* 例如给所有人加20分 `db.scores.aggregate({$project:{userId2:"$userId", score:1,newScore:{$add:["$score", 10]}}})` 首先按条件匹配出集合，通过对集合的运算产生新字段并返回。
* `$year` `$month` `$week` `$dayOfMonth` `$dayOfWeek` `$dayOfYear` `$hour` `$minute` `$second` 日期表达式，用于从指定字段中提取日期信息的表达式。使用方式不再赘述，可参考上面的示例。
* `$substr` `$concat` `$toLower` `$toUpper` 字符串表达式，使用方式不再赘述，可参考上面的示例。
* 逻辑表达式
  1. `$cmp` 比较两个表达式
  1. `$strcasecmp` 比较两个字符串
  1. `$eq` `$ne` `$gt` `$gte` `$lt` `$lte`
  1. `$and` `$or` `$not`
  1. `$cond`
  1. `$ifNull`
  1. `$group` 将文档依据特定的不同值进行分组。选定了分组字段过后，就可以把这些字段传递给 `$group` 函数的 `_id` 字段。
    ```json
    db.scores.aggregate({$group:{_id:"$userId"}} )
    db.scores.aggregate({$group:{_id:{"myUserId":"$userId"}}} );
    db.scores.aggregate({$group:{_id:{"myUserId":"$userId"}, count:{$sum:1}}} ,{$sort:{count:-1}})
    ```
* group支持的操作符 `$sum` `$avg` `$max` `$min` `$first` `$last` `$addToSet` `$push`
    1. `$unwind` 把数组中的每个值拆分成单独的文档  `db.user.aggregate({$unwind:"$score"})`
    1. `$sort` 如果要对大量文档进行排序，建议放到第一阶段以利用索引
    1. `$count` 用一返回集合中文档的数量
    1. `$distinct` 找出给定键的所有不同值，使用时必须指定集合和键


# MapReduce

MongoDB也支持数据的并行计算，不过它并没有完整实现MapReduce的所有功能，但基本的功能可以满足一定的需求，可以简单了一下，虽然在实际项目中可能不会用到。

* 找出集合中所有的键并统计每个键出现的次数
    ```json
    > var map = function() {
    ...         for (var key in this) {
    ...                 emit(key, {"count": 1});
    ...         }
    ... };
    > var reduce = function(key, emits) {
    ...         var total = 0;
    ...         for (var i in emits) {
    ...                 total += emits[i].count;
    ...         }
    ...         return {"count": total};
    ... };
    > db.runCommand({mapreduce:'scores', map:map, reduce:reduce, out:'myout2'});
    {
        "result" : "myout2",
        "timeMillis" : 464,
        "counts" : {
            "input" : 400,
            "emit" : 1600,
            "reduce" : 16,
            "output" : 4
        },
        "ok" : 1
    }
    > db.myout2.find()
    { "_id" : "_id", "value" : { "count" : 400 } }
    { "_id" : "course", "value" : { "count" : 400 } }
    { "_id" : "score", "value" : { "count" : 400 } }
    { "_id" : "userId", "value" : { "count" : 400 } }
    >
    ```
