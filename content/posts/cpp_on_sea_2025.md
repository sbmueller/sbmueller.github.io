+++
title = "C++ On Sea 2025 Takeaways"
date = "2025-06-29"

[extra]
comment = false
+++

Recently, I had the opportunity to join the [C++ on Sea](https://cpponsea.uk/)
conference in Folkestone, UK. I had a wonderful time immersing myself further
into the C++ world and learning both new concepts and ideas but also getting to
know a lot of awesome people. Thanks to all the people that talked to me and
made the conference a pleasure, specifically Matt, Jason, Hampus, Prithvi,
Kathleen, Tina, Hendrik, Timur, Walter, Christopher, Vincent, Evgenii and all
others I might have forgotten in this list (shame on me).

Now, I want to highlight my takeaways from the conference.

## Error Handling with Monads

In his talk [Safe and Readable Code: Monadic Operations in
C++23](https://cpponsea.uk/2025/session/safe-and-readable-code-monadic-operations-in-cpp23),
Robert Schimkowitsch introduced the power of using monadic operations in
existing code. I enjoyed the side-by-side examples that highlighted the benefit
over plain `if`-chains, that mix business logic and error handling logic.

Imagine a classic C++ (embedded) code that doesn't rely on exceptions. Often,
we find something like this:

```cpp
bool getIntCellValueNegative
  (CDb db, Key key, CLocation location, bool & result) {
    CElement element;
    if (!getElement(db, key, element)) {
      return false;
    }
    CTable table;
    if (!getTable(element, table)) {
      return false;
    }
    CCell cell;
    if (!getCell(table, location, cell)) {
      return false;
    }
    int value;
    if (!getNumericCellValue(cell, value)) {
      return false;
    }

    result = (value < 0);
    return true;

  }
```

This approach uses a return value based error handling (`false` in case of
failure, `true` otherwise). The error handling logic is deeply entangled with
the business logic, making the function unnecessary long and difficult to
understand.

A monad is a design pattern used in functional programming that helps chain
operations while handling context (like absence of a value, errors, or
asynchronous computations). It’s a wrapper that lets you compose a series of
operations without repeatedly checking the context (e.g., whether a value is
present or not). A more modern approach to this functionality is the use of
`std::optional` as monad:

```cpp
optional<bool> isIntCellValueNegative(CDb db, Key key, CLocation location) {
  return getElement(db, key)
    .and_then(getTable)
    .and_then([location](CTable table) {
      return getCell(table, location);
    })
    .and_then(getNumericCellValue)
    .transform(isNegative);
}
```

The `transform` function is typical for functors. It maps one type `T` to
another one `U`, or does nothing if no value is contained in the `optional`.
`and_then` is typical for monads, as it takes a transformation function that
has the content of the `optional` `T` as input and returns another
`optional<U>` with a transformed type. Note that no specific code needs to be
written to handle the both cases that the `optional` can contain a value or be
empty.

At this point, you might be asking: "But we're losing any information on where
an error happend!" and that is true. The example can be extended by using an
`std::expected` type instead of `std::optional` and follow up the chained
function calls with `.or_else()` to handle the error case while maintaining the
error context.

## Safety

In his talk [Will your program still be correct next
year?](https://cpponsea.uk/2025/session/will-your-program-still-be-correct-next-year)
by Björn Fahller, he shared years of experience in writing unit tests and
common fallacies.

I have now a better understanding that contracts (or assertions, for everyone
on older C++ standards) are a measure to _detect defects early_ with rigorous
testing. They should enforce conditions that can never be wrong if the program
is logically correct. For performance reasons, the assertions should be
disabled as soon as testing is completed.

Furthermore, there is an old debate in object oriented programming about how to
test private methods of a class. I'm convinced now that the following is a good
approach: If the logic is sufficiently simple, private methods should be tested
via the public interface of the class. If the logic is more complex, it should
be extracted into an own class along with own unit tests. There is no need for
making methods public just for the sake of testing, nor for the `friend`
approach.

John Lakos dived deeper into safety in his talk [What C++ Needs to be Safe
](https://cpponsea.uk/2025/session/what-cpp-needs-to-be-safe). He defined
safety as encompassing **correctness, memory safety, and the absence of
undefined behavior (UB)**, noting that memory safety is only one aspect. He
emphasized that no single tool can achieve complete safety, illustrating this
with a Venn diagram of techniques. These techniques include:

- **Contracts (C++26):** For ensuring correctness.
- **Profiles:** Limiting the language to a "safe" subset, orthogonal to
  contracts.
- **Law of Exclusivity:** Referencing tools like the Rust borrow checker and
  Circle compiler.
- **Runtime Enforcement with Ghost Data:** Debug-only data to track runtime
  conditions.

Timur Doumler followed up with his talk [Contracts, Safety, and the Art of Cat
Herding](https://cpponsea.uk/2025/session/contracts-safety-and-the-art-of-cat-herding).
He pointed out that "safety" is a vague term, being used very differently
depending on context. He identified different categories of safety:

- Functional Safety
- Regulator Safety
- Absence of UB
- Memory Safety
- etc.

It is crucial to define, what safety one is talking about.

He also made a point, that if a program is correct, it must be safe and secure
as well, as that is implied by correctness. So correctness can serve as generic
goal for software development. Lastly, he encouraged a swiss-cheese approach in
the endeavors to achieve safety. Different mechanisms target different aspects
of safety, and only combined can achieve the level of safety we'd like to see.
In addition to the measures mentioned by John, he added replacing UB by EB or
other defined behavior, which is currently taking place in C++26 and beyond.

## Performance

Jason Turner kicked off his talk [The Power and Pain of Hidden Symbols
](https://cpponsea.uk/2025/session/the-power-and-pain-of-hidden-symbols) by
introducing a peculiarity of Windows library development: Each symbol that
should become part of a public API needs to be exported explicitly using
`__declspec(dllexport)`. The same behavior can be achieved on Unix as well,
with the compiler flag `-fvisibilty=hidden`. This prevents any symbols from
being visible in the object file, unless declared with
`__attribute__((visibility("default")))`. This not only has an impact on the
binary size, but also on performance. If this is combined with link time
optimization (`-flto`), a significant performance increase can be observed. The
reason for this is, that the compiler can inline functions that are not public,
or optimize them away completely. We're basically equipping the compiler with
more information how we're going to use the program, enabling more freedom in
optimization.

Marcell Juhasz contributed to the performance topic with his talk [Balancing
Efficiency and Flexibility: Cost of Abstractions in Embedded
Systems](https://cpponsea.uk/2025/session/balancing-efficiency-and-flexibility-cost-of-abstractions-in-embedded-systems).
He showed that C++ is a sane choice for embedded development, despite the
criticism about runtime overhead. He showed that zero-cost abstractions work as
intended (encapsulation, inheritance, static polymorphism) and with the
advances in template metaprogramming and other compile-time functionality
(templates, concepts, constexpr, consteval), runtime cost can be eliminated
altogether.

## Behavioral Insights

Mike Shah & Chris Croft-White showed in their talk [Understanding large and
unfamiliar codebases](https://cpponsea.uk/2025/session/understanding-large-and-unfamiliar-codebases)
approaches to gain insights into unfamiliar code. They highlighted to always
start off with the `main` function and demonstrated tools like `gcc`, `perf`
and `valgrind` to navigate through the code rather than reading it in your IDE.
The audience also mentioned modern IDEs will build call hierarchies for you.

Tina Ulbrich & Hendrik Niemeyer raised awareness of environmental factors in
their talk [Green
Programming](https://cpponsea.uk/2025/session/green-programming). They showed
what metrics affect energy consumption in the software world, and how to make
smart decisions when environmental considerations should be taken into account.
C++, C and Rust are great choices for environmentally friendly languages, as
the main contributing factor for energy consumption seems to be runtime. Also,
it matters where the code is physically run, e.g. in Norway, where 88% of
energy originates from hydroelectricity, or in the US, where well over 50% of
energy is still generated via fossil resources.

In Timur Doumler's talk (mentioned above under "Safety"), he also mentioned an
interesting way that he is implementing in the WG21 to discuss approaches and
achieve consensus, which he calls "THE TABLE". That is a table that lists
different options in its columnds, and requirements or qualities in its rows.

|               | Approach 1 | Approach 2      | Approach 3 |
| ------------- | ---------- | --------------- | ---------- |
| Requirement 1 | ❌         | ✅              | ❌         |
| Requirement 2 | ✅         | ⚠️ Only, if ... | ❌         |
| Requirement 3 | ✅         | ✅              | ✅         |

THE TABLE helps to get everyone on the same page for a discussion, which is
often the biggest challenge. Also, solutions get more obvious.

Last but not least, Kristen Shaker ended the conference with her talk [
Why Software Engineering Interviews are Broken and How to Actually Make them
Better](https://cpponsea.uk/2025/session/why-software-engineering-interviews-are-broken-and-how-to-actually-make-them-better).
She demonstrated how technical interviews consisting of leetcode or hackerrank
problems don't lead to good hires, but companies are still widely relying on
that. Many engineers admit that if interviewed today, they would fail their
technical interviews, since everyday work is very different and requires a
different set of skills as interviews do. Instead, interviews should consist of
questions that have many right answers, like

> What is your favorite feature of C++ and how have you used it in the past?

This is an open question that is easy to answer for anyone with a certain C++
experience, but difficult to answer without having used C++.

## Personal Takeaways

Two unrelated notes that I took:

- Invest heavily in your tooling, both hardware and software
- Take a step back from daily business and focus more on fundamental questions

## Ending Notes

Who I didn't mention above, but still want to mention, are Mantrala Sandeep and
Walter E Brown, who gave a great refresher on C++ value categories.

Thanks to Phil and all the volunteers and speakers for the great conference!
