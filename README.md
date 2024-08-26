# Appendix for "Reevaluating the Defect Proneness of Atoms of Confusion in Java Systems"
You can find [a list of AoC types](#list-of-aoc-types), [what a human-readable AST is](#what-is-a-human-visible-ast-why-do-we-need-it), [project and language factors in AoC](#project--and-language-specific-factors-in-aoc) and [a discussion on the relation between AoC and technical debt](#are-aocs-related-to-technical-debt) in this appendix.

## List of AoC Types
Note: our study only concerns the AoC types marked with `*`. We group the __Removed Indentation__ AoC with the __Indentation__ AoC, and omit __Arithmetic as Logic__, __Constant Variables__, and __Dead, Unreachable, Repeated__ since Langhout and Aniche [[3]](#3) Gopstein et al. [[2]](#2) considered them as not confusing.
| AoC Types | Abbr. | Example | Transformed |
| - | - | - | - |
| Arithmetic as Logic | AL | `(n - 3) * (m - 4) != 0` | `n != 3 && m != 4` |
| *Change of Literal Encoding | CLE | `int n = 5 & 11;` | `int n = 0b0101 & 0b1011` |
| *Conditional Operator | CO | `int n = (m == 3) ? 2 : 1;` | <pre>int n;<br>if (m == 3) {<br>&nbsp;&nbsp;n = 2;<br>} else {<br>&nbsp;&nbsp;n = 1;<br>}</pre> |
| Constant Variables | CV | `n = m;` | `n = 5;` |
| Dead, Unreachable, Repeated | DUR | `n = 1; n = 2;` | `n = 2` |
| *Indentation | Ind | <pre>int m, n;<br>if (n == 2)<br>&nbsp;&nbsp;m = 3;<br>&nbsp;&nbsp;m = 1;</pre> | <pre>int m, n;<br>if (n == 2)<br>&nbsp;&nbsp;m = 3;<br>m = 1;</pre> |
| *Infix Operator Precedence | IOP | `int n = 2 - 4 / 2` | `int n = 2 - (4 / 2)` |
| *Logic as Control Flow | LCF | `boolean test = n == m && ++n > 0 \|\| ++m > 0;` | <pre>if (n == m) {<br>&nbsp;&nbsp;++n;<br>} else {<br>&nbsp;&nbsp;++m;<br>}</pre> |
| *Ommitted Curly Braces | OCB | `if(n <= 0) n++; n++;` | `if(n <= 0) {n++;} n++;` |
| *Post Increment/Decrement | PostID | <pre>int n = 2;<br>int m = 3 + n++;</pre> | <pre>int n = 2;<br>int m = n + 3;<br>n++;</pre> |
| *Pre Increment/Decrement | PostID | <pre>int n = 2;<br>int m = ++n - 2;</pre> | <pre>int n = 2;<br>n = n + 1;<br>int m = n - 2;</pre> |
| Removed Indentation | RI | <pre>while (m > 0)<br>&nbsp;&nbsp;m--;<br>&nbsp;&nbsp;n++;</pre> | <pre>while (m > 0)<br>&nbsp;&nbsp;m--;<br>n++;</pre> |
| *Repurposed Variables | RV | <pre>int v1[] = new int[5];<br>v1[4] = 3;<br><br>while (v1[4] > 0) {<br>&nbsp;&nbsp;v1[3 - v1[4]] = v1[4];<br>&nbsp;&nbsp;v1[4] = v1[4] - 1;<br>}</pre> | <pre>int v1[] = new int[5];<br>int n = 5;<br><br>while (n > 0) {<br>&nbsp;&nbsp;v1[3 - v1[4]] = v1[4];<br>&nbsp;&nbsp;a = a - 1;<br>}</pre> |
| *Type Conversion | TC | <pre>int n = 4;<br>char c = (char) n;</pre> | <pre>int n = 4;<br>char c = Character.forDigit(n, 10);</pre> |

## What is a Human-Visible AST? Why do we need it?
Gopstein *ea.* [[2]](#2) searched for AoC described in the following way:
> We searched for atoms in the portion of the abstract syntax tree (AST) which is visible in the source (e.g. we do not search parts of the tree generated by the preprocessor). We also search code that doesn’t directly result in AST nodes (e.g. preprocessor directive definitions). We call this slightly modified version of the AST the ‘human-visible AST’, as it is meant to replicate what a programmer can see directly when reading a program.

Their analyses deals with ratios, and project size should not affect these anlysis. Therefore, the count of AoC is divided by the number of AST nodes to mitigate the effect of commit size on the results.

To the best of our judgement, the Java Lexer tokens resemble C/C++ AST nodes the most, and they are also available to use with the Java AoC detection tool [[3]](#3). Therefore, we choose the Java Lexer tokens to mitigate the effect of PR size.

## Project- and Language-specific Factors in AoC
In this section, we list a few examples of AoC in different [projects](#project-specific-examples) and [languages](#language-specific-examples) to elaborate on how they affect the appearances of AoCs.

### Project-specific examples
In this section, we show that developers follow project-specific guidelines and conventions, thus affecting which AoCs are changed more often.
Here, we show one example from the [Elasticsearch](https://github.com/elastic/elasticsearch) project and another from the [Shardingsphere](https://github.com/apache/shardingsphere) project.

In our paper, we discover that the developers in the Elasticsearch project make frequent changes to the Conditional Operator AoC.
To be specific, the project use the function `randomBoolean()` which, as the name suggests, generates a random Boolean value, as the condition in a ternary conditional operator expression.
Such ternary expressions are frequently used in their testing files. In other words, the Conditional Operator AoCs help the developer with their debugging.
The process of debugging itself is already lengthy and difficult, and the use of ternary operators helps shorten the process.
With proper formatting, it would not be as confusing.

As for the Shardingsphere project, they had frequent changes to the Pre-Increment/Decrement AoCs, but mostly to two expressions: `++sequenceID` and `++currentSequenceID`, which were all removed during a major refactoring of the code base.

The two examples above show that when considering the relations between AoC changes and other factors, we should also consider project-specific factors.

### Language-specific examples
In this section, we show that the appearances of AoCs differ due to different languages.
Here, we show one example in the Apache's [Kafka](https://github.com/apache/kafka) project which is written primarily in Java, and another from Gopstein et al.'s study [[2]](#2)

In PR [#11393](https://github.com/apache/kafka/pull/11393) of the Kafka project, a [discussion](https://github.com/apache/kafka/pull/11393#discussion_r758432812) revolved around code such as the following:

<img width="700" src="https://github.com/user-attachments/assets/aa72c21a-72d5-46a1-adc2-cea28e4b6d56">

Eventually, they decide to change the data type of parameters `version` to `short` to prevent casting from `short` to `int`.
However, the Type Casting AoCs was probably not added "accidentally". 
In PR [#12135](https://github.com/apache/kafka/pull/12135), a developer [justified](https://github.com/apache/kafka/pull/12135#discussion_r869499993) added such a conversion to the codebase as follows:

<img width="590" src="https://github.com/user-attachments/assets/4aa51175-0bf2-496c-8a7e-3b083185d201">

and they justified their choice of this "unsafe type conversion":

> This is because Java's default type is int -- i.e. line 36/37 above would take the value 0/1 as int. So we basically need to either do the conversion in each line of 36/37 above, or just do the conversion once here.

This goes to show that the Type Conversion AoCs would occur more frequently in Java projects.

On the other hand, in C/C++ projects, the Macro Operator Precedence AoCs appear often.
Gopstein et al. [[2]](#2) described in their paper that the following macro was frequently used in Linux:
```
#define ABS(x) ((x) < 0? (-x) : (x))
```
This is logically incorrect since, when `x` is an expression instead of a number, the result would be incorrect. For example, let `x = 1-2`. The corect result would be 1, but if we substitute `x` into the expression above, it looks like
```
#define ABS(1-2) ((1-2) < 0? (-1-2) : (1-2))
```
which will evaluate to -3. This goes to show that the Macro Operator Precedence AoCs would occur more frequently in C/C++ projects.

## Are AoCs related to technical debt?
It has been statistically shown that AoCs are usually preserved after PRs[[1]](#1), and we have shown that the presence of AoCs is not correlated with the presence of bugs.
Therefore, it is unlikely that AoCs are related to technical debt.

Let us consider two things.
First, AoCs are not defects themselves.
They are just snippets of code that are **potentially** confusing to developers.
Developers do not need to invest large amounts of resources to make changes to them, since they do not pose actual threats to the software system.
Second, as the open-source project evolves, and developers become more senior and familiar with the repository, it is less likely that they would consider AoCs an issue, as discussed above.
In turn, it is unlikely that they would invest resources to remove them.

Hence, we claim that AoCs are not related to technical debt.

### References
<a id="1"[1]</a>
V. Bogachenkova, L. Nguyen, F. Ebert, A. Serebrenik and F. Castor. 2022.
Evaluating Atoms of Confusion in the Context of Code Reviews.
2022 IEEE International Conference on Software Maintenance and Evolution (ICSME).
Limassol, Cyprus, 2022, pp. 404-408.
https://doi.org/10.1109/ICSME55016.2022.00048.

<a id="2">[2]</a>
D. Gopstein, H. Zhou, P. Frankl, and J. Cappos. 2018.
Prevalence of Confusing Code in Software Projects: Atoms of Confusion in the Wild.
In Proceedings of the 15th International Conference on Mining Software Repositories (Gothenburg, Sweden) (MSR ’18).
Association for Computing Machinery, New York, NY, USA, 281–291.
https://doi.org/10.1145/3196398.3196432

<a id="3">[3]</a>
C. Langhout and M. Aniche. 2021. Atoms of Confusion in Java.
In 2021 IEEE/ACM 29th International Conference on Program Comprehension (ICPC).
IEEE, New York, NY, USA, 25–35.
https://doi.org/10.1109/ICPC52881.2021.00012


