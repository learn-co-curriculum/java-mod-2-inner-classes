# Inner Classes

## Learning Goals

- Explain inner classes
- Use inner classes in Java

## Introduction

All the classes we have seen so far have been publicly defined classes that have
existing in their own files. These class definitions are accessible to anyone
and objects of their type can be instantiated by anyone.

Since classes are data structures that help us group data and behaviors together
in ways that are useful for the problems we are trying to solve with our
programs, it's easy to imagine that we might sometimes have scenarios where we
do not actually want our classes to be visible and available to anyone outside
our specific program. In most cases, that is because the "secondary" classes are
useful to the "primary" classes, but we want to keep them "internal" because we
want to be able to change their implementation and behavior without having to
change how other classes interface with our "primary" class. The concept of
"hiding implementation details" is something we will cover in more detail in our
section about design principles.

## Inner Class Walkthrough

Let's get back to our student grade example. For this section, we will think
about needing to support grading on a curve, so we will start with a list of
students with a point grade out of 100, and we will look for ways to translate
this point grade into a letter grade.

As before, we will use a `HashMap` to store our list of students and their
grades:

```java
HashMap<String, String> studentGrades = new HashMap<String, String>();
studentGrades.put("Jamaal", "93");
studentGrades.put("Jennifer", "87");
studentGrades.put("Jules", "59");
```

The basic logic for translating point grades to letter grades goes something
like this - note that we're leaving +/- grades out here for simplicity, but it
would be easy to add them in once we have our basic structure in place.

1. If the point grade is less than 60, the letter grade should be "F"
2. If the point grade is less than 70, the letter grade should be "D"
3. ...
4. If the point grade is lower than 90, the letter grade should be "B"
5. Otherwise, the letter grade should be "A"

Note that the order of comparison is important here because any grade that is
less than 60 is also less than 70, so if we compared a point grade of 55 to 70
before we compared it to 60, we might think it's deserving of a "D" when it's
really only deserving of an "F", at least based on the rules above.

Here is code that returns the right values based on these rules:

```java
if (numberGrade < 60) {
    return "F";
}
if (numberGrade < 70) {
    return "D";
}
if (numberGrade < 80) {
    return "C";
}
if (numberGrade < 90) {
    return "B";
}
return "A";
```

Putting all this in a `StudentGradeTranslator` class gives us the following
code:

```java
public class StudentGradeTranslator {
   public String getLetterGrade(int numberGrade) {
      if (numberGrade < 60) {
         return "F";
      }
      if (numberGrade < 70) {
         return "D";
      }
      if (numberGrade < 80) {
         return "C";
      }
      if (numberGrade < 90) {
         return "B";
      }
      return "A";
    }
}
```

which we can use with the following code in a class that we'll name
`InnerClassRunner` since we're using it for this "Inner Class" section of the
course:

```java
public class InnerClassRunner {

    public static void main(String[] args) {
        StudentGradeTranslator gradeTranslator = new StudentGradeTranslator();

        HashMap<String, String> studentGrades = new HashMap<String, String>();
        studentGrades.put("Jamaal", "93");
        studentGrades.put("Jennifer", "87");
        studentGrades.put("Jules", "59");

        // get all the student and their grades using each entry
        System.out.println("List of students and their grades:");
        for (Map.Entry<String, String> studentGrade: studentGrades.entrySet()) {
            System.out.println(studentGrade.getKey() + "'s grade is " +
                    gradeTranslator.getLetterGrade(Integer.parseInt(studentGrade.getValue())));
        }
    }
}
```

This will produce the following output:

```
List of students and their grades:
Jennifer's grade is B
Jamaal's grade is A
Jules's grade is F
```

Let's summarize what we've done so far:

1. We have a "runner" class that creates the list of students and their point
   grades
2. We determined we needed to be able to translate those point grades into
   letter grades
3. So we created a class to help us to do that, and implemented a basic
   algorithm based on a flat grading system where each point grade translates to
   the corresponding letter grade based on the traditional A for 90 to 100
   classification
4. We created a simple loop that goes through every student and displays their
   translated letter grade

So far, so good!

Now one of our professors has determined that they need to be able to support
multiple ways to translate their numerical grades into letter grades based on
how difficult the class is or how difficult a specific test was.

