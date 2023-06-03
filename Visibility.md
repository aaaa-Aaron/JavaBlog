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