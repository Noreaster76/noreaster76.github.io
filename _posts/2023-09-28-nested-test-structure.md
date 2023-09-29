---
title: "Nested Test Structure FTW"
layout: default
excerpt_separator: <!--more-->
---

# Nested Test Structure FTW

In this blog post, I argue that unit tests should be written using a nested structure, rather than a flat one.
In short, doing so allows you to much more clearly cover all possible cases, as well as DRY out your setup code.

<!--more-->

## A stipulation

Let me first stipulate that your application's business logic (not the UI logic -- that's another discussion) needs as full branch coverage as possible. That is, every single branch of logic in your core business logic needs to have a corresponding, thorough unit test. After all:

**Someone is going to test your code. Who do you want that to be? You? Or your users?**

We are solely going to discuss unit testing here. This likely does not apply to higher level testing, such as integration testing, functional testing, system testing, UI testing, end-to-end testing, and so on. Those are at higher levels of the automated testing pyramid and therefore unlikely cannot justify the cost of testing every single scenario. (Nevertheless, the other benefits I state below would still apply.)

Now, let's first discuss flat test structure, which I'm pretty sure is the prevailing format in most software circles.

## What is flat test structure?

When writing unit tests, most of us probably use a flat test structure. All the unit tests are at the same level of indentation, and as the logic being tested grows more complex, so do the descriptions of said tests. As an example, let's say you are trying to add coverage to fizzbuzz.

```ruby
"Given an integer, when the integer is divisible by neither 3 nor 5, print the integer"
"Given an integer, when the integer is divisible by 3 and not 5, print 'fizz'"
"Given an integer, when the integer is divisible by 5 and not 3, print 'buzz'"
"Given an integer, when the integer is divisible by both 3 and 5, print 'fizzbuzz'"
```

