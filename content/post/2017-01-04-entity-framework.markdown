---
layout:     post
title:      "Entity Framework handbook"
subtitle:   ""
date:       2017-01-04 12:00:00
author:     "Shengwen"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - Entity Framework
    - ASP.NET
---

### Catalog
1. [Handle model change](#cat0)
2. [Seed database with test data](#cat1)
3. [Use stored procedures for inserting/editing/deleting in EF](#cat2)
4. [Use stored procedure for inserting/editing/deleting in code first approach](#cat3)
5. [Implement the repository class](#cat4)
6. [Entity splitting with code first approach](#cat5)
7. [Table splitting](#cat6)
8. [Conditional mapping](#cat7)
9. [Self-referencing association](#cat8)
10. [Table Per Hierarchy (TPH) Inheritance](#cat9)
11. [Table Per Type (TPT) Inheritance](#cat10)
12. [Many-to-Many relationship in EF](#cat11)
13. [Entity for Bridge Table in many to many relationship](#cat12)

---
<p id="cat0"/>
### Handle model change:

In Global.asax.cs, Application_Start():
```csharp
Database.SetInitializer(new DropCreateDatabaseIfModelChanges)
```

<p id="cat1"/>
### Seed database with test data:

1. Create a ProductDBContextSeeder.cs class:
```csharp
public class ProductDBContextSeeder: DropCreateDatabaseIfModelChanges<ProductDBContext>
{
    protected override void Seed(ProductDBContext context)
    {
        Category category1 = new Category()
        {
            Name = "Flower",
            Products = new List<Product>()
            {
                new Product()
                {
                    Name = "Rose",
                    Price = 10,
                },
                new Product(){…},
                new Product(){…}
	        };
            Category category2 = new Category(){…};
            Category category3 = new Category(){…};
            context.Categories.Add(category1);
            context.Categories.Add(category2);
            context.Categories.Add(category3);
            base.Seed(context);
        }
    }
}
```
2. Change the Application_start() code:
```csharp
Database.SetInitializer(new ProductDBContextSeeder());
```

<p id="cat2"/>
### Use stored procedures for inserting/editing/deleting in EF

By default, EF will use auto-generated SQL statements for insert/edit/delete, to use our stored procedure instead:
1. Right click on "Product" entity on "ProductModel.edmx" and select "Stored Procedure Mapping" option from the context menu.
2. In the "Mapping Details" windows specify the Insert, Update and Delete stored procedures that you want to use with "Product" entity

<p id="cat3"/>
### Use stored procedure for inserting/editing/deleting in code first approach

In EmployeeDBContext.cs, override OnModelCreating():
```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Entity<Employee>().MapToStoredProcedures();
    base.OnModelCreating(modelBuilder);
}
```

Then complete EmployeeRepository and add connection string in web.config. Stored procedures for inserting/editing/deleting will be auto generated.

Change the default name of these SPs and column names in table:
```csharp
modelBuilder.Entity<Employee>().MapToStoredProcedures
    (p => p.Insert(i => i.HasName("InsertEmployee")
    .Parameter(n => n.Name, "EmployeeName")
    .Parameter(n => n.Gender, "EmployeeGender")
    .Parameter(n => n.Salary, "EmployeeSalary"))
    .Update(u => u.HasName("UpdateEmployee")
    .Parameter(n => n.ID, "EmployeeID")
    .Parameter(n => n.Name, "EmployeeName")
    .Parameter(n => n.Gender, "EmployeeGender")
    .Parameter(n => n.Salary, "EmployeeSalary"))
    .Delete(d => d.HasName("DeleteEmployee")
    .Parameter(n => n.ID, "EmployeeID"))
        );

```

<p id="cat4"/>
### Implement the repository class:

```csharp
public class EmployeeRepository{
    EmployeeDBContext employeeDBContext = new EmployeeDBContext();
    public List<Employee> GetEmployees()
    {
        return employeeDBContext.Employees.ToList();
    }
    public void InsertEmployee(Employee employee)
    {
        employeeDBContext.Employees.Add(employee);
        employeeDBContext.SaveChanges();
    }
    public void UpdateEmployee(Employee employee)
    {
        Employee employeeToUpdate = employeeDBContext.Employees
            .SingleOrDefault(x => x.EmployeeId == employee.EmployeeId);
        employeeToUpdate.EmployeeId = employee.EmployeeId;
        employeeToUpdate.FirstName = employee.FirstName;
        employeeToUpdate.Gender = employee.Gender;
        employeeToUpdate.Email = employee.Email;
        employeeToUpdate.Mobile = employee.Mobile;
        employeeToUpdate.Landline = employee.Landline;
        employeeDBContext.SaveChanges();
    }
    public void DeleteEmployee(Employee employee)
    {
        Employee employeeToDelete = employeeDBContext.Employees
            .SingleOrDefault(x => x.EmployeeId == employee.EmployeeId);
        employeeDBContext.Employees.Remove(employeeToDelete);
        employeeDBContext.SaveChanges();
    }  
}
```

<p id="cat5"/>
### Entity splitting with code first approach

We want an entity Employee to be stored in 2 DB tables, Employees and EmployeeContactDetails. In EmployeeDBContext.cs, override OnModelCreating():
```csharp
public class EmployeeDBContext : DbContext
{
    public DbSet<Employee> Employees {get; set;}
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Employee>()
        // Specify properties to map to Employees table
        .Map(map =>
        {
            map.Properties(p => new
            {
                p.EmployeeId,
                p.FirstName,
                p.LastName,
                p.Gender
            });
            map.ToTable("Employees");
        })
        // Specify properties to map to EmployeeContactDetails table
        .Map(map =>
        {
            map.Properties(p => new
            {
                p.EmployeeId,
                p.Email,
                p.Mobile,
                p.Landline
            });
            map.ToTable("EmployeeContactDetails");
            });
        base.OnModelCreating(modelBuilder);
    }
}
```

<p id="cat6"/>
### Table splitting

Table Splitting is useful when you want to delay the loading of some properties with large data when using lazy loading.
After creating the entity of table:
1. add a new entity with key property;
2. Cut properties from old entity and paste them in new entity;
3. Add 1 to 1 association;
4. Go to properties, add referential constraint;
5. Table mapping

Code first approach:
1. Create DAO classes: Employee with Id, Name and Gender, and EmployeeContactDetail with Id, Email, Address
2. Create EmployeeDBContext.cs as below:
```csharp
public class EmployeeDBContext : DbContext
{
    public DbSet<Employee> Employees { get; set; }
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Employee>()
            .HasKey(pk => pk.EmployeeID)
            .ToTable("Employees");
        modelBuilder.Entity<EmployeeContactDetail>()
            .HasKey(pk => pk.EmployeeID)
            .ToTable("Employees");
        modelBuilder.Entity<Employee>()
            .HasRequired(p => p.EmployeeContactDetail)
            .WithRequiredPrincipal(c => c.Employee);
    }
}
```

<p id="cat7"/>
### Conditional mapping

Say a table Employee has a column IsTerminated, in our web app we only need employees who are not terminated:
Right click entity -> Table Mapping -> Add condition : When Is Terminated = false  -> Delete the original mapping of IsTerminated

Code first approach:
override OnModelCreating() in EmployeeDBContext:
```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Entity<Employee>()
        .Map(m => m.Requires("IsTerminated")
        .HasValue(false))
        .Ignore(m => m.IsTerminated);
    base.OnModelCreating(modelBuilder);
}
```

<p id="cat8"/>
### Self-referencing association

Rename the auto generated navigation properties to meaningful names.
Code first approach:
```csharp
public class Employee
{
    // Scalar properties
    public int EmployeeID { get; set; }
    public string EmployeeName { get; set; }
    public int ManagerID { get; set; }
    // Navigation property
    public Employee Manager { get; set; }
}
```
Override OnModelCreating() in EmployeeDBContext:
```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Entity<Employee>()
        .HasOptional(e => e.Manager)
        .WithMany()
        .HasForeignKey(m => m.ManagerID);
    base.OnModelCreating(modelBuilder);
}
```

<p id="cat9"/>
### Table Per Hierarchy (TPH) Inheritance

Say in our employee table we have two kinds of Employees: Permanent and Contract. Permanent has AnuualSalary column, contract has HourlyPay and HoursWorked columns. Both kinds has some common columns and a Discriminator column to indicate PermanentEmployee or ContractEmployee.

DB first approach:
1. Add two entities PermanentEmployee and ContractEmployee with Based type Employee
2. From Employee entity, cut AnuualSalary, HourlyPay and HoursWorked properties and paste to corresponding entities
3. In Table mapping, map entities to Employees table, and add condition to both entity, delete the origin mapping of Discriminator
4. Use OfType in code.
```csharp
switch(RadioButtonList1.SelectedValue)
{
    case "Permanent":
        GridView1.DataSource = employeeDBContext.Employees
            .OfType<PermanentEmployee>().ToList();
        GridView1.DataBind();
        break;
    case "Contract":
        GridView1.DataSource = employeeDBContext.Employees
            .OfType<ContractEmployee>().ToList();
        GridView1.DataBind();
        break;
    default:
        GridView1.DataSource = ConvertEmployeesForDisplay(
            employeeDBContext.Employees.ToList());
        GridView1.DataBind();
        break;
}
```

Code first approach: Create Employee class and its subclasses PermanentEmployee and ConrtractEmployee

<p id="cat10"/>
### Table Per Type (TPT) Inheritance

Downside of TPH: null values. To avoid that, use TPT. 3 tables, Employee has common columns, PermanentEmployee has only Employeeid(FK) and AnnualSalary, ContractEmployee has only Employeeid(FK), HourlyPay and HoursWorked.

Create entities, delete the association and add inheritance relationship, select Employee as Base Entity, then delete the EmployeeId property from PermanentEmployee and ContractEmployee entities since it will be inherited from base Employee entity.

Code first approach: same classes as in TPH code first, by default EF will use TPH and create one table. We need to use [Table("TableName")] attributes on 3 classes

<p id="cat11"/>
### Many-to-Many relationship in EF
Course and Student tables and a bridge table. Code first approach:
1. Create Student and Course classes
```csharp
public class Student{
    public int StudentID {get; set;}
    public string StudentName {get; set;}
    Public Icollection Courses{get; set;}
}
public class Courses{
    public int CourseID {get; set;}
    public string CourseName {get; set;}
    Public Icollection Courses{get; set;}
}
```
2. Create StudentDBContext.cs:
```csharp
public class EmployeeDBContext : DbContext
{
    public DbSet<Course> Courses { get; set; }
    public DbSet<Student> Students { get; set; }
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Student>()
            .HasMany(t => t.Courses)
            .WithMany(t => t.Students)
            .Map(m =>
            {
                m.ToTable("StudentCourses");
                m.MapLeftKey("StudentID");
                m.MapRightKey("CourseID");
            });
        base.OnModelCreating(modelBuilder);
    }
}
```
3. Show in GridView:
```csharp
GridView1.DataSource = (from student in employeeDBContext.Students
                        from c in student.Courses
                        select new
                        {
                            StudentName = student.StudentName,
                            CourseName = c.CourseName
                        }).ToList();
GridView1.DataBind();
```

<p id="cat12"/>
### Entity for Bridge Table in many to many relationship

When an entity will and will not be created by the entity framework for the bridge table in a many-to-many relationship?
An entity for the bridge table is NOT created when the bridge table has only the foreign keys. On the other if the bridge table has any other columns apart from the foreign key columns then a bridge table is created.

Now the bridge table (StudentCourse) has an extra DateEnrolled column. EF will create one more entity StudentCourse.
```csharp
GridView1.DataSource = (from student in employeeDBContext.Students
                        from studentCourse in student.StudentCourses
                        select new
                        {
                            StudentName = student.StudentName,
                            CourseName = studentCourse.Course.CourseName,
                            EnrolledDate = studentCourse.EnrolledDate
                        }).ToList();
// The above query can also be written as shown below
//GridView1.DataSource = (from course in employeeDBContext.Courses
//                        from studentCourse in course.StudentCourses
//                        select new
//                        {
//                            StudentName = studentCourse.Student.StudentName,
//                            CourseName = course.CourseName,
//                            EnrolledDate = studentCourse.EnrolledDate
//                        }).ToList();
```

Code first approach: 
```csharp
public class Course
{
    public int CourseID { get; set; }
    public string CourseName { get; set; }
    public IList<StudentCourse> StudentCourses { get; set; }
}
public class Student
{
    public int StudentID { get; set; } 
    public string StudentName { get; set; }
    public IList<StudentCourse> StudentCourses { get; set; }
}
public class StudentCourse
{
    public Course Course { get; set; }
    public Student Student { get; set; }
    [Key, Column(Order = 1)]
    public int StudentID { get; set; }
    [Key, Column(Order = 2)]
    public int CourseID { get; set; }
    public DateTime EnrolledDate { get; set; }
}
```



Reference: [kudvenkat's youtube playlist](https://www.youtube.com/playlist?list=PL6n9fhu94yhUPBSX-E2aJCnCR3-_6zBZx)

