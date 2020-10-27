## PG常用SQL

#### 重置id

```
select setval('page_info_id_seq',(select max(id) from page_info));
```

