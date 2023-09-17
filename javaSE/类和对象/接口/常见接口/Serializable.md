---
aliases: 
tags: 
---
*Serializable，标记接口，标志着此类是否可以序列化。*

### 将类序列化到文件中

```java
ObjectOutputStream objectOutputStream = new ObjectOutputStream( new                                                                            FileOutputStream("D:\\text.out"));
Student student = new Student();
student.setAge(25);
student.setName("jayWei");
objectOutputStream.writeObject(student);
objectOutputStream.flush();
objectOutputStream.close();
```

### 从文件中进行反序列化

```java
ObjectInputStream objectInputStream = new ObjectInputStream(new                                                               FileInputStream("D:\\text.out"));
Student student = (Student) objectInputStream.readObject();
System.out.println("name="+student.getName());
```

### transient

*被 transient 修饰的字段无法进行序列化和反序列化，transient 只能修饰成员变量，不能修饰方法和类。*