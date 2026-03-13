 For the last couple of years I've been interested in creating my own programming language to learn and get a better understanding of how things work. To be completely honest, it's not all that hard, it's just VERY confusing. It took me a couple of months to properly understand this, but once I did it's made everything so much easier. We can start with the two primary questions.

**First:** What is a programming language?
- The obvious answer is a "human-readable source-code for a computer program."

**Next:** What does that actually mean?
- Well, it's a written interpretation of how the data in a program is going to manipulated to perform the tasks that we want it to do. Alright, we've added more words to the definition but we still don't know HOW it does that.
# How it does that
This is where it gets fun. There are three (3) major steps that take place after you've finished writing the source-code. The Lexer, Parser and Interpreter (or Compiler).
## Lexer
The lexer is the first and most simple part of the programming language, you can think of it as a pair of scissors. It takes the source-code as it's input, cutting them into correct pieces, before outputs a stream (list) of "tokens" and "literals."

- **Tokens:** The pieces of syntax, like: `func`, `=`, `+`, or similar
- **Literals:** The actual data, like: `"my string"`, `12`, `32.500049`, any non-syntax that should be kept as it was inputted, literally.

That's about it. Depending on what you're planning to create, this can be as simple as "split by line" or as complex as C, Python and the rest. It all depends on what you need.
## Parser
The most fun and probably most difficult to understand part of the programming language is this part. That said, it's rather easy to implement once you wrap your head around it. To put it as simple as physically possible, the goal with the parser is to boil everything down to the simplest executable state.
> [!NOTE]
> Like inequalities in math, where you take two elements and remove them because they're identical.

Regardless of if a function immediately returns a value, or if it calculates the Fibonnacci sequence, the parser will break everything into specific and different "Nodes".
### Nodes
Everything in a programming language will be a node in an "Abstract Syntax Tree" which will be pushed to the final part. If you're familiar with C, you can think of these as custom structs that hold the pieces used to create the desired output. If you're familiar with Object Oriented Languages, then you can think of them as (class) objects.

Before we get into what that means, let's go over what a Node is first. To keep it simple, they're broken up into 3 main parts. 
- Type: What the Node is supposed to do, this is generally indicated by the stored name of the Node, like `functionNode`, `returnNode`, `literalNode`, etc.
- Internals: What is supposed to be calculated, split into 2 parts (left and right) 

We'll go into how they're created a little later.
### Node Example
Now we can get into the more technical side of what these Nodes actually look like in a programming language. We'll go with Python because it's dead simple and fairly easy to setup.
> [!NOTE]
> You can create a parent class that each node will reference as it's base, this makes it easier to type hint if that's something you prefer.
> ```python test
> class ASTNode:
> """ Base class for the rest of the AST Nodes """
>    pass
> ```

Each Node will be it's own class instance object, we'll go with a Unix Shell example because it's easier to understand for most people:
```python
class CommandNode:
""" Single command """
	def __init__(self, name, args):
		self.name = name
		self.args = args
```
This is the most simple part of a Unix Shell, a single command execution. So the only things this node needs to know is the name of the program it's going to run and any arguments that were passed to it.

Let's say you have a pipe (`|`) in your command, then the output would be something more simple, like:
```python
class PipeNode:
	""" Command output piped to another command """
	def __init__(self, left, right):
	    self.left = left
	    self.right = right
```

Once we get to the interpreter or compiler these will actually have a function, but currently they're containers that group the commands based on what they need to do later.
### AST (Abstract Syntax Tree)
This is one of the pieces that is incredibly difficult to understand if not explained properly. An Abstract Syntax Tree is a Tree of Abstract Syntax used by the interpreter or compiler to output a functional program. It's abstract because it's generated from your source-code, so it will be different based on every program that is written. The Syntax refers to, well, the syntax that you used to create the program in your source-code. And we call it a Tree because, when the source-code is properly parsed, it will fill out a tree-like structure of data with multiple connected branches.

