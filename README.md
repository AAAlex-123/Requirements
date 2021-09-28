# Requirements

## Intent
Decouple the request for an object, the object responsible for fulfiling that request and 
the fulfilment itself.

## Explanation

Real-world example

> A teacher in a classroom needs something to write on the board. After requesting that 
> object, a student takes the responsibility to fulfil the teacher's request and leaves 
> the classroom. By means unknown to the teacher, the student later returns with a marker 
> and gives it back to the teacher so that they can write on the board.

In plain words

> A Requirement object represents a request made by object A. Object B uses that 
> Requirement to fulfil it and gives it back to object A to use the requested object.

**Programmatic Example**

Let's define a compiler that needs to know its input and output files

```java
public class Compiler {

	public static Requirements getRequirements() {
		Requirements reqs = new Requirements();
		reqs.add("input_file");
		reqs.add("output_file");
		reqs.add("verbose", Arrays.asList(true, false));
		return reqs;
	}

	public void compile(Requirements reqs) {
		File    input_file  = new File(reqs.getValue("input_file", String.class));
		File    output_file = new File(reqs.getValue("output_file", String.class));
		boolean verbose     = reqs.getValue("verbose", Boolean.class);

		System.out.printf("Compiling from '%s'%n", input_file);
		System.out.printf("Outputing to   '%s'%n", output_file);
		System.out.printf("Printing %s messages%n", verbose ? "all" : "some");
		// read source code from input file
		// compile source code
		// write generated code to output file
	}
}
```
Here's how the compiler's Requirements can be used to modify its compilation parameters:

```java
public class App {

	public static void main(String[] args) {
		Requirements testReqs = Compiler.getRequirements();
		testReqs.fulfil("input_file", "test.c");
		testReqs.fulfil("output_file", "test.out");
		testReqs.fulfil("verbose", false);

		Requirements productionReqs = Compiler.getRequirements();
		productionReqs.fulfil("input_file", "src.c");
		productionReqs.fulfil("output_file", "a.out");
		promptAndFulfilVerbose(productionReqs);

		Compiler compiler = new Compiler();
		compiler.compile(testReqs);
		compiler.compile(productionReqs);
	}

	private static void promptAndFulfilVerbose(Requirements reqs) {
		Scanner scanner = new Scanner(System.in);
		System.out.println("Do you want to print all messages during compilation? (y/n)");
		boolean verbose = scanner.nextLine().equals("y");
		reqs.fulfil("verbose", verbose);
	}
}
```

**Notes**
* Altering the Requirements:
  * There are no methods for specifying the parameters (e.g.`setInputFile(String)` or 
`Compiler(String input, String output, boolean verbose)`), therefore altering what the 
compiler needs to function is as simple as changing the Requirements object returned. 
  * Similarly, the client code doesn't have to change its method or constructor calls 
whenever the compiler is changed, only the way the individual requirements that are 
fulfilled.

## Class diagram

![!TODO!](#lurl#)

## Applicability

!TOOD!
The Template Method pattern should be used

* To implement the invariant parts of an algorithm once and leave it up to subclasses to implement the behavior that can vary
* When common behavior among subclasses should be factored and localized in a common class to avoid code duplication. This is a good example of "refactoring to generalize" as described by Opdyke and Johnson. You first identify the differences in the existing code and then separate the differences into new operations. Finally, you replace the differing code with a template method that calls one of these new operations
* To control subclasses extensions. You can define a template method that calls "hook" operations at specific points, thereby permitting extensions only at those points


## Known uses

* [This project](https://github.com/AAAlex-123/Simple-CAD-Tool/) uses Requirements to change the behaviour of the Command and Actions classes.
