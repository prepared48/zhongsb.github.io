---
layout: post
title:  "springboot集成MongoDB"
date:   2019-12-31 15:14:54
categories: Java SpringBoot
tags: 微服务 MongoDB
mathjax: true
---

* content
{:toc}

思路：

Iservice 实现通用方法，增删改查，ServiceImpl实现具体方法。

业务Service接口增加具体业务方法，业务ServiceImpl实现业务Service接口





## 1 代码实现

### 1.1 yml文件配置：

	spring:
	    data:
	        mongodb:
	            host: IP
	            port: 27017
	            database: test
	            username: test
	            password: 123456

### 1.2 实体类

	package com.prepared.service.modules.mongodb.model;
	
	import org.springframework.data.annotation.Id;
	import org.springframework.data.mongodb.core.index.Indexed;
	import org.springframework.data.mongodb.core.mapping.Document;
	import org.springframework.data.mongodb.core.mapping.Field;
	
	import java.io.Serializable;
	
	/**
	 * mongodb测试用user实体类
	 *
	 * @date 2019/12/30
	 * @author z
	 */
	@Document(collection = "mongoDBUser")
	public class MongoDBUser implements Serializable {
	
	        private static final long serialVersionUID = -3258839839160856613L;
	        @Id
	        private Long id;
	
	        /**
	         * 声明username加索引，加快查询速度
	         */
	        @Indexed
	        private String userName;
	
	        @Field("pwd")
	        private String passWord;
	
	        public Long getId() {
	                return id;
	        }
	
	        public void setId(Long id) {
	                this.id = id;
	        }
	
	        public String getUserName() {
	                return userName;
	        }
	
	        public void setUserName(String userName) {
	                this.userName = userName;
	        }
	
	        public String getPassWord() {
	                return passWord;
	        }
	
	        public void setPassWord(String passWord) {
	                this.passWord = passWord;
	        }
	
	        @Override
	        public String toString() {
	                return "UserEntity{" +
	                        "id=" + id +
	                        ", userName='" + userName + '\'' +
	                        ", passWord='" + passWord + '\'' +
	                        '}';
	        }
	}

### 1.3 定义通用IService

	package com.prepared.service.datasource.mongodb;
	
	import java.util.List;
	
	/**
	 * MongoDB 通用IService
	 *
	 * @Author: z
	 * @Date: 2019/12/30 15:12
	 */
	public interface IService<T> {
	
	    /**
	     * 保存对象
	     *
	     * @param t 对象实体
	     */
	    void saveEntity(T t);
	
	    /**
	     * 根据ID修改实体
	     *
	     * @param t 对象实体
	     * @param id 主键
	     * @return
	     */
	    long updateEntityById(T t, Long id);
	
	    /**
	     * 根据ID进行删除
	     *
	     * @param id 主键
	     */
	    void deleteEntityById(Long id);
	
	    /**
	     * 查询所有实体
	     *
	     * @return
	     */
	    List<T> findAll();
	}

### 1.4 实现通用IService

	package com.prepared.service.datasource.mongodb.impl;
	
	import com.mongodb.client.result.UpdateResult;
	import com.prepared.service.datasource.mongodb.IService;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.data.mongodb.core.MongoTemplate;
	import org.springframework.data.mongodb.core.query.Criteria;
	import org.springframework.data.mongodb.core.query.Query;
	import org.springframework.data.mongodb.core.query.Update;
	import org.springframework.stereotype.Component;
	
	import java.lang.reflect.ParameterizedType;
	import java.lang.reflect.Type;
	import java.util.List;
	
	/**
	 * mongoDB通用查询实现类
	 *
	 * @author z
	 * @date 2019/12/30
	 */
	@Component
	public class ServiceImpl<T> implements IService<T> {
	
	    @Autowired
	    private MongoTemplate mongoTemplate;
	
	    @Override
	    public void saveEntity(T t) {
	        mongoTemplate.save(t);
	    }
	
	    @Override
	    public long updateEntityById(T t, Long id) {
	        Type superclass = getClass().getGenericSuperclass();
	        Type supertype = ((ParameterizedType) superclass).getActualTypeArguments()[0];
	        Class<T> clazz = (Class<T>) supertype;
	        Query query = new Query(Criteria.where("id").is(id));
	        Update update= new Update();
	        //更新查询返回结果集的第一条
	        UpdateResult result = mongoTemplate.updateFirst(query, update, clazz);
	        if(result!=null) {
	            return result.getMatchedCount();
	        } else {
	            return 0;
	        }
	    }
	
	    @Override
	    public void deleteEntityById(Long id) {
	        Type superclass = getClass().getGenericSuperclass();
	        Type supertype = ((ParameterizedType) superclass).getActualTypeArguments()[0];
	        Class<T> clazz = (Class<T>) supertype;
	        Query query = new Query(Criteria.where("id").is(id));
	        mongoTemplate.remove(query, clazz);
	    }
	
	    @Override
	    public List<T> findAll() {
	        Type superclass = getClass().getGenericSuperclass();
	        Type supertype = ((ParameterizedType) superclass).getActualTypeArguments()[0];
	        Class<T> clazz = (Class<T>) supertype;
	        return mongoTemplate.findAll(clazz);
	    }
	}
	
### 1.5 使用：

Service接口 继承 IService<Entity>

	public interface UserService extends IService<MongoDBUser> 

serviceImpl，继承 ServiceImpl<MongoDBUser>

	public class UserServiceImpl<T> extends ServiceImpl<MongoDBUser> implements UserService

### 1.6 测试：

	@Test
	public void testFindUserByUserName() {
		MongoDBUser test = userService.findUserByUserName("test");
		System.out.println("test>>> " + test);
	}

	@Test
	public void testSaveUser() {
		MongoDBUser user = new MongoDBUser();
		user.setId(111L);
		user.setUserName("test");
		user.setPassWord("123");
		userService.saveEntity(user);
		MongoDBUser user2 = new MongoDBUser();
		user2.setId(222L);
		user2.setUserName("test2");
		user2.setPassWord("456");
		userService.saveEntity(user2);
		System.out.println("test>>> " + 1111);
	}

	@Test
	public void testFindAllUser() {
		List<MongoDBUser> allUser = userService.findAll();
		System.out.println("test>>> " + allUser);
	}

	@Test
	public void testFindByName() {
		MongoDBUser allUser = userService.findUserByUserName("test");
		System.out.println("test>>> " + allUser);
	}

##  2 工具
MongoDB 官方可视化界面工具：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191231101630549.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1ByZXBhcmVk,size_16,color_FFFFFF,t_70)

文档：mongodb-compass-1.20.4-win32-x64.exe
链接：http://note.youdao.com/noteshare?id=a531253dc148580135a47e0cea9aa39b&sub=2106F347C6C84EE4ACA887197EDBDDCE