I know that's a lot of words, so let's go over an example so you can see it.
#### AST Example:
For now, we'll continue the theme of using a Unix Shell to explain this because it's very simple.

We'll go with a rather easy but complex combination of command that will print the permissions of a folder (in this case `/home/<user>/Videos`) to a file named `output.txt`.
```
Input: "ls -l | grep Videos > output.txt"
```

While the nodes are being processed, they're going to be sorted with an order of operations that you choose. In this case, we'll be splitting the command based on what it needs to do:
```
Tree structure:
    Pipe
    /    \
  Command  Redirect
  (ls -l)  /       \
        Command   File
      (grep txt)  (output.txt)
```
1. What does the command need to do?
	- It runs `ls` and prints the output to the terminal
	- It pipes the previous command into the next
	- It redirects the output of the previous command into the the next
2. Where do we split this?
	- Order of operations
### Order of Operations
Everyone should be familiar with the order of operations used in math, `P E M D A S`. Our programming language will use a very similar order (actually in some places it will be identical) to properly parse the tokens (that we created in the Lexer) into the Nodes that we need. Just to make sure we wrap our heads around this, we'll do a simple Math expression to explain.
#### PEMDAS Example:
**Example Expression:** `12 + (14 - 1) * 10 / (12 + 4)`
**Execution Patter:**
```example
Search for parenthesis
  evaluate 14 - 1
  update expression: 12 + 13 * 10 / (12 + 4)

search for parenthesis
  evaluate 12 + 4
  update expression: 12 + 13 * 10 / 16

search for parenthesis
  Not found, check expression for next step

search for exponents
  Not found, check expression for next step

search for multiplication
  evaluate 13 * 10
  update expression: 12 + 130 / 16

search for multiplication
  Not found, check expression for next step

search for division
  evaluate 130 / 16
  update expression: 12 + 8.125

search for division
  Not found, check expression for next step

search for addition
  evaluate 12 + 8.125
  update expression: 20.125

search for addition
  Not found, check expression for next step

search for subtraction
  Not found, check expression for next step

return result
  result is 20.125
```
Notice how, no matter what step you're in, you always (possibly subconsciously) look for the current expression before moving on. This is called recursion.
### Recursion
We've already done a little recursion in the previous example, but we'll go over it in more detail now, to make sure you properly understand. Recursion is a pretty basic principle in programming, but what it can do gets pretty confusing pretty quick. First and foremost, it's ***almost*** the same as repetition, the main difference being that everything that happens will also happen to the pieces inside of it (like checking inside parenthesis for more parenthesis).

You can create a simple recursive function by having the function return itself, but that can get pretty messy because it'll run forever. The best way to stop this is to actually have a check before the return that will "complete" the function. This "final" step, is the one that actually does all the work.
#### Recursion Example
We'll do this with a factorial because it's incredibly simple; we take a number and multiplying all of the previous numbers together to get it's "total value" (the factorial).
```python
def factorial(num):
	if num == 1:
		return 1
	else:
		return num * factorial(num - 1)
```

This will cycle over itself until the result is `1`, or the lowest number that can actually be multiplied while still returning a positive value. To keep it simple, we'll pass `factorial(5)` so it only loops 5 times.
1. `return 5 * factorial(4)`
2. `return 4 * factorial(3)`
3. `return 3 * factorial(2)`
4. `return 2 * factorial(1)`
5. `return 1`

Notice that every return that is not 1 also calculates the factorial of itself, this is the recursion. The function does not care where the data it is calculating comes from, so if it needs to get it from itself later, it knows how to do that. This specifically is the part that confused me when learning about recursion (it confused me to write this).

Let's go ahead and re-order this to make more Logical sense, we'll flip the food pyramid and expand out the function calls. To get from 5 to 4, we need to know what 4 is, which means we need to know what 3 is, which means we need to know what 2 is, which means we need to have at least 1. The'll all compound before actually being executed, so our final expression should look almost identical to the actual math.
1. `return 1`: factorial(1) will always return 1, so we can ignore the call all together.
2. `return 2 * 1`
3. `return 3 * 2 * 1`
4. `return 4 * 3 * 2 * 1`
5. `return 5 * 4 * 3 * 2 * 1`

