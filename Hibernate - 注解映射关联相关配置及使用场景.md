### 1.OneToMany/ManyToOne/ManyToMany注解<br>

### 2.Cascade.Type使用<br>

### 3.JoinColumn/mappedBy属性使用<br>

------

##### 场景：

###### 1.一个老师有多个学生，一个学生有多个老师。（多对多）

###### 2.一个教室有多个学生，一个学生只会有一个教室。（一对多）

<br>
<br>

ManyToMany

<pre lang="java">
// 学生实体类
@Entity
@Table(name = "student")
public class Student {
    private Integer studentId;
    private Set<Teacher> teacherSet;
    private String name;

    public Student() {}

    public Student(String name, Set<Teacher> teacherSet) {
        this.name = name;
        this.teacherSet = teacherSet;
    }

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "student_id")
    public Integer getStudentId() {
        return studentId;
    }
    public void setStudentId(Integer studentId) {
        this.studentId = studentId;
    }

    @ManyToMany(cascade = CascadeType.MERGE, fetch = FetchType.EAGER)
    @JoinTable(name = "teacher_student",
            joinColumns = {@JoinColumn(name = "studentId", referencedColumnName = "student_id")},
            inverseJoinColumns = {@JoinColumn(name = "teacherId", referencedColumnName ="teacher_id")})
    public Set<Teacher> getTeacherSet() {
        return teacherSet;
    }

    public void setTeacherSet(Set<Teacher> teacherSet) {
        this.teacherSet = teacherSet;
    }

    @Column(name = "name", length=32)
    public String getName() {return name;}
    public void setName(String name) {this.name = name;}
}

// 老师实体类
@Entity
@Table(name = "teacher")
public class Teacher {
    private Integer teacherId;
    private Set<Student> studentSet;
    private String name;

    public Teacher() {}
    public Teacher(String name) {this.name = name;}

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "teacher_id")
    public Integer getTeacherId() {
        return teacherId;
    }

    public void setTeacherId(Integer teacherId) {
        this.teacherId = teacherId;
    }

    @ManyToMany(fetch = FetchType.LAZY, mappedBy = "teacherSet")
    public Set<Student> getStudentSet() {
        return studentSet;
    }

    public void setStudentSet(Set<Student> studentSet) {
        this.studentSet = studentSet;
    }

    @Column(name = "name", length=32)
    public String getName() {return name;}
    public void setName(String name) {this.name = name;}
}

// 测试用例
Teacher a = new Teacher("Mr.A");
Teacher b = new Teacher("Mr.B");
teacherRepository.save(a);
teacherRepository.save(b);

Set<Teacher> teacherSet = new HashSet<Teacher>();
teacherSet.add(a);
teacherSet.add(b);
studentRepository.save(new Student("Alice", teacherSet));

</pre>

测试结果：

<pre lang="shell">
mysql> select * from teacher;
+------------+------+
| teacher_id | name |
+------------+------+
|          1 | Mr.A |
|          2 | Mr.B |
+------------+------+
2 rows in set (0.00 sec)

mysql> select * from student;
+------------+-------+
| student_id | name  |
+------------+-------+
|          3 | Alice |
+------------+-------+
1 row in set (0.00 sec)

mysql> select * from teacher_student;
+-----------+-----------+
| studentId | teacherId |
+-----------+-----------+
|         3 |         1 |
|         3 |         2 |
+-----------+-----------+
2 rows in set (0.00 sec)
</pre>

此时如果直接删除老师：会抛出异常，原因是存在约束，不可直接删除老师（由于mappedBy属性的存在，老师是被控方，文章后面详细叙述），应该从学生的teacherSet中先将所有关联的老师去除后，再删除老师条目。

<pre lang="java">
com.mysql.jdbc.exceptions.jdbc4.MySQLIntegrityConstraintViolationException: Cannot delete or update a parent row: a foreign key constraint fails (`tickets`.`teacher_student`, CONSTRAINT `FKqipv863hdf6fjitog8hgeky1k` FOREIGN KEY (`teacherId`) REFERENCES `teacher` (`teacher_id`))
</pre>

修改：

