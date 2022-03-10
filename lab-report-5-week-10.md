# Week 10 Lab Report 5: CommonMark

## Table of Contents

1. [Introduction](#1-introduction)
2. [Test 1: `22.md`](#2-test-1-22md)
3. [Test 2: `496.md`](#3-test-2-496md)

## 1. Introduction

Here is a link to my own markdown-parse repository: [click here!](https://github.com/jeremynguyencs/markdown-parse)

Here is a link to the markdown-parse repository that I will be reviewing: [click here!](https://github.com/ucsd-cse15l-w22/markdown-parse)

I found the tests with different results through running a bash script that tested the 652 `commonmark-spec` tests on both my implementation of `markdown-parse` and the `markdown-parse` repository that I will be reviewing. The results were printed to a `results.txt` file. I used the `diff` command to compare the results of the tests.

To determine the correct results for the links of the markdown files, I wrote a class called `LinkVisitor` in Java that uses the `commonmark-java` library and the `AbstractVisitor` class from the `commonmark-java` library. I used the `LinkVisitor` class to visit the nodes of the markdown files and print the links that were found. Here is the code for the `LinkVisitor` class:

`LinkVisitor.java`

```java
public class LinkVisitor extends AbstractVisitor {
  public ArrayList<String> links = new ArrayList<>();

  public LinkVisitor() {
    super();
  }

  public ArrayList<String> getLinks() {
    return links;
  }

  public void visit(Link link) {
    String url = link.getDestination();
    links.add(url);
  }
}
```

## 2. Test 1: `22.md`

Test 1 is the file `22.md` from the CommonMark spec.

The `diff` command produces this result, stating there was a difference in line 404 of the two `results.txt` files:

```
> [baz]
404c404
< [/bar\*]
```

For my own `markdown-parse` repository, here is the output and results of running the test.

```
[/bar\*]
```

For the `markdown-parse` repository that I will be reviewing, here is the output and results of running the test.

```
[]
```

To determine the correct output for parsing the markdown, I created the following JUnit test using the CommonMark library.

`MarkdownParseTest.java`

```java
public void markdownCommonmarkTest1() throws IOException {
  String contents = Files.readString(Path.of("test-files/22.md"));
  ArrayList<String> links = MarkdownParse.getLinks(contents);
  Node document = parser.parse(contents);
  document.accept(visitor);
  ArrayList<String> expected = visitor.getLinks();
  assertEquals("Should have returned links", expected, links);
}
```

Based on the CommonMark implementation, the correct output should be the only item being `/bar*`. The original implementation of `markdown-parse` did not return any items, while mine did, so I think mine is more correct. However, it did not parse it correctly, and returned `/bar\*`. I believe this is because while my implementation of `markdown-parse` did indeed get rid of the `"ti\*tle"` part of the code, it did not process the escaped characters correctly. I would have to implement that functionality to get the correct output. Likely, I would've added that functionality in the `returnLink()` method. The code is shown below.

`MarkdownParse.java`

```java
public static String returnLink(String line) {
  String link = "";
  // get the text between the ( and the )
  link = line.substring(line.indexOf("(") + 1, line.indexOf(")"));
  if (link.contains(" ")) {
    link = link.substring(0, link.indexOf(" "));
  }
  // throw error if link is not formatted correctly
  if (!determineLink(line)) {
    throw new IllegalArgumentException("Link is not formatted correctly");
  }
  return link;
}
```

## 3. Test 2 `496.md`

Test 2 is the file `496.md` from the CommonMark spec.

The `diff` command produces this result, stating there was a difference in line 1322 of the two `results.txt` files:

```
> [foo(and(bar))]
1322c1322

(theirs)
< [foo(and(bar]
```

For my own `markdown-parse` repository, here is the output and results of running the test.

```
[foo(and(bar]
```

For the `markdown-parse` repository that I will be reviewing, here is the output and results of running the test.

```
[]
```

To determine the correct output for parsing the markdown, I created the following JUnit test using the CommonMark library.

`MarkdownParseTest.java`

```java
public void markdownCommonmarkTest2() throws IOException {
  String contents = Files.readString(Path.of("test-files/496.md"));
  ArrayList<String> links = MarkdownParse.getLinks(contents);
  Node document = parser.parse(contents);
  document.accept(visitor);
  ArrayList<String> expected = visitor.getLinks();
  assertEquals("Should have returned links", expected, links);
}
```

Based on the CommonMark implementation, the correct output should be an empty list. This is what the original implementation of `markdown-parse` did, while mine returned a list with a single item of `foo(and(bar`. The issue with my code is that I did not account for multiple parentheses in the link. My code is shown below, and it only gets the index of the first set of parentheses, and does not account for multiple parentheses.

`MarkdownParse.java`

```java
...
link = line.substring(line.indexOf("(") + 1, line.indexOf(")"));
...
```

To resolve this, I would have to implement a function that could parse the line when there are multiple paranthesis and determine whether it should be considered a link with a URL or simply a line of plaintext. In this case, the link is not a link with a URL, so it shouldn't have been added to the list.