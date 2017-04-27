# VLOOKUP

## 快速类比

编号 A| 项目 B | 数量 C| 人工单价 D
----|----|----|----
1| 地面贴砖（平方）  |4.49 	|   25
2| 墙面贴砖（平方）  |21.00 	|   25
3| 包下水管（个）	|1.00 		|   120
4| 包燃气管（个）	|1.00 		|   50
5| 烟道加固（个）	|1.00 		|   200
6| 瓷砖磨边（米）	|11.00 		|   10

上表中查编号(A列)为3的人工单价是多少？

SQL: `SELECT D from A2:D7 where A=3 `

VLOOKUP: `=VLOOKUP( 3, A2:D7, 4, FALSE())`

查找在区域（A2:D7）中默认匹配第一列其值为3的行，返回该行的第4列数据。

其中4表示第4列，即D列，FALSE表示匹配时不做模糊查询。

用 SQL SELECT 语句去类比记忆，查找什么区域，其第一列值为多少的行，返回哪列值，返回的列就是你要的数据。

## 查找单个数据

VLOOKUP用于从选定区域（表）中根据指定条件查找单个数据。Excel开发人员应该直接借鉴SQL SELECT语句，我们也从这个角度来类比记忆。



在以上表格中查找*编号A=3* 这一项的人工单价D是多少。

```
# SQL 
# 用A2:D7代替这个表名，没毛病，注意这里列名字占据第1行

SELECT D from A2:D7 where A=3
# 结果 120

# 重点参数
# D = DES_COL，查找的值所在列
# A2:D7 = FROM_AREA，所在的表
# A = WHERE_COL，匹配条件
# 3 = 匹配值

# 把这个SQL查询封装成查找函数，最简单的：
=SELECT( DES_COL, FROM_AREA, WHERE_COL, VALUE );

# 如果限定 WHERE_COL 只能是所选区域FROM_AREA的第一列，那这个条件就可以省掉
=SELECT( DES_COL, FROM_AREA, VALUE );
=SELECT( 4, A2:D7, 3);

# 根据个人喜好，你喜欢把VALUE放最前面，就变成了：
=SELECT( VALUE, FROM_AREA, DES_COL);
=SELECT( 3, A2:D7, 4);

# 换个函数名，就是VLOOKUP了，VLOOKUP多考虑了是否支持模糊匹配
=VLOOKUP( 3, A2:D7, 4, FALSE())
```

# 复合使用

```
# 与IF判断复合使用，如果查找的值符合条件怎么办
=IF(
	VLOOKUP(B4, B2:D7, 3)=120,
	"等于120",
	"不等于120" )
	
# 与ISBLANK复合使用，如果查出来的值是空的怎么办
=ISBLANK( VLOOKUP(B4, B2:D7, 3) )
```