<pre lang="java">
// 使得Teacher也成为主控方
@ManyToMany(cascade = CascadeType.MERGE, fetch = FetchType.LAZY)
@JoinTable(name = "teacher_student",
            joinColumns = {@JoinColumn(name = "teacherId", referencedColumnName = "teacher_id")},
            inverseJoinColumns = {@JoinColumn(name = "studentId", referencedColumnName ="student_id")})
public Set<Student> getStudentSet() {
    return studentSet;
}
</pre>

删除老师A后：可以看到相关的中间表已经被删除。

<pre lang="shell">
mysql> select * from teacher;
+------------+------+
| teacher_id | name |
+------------+------+
|          2 | Mr.B |
+------------+------+
1 row in set (0.00 sec)

mysql> select * from student;
+------------+-------+
| student_id | name  |
+------------+-------+
|          3 | Alice |
+------------+-------+
1 row in set (0.00 sec)

mysql> select * from teacher_student;
+-----------+-----------+
| teacherId | studentId |
+-----------+-----------+
|         2 |         3 |
+-----------+-----------+
1 row in set (0.00 sec)
</pre>

#### FetchType使用：

FetchType.EAGER 急加载，意思是在取出对象时，会将关联对象一齐取出并存放在关联属性里。
FetchType.Type 懒加载，在取出对象后，如果需要关联对象就要进行下一步的查询。

#### 级联属性使用：

###### 1.CascadeType.PERSIST：级联保存。

效果：保存学生的同时将其构造函数中的Set老师也保存，如上例使用会抛异常。原因是数据库中本已经有了a、b两个老师的条目，保存学生会将a、b再保存一次造成冲突。

###### 2.CascadeType.MERGE：级联更新。（常用）

效果：保存学生的同时会将其构造函数中的Set老师更新，但应保证Set中的老师都已经存在数据库，否则会报错。

###### 3.CascadeType.REMOVE：级联删除

效果：删除学生的同时会将其中的Set老师全部一齐删除。

###### 4.CascadeType.REFRESH：级联刷新

效果：保证关联对象为最新，即会进行一次查询。

###### 分别对应entityManager的persist、merge、remove、refresh方法有效。

<br>
<br>

OneToMany 以及 ManyToOne使用：

<pre lang="java">
@Entity
@Table(name = "room")
public class Room {
    private Integer roomId;
    private Set<Student> studentSet;

    public Room() {
    }

    public Room(Set<Student> studentSet) {
        this.studentSet = studentSet;
    }

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "room_id")
    public Integer getRoomId() {
        return roomId;
    }

    public void setRoomId(Integer roomId) {
        this.roomId = roomId;
    }

    @OneToMany(cascade = {CascadeType.MERGE})
    public Set<Student> getStudentSet() {
        return studentSet;
    }

    public void setStudentSet(Set<Student> studentSet) {
        this.studentSet = studentSet;
    }
}

// 测试
Set<Student> studentSet = new HashSet<Student>();
studentSet.add(s);
Room r = new Room(studentSet);
roomRepository.save(r);

</pre>

<pre lang="shell">
// 默认生成中间表 room_student

mysql> select * from room;
+---------+
| room_id |
+---------+
|       4 |
+---------+
1 row in set (0.00 sec)

mysql> select * from room_student;
+--------------+-----------------------+
| Room_room_id | studentSet_student_id |
+--------------+-----------------------+
|            4 |                     3 |
+--------------+-----------------------+
1 row in set (0.00 sec)

</pre>

如果使用@JoinColumn(name="xxx_id")

<pre>
@OneToMany(cascade = {CascadeType.MERGE})
@JoinColumn(name="room_id")
public Set<Student> getStudentSet() {
    return studentSet;
}
</pre>

可以看到，由于外键是存储在“一”的一方，hibernate自动在student表中建立了一个room_id的列。

<pre lang="shell">
mysql> select * from student;
+------------+-------+---------+
| student_id | name  | room_id |
+------------+-------+---------+
|          3 | Alice |       4 |
+------------+-------+---------+
1 row in set (0.00 sec)


mysql> select * from room;
+---------+
| room_id |
+---------+
|       4 |
+---------+
1 row in set (0.00 sec)
</pre>

如果想要由学生端操控这一关系，则可以如下设置：

