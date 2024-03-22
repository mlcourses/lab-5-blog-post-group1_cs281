# Lab 5: Sequential Circuit Design

## Overview and Motivation

In this lab, we will be designing a circuit that reads a binary number and determines if that input is divisible by 3. The binary number will be fed to the circuit, one bit at a time, starting with the highest order bit and ending with the lowest order bit. Sounds a bit tricky, right? Actually, with some careful planning, you can also build your own circuit, and possibly even more complicated ones, as long as you follow our design pipeline! This lab should get you familiar with the sequential circuit design process, introduce you to the J-K flip-flop, drawing K-Maps, and get you some practice on circuit building. Let's get started!

## Activities:

1. Build a FSM as a Deterministic Finite Automata (DFA) state transition diagram.
2. Introduction to the J-K flip-flop
3. Create a function table
4. Use K-Maps to simplify logic for our outputs
5. Build our circuit

## Materials

1. PB-503 Breadboard
2. 7476 IC chip (J-K flip-flop), 7404 IC chip (NOT gates), 7408 IC chip (AND gates), 7486 IC chip (XOR gates)
3. Arduino
4. Resistors

## Project Steps

### 1. The DFA state transition diagram

Before our circuit materializes, we have to have an abstraction of it on paper. What are the different states our circuit can be in for this problem? Well, when talking about divisibility by 3, a number can be divisible by 3, have a remainder of 1 when divided by 3, or have a remainder of 2. Thus, at any given point of our input, the circuit will be in one of those 3 states. We will use a **Deterministic Finite Automata (DFA)** to show how these 3 states interact with each other based on our input.

A DFA is a graph that consists of vertices that represent our states, and edges that connect the vertices, which represent how we transition from one state to another based on our input. To better understand our DFA, let's consider the moment when we just started and our input is empty. Since we have 3 states, we will have 3 vertices in our DFA that we will name S<sub>0</sub>, S<sub>1</sub> and S<sub>2</sub>. The subscript number represents the remainder value, so we will start at S<sub>0</sub>, since our input is empty and has a value of 0 (divisible by 3).

Now, let's put in a 0. Our number is now still 0, so we stay at S<sub>0</sub>. This means there will be a an edge labeled 0 that goes from S<sub>0</sub> to itself. What if we put in a 1 instead? Then our number will be 1 (remainder of 1 when divide by 3), so we will transition to state S<sub>1</sub>. That means there will be an edge labeled 1 that goes from S<sub>0</sub> to S<sub>1</sub>. Okay, with our 1 now, what if we put in another 1? Then our number will be 11 in binary (3 in decimal, divisible by 3), so we will transition back to state S<sub>0</sub>. That means there will be another edge labeled 1 that goes from S<sub>1</sub> to S<sub>0</sub>. If we instead put in a 0, we will have the number 10 in binary (2 in decimal, remainder 2 when divide by 3), which means a transition to state S<sub>2</sub>. Then there will be an edge labeled 0 that goes from S<sub>1</sub> to S<sub>2</sub>. Do this until every state has 2 outgoing edges (1 and 0), and your DFA is complete! It should look like this:

