# Introduction

## Why read this book
Software design is a craft not a science. To master it you cannot just study a book that explains comprehensively all the theory behind it.
You have to read the literature of many great developers, looking for advices that repeat: that's what you then consider good software design.
However, the literature is vast, and it takes much time to get the big picture out of single sources.
This book makes three promises:
  * to be a catalogue of everything that constitutes good software design
  * to be very succinct
  * to mention trustworthy sources to go deeper 

## Who should read this book
The book is intended mainly for people who just started their journey into software design or who are in the middle of it.
If you have being seriously studying the topic for 5+ years, likely you are not going to find big surprises. However, I still 
suggest you to go through the table of contents: you might find one or two learnings worth a quick read. 
As a last consideration, the book is heavily skewed towards object-oriented programming though many advices apply to all [programming paradigms](http://www.cs.albany.edu/~sdc/CSI500/Downloads/ProgrammingParadigmsVanRoyChapter.pdf)

## Teach me back
I really appreciate any feedback about the book and my current understanding of software design.
You can share any feedback by creating a pull request.
If you do not know what a pull request is, take a look here [here](https://guides.github.com/activities/hello-world/#:~:text=Pull%20Requests%20are%20the%20heart,merge%20them%20into%20their%20branch.&text=You%20can%20even%20open%20pull,repository%20and%20merge%20them%20yourself.).

## Who I am
My name is Matteo Di Tucci, I currently work at ThoughtWorks.
I am a developer by accident. I picked up coding in university because I did not know what to do when growing up. 
I still do not know, but along the way I pleasantly discovered I like software and its design.

# Simple code
It is hard to define what good design looks like: nobody was able to come up with an unambiguous definition so far. 
The thing that came closer to that is the [C2 Wiki](https://wiki.c2.com/?XpSimplicityRules) simple code rules.  
Simple code:

* Passes all tests
* Expresses intent
* Does not repeat itself
* Does not contain superfluous parts

If a piece of code respects all those 4 rules then we can consider it good code.  
This book is divided in 4 sections: one for each the simple code rules.

# Passes all tests

### Test pyramid
### If testing is hard, inject state or collaborators
### Mock vs stub vs spy
### Test driven design (TDD)
### Test naming
### Test coverage is not enough: parameterised tests
### Tests must be reproducible (no Math.random() or LocalDate.now())
### Dry vs moist tests
### Outside-in vs inside-out TDD
### Classical vs mockist TDD
### Component tests vs end-to-end tests vs monitoring tradeoffs
### Contract testing
### Wrap external library
### Do not use production constants
### Performance tests
### Linting

# Expresses intent
### Naming
### Data structures over algorithm
### Small classes and short methods
### SOLID principles
### Usually composition is better than inheritance
### Reduce edge cases
### Narrow and deep public methods
### Immutability
### Do not over-engineer (YAGNI)
### Comments the why
### Visual indentation (kevlin Henney)
### Folder structure

# Does not repeat itself
### One single authoritative knowledge representation
### Encapsulation
### Do not abstract by pattern matching
### Polymorphism

# Does not contain superfluous parts
### Wrong abstraction is more costly than duplication
### Do not abuse design patterns
