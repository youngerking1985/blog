# pg全文检索加速方法尝试
## 背景描述
进行POI、地名检索的模糊搜索时，执行语句较慢。目前数据地名使用了rum索引，地理范围使用了基于GiST的R-Tree索引。因为Like语句的所有只能执行‘AB%’，如果执行‘%A%B%'，会导致失效，所以原有方案中引入rum所谓全文索引
### 问题分析：
1. 字符串匹配的gin效率较慢
2. 基于北京市进行地理范围包含计算效率较慢


## 相关概念
### gin全文索引
GIN是Generalized Inverted Index的缩写，就是所谓的倒排索引。GIN的主要应用领域是加速全文搜索
### tsvector
全文检索会用到的数据类型，tsvector的值是一个无重复的lexemes排序列表，比如将一个字符串转换为tsvector类型：
```
test=# select 'I am chinese'::tsvector;
      tsvector      
--------------------
 'I' 'am' 'chinese'
(1 row)

select string_to_tsvector('我是中国人');
         string_to_tsvector         
------------------------------------
 '中':3 '人':5 '国':4 '我':1 '是':2
(1 row)

```

### tsquery
全文检索的条件语句构成，tsquery查询条件并不是简单的正则，而是一组搜索术语，使用并且使用布尔操作符&（AND）、|（OR）和!（NOT）来组合它们，还有短语搜索操作符<->（FOLLOWED BY）。存储用于检索的词汇，并且使用布尔操作符&(AND),|(OR),!(NOT)来组合它们。!(NOT)结合的最紧密，而&(AND)结合的比|(OR)紧密，也可以用括号来强调分组。

```
test=# SELECT $$fat & rat$$::tsquery;
    tsquery    
---------------
 'fat' & 'rat'
(1 row)

test=# 
test=# SELECT $$fat & (rat | cat)$$::tsquery;
          tsquery          
---------------------------
 'fat' & ( 'rat' | 'cat' )
(1 row)

test=# 
test=# SELECT $$fat & rat & ! cat$$::tsquery;
        tsquery         
------------------------
 'fat' & 'rat' & !'cat'
(1 row)
```

### @@
全文检索操作符@@操作符支持隐式转换，对于text类型可以无需强类型转换(::tsvector或to_tsvector(config_name, text))，所以这个操作符实际支持的参数类型是这样的:

```
tsvector @@ tsquery
tsquery  @@ tsvector
text @@ tsquery
text @@ text
```
```
SELECT 'a fat cat sat on a mat and ate a fat rat'::tsvector @@ 'cat & rat'::tsquery;
 ?column?
----------
 t
```

### 分词检索
PostgreSQL默认分词是按照空格及各种标点符号来分词，但是对于国内更多的是中文文章，按照默认分词方式不符合中文的分词方式，目前中文分词使用最多的是zhparser和pg_jieba。

## 当前方法语句及执行效果
其中POI库表共1000w左右的数据记录，当前语句需要查询从全球POI中查询位于北京市的通州区相关POI，SQL语句如下
```
select *,st_astext(geom) from "POI"  where string_to_tsvector(name_cn) @@ string_to_tsquery('通州区') and st_contains((select geom from "BOUA" where name_cn like '北京市%'), geom) order by lv,length limit 7; 
```
执行分析如下，总共需要执行12s左右，主要的时间在string_to_tsvector：
```
gin_all_place=# explain analyze select *,st_astext(geom) from "POI"  where string_to_tsvector(name_cn) @@ strin "BOUA" where name_cn like '北京市%'), geom) order by lv,length limit 7; 
                                                                       QUERY PLAN                              
---------------------------------------------------------------------------------------------------------------
 Limit  (cost=364294.78..364296.05 rows=1 width=1952) (actual time=12724.197..12724.212 rows=7 loops=1)
   InitPlan 1 (returns $0)
     ->  Seq Scan on "BOUA"  (cost=0.00..1361.70 rows=1 width=31854) (actual time=50.164..50.184 rows=1 loops=1
           Filter: ((name_cn)::text ~~ '北京市%'::text)
           Rows Removed by Filter: 6055
   ->  Result  (cost=362933.08..362934.35 rows=1 width=1952) (actual time=12724.187..12724.197 rows=7 loops=1)
         ->  Sort  (cost=362933.08..362933.09 rows=1 width=1920) (actual time=12724.131..12724.133 rows=7 loops
               Sort Key: "POI".lv, "POI".length
               Sort Method: top-N heapsort  Memory: 26kB
               ->  Bitmap Heap Scan on "POI"  (cost=475.73..362933.07 rows=1 width=1920) (actual time=212.052..
                     Filter: ((string_to_tsvector((name_cn)::text) @@ '''通'' <-> ''州'' <-> ''区'''::tsquery) 
                     Rows Removed by Filter: 338177
                     Heap Blocks: exact=11673
                     ->  Bitmap Index Scan on "sidx_POI_geom"  (cost=0.00..475.73 rows=12708 width=0) (actual t
                           Index Cond: (geom @ $0)
 Planning Time: 3.071 ms
 Execution Time: 12725.075 ms
(17 rows)
```

