Navigation properties in Entity Framework Core (EF Core) are used to define relationships between entities and allow easy traversal of these relationships in both directions. They enable you to access related data without needing to manually write SQL joins or queries. EF Core manages relationships and foreign keys based on these navigation properties and foreign key properties.

Here’s a detailed explanation of how navigation properties are used:

### Types of Navigation Properties

1. **Reference Navigation Properties**:
   - These represent single objects or entities related to the current entity. 
   - Example: In the `Enrollment` class, the `Course` and `Student` properties are reference navigation properties because each `Enrollment` is associated with a single `Course` and a single `Student`.

   ```csharp
   public class Enrollment
   {
       public int EnrollmentID { get; set; }
       public int CourseID { get; set; }
       public int StudentID { get; set; }
       public Grade? Grade { get; set; }

       public Course Course { get; set; }  // Reference navigation property
       public Student Student { get; set; }  // Reference navigation property
   }
   ```

2. **Collection Navigation Properties**:
   - These represent collections of related entities (many related entities).
   - Example: In the `Course` class, the `Enrollments` property is a collection navigation property because a single course can have many enrollments.

   ```csharp
   public class Course
   {
       public int CourseID { get; set; }
       public string Title { get; set; }
       public int Credits { get; set; }

       public ICollection<Enrollment> Enrollments { get; set; }  // Collection navigation property
   }
   ```

### How Navigation Properties Are Used

Navigation properties let you load related data, and EF Core can automatically manage the relationships between the entities through foreign keys. Here are some examples of how navigation properties are used in practice:

#### 1. **Loading Related Data**

You can use navigation properties to load related entities. EF Core offers various ways to load related data, including **eager loading**, **lazy loading**, and **explicit loading**.

- **Eager Loading**:
  - You load related entities at the same time as the main entity using the `Include` method.

    ```csharp
    var courses = context.Courses
                         .Include(c => c.Enrollments)
                         .ThenInclude(e => e.Student)
                         .ToList();
    ```

    In this example:
    - The `Include(c => c.Enrollments)` ensures that when retrieving `Course` entities, the related `Enrollment` entities are also retrieved.
    - The `ThenInclude(e => e.Student)` further ensures that each `Enrollment`'s `Student` is loaded.

- **Lazy Loading**:
  - Related data is loaded automatically when you access the navigation property, but this requires configuration.
    - You need to install and configure the lazy-loading package (`Microsoft.EntityFrameworkCore.Proxies`).
  
    ```csharp
    var course = context.Courses.Find(1);
    var enrollments = course.Enrollments;  // Lazy loads the enrollments when accessed
    ```

    In this case, the `Enrollments` property is loaded when accessed, rather than when the `Course` entity is initially retrieved.

- **Explicit Loading**:
  - You can explicitly load related data when needed using the `Load` method.

    ```csharp
    var course = context.Courses.Find(1);
    context.Entry(course).Collection(c => c.Enrollments).Load();
    ```

    Here, the `Enrollments` navigation property for the `course` entity is explicitly loaded.

#### 2. **Navigating Relationships**

Once related data is loaded (either through eager, lazy, or explicit loading), you can use the navigation properties to access related entities.

For example:
- You can access the course related to a particular enrollment using the `Course` navigation property:
  
  ```csharp
  var enrollment = context.Enrollments.Find(1);
  var courseTitle = enrollment.Course.Title;
  ```

  In this case, you’re navigating from `Enrollment` to `Course` to retrieve the `Title` of the course in which the student is enrolled.

- Similarly, you can access all the students enrolled in a course by navigating the `Enrollments` collection:

  ```csharp
  var course = context.Courses.Find(1);
  var students = course.Enrollments.Select(e => e.Student).ToList();
  ```

  This retrieves all the students enrolled in a particular course.

#### 3. **Modifying Related Data**

You can also use navigation properties when modifying relationships between entities.

- **Adding an Enrollment to a Course**:
  ```csharp
  var student = context.Students.Find(1);
  var course = context.Courses.Find(2);
  var enrollment = new Enrollment
  {
      Student = student,
      Course = course,
      Grade = Grade.A
  };
  context.Enrollments.Add(enrollment);
  context.SaveChanges();
  ```

  This example shows how to create a new `Enrollment` by linking a `Student` and a `Course` using navigation properties.

- **Updating a Relationship**:
  If you want to change which student is enrolled in a course, you can modify the navigation properties:
  ```csharp
  var enrollment = context.Enrollments.Find(1);
  var newStudent = context.Students.Find(2);
  enrollment.Student = newStudent;  // Update the related entity
  context.SaveChanges();
  ```

#### 4. **Cascade Delete Behavior**

Navigation properties can also influence delete behaviors. For example, if a `Course` has related `Enrollments`, and you delete the `Course`, EF Core can be configured to automatically delete related `Enrollments` through **cascade delete**.

This behavior can be controlled by configuring relationships in your `DbContext` using the Fluent API or data annotations.

### Summary of Usage

- **Loading related data**: Navigation properties enable you to load related entities using different loading strategies (eager, lazy, or explicit).
- **Navigating relationships**: You can traverse relationships between entities through navigation properties, accessing related data easily.
- **Modifying relationships**: Navigation properties allow you to add, update, and delete relationships between entities.
- **Cascade delete**: Navigation properties play a role in cascade delete behaviors, where deleting a parent entity can delete related child entities automatically.

Navigation properties provide a powerful way to work with related data without manually writing complex SQL queries. They allow for smooth and efficient access and manipulation of related entities in your data model.
