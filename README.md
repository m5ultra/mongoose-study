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