The only step that performs the calculation is the very first one, every other one is just returning a piece of that calculation. So to sum up this recursive factorial, when you call `factorial(5)` it returns `5 * 4 * 3 * 2 * 1`, or `120`.
### How Nodes are Created
Now, we can get into the fun part, how the nodes are actually created. This is pretty simple, we're going to chain a bunch of recursive functions together, using a specific Order of Operations. Previously, our Lexer output a list of tokens, now, we can iterate over them and sort them into their own Nodes. This may sound confusing, but it's actually pretty simple.

Generally, it's preferred that your parse function has a generic entry point, mine will accept a list of tokens and call the first recursive function. The order of operations for a shell is a little different than what I've shown, but you should be able to understand it.
#### Shell Node Example
I'll use Python (because it's syntax is dead simple and easy to understand), and a simplified version of what the commands will do.

We'll start off by creating two (2) helper functions. The first will find and return the index value of an item in the list. The second will split the list based on that index value.

We'll use Python's built-in try block to handle the difficult stuff, and just have a basic return that uses the index method on our class. This way we don't need to hard-code it into every function in the parser. 
```python
def find_operator(tokens: list, operator: str) -> int:
	try:
		return tokens.index(operator)
	except ValueError:
		return None
```

Next, we'll create the one that splits the list:
```python
def split_token(tokens: list, delimiter: str) -> tuple:
	idx = find_operator(tokens, delimiter)
	
	if idx is None:
		return None
	
	return tokens[:idx], tokens[idx + 1:]
```
This takes in a list and a delimiter and returns two lists and no delimiter. Something along the lines of:
`tokens = ["mkdir", "test", "&&", "echo", "Done!"]` ->
`split_tokens(tokens, "&&")` ->
`return ["mkdir", "test"], ["echo", "Done!"]`

Now we get to the actual Order of Operations, this is where the parser will do the heavy lifting, sorting through each item in the order you specify. This is much more complicated for a programming language, so we'll continue with the shell and it's seven (7) basic types of commands.
1. sequence breaks (`;`) - This splits into two distinct commands.
2. double ampersand (`&&`) - Run the next command if and only if the previous one exited with a status of 0.
3. double pipe (`||`) - Logical OR: Run the next command IF the previous exited with a non-zero status.
4. pipe (`|`) - Use the previous command's output as the next command's input.
5. forward redirect (`>`) - Redirect the output of the command to something else (like printing to a file).
6. backward redirect (`<`) - Use a file instead of user input.
7. ampersand (`&`) - Run in the background.

There are obviously more that need to be added, and I don't intend on going over all of them, but I will go over a few so you can understand what's happening.

We start in the general "parser" function, this is where it all starts, but generally this wont do much other than make sure your command isn't completely empty. Think of it as the front-door to the building, and everything else are the rooms you have to visit to get the info you're looking for, BEFORE you can finally return to the entrance (exit).
```python
def parser(tokens: list) -> ASTNode:
	if not tokens:
		raise ParseError(f"No tokens were provided.")
	
	# If no "advanced" operator, treat as normal command	
	if not any(t in OPERATORS for t in tokens):
		return CommandNode(tokens[0], tokens[1:])
	
	# Find what operator this sequence is using
	return parse_sequence(tokens)
```
> [!NOTE]
> You may notice that I'm using type hinting, this is just to keep the code clean so you can fully understand what it is doing, realistically you could remove it and this will function the exact same.

Next, we can look for sequence breaks, these turn into two explicitly different commands, regardless of their individual sizes.
```python
def parse_sequence(tokens: list) -> ASTNode:
	result = split_token(tokens, ";")
	
	if result is None:
		return parse_and(tokens)
	
	l_token, r_token = result
	left = parse_sequence(l_token)
	right = parse_sequence(r_token)
	
	return SequenceNode(left, right)
```
Regardless of what the commands are, if there is a semi-colon splitting it, this function will split them into the correct two commands.

