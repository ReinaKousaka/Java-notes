
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


