# Lab 5: Finite State Machine Design

## Overview and Motivation

In this lab, we will be designing a circuit that reads a binary number and determines if that input is divisible by 3. Sounds a bit tricky, right? Actually, with some careful planning, you can also build your own circuit, and possibly even more complicated ones, as long as you follow our design pipeline! This lab should get you familiar with the finite state machine (FSM) process, introduce you to the J-K flip-flop, drawing K-Maps, and get you some practice on circuit building. Let's get started!

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

## Testing

## Conclusion




