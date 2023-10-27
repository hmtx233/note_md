
### navicat 工具 回滚数据到某一时刻

```sql
alter table table_name enable row movement;
flashback table table_name  to timestamp TO_TIMESTAMP('2022-08-08 13:42:00', 'yyyy-mm-dd hh24:mi:ss');
alter table table_name disable row movement;
```

### oracle创建序列和触发器实现字段自增

```sql
-- 创建序列
CREATE SEQUENCE TABLE_NAME_ID_SEQ
INCREMENT BY 1  -- 每次新增1
START WITH 1 -- 开始值
MAXVALUE 999999999 -- 最大值
NOCYCLE       -- 不循环
NOCACHE;      -- 没有缓存 

-- 删除序列
drop SEQUENCE TABLE_NAME_ID_SEQ;

-- 创建触发器
CREATE OR REPLACE TRIGGER TABLE_NAME_ID_SEQ_TRG
BEFORE INSERT ON "TABLE_NAME"
FOR EACH ROW
WHEN (NEW."ID" IS NULL)
BEGIN
  SELECT TABLE_NAME_ID_SEQ.NEXTVAL
  INTO :NEW."ID"
  FROM DUAL;
END;

-- 删除触发器
drop TRIGGER TABLE_NAME_ID_SEQ_TRG;
```