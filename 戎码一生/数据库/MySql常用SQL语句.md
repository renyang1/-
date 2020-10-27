## MySql常用SQL语句

**-- 查看数据库**

```mysql
show DATABASES;
```

**-- 创建数据库**

```mysql
CREATE DATABASE foodie;
```

**-- 切换数据库**

```mysql
USE DATABASE foodie;
```

**-- 查看数据库中所有表**

```mysql
show TABLES;
```

**-- 查看表结构**

```mysql
discribe user;
```

-- 建表语句

```mysql
-- 图片元素表
DROP TABLE IF EXISTS `element_image`;
CREATE TABLE `element_image`  (
  `id` bigint(20) UNSIGNED NOT NULL AUTO_INCREMENT,
  `no` varchar(32) NOT NULL DEFAULT '' COMMENT '业务主键',
  `image_url` varchar(500) NOT NULL COMMENT '图片地址',
  `image_link_url` varchar(500) NOT NULL COMMENT '图片链接地址',
  `is_delete` tinyint(4) NOT NULL DEFAULT 0 COMMENT '是否删除  0未删除 1删除',
  `create_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP(0) COMMENT '创建时间',
  `create_user` VARCHAR(32) NOT NULL DEFAULT ''	COMMENT '创建人编号',
  `update_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP(0) ON UPDATE CURRENT_TIMESTAMP(0) COMMENT '修改时间',
  `update_user` VARCHAR(32) NOT NULL DEFAULT '' COMMENT '修改人编号',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci COMMENT = '图片元素表' ROW_FORMAT = Dynamic;
```

