# Learning Journal Entry \- Reading Notes

## Effective Java

**Second Edition**  
Joshua Bloch

---

### Chapter 2: Creating and Destroying Objects

* When and how to create and destroy them
* When and how to avoid creating them
* Ensure they are destroyed in a timely manner
* Manage cleanup operations that proceed their destruction

#### Item 1: Consider Static Factory Methods

The normal way for a class to allow a client to obtain an instance of itself is to provide a public constructor.  
A class can provide a public static factory method, which is simply a method that returns an instance of the class.

```java
public static Boolean valueOf(boolean b) {
	return b ? Boolean.TRUE : Boolean.FALSE;
}
```

**One advantage of static factory methods is that, unlike constructors, they have names.** If the parameters of a constructor do not, in and of themselves, describe the object being returned, a static factory with a well known name is easier to use and the resulting client code easier to read.

In cases where a class seems to require multiple constructors with the same signature, replace the constructors with static factory methods and carefully chosen names to highlight their differences.

**A second advantage of static factory methods is that, unlike constructors, they are not required to create a new object each time they’re invoked.**  
This allows immutable classes to use preconstructed instances, or to cache instances as they;re constructed, and dispense with them repeatedly to avoid creating unnecessary duplicate objects.

The ability of static factory methods to return the same object from repeated invocations allows classes to maintain strict control over what instances exist at any time. Classes that do this are said to be *instance-controlled.* There are several reasons to write instance-controlled classes. Instance control allows a class to guarantee that it is a singleton or noninstantiable. Also, it allows an immutable class to guarantee that no two equal instances exist: a.equals(b) if and only if a==b. If a class makes this guarantee, then its clients can use the \== operator instead of the equals(Object) method, which may result in improved performance. Enum types provide this guarantee.

**A third advantage of static factory methods is that, unlike constructors, they can return an object of any subtype of their return type.** This gives you great flexibility in choosing the class of the returned object.  
One application of this flexibility is that an API can return objects without making their classes public. Hiding implementation classes in this fashion leads to a very compact API. This technique lends itself to *interface-based* frameworks where interfaces provide natural return types of static factor methods. Interfaces can’t have static methods, so by convention, static factory methods for an interface named *Type* are put in a noninstantiable class named *Types.*  
For example, the Java Collections Framework has 32 convenience implementations of its collection interfaces, providing unmodifiable collections, synchronised collections, and the like. Nearly all of these implementations are exported via static factory methods in one noninstantiable class (java.util.Collections).The classes of the returned objects are all nonpublic.  
The Collections Framework API is much smaller than it would have been had it exported 32 separate public classes, one for each convenience implementation. It is not just the bulk of the API that is reduced, but the *conceptual weight.* The user knows that the returned object has precisely the API specified by its interface, so there is no need to read additional class documentation for the implementation classes. Furthermore, using such a static factory method requires the client to refer to the returned object by its interface rather than its implementation class, which is generally good practice.  
…  
The class of the object returned by a static factory method need not even exist at the time the class containing the method is written. Such flexible static factory methods form the basis of *service provider frameworks,* such as the Java Database Connectivity API (JDBC). A service provider framework is a system in which multiple service providers implement a service and the system makes the implementations available to its clients, decoupling them from the implementations.

**A fourth advantage of static factory methods is that they reduce the verbosity of creating parameterized type instances.** Unfortunately, you must specify the type parameters when you invoke the constructor or a parameterized class even if they’re obvious from context. This typically requires you to provide type parameters twice in quick succession.

```java
Map<String, List<String>> m = 
	new HashMap<String, List<String>>();
```

This redundant specification quickly becomes painful as the length and complexity of the type parameters increase. With static factories, however, the compiler can figure out the type parameters for you. This is known as *type inference.* FOr example, supposed HahMpa provided this static factory:

```java
public static <K, V> HashMap<K, V> newInstance() {
	return new HashMap<K, V>();
}
```