How can we support multiple grade calculations? Very importantly, how can we
support multiple grade calculations without repeating a lot of code and without
making the `InnerClassRunner` class worry about the implementation details of
the actual calculations?

We could create a different version of the `getLetterGrade()` method for each
different level of curve - which would look something like this:

```java
public class StudentGradeTranslator {
   public String getLetterGrade(int numberGrade) {
      if (numberGrade < 60) {
         return "F";
      }
      if (numberGrade < 70) {
         return "D";
      }
      if (numberGrade < 80) {
         return "C";
      }
      if (numberGrade < 90) {
         return "B";
      }
      return "A";
    }

   public String getLetterGrade_SlightCurve(int numberGrade) {
      if (numberGrade < 55) {
         return "F";
      }
      if (numberGrade < 65) {
         return "D";
      }
      if (numberGrade < 75) {
         return "C";
      }
      if (numberGrade < 85) {
         return "B";
      }
      return "A";
   }
}
```

While this would do the job, it also means we need to change the code in our
`InnerClassRunner` class to call the new method:

```java
            System.out.println(studentGrade.getKey() + "'s grade is " +
                    gradeTranslator.getLetterGrade_SlightCurve(Integer.parseInt(studentGrade.getValue())));

```

If we are using the `getLetterGrade()` method in multiple places, we would have
to remember to call the right method, i.e. `getLetterGrade()` vs
`getLetterGrade_SlightCurve()` in every place. Similarly, if there was other
behavior that was conditional on the grade curve, I would also have to make sure
to create different methods to implement that behavior _and_ class the right
method from everywhere that functionality was used.

For example, I might have a method to tell me whether the student has a passing
grade:

```java
        public boolean isPassingGrade(int numberGrade) {
            if (numberGrade >= 60) return true;
            return false;
        }
```

This method would look different if we had a slight curve on the grades:

```java
        public boolean isPassingGrade_SlightCurve(int numberGrade) {
            if (numberGrade >= 60) return true;
            return false;
        }
```

So now I would have to change my `InnerClassRunner` to make sure it calls the
right `isPassingGrade` method based on the curve that needs to be applied,
yielding code that is becoming increasingly duplicative, like this:

```java
        System.out.println("List of students and their grades:");
        for (Map.Entry<String, String> studentGrade: studentGrades.entrySet()) {
            if (gradingMethod.equals("FLAT")) {
                System.out.println(studentGrade.getKey() + "'s grade is " +
                        gradeTranslator.getLetterGrade(Integer.parseInt(studentGrade.getValue())));
                System.out.println("Passing grade status: " + gradeTranslator.isPassingGrade(Integer.parseInt(studentGrade.getValue())));
            } else if (gradingMethod.equals("SLIGHT")) {
                System.out.println(studentGrade.getKey() + "'s grade is " +
                        gradeTranslator.getLetterGrade_SlightCurve(Integer.parseInt(studentGrade.getValue())));
                System.out.println("Passing grade status: " + gradeTranslator.isPassingGrade_SlightCurve(Integer.parseInt(studentGrade.getValue())));
            }
        }
```

The only difference between the code inside the two `if` statements is the
version of the `getLetterGrade()` and `isPassingGrade()` methods it calls. This
will need over time to a severe duplication of code in addition to being very
error prone because we must remember to call exactly the right method for every
type of grading method we want to use, and make sure this is right in all the
places where this choice is relevant.

A better approach would be to take advantage of the fact that we want the same
basic functionality from the `StudentGradeTranslator` class regardless of the
actual method we need to use. The basic functionality as we have discussed it
is:

1. Translate a numerical grade into a letter grade
2. Tell us is a numerical grade is a passing grade

Let's create an interface that defines the method we might implement to support
this functionality:

```java
interface GradeCalculator {
  public String getLetterGrade(int numberGrade);
  public boolean isPassingGrade(int numberGrade);
}
```

We can create an implementation of this interface for each different curve
algorithms:

```java
class FlatCurveGradeCalculator implements GradeCalculator {
    // ...
}
```

and

