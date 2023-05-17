
### Constructors

- When a constructor runs, it immediately calls its superclass constructors, all the way up the chain until you get to the class Object constructor
- To invoke a superclass constructor, the only way is call `super()`. It MUST be the first statement in a constructor
- Use `this()` to call a constructor from another overloaded constructor in the same class. It MUST be the first statement in a constructor (so a constructor can never have both super() and this())
```java
// if you do not provide a constructor, the compiler puts a default one:
public className() {
	super();
}

// if provide a constructor without a call to super()
// the compiler will call super() without arguments in each of your overloaded constructors
```

### Accesss levels

 - public: any code anywhere can access it
 - protected: same as default, except it also allows access from subclasses of the class
 - default (no modifiers): can be accessed from any class within the same package, but is inaccessible to classes in different packages.
 - private: private to the class, not to the object (Accessing the private field of another object of the same class is allowed)

### Generics

```java
// bounded wildcard generics, means Animal, or any subtype of Animal
// List<? extends Animal>

public void func(List<? extends Animal> animals) {
	animals.get(0).doSomething();	// OK
	animals.add(new Animal());	// Error
}

// generic type parameter T
// List<T extends Animal>

public <T extends Animal> void func(List<T> animals, T animal) {
	animals.add(animal);	// OK
}
```

 - When use a wildcard in method argument, the compiler will STOP you from doing anything that could hurt the list referenced by the method parameter (e.g. put new things except null in the list); But you can still do things with the list elements. The wildcard could represent any subtype of Animal, so it is unsafe to add an Animal or any subtype of Animal
 - `T` represents a single known subtype of Animal so you can add elements of the type

Note: In generics, "extends" means "extends or implements"

### Comparable & Comparator

```java
// Comparable
class Song implements Comparable<Song> {
	private String title, artist;

	// must implement compareTo()
	public int compareTo(Song s) {
		return title.compareTo(s.getTitle());
	}
}

// Comparator
class ArtistCompare implements Comparator<Song> {
	// must implement compare()
	public int compare(Song a, Song b) {
		return a.getArtist().compareTo(b.getArtist());
	}
}
...
ArtistCompare artistCompare = new ArtistCompare();
songList.sort(artistCompare);
```

Comparable: compare in only one way \
Comparator is external, separate class (can make as many as you like) \
If you pass a comparator to the sort method, the sort order is determined by the comparator; otherwise is determined by the element’s compareTo() method. Both Comparable and Comparator are interfaces.


### Reference equality & Object equality

 - Reference equality: two references, one object on the heap. (referring to the same object).
	Checked by `==` operator
 - Object equality: two references, two objects on the heap. But the objects are considered **meaningfully equivalent**. Checked by `equals()` method

E.g. `HashSet<E>` calls object's hashCode() method to check duplicates, which gives each object a unique hashcode value. So to treat two difference Song objects as equal by title, we must override both the hashCode() and equals() interited from class Object

```java
class Song implements Comparable<Song> {
	private String title, artist;

	public boolean equals(Object aSong) {
		Song other = (Song) aSong;
		return title.equals(other.getTitle());
	}

	public int hashCode() {
		return title.hashCode();
	}
}
```

### Lambda & Streams
SAM (single abstract method) interfaces, a.k.a. functional interfaces can be implmented as a lambda expression.
```java
// emptyt brackets means no parameters
// no semicolons needed
() -> System.out.println("Hello!")
// param types are optional, no need for round brackets if it's a single parameter without a type
str -> System.out.println(str)
// Lambda implements a functional interface
(s1, s2) -> s1.compareToIgnoreCase(s2)

@FunctionalInterface
public interface Comparator<T> {
	int compare(T o1, T o2);
}
```