## 优化方式
通过sql语句执行分析，可以看到主要耗时在tovector，考虑增加tsvector字段，将数据进行预处理，减少sql执行的计算量
alter table "POI" add column name_cn_tsv tsvector;
update "POI" set name_cn_tsv = to_tsvector(name_cn);
再次执行SQL语句，时间缩到400ms左右，提升为30倍(首次查询为800ms左右)
```
gin_all_place=# explain analyze select *,st_astext(geom) from "POI"  where name_cn_tsv @@ string_to_tsquery(' 通州区') and st_contains((select geom from "BOUA" where name_cn like '北京市%'), geom) order by lv,length limit 7; 
                                                                      QUERY PLAN                              
                                         
--------------------------------------------------------------------------------------------------------------
-----------------------------------------
 Limit  (cost=367826.31..367827.57 rows=1 width=2045) (actual time=379.711..379.729 rows=7 loops=1)
   InitPlan 1 (returns $0)
     ->  Seq Scan on "BOUA"  (cost=0.00..1361.70 rows=1 width=31854) (actual time=6.413..6.419 rows=1 loops=1)
           Filter: ((name_cn)::text ~~ '北京市%'::text)
           Rows Removed by Filter: 6055
   ->  Result  (cost=366464.61..366465.87 rows=1 width=2045) (actual time=379.709..379.724 rows=7 loops=1)
         ->  Sort  (cost=366464.61..366464.61 rows=1 width=2013) (actual time=379.695..379.703 rows=7 loops=1)
               Sort Key: "POI".lv, "POI".length
               Sort Method: top-N heapsort  Memory: 26kB
               ->  Bitmap Heap Scan on "POI"  (cost=776.43..366464.60 rows=1 width=2013) (actual time=93.154..
378.886 rows=965 loops=1)
                     Filter: ((name_cn_tsv @@ '''通'' <-> ''州'' <-> ''区'''::tsquery) AND st_contains($0, geo
m))
                     Rows Removed by Filter: 338177
                     Heap Blocks: exact=18490
                     ->  Bitmap Index Scan on "sidx_POI_geom"  (cost=0.00..776.43 rows=12802 width=0) (actual 
time=87.625..87.625 rows=339142 loops=1)
                           Index Cond: (geom @ $0)
 Planning Time: 0.663 ms
 Execution Time: 380.269 ms
(17 rows)
```

## 优化2
创建gin索引


## TODO
增加SQL并行计划；
构建GIN索引；
构建lv，length联合索引；

## 参考
1. 浅谈postgresql的GIN索引(通用倒排索引)：https://www.cnblogs.com/flying-tiger/p/6704931.html
2. 全文检索官方文档：http://www.postgres.cn/docs/10/textsearch-tables.html
3. gin索引：https://razeencheng.com/post/pg-like-index-optimize
4. rum索引：https://blog.csdn.net/dazuiba008/article/details/104653130
5. gin索引简单使用：https://blog.csdn.net/dazuiba008/article/details/103985791