# MPC's Background: Millionaires' Problem

> Reference : 《Protocol for Secure Computations》, From Andrew C, Yao In 1982
>
> MPC, which means Multi-Party Computation.

## Introduction 

One sentence to introduce this problem: Two millionaires want to find out who is richer Without leak any additional information about each other's wealth.

This article's structure:

- A precise formulation of Millionaires' problem
- Three ways to solve this problem
   - By using OWF (One-way function,which is the base of private-key cryptography, )

- discuss the problem about the scale (how many bits) of the communication between two parties and methods to prevent participates from cheating.
- Study the problem "What cannot be accomplished with OWF".

## Precise Formulation of Millionaires' problem

The OWF's two kinds of applications when it was proposed:

1. The encryption and transmission of messages to make them unreadable and unalterable for adversary
2. "Mental poker"

