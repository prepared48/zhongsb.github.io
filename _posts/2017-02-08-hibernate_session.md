---
layout: post
title: 持久化的三种状态
category: java
tags: java
comments: true
---

持久化的三种状态





  1、瞬时状态：对象已经创建，但是数据库中没有。  
  
    使用new创建的对象不是立刻就持久的。它们的状态是瞬时的，也就是说它们没有任何跟数据库表相
    关联的行为，只要应用不再引用这些对象（不再被任何其他对象所引用），它们的状态将会消失，并
    由垃圾回收机制回收。  
  2、持久状态：数据库中有，并且被session管理。
  
    持久实例是任何具有数据库标识的实例。它有持久化管理器session统一管理，持久实例是在事务中
    进行操作的————它们的状态在事务结束时同数据库进行同步。当事务提交时，通过执行SQL的INSERT
    、UPDATE和DELETE语句把内存中的状态同步到数据库中。  
  3、离线状态：数据库中有，并且没有被session管理。  
  
    session关闭之后，持久化对象就变为离线对象。离线表示这个对象不能再与数据库保持同步，它们
    不再受hibernate管理。    
    
代码说明：  

    public void testSave1(){
    	Session session = null;
	Transaction tx = null;
	try {
		session = HibernateUtils.getSession();
		tx = session.beginTransaction();
		//1、Transient 瞬时状态
		User user = new User();
		user.setName("zhangsan");
		user.setPassword("123");
		user.setCreateTime(new Date());
		user.setExpireTime(new Date());
		//2、Persistent持久状态
		session.save(user);
		user.setName("lisi");
		session.update(user);
		tx.commit();
	} catch (HibernateException e) {
		e.printStackTrace();
		//回滚
		if(tx != null){
			tx.rollback();
		}
	}finally{
		//关闭session
		HibernateUtils.closeSession(session);
	}
	//3、detached分离状态
	try {
		session = HibernateUtils.getSession();
		session.beginTransaction();
		//将detached状态的对象重新纳入session管理
		//此时将变为persistent状态的对象
		//persistent状态的对象，在清理缓存是会和数据库同步
		session.update(user);
		session.getTransaction().commit();
	}catch(Exception e) {
		e.printStackTrace();
		session.getTransaction().rollback();
	}finally {
		HibernateUtils.closeSession(session);
	}
		
	}