````java
```java
class SlightCurveGradeCalculator implements GradeCalculator {
    // ...
}
````

Now that we have an interface that defines the behavior we need, we can use the
appropriate implementation based on the algorithm we need, but the method calls
remain the same regardless:

```java
    GradeCalculator gradeCalculator;

    public StudentGradeTranslator(String gradingMethod) {
        if (gradingMethod == null) {
            this.gradeCalculator = new FlatCurveGradeCalculator();
        } else if (gradingMethod.equals("FLAT")) {
            this.gradeCalculator = new FlatCurveGradeCalculator();
        } else if (gradingMethod.equals("SLIGHT")) {
            this.gradeCalculator = new SlightCurveGradeCalculator();
        }
    }

    public String getLetterGrade(int numberGrade) {
        return gradeCalculator.getLetterGrade(numberGrade);
    }

    public boolean isPassingGrade(int numberGrade) {
        return gradeCalculator.isPassingGrade(numberGrade);
    }
```

We will discuss the specific implementations of each one of these methods a
little bit later, but first let's consider where this interface and these class
definitions should live.

The `GradeCalculator` is only useful in the context of the grade translation
functionality. Furthermore, we want to be able to change the internals of how a
"slight curve" is applied to grades without having to change how a letter grade
using that scale can be obtained from the `StudentGradeTranslator` class.

We can support this level of logic isolation by defining both the interface and
the implementations of the interface inside the `StudentGradeTranslator` class,
making them therefore only accessible within and by that class:

```java
public class StudentGradeTranslator {

    GradeCalculator gradeCalculator;

    public StudentGradeTranslator() {
        this.gradeCalculator = new FlatCurveGradeCalculator();
    }

    public StudentGradeTranslator(String gradingMethod) {
        if (gradingMethod == null) {
            this.gradeCalculator = new FlatCurveGradeCalculator();
        } else if (gradingMethod.equals("FLAT")) {
            this.gradeCalculator = new FlatCurveGradeCalculator();
        } else if (gradingMethod.equals("SLIGHT")) {
            this.gradeCalculator = new SlightCurveGradeCalculator();
        } else if (gradingMethod.equals("HEAVY")) {
            this.gradeCalculator = new HeavyCurveGradeCalculator();
        }
    }

    public String getLetterGrade(int numberGrade) {
        return gradeCalculator.getLetterGrade(numberGrade);
    }

    public boolean isPassingGrade(int numberGrade) {
        return gradeCalculator.isPassingGrade(numberGrade);
    }

    // the interface that dictates that methods each grade calculator
    // implementation must support
    interface GradeCalculator {
        public String getLetterGrade(int numberGrade);
        public boolean isPassingGrade(int numberGrade);
    }

    // an example implementation of the grade calculator interface
    class FlatCurveGradeCalculator implements GradeCalculator {
        public String getLetterGrade(int numberGrade) {
           // ...
        }

        public boolean isPassingGrade(int numberGrade) {
           // ...
        }
    }

    // another example implementation of the grade calculator interface
    class SlightCurveGradeCalculator implements GradeCalculator {
        public String getLetterGrade(int numberGrade) {
            // ...
        }

        public boolean isPassingGrade(int numberGrade) {
            if (numberGrade >= 55) return true;
            return false;
        }
    }
}
```

This has now drastically simplified the code that uses the
`StudentGradeTranslator` functionality in `InnerClassRunner`. While we still
need to indicate which curve algorithm we want, we only need to do so when we
initially create the `StudentGradeTranslator`. Everything else after that point
is the same regardless of the grading algorithm used:

```java
public class InnerClassRunner {

    public static void main(String[] args) {
        String gradingMethod = "SLIGHT";
        StudentGradeTranslator gradeTranslator = new StudentGradeTranslator();

        HashMap<String, String> studentGrades = new HashMap<String, String>();
        studentGrades.put("Jamaal", "93");
        studentGrades.put("Jennifer", "87");
        studentGrades.put("Jules", "59");

        // get all the student and their grades using each entry
        System.out.println("List of students and their grades:");
        for (Map.Entry<String, String> studentGrade: studentGrades.entrySet()) {
            System.out.println(studentGrade.getKey() + "'s grade is " +
                    gradeTranslator.getLetterGrade(Integer.parseInt(studentGrade.getValue())));
            System.out.println("Passing grade status: " + gradeTranslator.isPassingGrade(Integer.parseInt(studentGrade.getValue())));
        }
    }
}
```

This lets the `InnerClassRunner` class simply specify the algorithm it wants the
`StudentGradeTranslator` to use and then defer the logic of how that impacts
that class' behavior to the class itself, which is where it belongs.
Furthermore, because the `StudentGradeTranslator` class has defined the
interface and implementations it needs internally, it can now decide to change
how those behave without impacting anyone who uses that functionality.

Note: while it's _technically_ possible for an outside class to access another
class' inner class, it a) should not be done by the outside class and b) should
be protected from being possible by the outer class by defining its internal
classes as `private`, which is an access modifier we will cover in a later
section.

After we fill in the logic for each curve grade algorithm and add support for
both "Flat", "Slight" and "Heavy" grading, we end up with the following code for
`InnerClassRunner`:

```java
import java.util.HashMap;
import java.util.Map;

