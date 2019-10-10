---
title: Springboot+mybatis-plus实现restful风格接口
date: 2019-10-10 14:59:27
tags:
- springboot
- mybatis-plus
- mybatis
categories:
- 后端技术
---
**摘要**：本文将通过Springboot和 [MyBatis-Plus](https://mp.baomidou.com) 的整合实现restful风格接口，使用更强大但不侵入的mybatis-plus,简单满足日常需求，甚至可以达到不写dao层就能实现crud，分页等的效果，由于写博客经验欠缺，可能要点不够详细

## springboot

springboot按照通常情况创建工程

## [MyBatis-Plus](https://mp.baomidou.com)

### 导入依赖

首先现在pom文件中导入mp等必备的依赖

``` bash
		<dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!--mybatis-plus依赖-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.2.0</version>
        </dependency>
        <!--mybatis-plus代码生成器-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
            <version>3.2.0</version>
        </dependency>
        
    </dependencies>
```

### 配置application.xml

配置数据源，日志打印等常用配置

``` bash
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mp_db?characterEncoding=utf-8
    username: root
    password: root
    jackson:
      date-format: yyyy-MM-dd HH:mm:ss
      time-zone: GMT+8

logging:
  level:
#    root: info
    com.liu.dao: trace
  pattern:
    console: '%p%m%n'
```

### 建立实体类

建立实体类，注解实现映射

``` bash
package com.liu.model;

import com.baomidou.mybatisplus.annotation.*;
import com.baomidou.mybatisplus.extension.activerecord.Model;
import lombok.Data;

import java.time.LocalDateTime;
import java.util.Date;
import java.util.List;

@Data
//@TableName("user") 注解映射:表名
public class User extends Model<User> {
    //@TableId("id") 注解映射:主键
    //@TableId(value = "id",type = IdType.AUTO) //注解映射:主键,并且标注自增
    private Integer id;
    //@TableField("real_name") 注解映射:字段
    private String realName;
    private Integer pid;

    @TableField(fill=FieldFill.INSERT)//insert新增一个记录时自动填充
    private LocalDateTime createTime;

    @TableField(fill=FieldFill.UPDATE)//update更新一个记录时自动填充
    private LocalDateTime updateTime;

    @TableLogic//标注为逻辑删除字段(全局默认0未删除，1已删除)
    @TableField(select = false)//全局标注为不查询的字段
    private Integer deleted;

    @TableField(exist = false) //标注为user表中不存在的字段，但为实体类需求的属性
    protected List<String> list;
}
```

### dao层通过继承baseMapper（通用mapper），实现对实体类功能强大的基本操作

``` bash
@Repository
public interface UserMapper extends BaseMapper<User>{
}
```
### 使userMapper拥有以下功能强大，全面的方法
``` bash
public interface BaseMapper<T> extends Mapper<T> {
    int insert(T entity);

    int deleteById(Serializable id);

    int deleteByMap(@Param("cm") Map<String, Object> columnMap);

    int delete(@Param("ew") Wrapper<T> wrapper);

    int deleteBatchIds(@Param("coll") Collection<? extends Serializable> idList);

    int updateById(@Param("et") T entity);

    int update(@Param("et") T entity, @Param("ew") Wrapper<T> updateWrapper);

    T selectById(Serializable id);

    List<T> selectBatchIds(@Param("coll") Collection<? extends Serializable> idList);

    List<T> selectByMap(@Param("cm") Map<String, Object> columnMap);

    T selectOne(@Param("ew") Wrapper<T> queryWrapper);

    Integer selectCount(@Param("ew") Wrapper<T> queryWrapper);

    List<T> selectList(@Param("ew") Wrapper<T> queryWrapper);

    List<Map<String, Object>> selectMaps(@Param("ew") Wrapper<T> queryWrapper);

    List<Object> selectObjs(@Param("ew") Wrapper<T> queryWrapper);

    IPage<T> selectPage(IPage<T> page, @Param("ew") Wrapper<T> queryWrapper);

    IPage<Map<String, Object>> selectMapsPage(IPage<T> page, @Param("ew") Wrapper<T> queryWrapper);
}
```
### restful风格
``` bash
@Slf4j
@RestController
@RequestMapping("/rest")
public class PeopleController {
    //private final static Logger log = LoggerFactory.getLogger(PeopleController.class);

    @Autowired
    private UserService userService;

    @GetMapping("/people")
    public AjaxResponse getAll(){
        List<User> users = userService.getAll();
        if (users!=null){
            log.info("查询peoples:"+ users);
            return AjaxResponse.success(users);
        }else {
            log.info("查询集合peoples失败");
            return AjaxResponse.fail();
        }
    }
    @GetMapping("/people/{id}")
    public AjaxResponse getOne(@PathVariable("id") int id){
        User user =userService.getOne(id);
        if (user!=null){
            log.info("查询单个people:"+ user);
            return AjaxResponse.success(user);
        }else {
            log.info("查询people失败");
            return AjaxResponse.fail();
        }
    }
    @PostMapping("/people")
    public AjaxResponse add(@RequestBody User user){
        int result=userService.add(user);
        if (result>0){
            log.info("新增people:"+ user);
            return AjaxResponse.success(user);
        }else {
            log.info("新增people失败");
            return AjaxResponse.fail();
        }
    }
    @PutMapping("/people")
    // (x-www-form-urlencoded)表单提交时，使用public AjaxResponse edit( User user)或@RequestParam绑定参数

    //使用json提交时使用@RequestBody
    public AjaxResponse edit(@RequestBody User user){
        int result=userService.update(user);
        log.info("User"+user);
        log.info("result"+result);
        if (result>0){
            log.info("修改people:"+user.getUserId()+":"+ user);
            return AjaxResponse.success(user);
        }else {
            log.info("修改people:"+user.getUserId()+":失败");
            return AjaxResponse.fail();
        }

    }
    @DeleteMapping("/people/{id}")
    public AjaxResponse delete(@PathVariable("id")int id){
        int result=userService.delete(id);
        if (result>0){
            log.info("删除people:"+id);
            return AjaxResponse.success(id);
        }else{
            log.info("删除people:失败");
            return AjaxResponse.fail();
        }
    }
    //根据条件删除，deleteByExample(String userName)
    @DeleteMapping("/people/bynamedelete/{name}")
    public AjaxResponse deleteByExample(@PathVariable("name") String userName){
        int result=userService.deleteByExample(userName);
        if (result>0){
            log.info("删除people:"+userName);
            return AjaxResponse.success(userName);
        }else{
            log.info("删除people:失败");
            return AjaxResponse.fail();
        }
    }
    @RequestMapping("/text")
    public String text(){
        return "hello,Text~....";
    }
```
>1.mybatis-plus提供了功能强大的Wrapper，可实现多重复杂的条件查询
>2.可以实现数据库层的物理分页查询
>3.同时拥有通用service，不写dao层代码就实现对实体类基本常用的操作
>具体方法文档请浏览 [MyBatis-Plus官网](https://mp.baomidou.com)


