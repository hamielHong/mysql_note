mysql update
===

批量更新
---

如果是更新为同样的内容，没啥难度，直接在where里面下功夫就好了，大家都懂，我要说的是针对更新内容不一样的情况

首先，先看看网上转载的方法：

mysql 批量更新如果一条条去更新效率是相当的慢, 循环一条一条的更新记录,一条记录update一次，这样性能很差，也很容易造成阻塞。

mysql 批量更新共有以下四种办法：

1. replace into 批量更新

    > replace into test_tbl (id,dr) values (1,'2'),(2,'3'),...(x,'y');

2. insert into ...on duplicate key update批量更新

    > insert into test_tbl (id,dr) values (1,'2'),(2,'3'),...(x,'y') on duplicate key update dr=values(dr);

3. 创建临时表，先更新临时表，然后从临时表中update

    ``` sql
    create temporary table tmp(id int(4) primary key,dr varchar(50));
    insert into tmp values  (0,'gone'), (1,'xx'),...(m,'yy');
    update test_tbl, tmp set test_tbl.dr=tmp.dr where test_tbl.id=tmp.id;
    ```
    注意：这种方法需要用户有temporary 表的create 权限。

4. 使用mysql 自带的语句构建批量更新

    mysql 实现批量 可以用点小技巧来实现:

    ``` sql
    UPDATE yoiurtable
        SET dingdan = CASE id
            WHEN 1 THEN 3
            WHEN 2 THEN 4
            WHEN 3 THEN 5
        END
    WHERE id IN (1,2,3)
    ```

    这句sql 的意思是，更新dingdan 字段，如果id=1 则dingdan 的值为3，如果id=2 则dingdan 的值为4……

    where部分不影响代码的执行，但是会提高sql执行的效率。确保sql语句仅执行需要修改的行数，这里只有3条数据进行更新，而where子句确保只有3行数据执行。

    如果更新多个值的话，只需要稍加修改：

    ``` sql
    UPDATE categories
        SET dingdan = CASE id
            WHEN 1 THEN 3
            WHEN 2 THEN 4
            WHEN 3 THEN 5
        END,
        title = CASE id
            WHEN 1 THEN 'New Title 1'
            WHEN 2 THEN 'New Title 2'
            WHEN 3 THEN 'New Title 3'
        END
    WHERE id IN (1,2,3)
    ```

    到这里，已经完成一条mysql语句更新多条记录了。

php中用数组形式赋值批量更新的代码：

    ``` php
    $display_order = array(
        1 => 4,
        2 => 1,
        3 => 2,
        4 => 3,
        5 => 9,
        6 => 5,
        7 => 8,
        8 => 9
    );
    $ids = implode(',', array_keys($display_order));
    $sql = "UPDATE categories SET display_order = CASE id ";
    foreach ($display_order as $id => $ordinal) {
        $sql .= sprintf("WHEN %d THEN %d ", $id, $ordinal);
    }
    $sql .= "END WHERE id IN ($ids)";
    echo $sql;
    ```

更新 100000条数据的性能就测试结果来看，测试当时使用replace into性能较好。

replace into  和 insert into on duplicate key update的不同在于：

replace into 操作本质是对重复的记录先delete 后insert，如果更新的字段不全会将缺失的字段置为缺省值，用这个要悠着点！否则不小心清空大量数据可不是闹着玩的！！！

insert into 则是只update重复记录，不会改变其它字段。

还有一种是我偶尔在写临时脚本的时候用的懒方法，实现起来非常简单，速度肯定不如插入的方法，但是比起一条一条更新，效果也相当明显

就是直接在循环之前启动事务，循环结束后一起提交，省去每次连接数据库，解析SQL语句等时间。（注意：如果量太大，最好还是要分割一下，比如1000条分割一次，分批次提交）