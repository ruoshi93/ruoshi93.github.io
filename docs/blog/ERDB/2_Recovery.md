# 2. Recovery 恢复 Fehlerbehandlung

## Fehlerklassifikation
-   Lokaler Fehler in einer noch nicht festgeschriebenen (committed) Transaktion  
	    R1-Recovery 没有commit的事务出现错误，必须回滚
-   Fehler mit Hauptspeicherverlust  
	    R2-Recovery 已完成的事务，必须保存 Redo  
	    R3-Recovery 未完成的事务，必须回滚 Undo  
-   Fehler mit Hintergrundspeicherverlust 
        R4-Recovery 

## Seitenersetzungsstrategie: Steal & Force
### 替换缓冲页
	- ¬steal
	仍在活动事务的页不可替换，因此 kein Undo
	- steal
	任何没有被固定的页都是替换的候选者，因此需要 Undo
### 放置已完成事务的更改
	- force
	修改在事务结束后写入硬盘，因此 kein Redo
	- ¬force
	被修改的页可以留在Puffer里，需要Redo
*steal & ¬force 运行最快速，但恢复时代价更大。*

## Einbringungsstrategie 写入策略
	- Update in Place
	  每页在硬盘上有固定位置，页的旧状态会被覆写
	- Twin-block (Twin-Block-Verfahren)
	  每个页有副本，且位置连续
	- Shadow-storage (Schattenspeicherkonzept)
      只有被修改的页有副本，比Twin Block冗余更少

## 日志

### 日志条目结构
[LSN, TransaktionsID, PageID, Redo, Undo, PrevLSN]  
- LSN:  Log Sequence Number
- PrevLSN:   
指示前置的日志条目，对于R1-Recovery 和 Loser事务的回滚很重要

### 日志类型
- Physische Protokollierung 物理日志  
	- before-image 操作执行前的状态
    - after-image 操作执行后的状态
- Logische Protokollierung 逻辑日志
	- before-image通过执行Undo从after-image中产生  
    - after-image通过执行Redo从before-image中产生  

Speicherung der Seiten-LSN:   
每页都储存了关于该页最新日志条目的LSN

### 日志至少会被两次写入
1. Log-Datei für schnellen Zugriff
	R1, R2, R3-Recovery
2. Log-Archiv 归档
	R4-Recovery
	
### WAL原则：Write Ahead Log
1. 在事务commit之前，必须把和这个事务有关的所有日志写进硬盘。
2. 在置换脏页进入硬盘之前，这个页的日志也必须先写到硬盘。

## 数据库恢复三阶段
1. 分析
- 从头到尾分析临时日志
- 检查Winner，归入一个列表
- 检查Loser，归入一个列表
2. Redo  
Redo的目的是恢复到意外出现前数据库的状态
- 按执行顺序写入所有日志里的修改
- 日志条目的LSN和页LSN对比，确定redo是否必要  
  *若当前日志里扫描到的那条的LSN比写入数据库写入结果保存的那条LSN更加小，意味着后面已经写入硬盘，这里不需要重做。*
- 页里的LSN条目更新
3. Undo  
Loser的修改操作: 按照倒叙撤销他们的原本操作

### CLR: Compensating log record
UndoNxtLSN: 保存恢复的进度，防止Undo时再次崩溃  
CLR结构：  
<LSN, TA-Identifikator, betroffene Seite, Redo, PrevLSN, UndoNxtLSN>

## 三种检查点 Drei unterschiedliche Sicherungspunkt-Qualitäten 
- transaktionskonsistent 事务一致Checkpoint
- aktionskonsistent 行为一致Checkpoint
- unscharf (fuzzy) 模糊Checkpoint

## ARIES: Algorithms for Recovery and Isolation Exploiting Semantics
特点：
1. steal
2. ¬force 
3. Update in Place
4. Kleine Sperrgranulate 小锁粒度  
   在单句级别。脏数据和已提交更新同时存在在页、缓存、硬盘中

采用WAL原则，CLR，fuzzy检查点



