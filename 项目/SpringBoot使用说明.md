## mybatis-plus的使用

### 开启逻辑删除

在高版本mybatis-plus中，需要两步

1.在application配置文件中开启逻辑删除选项

```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-value: 1
      logic-not-delete-value: 0
```

2.在实体类中数据库对应的属性加入注解，其中value表示正常显示的值，delval表示逻辑删除后的值

`@TableLogic(value = "1", delval = "0")`



