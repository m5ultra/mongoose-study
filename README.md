# mongoose-study
mongoose学习
### 安装 mongoose
`yarn add mongoose`
### 链接数据库
```
  const mongoose = require("mongoose")
  mongoose.connect("mongodb://127.0.0.1:27017/users", {useNewUrlParser: true}, function(err) {
    if(err) {
      console.log(err)
    }
    console.log('数据库连接成功!')
  })
  // 如果数据库设置了密码
  mongoose.connect("mongodb://account:password@127.0.0.1:27017/users")
```
### 定义Schema Schema里面的对象需要和数据库中的字段(fields)一一对应
```
  const UserSchema = mongoose.Schema({
    name: String,
    age: Number,
    status: {type: Number, default: 0}
  })
```

### 通过Schema定义model
> 1.首字母大写 
> 2.要和数据库表(集合) 名称对应 
> 3. 第三个参数可以指定数据库中表的名称, 不指定为 第一个参数 小写 的复数 还有别的规则 后续补充
```
  var User = mongoose.model('User', UserSchema, 'users')
```

### 查找数据 Retrieve
```
  User.find({}, function(err, doc) {
    if(err) { 
      console.log(err)
      return false;
    }
    
   console.log(doc)
  })
```


### 增加数据 Create
> 通过实例化Model 调用Model的实例的save方法
```
  var u = new User({ name : '李四', age: 20, status: 1})
  u.save(function(err) {
    if(err) {
      console.log(err)
      return false;
    }
    cosnole.log('成功')
  })
```

### 更新数据 Update
```
  User.updateOne({_id: ''}, 更新对象内容, function(err, doc) {
     if(err) {
      console.log(err)
       return
     }
     console.log(doc)
  })
```

### 删除数据 Delete
```
 User.deleteOne({}, function(err, doc) {
    if(err) return;
    console.log(doc)
 })
```
### mongoose 预定义模式修饰符
lowercase uppercase trim
> trim 设置为true 对数据name 做去空格操作
```
  const UserSchema = mongoose.Schema({
    name: {type: String, trim: true},
    age: Number,
    status: {type: Number, default: 0},
    // 自定义事件修饰符, 出来用户redirect字段
    redirect: {
      type: 'string',
      set(params) {
        if(!params)  return ''
        if(params.indexOf('http://' !== 0 && 'https://' !== 0) {
          return 'http://' + params
        }
        ruturn params
      }
    }
  })
```