(I'll try to keep things in pseudocode as much as possible, so as to resonate with as many of us engineers as possible, regardless of coding language.) With a flat structure, the tests would look something like:

```ruby
testInputIsDivisibleByNeither3Nor5 {
  assert(fizzbuzz(4) == '4')
}
testInputIsDivisibleBy3ButNot5 {
  assert(fizzbuzz(9) == 'fizz')
}
testInputIsDivisibleBy5ButNot3 {
  assert(fizzbuzz(10) == 'buzz')
}
testInputIsDivisibleByBoth3And5 {
  assert(fizzbuzz(45) == 'fizzbuzz')
}
```

I suppose that's manageable, but keep in mind that we're dealing with a very simple system under test that consists of just a couple of logical branches. What happens if the business logic grows more complex?

```ruby
/* replace this requirement */
"Given an integer, when the integer is divisible by both 3 and 5, print 'fizzbuzz'"

/* with these two requirements */
"Given an integer, when the integer is divisible by both 3 and 5 and is odd, print 'fizzbuzz'"
"Given an integer, when the integer is divisible by both 3 and 5 and is even, print the integer"

/* and add this one */
"Given an integer, when the integer is divisible by neither 3 nor 5 but is divisible by 7, print 'seven'"
```

Then you would have:

```ruby
testInputIsDivisibleByNeither3Nor5Nor7 {
  assert(fizzbuzz(4) == '4')
}
testInputIsDivisibleByNeither3Nor5ButIsDivisibleBy7 {
  assert(fizzbuzz(28) == 'seven')
}
testInputIsDivisibleBy3ButNot5 {
  assert(fizzbuzz(9) == 'fizz')
}
testInputIsDivisibleBy5ButNot3 {
  assert(fizzbuzz(10) == 'buzz')
}
testInputIsDivisibleByBoth3And5AndIsOdd {
  assert(fizzbuzz(45) == 'fizzbuzz')
}
testInputIsDivisibleByBoth3And5AndIsEven {
  assert(fizzbuzz(90) == '90')
}
```

This is becoming a mess. How do we ascertain that we have covered all cases? In addition, is this scalable? As the system under test grows more complex, and requires more test harness setup, are we going to be able to keep our tests clean and understandable, so that we can revisit them later and easily perform necessary maintenance or alter the logic?

Moreover, what if our `fizzbuzz` method is only one of several in our production class? That is, what if there are several other production methods that we also have to test? Certainly, one option is to segregate the tests for each production method into their own file. And that's not a bad option, but it's certainly not widely practiced, and it also doesn't solve the readability problem mentioned above.

(Remember, the system under test above is super simple, and requires no setup code. The pseudocode above is also quite terse. Just imagine if this were for a reasonably complex method, required a bunch of setup, and written in a verbose language like, say, Java! In addition, notice that the test names above only describe the test scenario and not the expected result -- imagine how much longer and more difficult they would be to read if they also described the expected result, such as `testInputIsDivisibleByBoth3And5AndIsEvenReturnsTheInput`!)

Flat test structure is in fact *not* scalable. Additionally, and just as importantly, it quickly becomes difficult to discern whether all logical branches have been covered.

## What is nested test structure?

As you may have seen in BDD-compatible testing frameworks such as JUnit 5 (Java), Quick (Swift), RSpec (Ruby; perhaps the progenitor of this testing style), Spek (Kotlin), or ScalaTest (take a guess), a nested test structure enables us to nest our tests using as many levels as we want; descriptive annotations (such as `describe` or `context`) and setup state (often implemented in something like a `beforeEach` closure) impact not only the current node of the tree, but also all subnodes as well. Here's an example:

```ruby
describe('fizzbuzz()') {

  describe('when the input is divisible by 3') {
    describe('but the input is not divisible by 5') {
      it('returns fizz') {
        assert(fizzbuzz(3) == 'fizz')
      }
    }
    describe('and the input is divisible by 5') {
      describe('and the input is odd') {
        it('returns fizzbuzz') {
          assert(fizzbuzz(45) == 'fizzbuzz')
        }
      }
      describe('and the input is even') {
        it('returns the input') {
          assert(fizzbuzz(90) == '90')
        }
      }
    }
  }
  describe('when the input is not divisible by 3') {
    describe('but the input is divisible by 5') {
      it('returns buzz') {
        assert(fizzbuzz(5) == 'buzz')
      }
    }
    describe('and the input is not divisible by 5') {
      describe('but the input is divisible by 7') {
        it('returns seven') {
          assert(fizzbuzz(7) == 'seven')
        }
      }
      describe('and the input is not divisible by 7') {
        it('returns the input') {
          assert(fizzbuzz(8) == '8')
        }
      }
    }
  }

}
```

(Note that the actual syntax will differ, depending on the language and BDD framework. The above is simply a general gist that presents the same tests as before, but in a nested structure.)

Yes, it takes a little getting used to. After all, what are these free-form strings that follow the `describe` keywords above? And what's this `it`? I will leave those questions to the docs of your BDD framework of choice, since they don't all use the exact same terms, but there are actually several benefits.

## Why should you use nested?

### Better organization

It's an easy way to segregate method A's tests from method B's tests:

```ruby
describe('MyObject') {

  describe('#methodA()') {
    /* any setup and tests for methodA go here */
  }

  describe('#methodB()') {
    /* any setup and tests for methodB go here */
  }
  
  describe('#methodC()') {
    /* any setup and tests for methodC go here */
  }
  
}
```
(Certainly, you could stop here and just use a flat test structure within this way of grouping tests together, but doing so robs you of the real power of nested tests.) 
Keeping all tests segregated by method facilitates moving those tests to another class later on; you don't have to hunt and peck through your test file for where you're testing `methodA`, for example.

Let's keep going.

### Improved readability

The next three lines that follow the method name are as follows:

```ruby
describe('when the input is divisible by 3') {
  describe('but the input is not divisible by 5') {
    it('returns fizz') {
```

If we snip everything out except what is between quotes, we get the following (more or less):

```
when the input is divisible by 3
  but the input is not divisible by 5
    'it' [in this case, the fizzbuzz method!] returns 'fizz'
```

**That's a nicely legible, English sentence, which describes exactly the behavior we are looking for!** In addition, when reporting on the test run (say, to report a test failure), whatever BDD framework or testing library you're using will usually concatenate those clauses together in the same fashion. So once you have gotten the hang of reading the tests by visually walking the tree, not only will you have a legible test file, you'll also have a legible test report:

```
TEST FAILED:

fizzbuzz() when the input is not divisible by 3 and the input is not divisible by 5 but the input is divisible by 7 returns seven

Expected: seven
Actual: 45
```

Or something to that effect.

Furthermore, if your requirements are expressed in [Gherkin language](https://en.wikipedia.org/wiki/Cucumber_(software)#Gherkin_language) (the "given/when/then" structure), they happily map directly to the test code!

### More certainty of covering all cases

At each level of the tree, your contexts (the `describe` blocks, often) fully exhaust all branches of some logical test in your code.

#### If the logical test is a boolean...

Let's say you're testing a branch of logic in your code that only has two possible outcomes, like whether a request is valid or not. That gets expressed easily as `when the request is valid` and its converse, `when the request is not valid`, say.

For example:

```ruby
describe('#myMethod()') {
  
  describe('when the request is valid') {
    beforeEach { /* perform some setup that sets up a valid request */ }
    
    /* here go all the tests that apply to when the request is valid */
    it('calls out to XYZ service') { /* test goes here */ }
    it('logs a request to Grafana') { /* test goes here */ }
  }
  
  describe('when the request is not valid') {
    beforeEach { /* perform some setup that sets up an invalid request */ }
    
    /* here go all the tests that apply to when the request is invalid */
    it('throws a NotReadyException') { /* test goes here */ }
    it('logs an error') { /* test goes here */ }
  }
}
```

#### If the logical test has three or more possible outcomes...

Let's say you're using a `switch` or `case` statement to branch on various values of an enum, or let's say there are a few different kinds of exceptions that can be thrown from your method. In this case you would similarly need the corresponding contexts in your tests. But since you're only exploring one question at each level of the tree, again, it's much easier to keep track of whether you have all possible outcomes covered.

```ruby
/* let's say there's enum that can have values ALICE, BOB, and CHARLIE */

describe('#myMethod()') {

  describe('when the input is ALICE') {
    beforeEach { input = 'ALICE' }
    it('moos the hoojamadinga') { /* test goes here */ }
  }
  describe('when the input is BOB') {
    beforeEach { input = 'BOB' }
    it('says rah rah') { /* test goes here */ }
  }
  describe('when the input is CHARLIE') {
    beforeEach { input = 'CHARLIE' }
    it('throws an InvalidInputException') { /* test goes here */ }
  }

}
```

(Again, do make sure to cover all outcomes in your unit tests! You don't want to be that engineer who is fighting a production fire because, well, "I didn't think that that case was very likely or important.")

### Duplication of setup code is eradicated

All tests in a flat test structure share the same `setup` or `beforeEach` block. That means that any differentiation you need in your test scenarios needs to be within the tests themselves, leading to a lot of duplication! Setup code in a nested test structure, on the other hand, applies only to its containing scope and to any scopes nested within:

```ruby
describe('#myMethod()') {
  
  describe('when the request is valid') {
    beforeEach { /* perform some setup that sets up a valid request */ }
    
    describe('but the widgets are not ready') {
      beforeEach {
          /* perform some setup that reflects widgets that are not ready;
           * but the beauty is that we don't have to repeat setting up the valid
           * request, since we inherit it from the parent scope
           */
      }
      
      it('throws a WidgetNotReadyException') { /* test goes here */ }
    }
    
    describe('and the widgets are ready') {
      beforeEach {
          /* perform some setup that reflects widgets that are ready;
           * but the beauty is that we don't have to repeat setting up the valid
           * request, since we inherit it from the parent scope
           */
      }
      
      it('foobars the thingamabob') { /* test goes here */ }
    }
  }
  
  describe('when the request is not valid') {
    beforeEach { /* perform some setup that sets up an invalid request */ }
    
    /* here you put the tests that apply to when the request is invalid;
     * if you need to explore some other logic that only applies here, then
     * you have the ability to nest said tests within this node, yet inherit
     * the invalid request setup code from above
     */
  }
}
```

#### For Best Results
- Make sure that each level of `describe` statements ("when ABC is true" and "when ABC is not true", for example) covers just a single logical branch. If you have more than one logic clause in your `describe` statement ("when ABC is true and XYZ has not yet occurred"), then you're trying to do too much at once. (This actually ties in with the next bullet; read on.)
- Make sure that the setup method (`beforeEach` in these examples, but that will likely differ depending on your testing library) contained within each `describe` statement embodies only that which is described in the `describe` statement. Doing so will make it much more straightforward exactly what is getting inherited by subnodes nested within.

## Why should you _not_ use nested?

I was attempting to be a bit even-handed by including this here, but aside from perhaps having a higher learning curve, I seriously cannot think of a decent reason. Flat test structure is really almost like taking a 3-D object (imagine the nested test structure is the 3-D object) and smooshing it or projecting it onto a 2-D surface, thereby losing all of the richness and fidelity of its 3-D volume. Or, more directly related to this discussion, a flat test structure is what you get when you collapse a nested structure into a single level, needlessly foregoing any and all of the power and richness of the latter.

## Beware!

- Avoid mutating the objects you set up for your tests! Instead, treat them as immutable: organize your `setUp`/`beforeEach` blocks so that, for a given walk through the nested structure, each object you need for a given test gets set exactly once.
- Avoid making your tests too complex. If you end up refactoring and extracting a lot of test logic into test helper methods, and your tests grow to be as complex as production code, something is wrong. More or less, you or another engineer should be able to read a test without having to hunt and peck all over the codebase to figure out what the test is doing. Limit indirection, and err on the side of duplicating test code rather than getting fancy and making your test code hard to follow. (Though if you have your nested tests properly structured, then there should in theory be no reason to have to duplicate anything.)

## Examples

You've made it _this_ far? Wow. Strong work. As a reward, here are examples of using nested test structure in some commonly used languages:
- [Java](https://github.com/Noreaster76/nested_test_structure/blob/main/java/src/test/java/org/example/FizzBuzzerTest.java)
- [Kotlin](https://github.com/Noreaster76/nested_test_structure/blob/main/kotlin/src/test/kotlin/org/example/FizzBuzzerTest.kt)
- [Swift](https://github.com/Noreaster76/nested_test_structure/blob/main/swift/SwiftNestedTestStructureExampleTests/FizzBuzzerTest.swift)

## Conclusion

In this blog post, I made a case for structuring your tests in a nested fashion, rather than putting everything at the same level. Though there are several benefits, the most important one is that it makes it much easier to make sure you have enumerated all logical branches from your production code. That is something you owe yourself, your team, your stakeholders, and your users alike.