<pre lang="java">
// *** 首先删除room表中的studentSet

// 在Student中添加注解
@ManyToOne(cascade = {CascadeType.MERGE})
@JoinColumn(name="room_id")
public Room getRoom() {
    return room;
}

// 测试过程代码

Room r = new Room();
roomRepository.save(r);
// 存入时还有room r（更改构造函数）
Student s = new Student("Alice", teacherSet, r);
studentRepository.save(s);
</pre>

结果：

<pre lang="shell">
mysql> select * from room;
+---------+
| room_id |
+---------+
|       3 |
+---------+
1 row in set (0.00 sec)

mysql> select * from student;
+------------+-------+---------+
| student_id | name  | room_id |
+------------+-------+---------+
|          4 | Alice |       3 |
+------------+-------+---------+
1 row in set (0.00 sec)
</pre>

#### 可知：

1. 无论是ManyToOne还是OneToMany，@JoinColum的效果都是在One的一端添加一个列名为name的外键列。
2. 如果没有JoinColumn注解，则通过中间表连接。
3. 系映射仅仅只是hibernate提供的一种方式，数据库实现都是在One的一端建立外键，并将这一层关系通过映射管理起来。如果是单向映射，则由该方独立管理该映射关系。


###### ManyToOne和OneToMany中的JoinColumn用于指定“控制方”，如果如下配置：

<pre lang="java">
// Room表中
@OneToMany(cascade = {CascadeType.MERGE})
@JoinColumn(name="room_id")
public Set<Student> getStudentSet() {
    return studentSet;
}

// Student表中
@ManyToOne(cascade = {CascadeType.MERGE})
@JoinColumn(name="room_id")
public Room getRoom() {
    return room;
}
</pre>

根据Merge的级联属性，意味着双方都会更新连接表
测试代码：

<pre lang="java">
Room r = new Room();
roomRepository.save(r);

Student s = new Student("Alice", teacherSet, r);
studentRepository.save(s);

// 以下相当于给学生s指定了一个新的教室r2
Set<Student> studentSet = new HashSet<Student>();
studentSet.add(s);
Room r2 = new Room(studentSet);
roomRepository.save(r2);
</pre>

猜测：第一次存入Student时，由于指定了一个Room r给s，因此其现在的room_id是r的；但是随后，在r2存入时，由于r2指定了其set中存在s，会更新外键列，结果：

<pre lang="shell">
mysql> select * from room;
+---------+
| room_id |
+---------+
|       3 |
|       5 |
+---------+
2 rows in set (0.00 sec)

mysql> select * from student;
+------------+-------+---------+
| student_id | name  | room_id |
+------------+-------+---------+
|          4 | Alice |       5 |
+------------+-------+---------+
1 row in set (0.00 sec)
</pre>

与预料一致。

<h6>mappedBy属性：用在被控方，用于维护关系，如下使用，那么在新建该room时，并不会更新数据库中的关联字段</h6>

<pre lang="java">
// 修改Room增加mappeBy属性
@OneToMany(mappedBy = "room")
public Set<Student> getStudentSet() {
    return studentSet;
}

// 测试代码
Room r = new Room();
roomRepository.save(r);

Student s = new Student("Alice", teacherSet, r);
studentRepository.save(s);

Set<Student> studentSet = new HashSet<Student>();
studentSet.add(s);
Room r2 = new Room(studentSet);
roomRepository.save(r2);
</pre>

结果：可以看到，存入r2的操作并不会对外键列产生影响。

<pre lang="shell">
mysql> select * from student;
+------------+-------+---------+
| student_id | name  | room_id |
+------------+-------+---------+
|          4 | Alice |       3 |
+------------+-------+---------+
1 row in set (0.00 sec)

mysql> select * from room;
+---------+
| room_id |
+---------+
|       3 |
|       5 |
+---------+
2 rows in set (0.00 sec)
</pre>

###### mappedBy可以理解为，建立双向关系于被控方，使得被控方也能够通过fetchType等方式获得关联对象。

<br>
<br>

##### 另外注意：

######维护权（主控）尽量放在多的一方，因为外键的建立在多的一方表中的，逻辑更清晰。