An Example of Stream pipeline: \
Intermediate operations are lazy. So it performs operations as efficiently as possible. \
Terminal operations are eager, i.e. are run as soon as they're called.
```java
import java.util.stream.*;

List<String> strings = List.of("I", "am", "a", "list", "of", "Strings");
List<String> result = strings
	.stream()			// 1. Get the stream from a source collection
	.limit(4)			// 2. Call zero or more intermediate operations
	.collect(Collectors.toList());	// 3. Get results with a terminal operation

```

 - Stream operations do NOT change the original collection. The Streams API is a way to query.
 - Can NOT reuse streams.

```java
// filters out elements we don't want
Stream<T> filter(Predicate<? super T> predicate)
.filter(song -> song.getGenre().contains("Rock"))
// map: turn to another type, equivalent as Song::getGenre
.map(song -> song.getGenre())
// remove duplicates
.distinct()
// sort
.sorted((o1, o2) -> o1.getYear() - o2.getYear())

// where Predicate is defined as:
@FunctionalInterface
public interface Predicate<T> {
	boolean test(T t);
	...	// other methods
}
// Predicate can be used as the assignment target for a lambda expression or method reference. 
```

### Exception handling

Two categories of exceptions:
 - Checked exceptions: those are NOT subclasses of RuntimeException are checked by the compiler
 - Unchecked exceptions: RuntimeExceptions are NOT checked by the compiler

Must declare the exception if the code is risky: `public void takeRisk() throws BadException {}` \
Use try-catch when calls the risky method \
`RuntimeException` can be thrown anywhere, with or without declarations or try/catch blocks. The compiler does NOT pay attention to it.

### Threads

When a thread has been startedm a new stack is created with the Runnable's run() method on the bootom of the stack. The thread is now in the RUNNABLE state.
```java
@FunctionalInterface
public interface Runnable {
	public abstract void run();
}
```
To launch a new thread
```java
Runnable task = () -> {
    System.out.println("Hello from a new thread!");
};
// or
public class MyRunnable implements Runnable {
	@Override
	public void run() {}
}
// approach 1 (not recommended)
Thread thread = new Thread(task);
thread.start();

// approach 2, ExecutorService
import java.util.concurrent.Executors;

ExecutorService executor = Executors.newSingleThreadExecutor();		// single thread executor
executor.execute(task);		// run the job
executor.shutdown();		// shut down the ExecutorService
```

### Synchronization

Use `synchronized` keyword on a method or an object, to lock only one thread can use it at a time
```java
// on objects
private void goShoppint(int amount) {
	synchronized (account) {
		if (account.getBalance() >= amount) {;}
	}
}

public synchronized void spend(String name, int amount) {}

// synchronize a block of code
public void go() {
	doStuff();

	synchronized(this) {
		criticalStuff();
	}
}
```

#### Atomic variables

Atomic variables make a variable can be used safely by a thread without worring about another thread changing the object's values at the same time
```java
import java.util.concurrent.atomic.AtomicInteger; 	// AtomicLong, AtomicBoolean, AtomicReference

class Balance {
	AtomicInteger balance = new AtomicInteger(0);

	public void increment() {
		count.incrementAndGet();	// adds one to the value, no need to add "synchronized"
	}

	public void spend(String name, int amount) {
		int initialBalance = balance.get();
		// CAS: `compareAndSet(expectedValue, newValue)`
		// sets to new value only if the current value is the as the expected.
		boolean success = balance.compareAndSet(initialBalance, initialBalance - amount);
		if (success) {;} else {;}
	}
}
```
When an object's data cannot be changed, we call it an **immutable object**.
Make an object immutqable if you are going to share it between threads and do not want the threads to change its data. \
Note making an object field `final` does NOT guarantee the **data inside** that object won't change, just that the **reference** won't change. E.g. we might want to use `CopyOnWriteArrayList`, which is a thread-safe variant of `ArrayList`
```java
// we don't want to allow subclasses that might add mutable values
public final class Chat {
	private final String message;	// thread-safe

	public Chat(String message) {
		this.message = message;
	}
}
```