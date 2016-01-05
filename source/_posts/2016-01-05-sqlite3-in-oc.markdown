---
layout: post
title: "Objective-C中使用sqlite3"
date: 2016-01-05 11:39:17 +0800
comments: true
categories: 
---
1.打开数据库
使用sqlite3_open
2.查询数据库
 每一次使用sql语句查询 使用sqlite3_prepare_v2创建一个sqlite3_stmt.
  {% highlight objc %}
    sqlite3_stmt *stmt = NULL ;
    int result = sqlite3_prepare_v2(_db, sql.UTF8String, -1, &stmt, NULL);
    if (result != SQLITE_OK) {
        if (_errorLogsEnabled) NSLog(@"%s line:%d sqlite stmt prepare error (%d): %s", __FUNCTION__, __LINE__, result, sqlite3_errmsg(_db));
        return nil;
    }
    {% endhighlight  %}
 然后使用sqlite3_bind_text和sqlite3_bind_int,sqlite3_bind_blob插入数据到sqlite3_stmt 实例
  {% highlight objc %}
    //stmt是上文得到的stmt ,index是你需要插入的参数值的索引,key是你的参数值,-1是填入的字符长度-1表示到/0结束,NULL表示完成后如何释放
    sqlite3_bind_text(stmt, index , key.UTF8String, -1, NULL);
  {% endhighlight  %}
3 得到结果
 然后使用sqlite3_step函数执行2步骤里面的stmt ，会返回SQLITE_BUSY, SQLITE_DONE, SQLITE_ROW等消息
如果是SQLITE_ROW 则使用sqlite3_column_text或者其他类型的sqlite3_column得到返回的这一row的对应的值
{% highlight objc %}
do{
        result = sqlite3_step(stmt);
        if (result == SQLITE_ROW) {
            //得到这一行的数据
            char *key = (char *)sqlite3_column_text(stmt, i++);
            char *filename = (char *)sqlite3_column_text(stmt, i++);
            int size = sqlite3_column_int(stmt, i++);
        }else if (result == SQLITE_DONE){
            //完成
            break;
        }else{
            //error
            break;
        }
    }while(1);
    {% endhighlight  %}
