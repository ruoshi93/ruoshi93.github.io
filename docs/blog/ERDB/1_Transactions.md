# 1. Transactions 事务 Transaktionsverwaltung

## 传统事务系统中的操作
*   begin of transaction (BOT)
*   commit 提交 / abort 中止

## 新的数据库应用中
*   define savepoint
*   backup transaction

## 事务的特性：ACID
-  **A**tomicity 原子性 Atomarität  
    *alles oder nichts!*  
    某个事物的所有动作在数据库中要么全部反映出来，要么全部不反映。
-   **C**onsistency 一致性  
    银行系统中的两个账户A和B，执行转账等事务后，事务的执行不改变A、B之和。
-   **I**solation 隔离性  
    隔离性保证了并发执行多个事物对系统的状态的影响和串行化执行多个事物对系统的状态的影响是一样的。

-   **D**urability 持久性 Dauerhaftigkeit  
    执行成功的事物，其更改不会丢失。

##  SQL中的事务管理
*   commit work
*   rollback work 回滚

---

*   winner事务：其效果必须是完全可追踪的
*   loser事务：在中断发生时仍旧活跃，需要重做