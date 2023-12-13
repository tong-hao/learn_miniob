
## 整体流程

Parser->Plan Cache -> Resolver -> Transformer -> Optimizer -> Executor -> Storage Engine

## observer 代码
- sql/parser
- sql/plan_cache
- sql/optimizer
- sql/executor
- storage


## server 启动
```
src/observer/main.cpp
 Server::serve()

```

## SQL执行流程
```cpp
SessionStage::handle_request
 SessionStage::handle_sql
  ParseStage::handle_request
   yacc_sql.cpp::sql_parse
  // 将解析后的SQL语句，转换成各种Stmt(Statement), 同时会做错误检查
  ResolveStage::handle_request 
  OptimizeStage::handle_request
   OptimizeStage::create_logical_plan
   OptimizeStage::rewrite
   OptimizeStage::optimize
   OptimizeStage::generate_physical_plan
  ExecuteStage::handle_request
   CommandExecutor::execute
    ShowTablesExecutor::execute
     Db::all_tables
```

## mvcc

begin:
```cpp
TrxBeginExecutor::execute
 // 创建transaction
 Session::current_trx()
  // 事务需要日志管理器log_manager
  MvccTrxKit::create_trx(CLogManager *log_manager)
  MvccTrx::start_if_need()
   CLogManager::begin_trx
    CLogManager::append_log
     CLogBuffer::append_log_record
```

end:
```
TrxEndExecutor::execute
 Session::current_trx()
 MvccTrxKit::create_trx(CLogManager *log_manager)
 MvccTrx::commit()
 MvccTrx::commit_with_trx_id
  CLogManager::commit_trx
  append_log
  sync
   CLogBuffer::flush_buffer
```
