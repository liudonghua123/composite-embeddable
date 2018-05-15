# Composite key handling, using @EmbeddedId annotation in Spring boot java

To get brief understanding about how spring data JPA process composite key
relationship, read my previous blog.

From the two methods I've explained in that blog I'm gonna explain @EmbeddedId
annotation based implementation in this blog.

I'm gonna use the same one to many relationship among employee and department in
this blog as well.

![](https://cdn-images-1.medium.com/max/800/1*k4imuDv2S_YFl1X8t9hB2Q.png)
<span class="figcaption_hack">Department — Employee</span>

This time I'm gonna use Department as my example entity. So first clone my
github repository relevant to this blog.

[https://github.com/bhagyaj/composite-embeddable.git](https://github.com/bhagyaj/composite-embeddable.git)

I've selected Department entity as my entry point. Let's go through below code
extraction from Department model class.

```java
@Entity
@Table(name = "Department")
public class Department {
    @EmbeddedId
    DepartmentId departmentId;
    @Column(name = "EmpNumber")
    private Integer empNumber;
    @OneToMany(fetch = FetchType.
, cascade = {CascadeType.
}, mappedBy = "department")
    private Set<Employee> employees;
```

The main difference from @Idclass annotation is, there's no @Id annotation to
represent key fields in this model class. Instead we use Id class object to
represent key fields. In this method Id class introduction is done by
@EmbeddedId annotation.

As I explained in previous blog mapping details are added in this entity as
well. This time our Id class is lot more different than previous DepartmentId
class.

```java
@Data
@Embeddable
public class DepartmentId implements Serializable {
    @Column(name = "DepartmentName")
    private String name;
    @Column(name = "DepartmentLocation")
    private String location;
}
```

This class uses @Embeddable annotation to express this class as an Id class,
which embedded in an entity. Other variables are key fields in the class. In
this time our child class (Employee) has taken its common fields from
DepartmentId class. So no repetitions gonna happen. After adding DepartmentId,
EmployeeId class is below.

```java
@Data
@Embeddable
public class EmployeeId implements Serializable {
    DepartmentId departmentId;
    @Column(name = "Name")
    private String name;
}
```

Let's see ManyToOne mapping from employee end. That is exctly same mapping which
we use with @Idclass annotation.

```java
@Entity
@Table(name = "Employee")
public class Employee {
    @EmbeddedId
    EmployeeId employeeId;
    @Column(name = "Designation")
    private String designation;
    @ManyToOne
    @JoinColumns({
            @JoinColumn(name = "DepartmentName", referencedColumnName = "DepartmentName", insertable = false, updatable = false),
            @JoinColumn(name = "DepartmentLocation", referencedColumnName = "DepartmentLocation", insertable = false, updatable = false),
    })
    @JsonIgnore
    private Department department;
    @Column(name = "Salary")
    private Double salary;
```

To be more clearer, I've thought of comparing two select queries from both
implementations.

With `@IdClass` you write:

    select e.name from Employee e

and with `@EmbeddedId` you have to write:

    select e.employeeId.name from Employee e

With embeddable method we don't need to pass each and every key field, instead
we can pass Id class object to get required output. Below is the method extract
which I used to retrieve department information from Department entity using
DepartmentId.

```java
public Department getDepartment(String rnD, String kalutara) {
    DepartmentId departmentId = new DepartmentId();
    departmentId.setName("RnD");
    departmentId.setLocation("Kalutara");
    return departmentRepository.findByDepartmentId(departmentId);
}
```

I'm only passing departmentId object to retrieve Department. That will select
relevant employee from the table. My resulted output is below.

**Department by Id**

```json
{
 "departmentId": {
 "name": "RnD",
 "location": "Kalutara"
 },
 "empNumber": 5,
 "employees": [
 {
 "employeeId": {
 "departmentId": {
 "name": "RnD",
 "location": "Kalutara"
 },
 "name": "Bhagya"
 },
 "designation": "Architect",
 "salary": 500000
 },

{
 "employeeId": {
 "departmentId": {
 "name": "RnD",
 "location": "Kalutara"
 },
 "name": "Singam"
 },
 "designation": "Engineer",
 "salary": 50000
 }
 ]
}
```

From the above output we can see that departmentId object is come within the
employeeId object representing employee belongs to department.

Same way by passing Employee Id Employee data will return below output.

```json
{
 "employeeId": {
 "departmentId": {
 "name": "RnD",
 "location": "Kalutara"
 },
 "name": "Bhagya"
 },
 "designation": "Architect",
 "salary": 500000
}
```

**Advantages over @Idclass annotation**

By using @Idclass you cannot access whole composite key at once. But in here we
can perform operations for whole composite key using Embeddable class.

Most people argue that it is not possible to use @generatedValue with embeddable
Id. They highlight is as a drawback of this method. But it is argumentative
using auto incremented value within composite key since it's by default unique.
So at the end of the day there are different pros and cons for @Idclass and
@EmbeddedId annotations. You should be wise enough to choose what is right for
your implementation. Hope my two blogs will help for your decision.

* [Java](https://medium.com/tag/java?source=post)
* [Spring Boot](https://medium.com/tag/spring-boot?source=post)
* [Spring Data](https://medium.com/tag/spring-data?source=post)
* [Jpa](https://medium.com/tag/jpa?source=post)
* [Embedded](https://medium.com/tag/embedded?source=post)

From a quick cheer to a standing ovation, clap to show how much you enjoyed this
story.

### [Bhagya Rupasinghe](https://medium.com/@bhagyajayashani)
