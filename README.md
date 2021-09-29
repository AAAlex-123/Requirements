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

> A teacher in a classroom needs something to write on the board and something to erase 
> from it. They write in a piece of paper all the items they need and select the 
> appropriate tray to carry these items. A student takes the responsibility to fulfil the 
> teacher's request and leaves the classroom holding that piece of paper and the tray. By 
> means unknown to the teacher, the student later returns with a marker and a sponge, 
> which are placed on the tray, and gives the tray back to the teacher so that they can 
> retrieve the requested objects from it.

In plain words

> A Requirement object represents a request made by object A. Object B uses that 
> Requirement to fulfil it and gives it back to object A to use the requested object.

 ## Simple Example

Let's define a compiler that needs to know its input and output files, and whether or not 
to output all messages during compilation

```java
public class Compiler {

	public static Requirements getRequirements() {
		Requirements reqs = new Requirements();
		
		// define required objects
		reqs.add("input_file", StringType.FILENAME);
		reqs.add("output_file", StringType.FILENAME);
		reqs.add("verbose", Arrays.asList(true, false));
		return reqs;
	}

	public void compile(Requirements reqs) {
		// retrieve required objects
		File    input_file  = new File(reqs.getValue("input_file", String.class));
		File    output_file = new File(reqs.getValue("output_file", String.class));
		boolean verbose     = reqs.getValue("verbose", Boolean.class);

		// use required objects
		System.out.printf("Compiling from '%s'%n", input_file);
		System.out.printf("Outputing to   '%s'%n", output_file);
		System.out.printf("Printing %s messages%n", verbose ? "all" : "some");
	}
}
```
Here's how the Compiler's Requirements can be used to modify its compilation parameters:

```java
public class App {

	public static void main(String[] args) {
		// get the Requirements and fulfil them as deemed appropriate by the App
		Requirements testReqs = Compiler.getRequirements();
		fulfilTestReqs(testReqs);

		Requirements productionReqs = Compiler.getRequirements();
		fulfilProductionReqs(productionReqs);

		// use the Requirements to compile in 2 different ways, as specified by
		// the different values given to the parameters in the Requirements
		Compiler compiler = new Compiler();
		compiler.compile(testReqs);
		compiler.compile(productionReqs);
	}

	private static void fulfilTestReqs(Requirements reqs) {
		reqs.fulfil("input_file", "test.c");
		reqs.fulfil("output_file", "test.out");
		reqs.fulfil("verbose", false);
	}

	private static void fulfilProductionReqs(Requirements reqs) {
		reqs.fulfil("input_file", "src.c");
		reqs.fulfil("output_file", "a.out");

		Scanner scanner = new Scanner(System.in);
		System.out.println("Do you want to print all messages during compilation? (y/n)");
		boolean verbose = scanner.nextLine().equals("y");
		reqs.fulfil("verbose", verbose);
	}
}
```
**Notes**
* Altering the Requirements is as simple as:
  * defining different required objects when constructing the Requirements object
  * fulfiling different Requirements
  * retrieving different values when needed
* Instead of private String and boolean fields, the compilation information is contained 
within a single Requirements object
* Different parameteres for accomplishing the same task can be used by simply using a 
different Requirements object, without needing to call multiple setter methods

## Known uses

* [This project](https://github.com/AAAlex-123/Simple-CAD-Tool/) uses Requirements to 
change the behaviour of the Command and Actions classes.
* [This project](https://github.com/AAAlex-123/SML-compiler-runtime/) uses Requirements as 
shown in the example

## Acknowledgements

* [dimits](https://github.com/dimits-exe/) for the ListRequirement and its Graphic
