---
title: "Advanced Maven Reactor use-cases" # todo revisit title (use-cases -> edge cases?)
url: advanced-maven-reactor-use-cases
date: 2021-06-30T20:14:00+02:00 # todo
draft: true
---

For all folks using Maven, and especially those who use multi module projects, Maven 4 will be a treat for you.
I had the opportunity to work together with [@mthmulders](https://twitter.com/mthmulders) on several issues and bug fixes around the Maven reactor.
The reactor is the backbone of multi module projects in Maven. 
Most of the things you need to know about the reactor is [documented here](https://maven.apache.org/guides/mini/guide-multiple-modules-4.html).
In this article I would like to show you the real advanced use-cases. 
It's a collection of use-cases of which some are useful, some are rarely needed and some are just so exotic you should question yourself what you are actually doing.

# Maven Reactor

In Maven, you can control most of how Maven builds multi module projects using CLI flags. 
Most of the use-cases you will find below are based on the combination of those flags. 
First, a short summary of the relevant ones (taken from the [original docs](https://maven.apache.org/guides/mini/guide-multiple-modules-4.html)):

| Long | Short | Description |
|:--|:--|---|
| `--file` | `-f` | Selects an alternative POM file or directory containing a POM file. |
| `--projects` | `-pl` | Specifies the modules to include or exclude from a build. |
| `--also-make` | `-am` |	Builds the specified modules, and any of their dependencies in the reactor. |
| `--also-make-dependents` | `-amd` | Builds the specified modules, and any that depend on them. |
| `--resume-from` | `-rf` | Resumes a reactor from the specified project (e.g. when it fails in the middle). |

# Project structure

The use-cases below will take the following project structure in mind. 
To keep it easy, the same project structure is used as the [original docs](https://maven.apache.org/guides/mini/guide-multiple-modules-4.html).

![Multi module project structure](/images/posts/maven-reactor-advanced-use-cases-project.png "Multi module project structure")

# Use-cases

## 1. Building modules with their relevant inter-project dependencies

We start off simple. 
This is most likely one of the more common and useful scenarios.

When working on big multi module projects, some developers have grown custom to get coffee every time the project builds. 
Usually there is plenty of time, after all!
Even if only one module changed, they do that. 
But, among other reasons (such as taking care of their caffeine levels), people should use certain CLI flags to optimize this process.

A single module can easily be built by using the `-f` or `-pl` flags.
But, modules in a project often depend on others... which also should be rebuilt to include the newly built module.

Maven luckily can easily take care of that for you: `-am` and `-amd` are created for that. 

### Examples
1. `mvn <goal> -pl module-c-2 -am`  
  results in Maven building `parent`, `module-c`, `module-a`, `module-b` and finally `module-c-2`.

2. `mvn <goal> -pl module-b -amd`  
  results in Maven building `module-b` and then `module-c-2`.

## 2. Excluding the project we are resuming from

When a multi-module project fails building somewhere halfway, Maven 3 shows a hint how to continue the build:

```
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <args> -rf :<artifact-id>
```

In Maven 4, Maven keeps a record of the last failed module and allows you to resume the build from the last failed module using just `mvn <args> -r`.

But this use-case is not about that, because that would not be an advanced scenario. 
This one is about skipping the failed module and continuing the rest of the build. 
While I can imagine some scenarios where it can be useful, if the module is used as a dependency from later modules, this is discouraged.
It could result in the later module including an old artifact from the local repository as dependency.

### Examples

1. `mvn <goal> -rf :module-b -pl !:module-b`  
  results in Maven building just `module-c-2`.

2. _(Assuming `module-b` failed in an earlier run)_  
  `mvn <goal> -r -pl !:module-b`  
  results in Maven building just `module-c-2`.

## 3. Excluding a module activated by also-make does take its transitive dependencies

It gets trickier here. 

Remember that `--also-make` or `-am` also builds all inter-module dependencies of a selected module?
Because the reactor first handles including modules (in several ways), and finally excludes modules, it's possible to exclude a dependency in the transitive (multi-module) dependency graph.
The examples below expand on the first example of use-case 1 (combining `-pl` and `-am`).

Now, why this is useful... I don't know. I haven't encountered a valid use-case for it yet. 
But since Maven is used in so many different forms, I'm sure it's useful somewhere! 

### Examples
1. `mvn <goal> -pl :module-c-2,!:module-b -am`  
  results in Maven building `parent`, `module-c`, `module-a` and finally `module-c-2`.

2. `mvn <goal> -rf :module-c-2 -pl !:module-b -am`  
  also results in Maven building `parent`, `module-c`, `module-a` and finally `module-c-2`.

# Conclusion

In this article I showed you how to combine several CLI flags to unlock advanced behavior in the Maven reactor. 
Even though it might not be applicable directly, perhaps it helped you understand the way how the Maven parses the flags. 
All of these exotic cases, but also the usual ones, are [covered in unit tests](https://github.com/apache/maven/blob/master/maven-core/src/test/java/org/apache/maven/graph/DefaultGraphBuilderTest.java#L73), which should be pretty easy to interpret.
Can you find other exotic ones? 
Please let us know if we are still missing test cases! 
Either in a PM, JIRA issue or a direct PR ;) 

# TODO - fix syntax highlighting of single-line code snippets