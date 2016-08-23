# mongodb安装以及配置
1. 安装mongodb 

2. 配置mongodb数据库存放路径以及日志文件路径（新建db目录以及mongodb.log日志文件）

3. 进入mongodb安装目录下的bin文件夹，启动命令行，创建mongodb服务

    ```
    mongod --storageEngine mmapv1 --dbpath G:\mongodb\db --logpath G:\mongodb\log\mongo.log --install --serviceName mongodb
    ```

4. 在命令行输入mongo。进入mongodb的客户端

5. windows下删除服务

    ```
    sc delete serviceName
    ```

# mongodb 数据库级命令
1. db 查看当前数据库 

    ```
    db
    ```

2. show dbs 查看所有的数据库

    ```
    show dbs
    ```

3. use dbName 切换数据库（没有改数据库也不会出错）

    ```
    use blog 切换到blog这个数据库下，以后的操作都是针对这个数据库的
    ```

# mongodb 集合（表）级命令

1. show collections 显示当前数据库下的所有集合

    ```
    show collections
    ```

2. db.createCollection('collectionName') 创建一个集合（表）

    ```
    db.createCollection('users')
    ```

# mongodb 增加数据

1. insert  添加一条文档（记录）  默认会自动添加_id字段并且作为主键，如果数据库中含有该_id，那么会报错

    ```
    db.users.insert({name:'zhangsan',pwd:"123"})
    ```

2. save 插入或者更新数据。如果参数中没有加主键_id，那么相当于insert操作，如果有_id,那么相当于update操作

    ```
    db.users.save({name:"lisi",pwd:"123"}) --相当于insert
    db.users.save({_id:1,name:"wangwu",pwd:"123"})  ---相当于update
    ```

# mongo 更新操作
1. update 文档替换（更新一条记录）。update后的值会和update前的值的主键_id做比较，如果update后没有设置_id，那么会用update前的_id作为主键，如果update后的值有_id,会比较2次的_id是否一致，如果不一致则会报错

    ```
    db.users.update({name:"lisi"},{name:"lisi",age:10,gender:"man"})  ---如果字段比较多，那么会很麻烦
    var user=db.users.findOne({name:"lisi"})
    user.age=10;
    user.gender="man";
    //db.users.update({name:"lisi"},user)   --如果记录中不止一条含有{那么：“lisi”}的记录可能会报错
    db.users.update({_id:user._id},user)   --这样就不会报错，在关系型数据中也是这么更新的
    ```

2. $inc 修改器,修改器只更新更新修改器设置的字段，所以是局部更新。给某个字段进行自增（没有该字段，则新增这个字段，并把值设置为这个自增的值）

    ```
    db.users.update({name:"lisi"},{$inc:{age:4}})   ---age字段只能是数值类型
    ```

3. $set 修改器  给文档/更新增加若干个字段，如果存在该字段，则会更新。更新内嵌文档，通过"filed.key"来更新

    ```
    db.users.update({name:"lisi"},{$set:{age:20,hobby:{sign:"qq爱",word:"hello"}}}})
    db.users.update({name:"lisi"},{$set:{"hobby.0":"dance"}})  ---更新内嵌文档。Object对象也是这样更新
    ```

4. $unset 修改器  删除文档的若干个字段

    ```
    db.users.update({name:"lisi"},{$unset:{age:20}})
    ```

5. $push 数组修改器  向数组中追加一个元素，如果数组不存在，则自动添加数组

    ```
    db.users.update({name:"lisi"},{$push:{hobby:1}})  ---向文档中的hobby字段添加一个元素1
    ```

6. $push+$each 向数组中添加多个元素

    ```
    db.users.update({name:"abc"},{$push:{hobby:{$each:[2,3,4,5,6]}}})  ---向hobby数组中追加2，3，4，5，6
    ```

