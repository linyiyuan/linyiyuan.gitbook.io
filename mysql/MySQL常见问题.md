# MySQL常见问题


**1\. 理解select Count (*)和Select Count(1)以及Select Count(column)区别**

  一般情况下，Select Count (*)和Select Count(1)两着返回结果是一样的

    假如表沒有主键(Primary key), 那么count(1)比count(*)快，

    如果有主键的話，那主键作为count的条件时候count(主键)最快

    如果你的表只有一个字段的话那count(*)就是最快的

   count(*) 跟 count(1) 的结果一样，都包括对NULL的统计，而count(column) 是不包括NULL的统计

