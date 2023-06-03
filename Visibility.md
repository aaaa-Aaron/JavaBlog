# What is Memory Visibility ?
Memory visibility refers to the concept of how changes made to shared variables in multithreaded program become visable to other threads.In concurrent programming, multiple threads may execute simultaneously and access shared data in memory.

However, due to factors like CUP caching and optimizations, changes made by one thread to a shared variable may not immediately be visible to other threads

# Example about memory problem
```java
public class NoVisibility { 
	private static boolean ready; 
	private static int number; 
	private static class ReaderThread extends Thread { 
		public void run() { 
			while (!ready) 
				Thread.yield(); 
			System.out.println(number);
		} 
	} 
	public static void main(String[] args) { 
		new ReaderThread().start(); 
		number = 42; 
		ready = true; 
	} 
}
```
NoVisibility could loop forever because the value of ready might never becom visible to the reader thread. Even more strangely, NoVisibility could print zero because the write to ready might be made visible to the reader thread before the write to number, a phenomenon known as reordering.

# confuse
```
public class NoVisibility { 
    private static boolean ready; 
    private static int number; 
    private static class ReaderThread extends Thread { 
        public void run() { 
            while (!ready) 
                Thread.yield(); 
            System.out.println(number);
        } 
    } 
    public static void main(String[] args) { 
        new ReaderThread().start(); 
         Thread.sleep(10);
        ready = true; 
		number = 42;
    } 
}
```

https://wiki.sei.cmu.edu/confluence/display/java/TSM03-J.+Do+not+publish+partially+initialized+objects


The `ReaderThread` may indeed get stuck in an infinite loop, even though the `ready` variable is marked as `static`. The reason for this behavior is the lack of proper synchronization in the code.

Although the Java memory model provides "safe publication" guarantees for `static` variables, it does not provide any synchronization guarantees between threads without explicit synchronization mechanisms.

In this case, the `ReaderThread` may not see the updated value of `ready` because there is no explicit synchronization between the `main` thread and the `ReaderThread`. The `ReaderThread` may cache the value of `ready` and not observe the change made by the `main` thread.

When you remove the `Thread.sleep` call, it can introduce a different timing and thread scheduling, which may affect the visibility of shared variables. It can create a higher probability for the `ReaderThread` to observe the updated value of `ready`.

To ensure proper visibility and synchronization, you should use explicit synchronization mechanisms such as `synchronized` blocks or `volatile` variables. For example, marking the `ready` variable as `volatile` would ensure that changes made by one thread are immediately visible to other threads:

```java
private static volatile boolean ready;
```

Using explicit synchronization or volatile variables ensures that changes made to shared variables are properly visible to other threads, resolving the issue of the `ReaderThread` getting stuck in an infinite loop.

# Summary
java have a concept called "safe publication" 

The Java memory model (JMM) allows multiple threads to observe the object after its initialization has begun but before it has concluded.

because of this ReaderThread sometime can notice when static variable 'ready' is updated, but sometime it can't recognize it 