7. $addToSet 相当于$push+$ne  首先会判断数组中有木有这个元素，没有就添加，有则忽略

    ```
    db.users.update({name:"abc"},{$addToSet:{hobby:5}})  ---如果hobby中有5则直接忽略
    db.users.update({name:"abc"},{$addToSet:{hobby:{$each:[3,4,5,6,7,8,9]}}}})  ---向hobby中添加多个元素，存在的则忽略，否则追加
    ```

8. $pop 数组删除修改器

    ```
    db.users.update({name:"abc"},{$pop:{hobby:1}})  --- 1:从末尾删除  -1:从头部删除
    db.users.update({name:"abc"},{$pop:{hobby:-1}})  --- 1:从末尾删除  -1:从头部删除
    ```

9. $pull 数组删除修改器，删除所有匹配到的元素

    ```
    db.users.update({name:"abc",{$pull:{hobby:2}})  ---不能喝$each组合批量删除
    ```

10. $ 定位修改器

    ````
    若数据如下：{name:"abc",firends:[{name:"zhangsan",age:11},{name:"lisi",age:12}]}  ---数组内嵌文档
    db.users.find({"firends.name":"lisi"})  ---可以直接使用内嵌文档的属性来定位
    db.users.update({"firends.name":"lisi"},{$set:{"friends.$.age":22}})  ---$定位到数组的第二个元素
    ```

11. upsert  update的第三个参数  如果没有匹配到文档，如果设置upsert=true，则会添加这个文档，相当于insert

    ```
    db.users.update({name:"who"},{$set:{sex:'man'},true})  ---没匹配到name:who的文档，所以直接增加，并添加了字段sex
    ```

12. multi  update的第四个参数  是否更新所有匹配到的文档

    ```
    db.users.update({name:"lisi"},{$inc:{age:2}},false,true)  ---所有name="lisi"的文档的age=age+2；
    ```

13. findAndModify:修改(更新/删除)文档并返回修改前（后）的文档。有以下几个参数。update不能知道结果，而findAndModify会返回结果，所以速度也会比较慢

    >query:查询条件

    >update:更新的值

    >remove:是否删除

    >sort:排序方式 1：倒叙 -1：顺序  更新或者删除是根据排序后的文档来进行操作的

    >new:是否返回更新后的值  true是 false返回更新前的文档

    >注意点：1.update和remove只能同时存在一个 2.一次只能操作一个文档   3.不能执行upsert操作，只能更新已存在的文档

    ````
    db.users.findAndModify({query:{name:"abc"},update:{$set:{age:18}},new:true,sort:1})

    db.users.findAndModify({query:{name:"abc"},remove:true,new:true,sort:1})
    ```
   
# mongodb 查询

1. find(condition={},showFiled) 获取所有匹配的文档,第一个参数默认是{}，即获取所有，第二个参数设置需要显示的字段 1：显示 0：不显示。默认都会显示_id字段。设置_id:0就不会显示

    ```
    db.users.find({},{_id:0,name:11,age:1})
    ```
2. 条件查询

    > $gt   大于

    > $gte  大于等于

    > $lt   小于

    > $lte  小于等于

    > $ne   不等于

    > //   模糊查询

    ```
    db.users.find({age:{$lte:35}})  ---查询年龄<=35的所有文档
    db.users.find({age:{$gte:35}})  ---查询年龄>=35的所有文档
    db.users.find({name:/吴/})      ---查询所有名字中有"吴"的所有文档
    ```

3. 多条件查询

    > $or 满足其中一个条件即可

    > $in、$nin 在某个(不在)集合中即可

    > $not 不满足即可  一般用在别的条件前面

    > {age:18,gender:'man'}  相当于and

    ```
    db.users.find({$or:[{age:{$gt:60}},{age:{$lt:18}}]})  ---匹配age>60 or age<18
    db.users.find({age:{$in:[15,16,17,18]}})  ---匹配年龄为15，16，17，18的文档
    db.users.find({age:{$gte:18},gender:'man'})  ---匹配age>=18 and gender='man'的文档
    db.users.find({age:{$not:{$in:[11,12]}}})  ---匹配年龄不在11，12这个范围的文档，类似$nin   
    ```