public class InnerClassRunner {

    public static void main(String[] args) {
        String gradingMethod = "SLIGHT"; // <---- simply change this value to change the way grades are curved
        StudentGradeTranslator gradeTranslator = new StudentGradeTranslator();

        HashMap<String, String> studentGrades = new HashMap<String, String>();
        studentGrades.put("Jamaal", "93");
        studentGrades.put("Jennifer", "87");
        studentGrades.put("Jules", "59");

        // get all the student and their grades using each entry
        System.out.println("List of students and their grades:");
        for (Map.Entry<String, String> studentGrade: studentGrades.entrySet()) {
            System.out.println(studentGrade.getKey() + "'s grade is " +
                    gradeTranslator.getLetterGrade(Integer.parseInt(studentGrade.getValue())));
            System.out.println("Passing grade status: " + gradeTranslator.isPassingGrade(Integer.parseInt(studentGrade.getValue())));
        }
    }
}
```

And the following code for `StudentGradeTranslator`:

```java
public class StudentGradeTranslator {

    GradeCalculator gradeCalculator;

    public StudentGradeTranslator() {
        this.gradeCalculator = new FlatCurveGradeCalculator();
    }

    public StudentGradeTranslator(String gradingMethod) {
        if (gradingMethod == null) {
            this.gradeCalculator = new FlatCurveGradeCalculator();
        } else if (gradingMethod.equals("FLAT")) {
            this.gradeCalculator = new FlatCurveGradeCalculator();
        } else if (gradingMethod.equals("SLIGHT")) {
            this.gradeCalculator = new SlightCurveGradeCalculator();
        } else if (gradingMethod.equals("HEAVY")) {
            this.gradeCalculator = new HeavyCurveGradeCalculator();
        }
    }

    public String getLetterGrade(int numberGrade) {
        return gradeCalculator.getLetterGrade(numberGrade);
    }

    public boolean isPassingGrade(int numberGrade) {
        return gradeCalculator.isPassingGrade(numberGrade);
    }

    interface GradeCalculator {
        public String getLetterGrade(int numberGrade);
        public boolean isPassingGrade(int numberGrade);
    }

    class FlatCurveGradeCalculator implements GradeCalculator {
        public String getLetterGrade(int numberGrade) {
            if (numberGrade < 60) {
                return "F";
            }
            if (numberGrade < 70) {
                return "D";
            }
            if (numberGrade < 80) {
                return "C";
            }
            if (numberGrade < 90) {
                return "B";
            }
            return "A";
        }

        public boolean isPassingGrade(int numberGrade) {
            if (numberGrade >= 60) return true;
            return false;
        }
    }

    class SlightCurveGradeCalculator implements GradeCalculator {
        public String getLetterGrade(int numberGrade) {
            if (numberGrade < 55) {
                return "F";
            }
            if (numberGrade < 65) {
                return "D";
            }
            if (numberGrade < 75) {
                return "C";
            }
            if (numberGrade < 85) {
                return "B";
            }
            return "A";
        }

        public boolean isPassingGrade(int numberGrade) {
            if (numberGrade >= 55) return true;
            return false;
        }
    }

    class HeavyCurveGradeCalculator implements GradeCalculator {
        public String getLetterGrade(int numberGrade) {
            if (numberGrade < 50) {
                return "F";
            }
            if (numberGrade < 60) {
                return "D";
            }
            if (numberGrade < 70) {
                return "C";
            }
            if (numberGrade < 80) {
                return "B";
            }
            return "A";
        }

        public boolean isPassingGrade(int numberGrade) {
            if (numberGrade >= 50) return true;
            return false;
        }
    }
}
```

Inner classes also have access to all the members of their outer class, which
gives them additional flexibility to implement their necessary logic.
