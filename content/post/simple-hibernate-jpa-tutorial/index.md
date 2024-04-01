---
title: "Simple Hibernate + JPA tutorial"
date: "2017-02-08"
categories:
  - "Coding"
tags:
  - "JPA"
  - "Java"
---

In this small tutorial, we will see how to start using Hibernate framework. In further lessons, I will show how to create one-to-one, one-to-many, and many-to-many relationships. Now we are going to create a new Maven project, add some dependencies, configure JPA and create our first entity. Let's get started.

By the way, the source code from this lesson is available on [GitHub](https://github.com/savva-k/HibernateSimpleApp).

I assume that you already have JDK, Eclipse (or whatever), Maven, and MySQL installed on your machine. Hibernate works with other databases as well. If you choose another one, you should change Hibernate dialect and drive properties in **persistence.xml.**

Creating our first Hibernate application step by step:

#### Create a table in your database

```sql
CREATE TABLE employee (
    id BIGINT SIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    surname VARCHAR(100) NOT NULL,
    added_at TIMESTAMP
);
```

#### Create a new Maven project

Simply run File → New → Project... → Choose "Maven Project", archetype "maven-archetype-quickstart", set proper groupId, artifactId and so on.

##### Add Hibernate dependencies

My pom.xml dependencies section looks like this (here I put artifacts versions right in dependency entries, but it's a good idea to use `<properties>` section.

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>4.3.5.Final</version>
    </dependency>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-entitymanager</artifactId>
        <version>4.3.5.Final</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-simple</artifactId>
        <version>1.7.5</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.0.5</version>
    </dependency>
</dependencies>
```

SLF4J is used by Hibernate to log events.

Please notice, that if you use a different database you should use different database connector.

##### Create an entity class

A JPA entity is a usual POJO annotated with JPA annotations. An entity can also be made with XML definition file. This is an entity class (with setters omitted for the sake of shortness).

```java
@Entity
@Table(name = "EMPLOYEE")
public class Employee implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "ID", nullable = false, unique = true)
    private long id;

    @Column(name = "NAME", nullable = false)
    private String name;

    @Column(name = "SURNAME", nullable = false)
    private String surname;

    @Column(name = "ADDED_AT", nullable = false)
    private Calendar addedAt;

}
```

Let's see the meaning of these annotations:

**@Entity** — specifies that the class is an entity

**@Table** — specifies the primary table for the annotated entity

**@Id** — specifies the primary key of an entity

**@GeneratedValue** — denotes a way of incrementing annotated value. If you choose another DB, you may probably want to change this strategy

**@Column** — specifies a DB column to map the field to

##### Configure persistence unit

```xml
<?xml version="1.0" encoding="UTF-8"?>

<persistence version="2.0"
    xmlns="http://java.sun.com/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/persistence
   http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd">

    <persistence-unit name="hibernate_test">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <class>com.imsavva.hibernatetestapp.entities.Employee</class>

        <properties>
            <property name="javax.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/test" />
            <property name="javax.persistence.jdbc.user" value="root" />
            <property name="javax.persistence.jdbc.password"
                value="12345" />
            <property name="javax.persistence.jdbc.driver"
                value="com.mysql.jdbc.Driver" />

            <property name="hibernate.show_sql" value="true" />
        </properties>

    </persistence-unit>
</persistence>
```

Here we set that our persistence provider is Hibernate, our only entity class is Employee and we also set database connection properties.

**Notice:** turn "hibernate.show_sql" property to false on a production server.

##### Get entity manager and persist an entity!

We are ready to try to do something with our entity. Here is a simple class with a main method that creates an entity and saves it to the database.

```java
public class HibernateTestApp {

    public static void main(String[] args) {
        EntityManagerFactory factory = Persistence.createEntityManagerFactory("hibernate_test");
        EntityManager manager = factory.createEntityManager();
        manager.getTransaction().begin();
        manager.persist(getTestEmployee());
        manager.getTransaction().commit();
        manager.close();
        factory.close();
    }

    private static Employee getTestEmployee() {
        Employee employee = new Employee();
        employee.setName("John");
        employee.setSurname("Smith");
        employee.setAddedAt(Calendar.getInstance());
        return employee;
    }
}
```

The full source code is available on [GitHub](https://github.com/savva-k/HibernateSimpleApp), there's also an Employee DAO interface, implemented using Hibernate:

```java
void save(Employee employee);
void update(Employee employee);
void delete(long id);
Employee findById(long id);
List<Employee> findByName(String name);
List<Employee> getAll();
```

Next time we'll see relationships between entities.

Thanks for reading.