4. 数组查询
    >1.根据数组中元素查询
    
    ```
    db.users.find({fruits:"apple"})  ---查询fruits字段的数组中是否有"apple"
    ```

    > 2.$all 查询数组中元素满足多个条件

    ```
    db.users.find({fruits:{$all:["apple","banana"]}})  ---查询fruit是数组中既有"apple"，又有"banana"的文档，$all的条件没有先后顺序
    ```
    > 3.完全匹配数组中的多个条件

     ```
    db.users.find({fruits:["apple","banana"]})  ---查询fruit是["apple","banana"]的文档，个数和先后书序要完全一致才匹配
    ```

    > 4.根据下标匹配

    ```
    db.users.find({"fruit.0":"apple"})  ---匹配fruit[0]="apple"的文档集合
    ```
    > 5.查询数组中元素为某个长度的文档几个,不能和$gt这些条件组合使用

    ```
    db.users.find({fruit:{$size:3}})
    ```

    > 6.$slice 查询数组的子集，只能作为find的第二个参数，表示返回结果中过滤几个元素

    ```
    db.users.find({shop:"A"},{fruit:{$slice:[1,2]}})  ---满足shop:"A"的集合，从中把fruit字段中index为[1,2)的结果返回
    ```

5. 内嵌文档匹配

 假设数据如下：

 {shop:"A",fruits:{name:"apple",count:10}}

 {shop:"B",fruits:{name:"peach",count:20}}

 1.精确匹配  精确匹配会按照键的数量和顺序完全一致才匹配

    ```
    db.users.find({fruits:{name:"apple",count:10}})
    db.users.find({fruits:{count:10,name:"apple"}})   ---这样是查不出结果的，
    ```
2.键值匹配
    ```
    db.users.find({"fruits.name":"apple","fruits.count":10})
    db.users.find({"fruits.count":10,"fruits.name":"apple"})  ---2者写法都是正确的
    ```
3. 数组中嵌套文档

    假设数据如下：

    {shop:"A",fruits:[{name:"apple",count:10},{name:"peach",count:20}]}

    ```
    db.users.find({fruits:{$elemMatch:{name:"apple",count:10}}})
    db.users.find({fruits:{$elemMatch:{count:10,name:"apple"}}})   ---2者都是正确写法
    ```

4. $where查询

    假设数据如下：  匹配apple=peach的文档

   {apple:5,peach:5})

   {apple:5,peach:4})

   {apple:4,peach:5})

   {apple:4,peach:4})

   ```
   db.users.find({$where:function(){return this.apple===this.peach}})
   db.users.find({$where:"this.apple===this.peach"})
   ```

5. limit  限制返回多少结果集

    ```
    db.users.find({name:"abc"}).limit(5)  ---取出查询结果的前5条
    ```

6. skip   略过匹配到的前多少条数据

    ```
    db.users.find({name:"abc"}).skip(5)   ---略过前5条数据，从第6条开始取
    db.users.find({name:"abc"}).skip(5).limit(5)  ---去第6-10之间的5条数据
    ```

7. sort  根据文档的键排序   1：升序   -1：降序

    ```
    db.users.find({name:"abc"}).sort({age:1,score:-1})
    ```

8. null匹配查询

    ```
    db.users.find({name:null})  ---匹配name=null 或者没有name字段的文档
    db.users.find({name:{$in:[null],$exists:true}})  ---匹配name字段存在并且name=null的文档
    ```

9. distinct  按键去重， 返回键对应值的一个数组

    ```
    db.users.distinct("name",{age:{$gt:30}})  ---返回年龄大于30的所有文档的不同name集合
    ```

10. group 分组

    ```
    db.users.group({key:{score:1},$reduce:function(doc,pre){
    pre.arr.push(doc.name)
    },initial:{arr:[]}})   

    group理解：先按照key来分组，然后根据initial来为每个分组初始化一个累加器，然后在分组后的每个文档执行$reduce函数，把需要的字段插入到累加器中

    ```