### mongoose 索引 mongoose 内置CURD方法 扩展Mongoose Model 的静态方法和实力方法
 new mongoose.Schema({
##### 设置索引
```
    sn: {
      type: Number,
      //  唯一索引
      unique: true
    },
    name: {
      type: String,
      // 普通索引
      index: true
    }
 })
```
#### 扩展静态方法

```
const mongoose = require('mongoose');
const { Schema } = mongoose;
const UserSchema = new Schema({
  name: {type: String, require: true},
  sn: {
    type: String,
    index: true
  },
  age: Number,
  status: {
    type: Number,
    default: 1,
    enum: [1, 2, 3]
  }
})
// 增加新的静态方法
UserSchema.static.findBySn = function(sn, cb) {
  this.find({sn}, function(err, docs) {
    cb(err, docs)
  })
}

// 使用静态方法 ???封装为Promise不香吗???
UserModel = mongoose.model('User', UserSchema, 'user')
UserModel.findBySn('sncode', function(err, docs) {
     if (err) return false;
     console.log(docs);
})
```

#### 扩展实例方法
```
const mongoose = require('mongoose');
const { Schema } = mongoose;
const UserSchema = new Schema({
  name: {type: String, require: true},
  sn: {
    type: String,
    index: true
  },
  age: Number,
  status: {
    type: Number,
    default: 1,
    enum: [1, 2, 3]
  }
})
// 增加新的静态方法
UserSchema.methods.findBySn = function(sn, cb) {
  this.find({sn}, function(err, docs) {
    cb(err, docs)
  })
}

// 使用实例方法 ???封装为Promise不香吗???
UserModel = mongoose.model('User', UserSchema, 'user')
let u = new UserModel({
  name: '...',
  ...
})
u.findBySn() // 
```

### 数据校验
```
  required: 字段比传入
  max: 数据最大值
  min: 最小值
  enum: [1, 2, 3] // 传入值必须是数组中的值,
  maxlength: 字符串最大长度
  minlength: 字符串最小长度
  match: /^sn(.*)/ // 符合正则规则
  validite: function(params) {
    return true/ false // true代表验证通过 false代表验证失败
  }
```

### mongoose中使用aggregate 聚合管道
mongodb 原始驱动聚合实现
```
 db.order.aggregate([
    {
      $lookup: {
        from: 'order_item', // 关联的表
        localField: "order_id", // order中关联order_item
        foreignField: "order_id", // order_item中的id
        as: "items"
      }
    },
    {
      $match: {"all_price": {$gte: 90}}
    }
 ])
```

```
const OrderModel =  require('./model/order.js')
 OrderModel.aggregate([
    {
      $lookup: {
        from: 'order_item', // 关联的表
        localField: "order_id", // order中关联order_item
        foreignField: "order_id", // order_item中的id
        as: "items"
      }
    },
    {
      $match: {"all_price": {$gte: 90}}
    }
 ], function(err, docs) {
    console.log(docs)
 })
```
### Mongoose: aggregate聚合 $group使用说明
> aggregate聚合是通过管道操作实现的。聚合管道里的每一步输出，都会作为下一步的输入，每一步在输入文档执行完操作后生成输出文档。
#### 聚合管道：
```
 $project 修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。对应project()方法 

 $match 用于过滤数据，只输出符合条件的文档。$match使用MongoDB的标准查询操作。对应match()。 

 $limit 用来限制MongoDB聚合管道返回的文档数。对应limit()方法 

 $skip 在聚合管道中跳过指定数量的文档，并返回余下的文档。对应skip()。 

 $unwind 将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。对应unwind()方法 

 $group 将集合中的文档分组，可用于统计结果。对应group()方法 

 $sort 将输入文档排序后输出。对应sort()方法 

 $geoNear 输出接近某一地理位置的有序文档。对应near()。 
```
#### $group表达式说明： 
```
$sum  计算总和 

 $avg  计算平均值  

 $min  获取每一组集合中所有文档对应值得最小值 

 $max  获取每一组集合中所有文档对应值得最大值 

 $push  在结果文档中插入值到一个数组中 

 $addToSet  在结果文档中插入值到一个数组中，但不创建副本 

 $first  根据资源文档的排序获取第一个文档数据

 $last  根据资源文档的排序获取最后一个文档数据 
```

#### 举例说明aggregate聚合 $group使用
> 商品属性：_id,  createTime,  nowPriceL,  nowPriceH,  number 统计每一天内店铺商品的最低价和最高价，平均最低价
> 执行完$match管道后，得到的查询结果会输入到$project管道，执行完$project管道，得到的结果格式为{day,nowPriceL,nowPriceH},把这个结果输入$group管道，$group管道执行完毕，输出的结果输入到$sort管道，$sort执行完毕，输出最终结果集

```
Goods.aggregate([
      {
        $match: {
          number: {$gte:100} //匹配number>=100的记录
       }
      },
      {
         $project : {
             day : {$substr: [{"$add":["$createTime", 28800000]}, 0, 10] },//时区数据校正，8小时换算成毫秒数为8*60*60*1000=288000后分割成YYYY-MM-DD日期格式便于分组
             "nowPriceL": 1, //设置原有nowPriceL为1，表示结果显示原有字段nowPriceL
             "nowPriceH":1, //设置原有nowPriceH为1，表示结果显示原有字段nowPriceH
             avgNowPriceL:{$toDouble:"$nowPriceL"},//把最低价转换为小数
             avgNowPriceH:{$toDouble:"$nowPriceH"},//把最高价转换为小数
             "dayNumber":1 //每组内有多少个成员
         },
      },
    
    { 
      $group: { 
        _id:"$day", //按照$day进行分组（一组为1天） 
        nowPriceL:{$min: "$nowPriceL"}, //查找组内最小的nowPriceL 
        nowPriceH:{$max: "$nowPriceH"}, //查找组内最大的nowPriceH  
        avgNowPriceL:{$avg:"$avgNowPriceL"},//统计每一天内店铺商品的平均最低价
        avgNowPriceH:{$avg:"$avgNowPriceH"},//统计每一天内店铺商品的平均最高价
        dayNumber:{$sum:1}   
      } 
    }, 
    { 
      $sort: {
        nowPriceL: 1//执行完 $group，得到的结果集按照nowPriceL升序排列
      }
    }]).exec(function (err, goods){
    //返回结果  
    console.log(goods);
    });
```
##### 注意：
> mongoodb存储的数据是按照世界时间存储的，因此进行分割操作时需要对时间进行时区校正，使用$add加上时区差8小时(毫秒数)才能得到正确的数据
`$substr: [ <string>, <start>, <length> ] }`
https://docs.mongodb.com/manual/reference/operator/aggregation/toDouble/#exp._S_toDouble

### mongodbd导入导出数据
```
mongodump -h dbhost -d dbname -o dbdirectory
mongorestore -h dbhost -d dbanme <path>
```
