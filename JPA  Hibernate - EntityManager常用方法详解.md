## 1. 数据库操作顶层

###### 1.1. CrudRepository接口实现：org.springframework.data.jpa.repository.support.SimpleJpaRepository

###### 1.2. 更改数据库主要方法：save、delete.

<pre lang="java">@Transactional
public &lt; S extends T&gt; S save(S entity) {
    if(this.entityInformation.isNew(entity)) {
        this.em.persist(entity); // 调用了EntityManager的方法
        return entity;
    } else {
        return this.em.merge(entity);
    }
}
</pre>

<pre lang="java">@Transactional
public void delete(T entity) {
    Assert.notNull(entity, "The entity must not be null!");
    this.em.remove(this.em.contains(entity)?entity:this.em.merge(entity)); // 调用了EntityManager的方法
}
</pre>

###### 1.3. EntityManager是jpa访问数据库的入口，其中EntityManager再通过调用各框架实现的Session进行操作。如Hibernate实现EntityManager接口，其中的persist方法实现：

<pre lang="java">public void persist(Object entity) {
    this.checkOpen();

    try {
        this.internalGetSession().persist(entity);// 调用了代理的session（如hibernate session）的方法
    } catch (MappingException var3) {
        throw this.convert((RuntimeException)(new IllegalArgumentException(var3.getMessage())));
    } catch (RuntimeException var4) {
        throw this.convert(var4);
    }
}
</pre>

## 2. Hibernate 实体四状态转换

<img src="http://119.29.166.163/wp-content/uploads/2017/05/hibernate-entity-states.png" alt="YifengWong" />

> 瞬时态（Transient）：new新创建的对象，且尚未与Hibernate Session 关联的对象就是瞬时的。其与数据库、session等没有任何关联。数据库中通常无此记录。瞬时状态实体对象并未生成唯一标识。
> 
> 持久态（Persistent）：持久的实体存在于相关联的Session内。 Hibernate会检测到处于持久状态的对象的任何改动，任何操作都会引起对象数据（state）与数据库同步，不需要调用UPDATE相关操作。
> 
> 脱管态（Detached）：也叫游离态，与持久对象关联的Session被关闭后，对象就变为脱管的。脱管态不能直接持久化，需要重新保存。数据库中可能有、也可能没有此记录。脱管态实体对象中有生成过的唯一标识。
> 
> 删除态（Deleted）：调用Session的delete方法之后，对象就变成删除态，此时Session中仍然保存有对象的信息，对删除态对象的引用依然有效，对象可继续被修改。删除态对象如果重新关联到某个新的 Session 上（也就是执行持久化操作）， 会再次转变为持久的（在Deleted其间的改动将被持久化到数据库）。删除态下的对象在数据库中无记录。



###### 所有状态都是基于session过程的，脱管状态指从session中移除不再管理的对象，瞬时状态一般指new的对象，其余状态均被session所关联。</h6>

###### session中存在一级缓存，加上事务的管理，其中的操作并不是立即更新到数据库，便于过程的优化，减少与数据库交互次数。</h6>

## 3. EntityManager常用方法：

##### Hibernate-5中EntityManager的具体实现在org.hibernate.jpa.internal.EntityManagerImpl，其中persist等方法实现在org.hibernate.jpa.spi.AbstractEntityManagerImpl。

##### 3.1. persist：将对象交给entityManager托管，对象成为持久态，也就是，当前对象已经被session纳入管理，在session中存放着该对象，该对象的直接操作均会影响数据库内容。

<pre lang="java">@org.junit.Test
public void test1() throws Exception {
    String userId = "13512345678";
    // 获得一个新建对象，默认Gender属性为"male"
    User user = newACompleteUser(userId);
    em.persist(user);
    Log.info(user.toString());

    // 此时从entityManager中获取的对象1
    User find1 = em.find(User.class, userId);
    Log.info(find1.toString() + " test1: " + find1.getGender());

    // 直接操作对象修改，由结果可以看到会生效
    user.setGender("female");
    // 此时从entityManager中获取的对象2
    User find2 = em.find(User.class, userId);
    Log.info(find2.toString() + " test1: " + find2.getGender());
}

// 测试结果：
com.sysu.yizhu.business.entities.User@62a68bcb
com.sysu.yizhu.business.entities.User@62a68bcb test1: male
com.sysu.yizhu.business.entities.User@62a68bcb test1: female

// 三个为同一对象

</pre>

##### 3.2. merge：操作将会生成一个拷贝的实体对象放入托管中，如果是已存在id的实体，则更新内容至被托管对象中。

<pre lang="java">@org.junit.Test
public void test2() throws Exception {
    String userId = "13512345678";
    // 获得一个新建对象，默认Gender属性为"male"
    User user = newACompleteUser(userId);
    em.merge(user);
    Log.info(user.toString());

    // 此时从entityManager中获取的对象1
    User find1 = em.find(User.class, userId);
    Log.info(find1.toString() + " test2: " + find1.getGender());

    // 直接操作对象修改，没有生效
    user.setGender("female");
    // 此时从entityManager中获取的对象2
    User find2 = em.find(User.class, userId);
    Log.info(find2.toString() + " test2: " + find2.getGender());

    // 再次merge之后，将user内容同步到了托管的对象中
    em.merge(user);
    User find3 = em.find(User.class, userId);
    Log.info(find3.toString() + " test2: " + find3.getGender());

    // 可以调用user = em.merge(user)获取merge操作托管的实体对象
}