![IMG_3C897D1DFF34-1](https://github.com/mb-linh/lab-5-blog-post-group1_cs281/assets/97915038/094e0a4b-9df9-4f5d-b6dd-ce435ec785b5)

Note how the state S<sub>0</sub> has 2 circles instead of 1. That means it is our **accepting state**. Since we are trying to determine if our number is divisible by 3, our circuit should accept our input (by lighting up an LED, for example) if our input leads to the circuit being at S<sub>0</sub>, and nothing otherwise. In our circuit, we will use the logic indicator on our breadboard that should show HIGH if our input is divisible by 3, and LOW otherwise. To verify our DFA, let's try a couple of inputs. What state will our circuit be in if we put in the number 10101 (21 in decimal)? Tracing our DFA, we will go in order: S<sub>0</sub> --> S<sub>1</sub> --> S<sub>2</sub> --> S<sub>2</sub> --> S<sub>1</sub> --> S<sub>0</sub>, which is correct since 21 is divisble by 3. What about 10111 (23 in decimal)? Our order will be S<sub>0</sub> --> S<sub>1</sub> --> S<sub>2</sub> --> S<sub>2</sub> --> S<sub>2</sub> --> S<sub>2</sub>, which is correct since 23 divide by 3 leaves a remainder of 2. Looks like our DFA is correct! Now, let's talk about how you can represent these states in our circuit, using the J-K flip-flop.

### 2. The J-K flip-flop

The J-K flip-flop is essentially a device that can retain 1 bit of information, called its state. It takes 2 inputs, called J and K, and outputs its state Q and its complement ~Q. At time t, its state is Q<sub>t</sub>, and the state Q<sub>t+1</sub> depends on J, K and Q<sub>t</sub>, according to the following table:

<img width="391" alt="image" src="https://github.com/mb-linh/lab-5-blog-post-group1_cs281/assets/97915038/70bd8008-0da3-4439-878b-678ee4b2d3af">

So if Q<sub>t</sub> is 0, and we want Q<sub>t+1</sub> to also be 0, we can either have J and K be both 0, or J be 0 and K be 1. Either case, J must be 0 but K can be either 0 or 1. In this case, we call the value of K a "don't care". If, on the other hand, Q<sub>t</sub> is 0 and we want Q<sub>t+1</sub> to be 1, we can either have J be 1 and K be 0, or J and K both be 1. In other words, J must be 1 and K is a "don't care". We can summarize all transitions from Q<sub>t</sub> to Q<sub>t+1</sub> in the table below (Q<sub>t+1</sub> is denoted as Q', and "don't care" is denoted as X):

<img width="267" alt="image" src="https://github.com/mb-linh/lab-5-blog-post-group1_cs281/assets/97915038/9ec97a43-bfbb-41d4-a396-91cf48564475">

Since Q can be either 0 or 1, we can use a single flip-flop to denote 2 states of our circuit. But since our a circuit have 3 states (according to our DFA), we will need 2 flip-flops to be able to represent 3 states. Let the 2 flip-flops be Q<sub>0</sub> and Q<sub>1</sub>, so we can denote our 3 states as follow (since we only have 3 states, we won't really care what happens when both flip-flops are 1):

| Q<sub>0</sub> | Q<sub>1</sub> | State |
| ------------- | ------------- | ----- |
| 0 | 0 | S<sub>0</sub> |
| 0 | 1 | S<sub>1</sub> |
| 1 | 0 | S<sub>2</sub> |
| 1 | 1 | X |

Now that we have a grasp on flip-flops, let's combine it with our DFA to create a function table!

### 3. Function table

Now, we will draw our function table, which tells us what we need to know about our circuit. Let's take a look at it first:

<img width="818" alt="image" src="https://github.com/mb-linh/lab-5-blog-post-group1_cs281/assets/97915038/80c0db3a-f49f-4c61-849b-f457b53a3ced">

This is a BIG table! Let's break it down:

- The first column, "Curr", is the current state of our circuit. It can be S<sub>0</sub>, S<sub>1</sub> or S<sub>2</sub>.
- The next 2 columns, Q<sub>1</sub> and Q<sub>0</sub>, are the flip-flop values corresponding to our current state in the first column. Note that since we don't care when both of them are 1, the last 2 rows are filled with Xs.
- The next column, "x", is the next digit of our input. By putting x in, it may change the state of our circuit.
- The next column, "Next", represents that state that our circuit will be in AFTER reading the digit in the previous column. So for example, in the first row, when we are currently in state S<sub>0</sub>, if we read in a 0, we will still be in state S<sub>0</sub>.
- The next 2 columns, Q<sub>1</sub>' and Q<sub>0</sub>', are the flip-flop values corresponding to the state specified in the "Next" column.
- The next 4 columns, J<sub>1</sub>, K<sub>1</sub>, J<sub>0</sub> and K<sub>0</sub>, are the J and K values needed to transform Q<sub>1</sub> and Q<sub>0</sub> values to Q<sub>1</sub>' and Q<sub>0</sub>' values. The subscript of each J and K represents which flip-flop they belong to.
- The final column, F, represent whether the CURRENT ("Curr" column, not "Next") state is the accepting state S<sub>0</sub>. If it is S<sub>0</sub>, F is 1 and otherwise F is 0.

With this table, we can see how a combination of Q<sub>1</sub>, Q<sub>0</sub> and x (input) can lead to a combination of J<sub>1</sub>, K<sub>1</sub>, J<sub>0</sub>, K<sub>0</sub> and F (output). In order to map these inputs to our desired outputs, we will use K-Maps.

### 4. K-Maps

A K-Map looks like this:

<img width="361" alt="image" src="https://github.com/mb-linh/lab-5-blog-post-group1_cs281/assets/97915038/14af9ad7-38ab-4a0e-a6ec-317f7de014b9">

This is the K-Map for J<sub>1</sub>. We can see that our inputs Q<sub>1</sub>, Q<sub>0</sub> and x create a 2D matrix, with Q<sub>1</sub> on one side and Q<sub>0</sub> and x combined on the other side. We will fill in the value of J<sub>1</sub> based on our function table above. So for example, when the inputs are all 0, J<sub>1</sub> is 0. So 0 is filled in the cell corresponding to the input values. Note that when you have more than 2 inputs on a side (our Q<sub>0</sub> and x in this example), you should only change 1 bit at a time going from row to row (or column to column in our example). Thus, our Q<sub>0</sub> and x values go from 00 to 01 (x change), then 11 (Q<sub>0</sub> change), then 10 (x change) instead of the familiar 00, 01, 10, 11 sequence.

After filling out the K-Map, our goal is to draw rectangles that will be our boolean expression that represent that particular input-output combination. The rules for drawing rectangles are as follow:
- All 1s in the K-Map must be in a rectangle.
- Rectangles can overlap.
- The rectangle size must be powers of 2.
- You should draw as big rectangles as possible, and use as few rectangles as possible.
- Rectangles can "wrap around" both horizontally and vertically, meaning that, for example, the furthest right cell is actually adjacent to the furthest left cell.
- "Don't cares" (X) are wild-cards: you can treat them as 1s if they help you draw bigger rectangles, or 0 if they don't.

So in the example above, we only have 1 cell with the value 1. We can technically just use a 1x1 square to cover that 1, but we can use the X below it to create a bigger 1x2 rectangle. The remaining Xs are unhelpful, so we leave them out.

After drawing the rectangles, we will create our boolean expression for our output, which in this case is J<sub>1</sub>. In the top cell, the inputs are ~Q<sub>1</sub>Q<sub>0</sub>~x, and in the bottom cell the inputs are Q<sub>1</sub>Q<sub>0</sub>~x, so our boolean expression is ~Q<sub>1</sub>Q<sub>0</sub>~x + Q<sub>1</sub>Q<sub>0</sub>~x = (~Q<sub>1</sub> + Q<sub>1</sub>)Q<sub>0</sub>~x = Q<sub>0</sub>~x.

Now draw the remaining K-Maps and create boolean expressions for the remaining outputs K<sub>1</sub>, J<sub>0</sub>, K<sub>0</sub> and F. If you did your work correctly, it should look like this:

<img width="356" alt="image" src="https://github.com/mb-linh/lab-5-blog-post-group1_cs281/assets/97915038/c5bc15be-c403-440f-9dd0-54ee65d2eb4b">

<img width="359" alt="image" src="https://github.com/mb-linh/lab-5-blog-post-group1_cs281/assets/97915038/68afa66e-0ed4-47c7-8f63-2ed87af42531">

<img width="386" alt="image" src="https://github.com/mb-linh/lab-5-blog-post-group1_cs281/assets/97915038/1f7d5949-8f0a-460c-8bc3-9e987e369bc2">

<img width="325" alt="image" src="https://github.com/mb-linh/lab-5-blog-post-group1_cs281/assets/97915038/c81b005b-292b-43de-8c3a-4259fb14e77c">

And the boolean expressions should look like this:

<img width="287" alt="image" src="https://github.com/mb-linh/lab-5-blog-post-group1_cs281/assets/97915038/9d2f8e04-46a4-456a-a099-d63063f3e2df">

You figured out all the necessary logic to build our circuit! Time to test it with Logisim.

### 5. Building our circuit

If we look at the boolean expression for J<sub>0</sub>, it is essentially Q<sub>1</sub> xor x. Besides that, we just need an output pin for F, and everything else is pretty straight forward. Your Logisim circuit should look like this:

<img width="832" alt="Screenshot 2024-03-06 at 6 43 00 PM" src="https://github.com/mb-linh/lab-5-blog-post-group1_cs281/assets/97915038/a9bf8e68-c44f-4255-ac77-aff4e5b8f4c1">

There are some things on the circuit that might look unfamiliar to you. First is a pin called "clock". What this does is that it ensures the state of the flip-flop is updated only when the clock is high. This allows us determine when to update our states. Another unfamiliar pin is the "reset" pin. In both the Logisim flip-flop and the real-life flip-flop, there are 2 pins called "Preset" and "Clear". The former ensures that the flip-flop stays on 1, while the latter ensures that the flip-flop stays on 0. We have the "reset" pin wired to the "Clear" pin on both flip-flops so that when we need to reset our circuit, we can just toggle that "reset" pin. The values of the 2 flip-flops will be set back to both 0. For our purposes, we don't need to wire the "Preset" pins.

Try passing a couple of inputs in your Logisim model to see if the F output pin lights up when expected. When you're ready, we will build the real thing!

## Testing

Several things to note when building our real-life circuit:

- The 7476 IC chip has inverted "Preset" and "Clear" pins, which means that we have to wire the "Clear" pin to a logic switch set to HIGH by default (We use switch S<sub>8</sub>). When we want to reset the circuit, toggle the switch to LOW, then back to HIGH again. The layout of the 7476 chip is as follow (each chip contains 2 J-K flip-flops, so we will only need 1 chip for our circuit):
<img width="207" alt="image" src="https://github.com/mb-linh/lab-5-blog-post-group1_cs281/assets/97915038/4fa3c6fd-9761-4592-b76d-e92f7506aef3">

- Our clock will be 1 of the 2 debounced pushbuttons on the breadboard. Explaining how they work is outside the scope of this lab, but what you need to do is use the NC (Normally Closed) pins on the button. Use a resistor and wire one end to +5V on the breadboard, and the other end to the button. Doing this ensures that when the button is depressed, it sends a LOW signal and when it is pressed the signal will be HIGH. You can use a wire and plug the button straight to the clock pin on the chip. Whenever you want to update the state of the flip-flops, just press the button.

- We will use switch S<sub>1</sub> as our input x. When testing, we will set our desired value of x, then press the button to pass it into the circuit.

That's it! Here is the video demonstrating our circuit at work:

https://github.com/mb-linh/lab-5-blog-post-group1_cs281/assets/97915038/3dc7c9a4-ad4d-4897-9bc8-94cc8db99eca

## Conclusion

This is a hefty lab with different components, either on paper, on the computer or in real-life. However, it also introduces us to a lot of new concepts, which all fall under the sequential circuit design process. We get to know the J-K flip-flop, drawing K-Maps to simplify boolean expressions, and building a reasonably complicated circuit. We hope this has been helpful for you, and please give it a try!




