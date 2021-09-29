# Requirements

## Intent
Decouple the request for an object, the object responsible for fulfiling that request and 
the fulfilment itself.

## Motivation
Sometimes a class has a number of private fields, along with their setters, which can be 
set to customize the behaviour of an object of the class. Calling different methods to 
specify different parameters makes extending or modifying the parameters of the object 
rather difficult since new methods must be defined or the existing ones must have a 
different signature, which heavily impacts client code. Having many private fields can 
also be impractical for the class itself and the number of setter methods pollute the 
public API of the class as well as its source files. Giving freedom to users of this class 
to set these private fields at any time necessitates multiple additional checks (possibly 
at every setter) to ensure that the data reains consistent.

A Requirements object (a collection of individual Requirements) solves all of the above 
problems. It encapsulates all of the information contained in the private fields and 
provides a way to set each and every one of them, without the need for separate setter 
methods. Additionally, extending or modifying the parameters needed is as simple as 
adding, removing or changing a Requirement in the collection. The class itself can access 
the values of the parameters in a way similar to the get operation of a map. To protect 
the Requirements object from being altered with new values at inappropriate times, the 
class shall take responsibility to provide a safe way to use the Requirements object.

## Applicability
Use Requirements
* to encapsulate the parameters that a class needs to function
* to provide a uniform way to specify these parameters
* to allow the class that uses to easily access their values

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

## Known uses

* [This project](https://github.com/AAAlex-123/Simple-CAD-Tool/) uses Requirements to change the behaviour of the Command and Actions classes.