The rest of them take the same or a very similar form, splitting on their respective pieces and recursively parsing them.
## Run the Code
This section may be shorter than the rest, but it's not going to be any less dense. We've completed all of the necessary setup for our programming language (or shell) to actually execute what it needs. The human readable source-code was turned into tokens that were then sorted into an AST of Nodes. Now, we can look in to how to actually execute it, this comes in three (3) basic forms: An Interpreter, a compiler and a Just-In-Time compiler.
### Interpreter
An interpreter is the most simple way to run a program, this is what Python started with (it has now transitioned to JIT, which is why we see `.pyc` files). This will, more or less, run the program line-by-line at run-time, instead of creating a separate binary file.

If you've written your custom language in Python, then it will use python's built-in functions to execute the commands (and in-turn C because Python is written in C). Say we wanted to recreate the same Factorial function in our custom language, all of the math will be done BY Python (which in turn does it's math with C).

This can be up to you on how to execute, but the most common is to create a "base" function with a large switch/case or if/else tree that will choose the type of Node that it's looking at and execute it's contents. Everything SHOULD be broken down into it's smallest possible calculation, so all you really have to worry about is HOW each is executed.

Let's go over an example of how, say, an AdditionNode will be executed.
#### AdditionNode Example
If you remember, the majority of our math nodes should be a class instance with a left and right. Skipping the entry-point function, we can break down the execution into something as simple as:
```python
def executeAddition(Node):
	left = Node.left
	right = Node.right
	
	return left + right
```

That's about it, there's obviously more steps and checks to make sure everything works the way you need it to. For instance, making sure the things being added together are actually numbers.
### Compiler
Depending on how far back you plan to go in their history, you can see compilers built in pretty much anything. If we go back to the 1970s we can see the first C compiler, written in the assembly language for the PDP-11, by Dennis Ritchie, for his pet project Unix. When he wrote it, it was a 1-to-1 translation from C to Unix Syscalls. Now-a-days when you create a language, it uses an intermediate representation, like LLVM.

I should mention that this version of the C Compiler only lasted a couple of hours at most, it was used to Bootstrap the language, with every version after being written in the C language itself.
#### Bootstrapping
Bootstrapping is the process of using another language to create the first functional version of your language. Pretty much every language, outside of direct machine code, uses some kind of base language to write their own. For instance, the C language was bootstrapped in the B programming language. The B language was written in the Basic Combined Programming Language (BCPL), which itself was written in ALGOL.
#### Intermediate Representation (IR)
Every compiler that we use will have an Intermediate Representation, or internal language, that the programming language will use to convert between the source-code and the compiled binary. This in-between language can be a custom-made one, specific for your language, or a premade one like LLVM.

The top two C Compilers use different intermediate representations. The GNU C Compiler (GCC) uses it's own custom one named GIMPLE, while CLANG uses LLVM.
### JIT Compiler
Finally, we can talk about one of the coolest things to come from so many people making their own languages. Just-in-time Compilation. We can skip over the first implementation and get to the one that made it popular; everyone's favorite language, Java.

In 1997, when Java 1.1 was released, it came with the advantage of JIT Compilation, making it one of the biggest names in the programming language space. It has since been improved and updated over the years, but even the first version was a massive performance improvement.

Just-in-time compilation is a tool, same as the Lexer or Parser, that lives inside (or along-side) the Interpreter. It's whole goal is to look for "hot code", or code that is used frequently, and translate it into native machine code. This both speeds up execution (which increases performance) and clean up/memory management, with the downside of needing far more setup. 

A fully featured JIT Compiler, like with Java or Python, uses techniques to handle how memory is done so you don't have to think about it the same way as a lower level language like C. Most commonly with a process called "Garbage Collection" where the pointers or mallocs that you write are added to a list (over-simplification) and purged from the list 1-by-1 until it is all freed, keeping your program free of memory leaks.
