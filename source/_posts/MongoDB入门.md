---
title: "MongoDB入门"
tag: ["Database", "中文","入门","未完结","MongoDB"]
date: 2020-04-05T15:38:22+08:00
draft: false
---

## 概念  
* [MongoDB官网](https://www.mongodb.com/ "MongDB")

* 属于nosql

* 上手快

* 相对与RDBMS的table，MongoDB采用了collection

* document相对于RDBMS的row，与json比较相似,比如  

  `{name:"aaa",age:14}`  
  
* field相对于RDBMS的column  

------

## 基本命令介绍  

* 启动命令:`mongo`默认为`mongo 127.0.0.1:27017`  

* 查看有哪些数据库  

  ```sq
  > show dbs
  admin   0.000GB
  config  0.000GB
  local   0.000GB
  ```

  默认会使用test数据库,但一开始没数据所以看不到

* 在test中的user集合(自动会创建)中插入一条数据,结果返回WriteResult对象,"`nInserted`"表示影响的记录数  

  ```sql
  > use test
  switched to db test
  > db
  test
  > db.user.insertOne({name:"aaa",age:14})
  WriteResult({ "nInserted" : 1 })
  > db.user.insertOne({name:"bbb",age:18,"address":"北京"})
  WriteResult({ "nInserted" : 1 })
  > doc=({name:"dd",age:11})
  { "name" : "dd", "age" : 11 }
  > db.user.insertOne(doc)
  WriteResult({ "nInserted" : 1 })
  ```

  新的字段会自动添加，方便扩展，变量也可以插入数据  

* 集合的基本操作  

  ```sql
  > db.createCollection("runoob")
  { "ok" : 1 }
  > show collections
  student
  user
  > db.student.drop()
  > show tables
  user
  ```

  查看集合可以用tables，但不够严谨

* 查询集合中数据  

  ```sql
  > db.user.find()
  { "_id" : ObjectId("5e89884d983ca629b5e9b574"), "name" : "aaa", "age" : 20 }
  { "_id" : ObjectId("5e898875983ca629b5e9b575"), "name" : "bbb", "age" : 18, "address" : "北京" }
  > db.user.find({name:"aaa"})
  { "_id" : ObjectId("5e89884d983ca629b5e9b574"), "name" : "aaa", "age" : 20 }
  > db.user.find({age:{$gt:18}})
  { "_id" : ObjectId("5e89884d983ca629b5e9b574"), "name" : "aaa", "age" : 21 }
  > db.user.find({$or:[{age:{$gt:18,$lt:22}},{"name":"bbb"}]})
  { "_id" : ObjectId("5e89884d983ca629b5e9b574"), "name" : "aaa", "age" : 21 }
  { "_id" : ObjectId("5e898875983ca629b5e9b575"), "name" : "bbb", "age" : 18, "address" : "北京" }
  ```

  可以看到主键"`_id`"是自动生成的，find方法可以加入条件，支持$gt等比较运算，以及$or条件,$regex正则匹配等  

* update方法  

  ```sql
  > db.user.update({name:"aaa"},{$set:{age:20}})
  WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
  ```

  还有三个可选参数**upsert**， **multi**，  **writeConcern**

* 删除数据

  ```sql
  > db.user.deleteMany({age:30})
  WriteResult({ "nRemoved" : 2 })
  > db.user.deleOne({_id:ObjectId("1231e241")})
  ```

  满足条件的document都会删掉，还有deleteOne方法只会删除一个document

* 删除数据库  

  ```sql
  > use test
  switched to db test
  > db.dropDatabase()
  { "dropped" : "test", "ok" : 1 }
  ```

------

## 使用python操作MongoDB    

* 连接  

  ```python
  import pymongo
  
  client = pymongo.MongoClient(host="localhost", port=27017)
  db = client['test'] # 指定数据库为test
  collection = db['user'] # 指定collection为user
  ```

* 插入操作  

  ```python
  user = {
      'name': 'python',
      'age': 50
  }
  users = [user.copy() for i in range(4)]
  
  collection.insert_one(user)  # 插入单条document
  collection.insert_many(users)  # 利用可迭代对象插入多条document
  ```

* 查找与更新  

  ```python
  result = collection.find_one({'name': 'python'})  # 查找单条
  print(result)
  # {'_id': ObjectId('5e89be93e3a85de07d04cb32'), 'name': 'python', 'age': 50}
  
  collection.find_one_and_update({'name': 'python'}, {'$set': {'age': 30}})  # 查找一条并修改
  
  result = collection.find({'name': 'python'})  # 查找多条,返回一个生成器
  
  print(collection.count_documents({'name': 'python'}))  # 查找结果总数
  # 1
  
  # 根据主键查找
  from bson.objectid import ObjectId
  print(collection.find_one({'_id': ObjectId('5e89b91f4a7a179ff823b9a3')}))
  # {'_id': ObjectId('5e89be93e3a85de07d04cb32'), 'name': 'python', 'age': 50}
  ```

* 删除与更新  

  ```python
  collection.find_one_and_delete({'name': 'python'})  # 根据条件找到一条并删除
  collection.delete_many({'name': 'python'})  # 删除所有满足条件的结果
  ```

* 对结果排序  

  ```python
  result = collection.find().sort('age',pymongo.DESCENDING)
  for r in result:
      print(r)
  '''
  {'_id': ObjectId('5e89c067a49025c2a690cb5f'), 'name': 'python', 'age': 50}
  {'_id': ObjectId('5e89884d983ca629b5e9b574'), 'name': 'aaa', 'age': 21.0}
  {'_id': ObjectId('5e898875983ca629b5e9b575'), 'name': 'bbb', 'age': 18, 'address': '北京'}
  {'_id': ObjectId('5e89a787ba556f887d48c61d'), 'name': 'dd', 'age': 11.0}
  '''
  ```

------

## 使用node.js操作MongoDB  



