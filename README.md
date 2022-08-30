# Inner Classes

## Learning Goals

- Explain inner classes
- Use inner classes in Java

## Introduction

All the classes we have seen so far have been publicly defined classes and
exist in their own files. These class definitions are accessible to anyone
and objects of their type can be instantiated by anyone.

In this lesson, we are going to be looking at nested classes. A **nested class**
is a class that is declared within another class. There are two types of
nested classes: static nested classes and non-static nested classes, which can
also be referred to as **inner classes**. For this lesson, we will only concern
ourselves about inner classes.

## Inner Class Walkthrough

Sometimes we want to have a specific set of methods that is only needed by one
other class. This is where **inner classes** can be useful. Inner classes have
access to all of its outer class' methods and variables; however, the outer
class does will not be able to invoke one of the inner class' methods directly
without instantiating it first. Let's look at an example here:

```java
public class Outer {
    
    // Inner class
    private class Inner {
        public void display() {
            System.out.println("In an inner class method!");
        }
    }
    
    public void showInner() {
        Inner inner = new Inner();
        inner.display();
    }
}
```

In this very simplistic example, we can see that the `Inner` class is enclosed
by the `Outer` class. The `Inner` class has one method and the `Outer` class
can call that method after instantiating an instance of the `Inner` class. Also
notice how the inner class can have a `private` access modifier. Most of the
time, with inner classes, they will be used for relationships that are
exclusive to the outer class in such a way that the inner class could not even
exist if it weren't for the outer class. But we also could have attached any of
the access modifiers to the inner class if we wanted too instead.

## Purpose of Inner Classes

So by now, we might be thinking "what is the point of having an inner class?"
Like we mentioned above, most of the time inner classes are exclusive to the
outer classes that encloses them. They might have a "has-a" relationship, like
we discussed when talking about composition earlier. Let's look at one of those
relationships to further explain the purpose of inner classes:

```java
public class GroceryStore {
   String storeName;
   int numberOfDepartments;
   Department[] departments;

   public GroceryStore(String storeName, int numberOfDepartments) {
      this.storeName = storeName;
      this.numberOfDepartments = numberOfDepartments;
      departments = new Department[numberOfDepartments];
   }

   public String getStoreName() {
      return storeName;
   }

   public void setStoreName(String storeName) {
      this.storeName = storeName;
   }

   public int getNumberOfDepartments() {
      return numberOfDepartments;
   }

   class Department {
      String departmentName;
      int numberOfEmployees;

      public Department(String departmentName) {
         this.departmentName = departmentName;
         this.numberOfEmployees = 0;
      }

      public String getDepartmentName() {
         return departmentName;
      }

      public void setDepartmentName(String departmentName) {
         this.departmentName = departmentName;
      }

      public int getNumberOfEmployees() {
         return numberOfEmployees;
      }

      public void setNumberOfEmployees(int numberOfEmployees) {
         this.numberOfEmployees = numberOfEmployees;
      }
   }
}
```

In the above example, we have a `GroceryStore` class. In a grocery store, we can
find different departments: the deli, the produce department, the meat
department, the seafood department, etc. If the grocery store did not exist,
then neither would the various departments that make up the grocery store. It
probably wouldn't make sense to have a produce department without a store. So
instead of creating a separate class outside the `GroceryStore` class, we could
just have the `GroceryStore` class encompass the `Department` class.

Inner classes allow us to logically group classes together in one place. They
help optimize code by writing less duplicate code along with helping
make the code more readable and maintainable. Inner classes also provide a
security mechanism in Java in that the inner class can be `private`, making it
inaccessible to other classes except for the outer class that encloses it.
