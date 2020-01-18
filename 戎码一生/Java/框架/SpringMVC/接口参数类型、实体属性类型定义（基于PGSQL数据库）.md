## 接口参数类型、实体属性类型定义（基于PGSQL数据库）

### 数据库表设计

![image-20191218002848190](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20191218002848190.png)

```sql
DROP TABLE IF EXISTS "public"."test_user";
CREATE TABLE "public"."test_user" (
  "id" serial8 NOT NULL ,
  "no" varchar(32) COLLATE "pg_catalog"."default" NOT NULL DEFAULT ''::character varying,
  "name" varchar(32) COLLATE "pg_catalog"."default" NOT NULL DEFAULT ''::character varying,
  "account" varchar(32) COLLATE "pg_catalog"."default" NOT NULL DEFAULT ''::character varying,
  "org_no" varchar(32) COLLATE "pg_catalog"."default" NOT NULL DEFAULT ''::character varying,
  "sex" int2 NOT NULL DEFAULT 0,
  "contact_number" varchar(16) COLLATE "pg_catalog"."default" NOT NULL DEFAULT ''::character varying,
  "employee_no" varchar(32) COLLATE "pg_catalog"."default" NOT NULL DEFAULT ''::character varying,
  "birthday" date NOT NULL DEFAULT CURRENT_TIMESTAMP,
  "start_time" time(6) NOT NULL DEFAULT CURRENT_TIMESTAMP,
  "end_time" time(6) NOT NULL DEFAULT CURRENT_TIMESTAMP,
  "create_time" timestamp(6) NOT NULL DEFAULT CURRENT_TIMESTAMP,
  "update_time" timestamp(6) NOT NULL DEFAULT CURRENT_TIMESTAMP,
  "create_user" varchar(32) COLLATE "pg_catalog"."default" NOT NULL DEFAULT ''::character varying,
  "update_user" varchar(32) COLLATE "pg_catalog"."default" NOT NULL DEFAULT ''::character varying,
  "is_deleted" int2 NOT NULL DEFAULT 0
)
;
COMMENT ON COLUMN "public"."test_user"."id" IS '主键';
COMMENT ON COLUMN "public"."test_user"."no" IS '业务主键';
COMMENT ON COLUMN "public"."test_user"."name" IS '名称';
COMMENT ON COLUMN "public"."test_user"."account" IS '账号';
COMMENT ON COLUMN "public"."test_user"."org_no" IS 'boss组织表业务主键';
COMMENT ON COLUMN "public"."test_user"."sex" IS '性别 0女 1男 2未知';
COMMENT ON COLUMN "public"."test_user"."contact_number" IS '联系方式';
COMMENT ON COLUMN "public"."test_user"."employee_no" IS '员工号';
COMMENT ON COLUMN "public"."test_user"."create_time" IS '创建时间';
COMMENT ON COLUMN "public"."test_user"."update_time" IS '更新时间';
COMMENT ON COLUMN "public"."test_user"."create_user" IS '创建人';
COMMENT ON COLUMN "public"."test_user"."update_user" IS '更新人';
COMMENT ON COLUMN "public"."test_user"."is_deleted" IS '是否删除:0 不删除,1 删除';
COMMENT ON COLUMN "public"."test_user"."birthday" IS '生日';
COMMENT ON COLUMN "public"."test_user"."start_time" IS '开始时间';
COMMENT ON COLUMN "public"."test_user"."end_time" IS '结束时间';
COMMENT ON TABLE "public"."test_user" IS '人员测试表';

-- ----------------------------
-- Indexes structure for table test_user
-- ----------------------------
CREATE INDEX "test_user_split_word_binary_idx" ON "public"."test_user" USING gin (
  split_word_binary(ARRAY[name::text]) COLLATE "pg_catalog"."default" "pg_catalog"."array_ops"
);

-- ----------------------------
-- Primary Key structure for table test_user
-- ----------------------------
ALTER TABLE "public"."test_user" ADD CONSTRAINT "test_user_pkey" PRIMARY KEY ("id");

ALTER TABLE "public"."test_user" ADD CONSTRAINT "test_user_no_key" UNIQUE ("no");
```

### Controller层出、入参对象属性定义

#### 入参

​	所有入参均可以使用String类型接收，再进行转换。但对于时间、枚举类型入参，可以使用Date、enum类型来进行接收，Date类型可以使用**@JsonFormat**注解来指定时间格式，枚举类型需要使用枚举名称（name）进行匹配，若使用数字的话，其实是按enum的下标顺序匹配，会有问题。

```java
@Data
public class TestUserDo extends BaseEntity implements Serializable {
    private static final long serialVersionUID = 42L;

    /** 名称 */
    private String name;

    /** 账号 */
    private String account;

    /** boss组织表业务主键 */
    private String orgNo;

    /** 性别 FEMALE=女, MALE=男, UNKNOWN未知 */
    private SexEnum sex;

    /** 联系方式 */
    private String contactNumber;

    /** 员工号 */
    private String employeeNo;

    /** 生日 */
    @JsonFormat(pattern = "yyyy-MM-dd")
    private Date birthday;

    /** 开始时间 HH:mm:ss */
    @JsonFormat(pattern = "HH:mm:ss")
    private Date startTime;

    /** 结束时间 yyyy-MM-dd HH:mm:ss */
    @JsonFormat(pattern = "HH:mm:ss")
    private Date endTime;

    /** 创建时间 */
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private Date creatTime;
}
```

**Date类型**

​	时间(包括yyyy-MM-dd HH:mm:ss、yyyy-MM-dd、HH:mm:ss格式)统一通过Date类型来接收，可以使用**@JsonFormat**注解来指定时间格式将前端传的字符串进行转换。需要注意当我们接收的时间格式不正确时，形如"2019--12-32-"、"2019-12-32-"格式正常通过，形如"2019+-12-32-"会失败，即部分错误的格式可以被处理，但处理后的时间不是我们传入的期望时间。

![image-20191221160628452](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20191221160628452.png)

**枚举类型**

​	枚举类型可以使用枚举name进行传值匹配，若 匹配不到抛出以下异常

```java
org.springframework.http.converter.HttpMessageNotReadableException: JSON parse error: Cannot deserialize value of type `com.newboss.api.base.enums.SexEnum` from String "MALE1": value not one of declared Enum instance names: [UNKNOWN, FEMALE, MALE]; 
nested exception is com.fasterxml.jackson.databind.exc.InvalidFormatException: Cannot deserialize value of type `com.newboss.api.base.enums.SexEnum` from String "MALE1": value not one of declared Enum instance names: [UNKNOWN, FEMALE, MALE]

```

### 