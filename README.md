# mongoose-study
mongoose学习
### 安装 mongoose
`yarn add mongoose`
### 链接数据库
```
  const mongoose = require("mongoose")
  mongoose.connect("mongodb://127.0.0.1:27017/users")
  // 如果数据库设置了密码
  mongoose.connect("mongodb://account:password@127.0.0.1:27017/users")
```
### 定义Schema Schema里面的对象需要和数据库中的字段(fields)一一对应
```
  const UserSchema = mongoose.Schema({
    name: String,
    age: Number,
    status: Number
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


### 