// 测试结果：
com.sysu.yizhu.business.entities.User@46dcbeab
com.sysu.yizhu.business.entities.User@60a7e509 test2: male
com.sysu.yizhu.business.entities.User@60a7e509 test2: male
com.sysu.yizhu.business.entities.User@60a7e509 test2: female

// 后三个find和第一个user不是同一对象
</pre>

##### 3.3. flush：缓存中的更改写入数据库，会立即执行SQL语句

<pre lang="java">@Transactional
public void test2() throws Exception {
    String userId = "13512345678";
    // 获得一个新建对象
    User user = newACompleteUser(userId);
    em.persist(user);
    user.setGender("male");
    user.setGender("female");
    user.setGender("male");
    em.flush();
}
// 执行的SQl，仅有一句插入，后面的所有更新都被合并
Hibernate: insert into user (birth_date, gender, head_img_url, location, name, password, user_id) values (?, ?, ?, ?, ?, ?, ?)


@Transactional
public void test2() throws Exception {
    String userId = "13512345678";
    // 获得一个新建对象
    User user = newACompleteUser(userId);
    em.persist(user);
    user.setGender("male");
    user.setGender("xxx");
    user.setGender("female");
    em.flush();
}

// 执行的SQl，包括了一句插入和一句更新
Hibernate: insert into user (birth_date, gender, head_img_url, location, name, password, user_id) values (?, ?, ?, ?, ?, ?, ?)
Hibernate: update user set birth_date=?, gender=?, head_img_url=?, location=?, name=?, password=? where user_id=?


</pre>

##### 3.4. detach：脱管实体对象，如果未进行过flush等操作（或事务结束之前），那么更改不会写入数据库。

<pre lang="java">@Transactional
public void test2() throws Exception {
    String userId = "13512345678";
    // 获得一个新建对象
    User user = newACompleteUser(userId);
    // 对象被管理
    em.persist(user);
    // 改动实时修改在EntityManager中
    user.setGender("male");
    // 仓库调用的也是EntityManager的方法，因此此时获取的是在缓存中的对象，没有执行SQL语句
    Log.info(userRepo.findOne(userId).getGender());
    // 对象脱管，所有更改都没有写入数据库，数据库无记录
    em.detach(user);
    // 修改仅仅只是修改对象内容
    user.setGender("female");
    // 此时，由于EntityManager中没有userId对应的对象，因此会发起SQL语句，无记录，返回空对象。
    Assert.assertEquals(userRepo.findOne(userId), null);
}

// 运行结果：
male
Hibernate: select user0_.user_id as user_id1_4_0_, user0_.birth_date as birth_da2_4_0_, user0_.gender as gender3_4_0_, user0_.head_img_url as head_img4_4_0_, user0_.location as location5_4_0_, user0_.name as name6_4_0_, user0_.password as password7_4_0_ from user user0_ where user0_.user_id=?

</pre>

##### 3.5. remove：删除实体，实体状态从持久态更改为删除态

<pre lang="java">@Transactional
public void test2() throws Exception {
    String userId = "13512345678";
    // 获得一个新建对象
    User user = newACompleteUser(userId);
    em.persist(user);
    user.setGender("male");
    Log.info(userRepo.findOne(userId).getGender());
    // 删除，如果执行flush等操作，会执行数据库语句
    em.remove(user);
    user.setGender("female");
    Assert.assertEquals(userRepo.findOne(userId), null);

    em.flush();
}

// 运行结果
male
Hibernate: insert into user (birth_date, gender, head_img_url, location, name, password, user_id) values (?, ?, ?, ?, ?, ?, ?)
Hibernate: delete from user where user_id=?

</pre>

##### 3.6. refresh：更新实体对象与数据库同步，如果数据库无记录会抛出异常EntityNotFoundException；如果对象未被管理，会抛出异常IllegalArgumentException。

<pre lang="java" class="">@Transactional
public void test2() throws Exception {
    String userId = "13512345678";
    // 获得一个新建对象
    User user = newACompleteUser(userId);
    em.persist(user);
    em.flush();

    user.setGender("female");
    Log.info(user.getGender());

    // 从数据库查询出其数据库状态并更新到实体中
    em.refresh(user);
    Log.info(user.getGender());
}

// 运行结果：
Hibernate: insert into user (birth_date, gender, head_img_url, location, name, password, user_id) values (?, ?, ?, ?, ?, ?, ?)
female
Hibernate: select user0_.user_id as user_id1_4_0_, user0_.birth_date as birth_da2_4_0_, user0_.gender as gender3_4_0_, user0_.head_img_url as head_img4_4_0_, user0_.location as location5_4_0_, user0_.name as name6_4_0_, user0_.password as password7_4_0_ from user user0_ where user0_.user_id=?
male


</pre>

## 参考

<p><a href="http://blog.csdn.net/abguorui0928/article/details/33779679" title="Hibernate实体对象四大状态">http://blog.csdn.net/abguorui0928/article/details/33779679</a>