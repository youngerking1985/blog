# pg事件触发器
## 背景描述
进行drop table删除时，需要进行附加表的清理工作；基于postgres默认pgsql触发器编写方式，难以满足该需求场景

## 相关概念
pg事件触发器包括ddl_command_start, ddl_command_end, table_rewrite 和 sql_drop4大类，各个事件触发器所能支持的command可参考[触发器命令说明](https://www.postgresql.org/docs/13/event-trigger-matrix.html)

### ddl_command_start
该触发器主要在执行ddl命令之前触发，包括 CREATE, ALTER, DROP, SECURITY LABEL, COMMENT, GRANT or REVOKE等命令

### ddl_command_end
该触发器主要在执行ddl命令之后触发，包括 CREATE, ALTER, DROP, SECURITY LABEL, COMMENT, GRANT or REVOKE等命令，可以通过pg_event_trigger_ddl_commands，获取相关的表，schema相关信息；

### table_rewrite
该触发器主要在表结构被修改时触发，如alter table

### sql_drop
该触发器可在执行drop命令时触发，如drop table。可以通过pg_event_trigger_dropped_objects获取相关的操作信息。不过该触发器执行时，表已经被删除

## 当前drop存在的问题
### 方案1，采用sql drop触发器问题
目前需要做的工作是drop table时，对table记录内关联的附表数据进行清理，所以直接调用sql_drop将无法获取表内容；
### 方案2， 采用ddl_command_start
ddl_command_start触发时，无法通过pg_event_trigger_dropped_objects，pg_event_trigger_ddl_commands获取到相应的事件消息， 只能通过tg_event, tg_tag获取简单的tag信息，无法获取表信息

## 当前采用方式
详细代码参见：best_georaster--6.0.0.sql/drop_before_raster_table_trigger
通过ddl_command_start触发c function，从c代码中获取EventTriggerData以及DropStmt信息，这种方式下，可以获取到schema和table等信息
``` sql
CREATE OR REPLACE FUNCTION _GR_DropRasterTableBeforeC_()
RETURNS event_trigger
AS '$libdir/libbestdb', '_GRDropRasterTableBeforeC_'
LANGUAGE c IMMUTABLE STRICT;

drop event trigger if exists drop_before_raster_table_trigger;
CREATE  EVENT TRIGGER drop_before_raster_table_trigger ON ddl_command_start
WHEN TAG IN('DROP TABLE')
EXECUTE FUNCTION _GR_DropRasterTableBeforeC_();

```
``` c++
PG_FUNCTION_INFO_V1(_GRDropRasterTableBeforeC_);
Datum  _GRDropRasterTableBeforeC_(PG_FUNCTION_ARGS)
{
  EventTriggerData *trigdata;
  DropStmt *drop;
  ObjectAddresses *objects;
  char relkind;
  ListCell *cell;
  int flags = 0;
  LOCKMODE lockmode = AccessExclusiveLock;
  int ret;

  if (!CALLED_AS_EVENT_TRIGGER(fcinfo))
    elog(ERROR, "auto_create_schema can only be called as an event trigger.");

  trigdata = (EventTriggerData *)fcinfo->context;
  drop = (DropStmt *)trigdata->parsetree;
    foreach (cell, drop->objects)
    {
        RangeVar *rel = makeRangeVarFromNameList((List *)lfirst(cell));
        if (!rel)
        continue;

        /* Check if schema was specified */
        char *schema = rel->schemaname;
        char *relname = rel->relname;
    }
}
```


## 参考
1. 官方文档： https://www.postgresql.org/docs/13/event-trigger-definition.html
2. 触发器与sql command触发关系：https://www.postgresql.org/docs/13/event-trigger-matrix.html