Then you could replace the wordy declaration above with this succinct alternative:

```java
Map<String, List<String>> m = HashMap.newInstance();
```

Someday the language may perform this sort of type inference on constructor invocation as well as method invocations, but as of release 1.6 it does not.  
Unfortunately, the standard collection implementations such as HashMap do not have factory methods  as of release 1.6., but you can put these methods in our own utility classes. More importantly, you can provide such static factories in our own parameterized classes.

**The main disadvantage of providing only static factory methods is that classes without public or protected constructors cannot be subclassed.** The same is true for nonpublic classes returned by public static factories. For example it is impossible to subclass any of the convenience implementation classes in the Collections framework. Arguably this can be a blessing in disguise as it encourages programmers to use composition instead of inheritance. **A second disadvantage of static factory methods is that they are not readily distinguishable from other static methods.** They do not stand out in API documentation in the way that constructors do, so it can be difficult to figure out how to instantiate a class that provides static factory methods instead of constructors. The Javadoc tool may someday draw attention to static factory methods. In the meantime, you can reduce this disadvantage by drawing attention to static factories in class or interface comments, and by adhering to common naming conventions. In summary, static factory methods and public constructors both have their uses, and it pays to understand their relative merits. Often static factories are preferable, so avoid the reflection to provide public constructors without first considering static factories.

---

This passage was directly relevant to a coding problem I encountered in my work for PacSol recently.

---

#### Item 2: Consider a builder when faced with many constructor parameters

```java
// Builder Pattern
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final in carbohydrate;
    
    public static class Builder {
        // Required Parameters
        private final int servingSize;
        private final int servings;
        
        // Optional Parameters - initialized to default values
        private int calories        = 0;
        private int fat             = 0;
        private int carbohydrate    = 0;
        private int sodium          = 0;
        
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }
        
        public Builder calories(int val) {
            { calories = val;       return this; }
        }

        public Builder carbohydrate(int val) {
            { carbohydrate = val;       return this; }
        }

        public Builder sodium(int val) {
            { sodium = val;       return this; }
        }
        
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
    
    private NutritionFacts(Builder builder) {
        servings        = builder.servings;
        servingSize     = builder.servingSize;
        calories        = builder.calories;
        fat             = builder.fat;
        sodium          = builder.sodium;
        carbohydrate    = builder.carbohydrate;
    }
}

```

Note that NutritionFacts is immutable, and that all parameter default values are in a single location. THe builder's setter methods return the builder itself so that invocations can be chained. Here's how the client code looks:

```java
NutritionFacts cocoCola = new NutritionFacts.Builder(240, 8).
    calories(100).sodium(35).carbohydrate(27).build();
```

The client code is easy to write and, more importantly, to read. **The Builder pattern simulates named optional parameters** as found in Ada and Python.  
Another way to impose invariants involving multiple parameters is to have setter methods take entire groups of parameters on which some invariant must hold. If the invariant isn't satisfied, the setter methods throw an IllegalSArugmentSException. THis has the advantage of detecting the invariant failure as soon as the invalid parameters are passed,instead of waiting for build to be invoked.

- The Builder pattern does have disadvantages of its own. In order to create an object, you must first create its builder. While the cost of creating the builder is unlikely to be noticeable in practice, it could be a problem in some performance critical situations. Also, the Builder pattern is more verbose than the telescoping constructor pattern, so it should be used only if there are enough parameters, say four or more. But keep in mind that you may want to add parameters in the future. If you start out with a constructor or static factories, and add a builder when the class evolves to the point where the numbers of parameters start to get out hand, the obsolete constructors or static factories will stick out like a sore thing., Therefore it's often better step start with a builder in the first place
- In summary, **the Builder pattern is a good choice when designing classes whose constructors or static factories would have more than a handful of parameters**, especially if most of those parameters are optional. Client code is much easier to read and write with builders than with the traditional telescoping constructs pattern, and builders are much safer than JavaBeans.