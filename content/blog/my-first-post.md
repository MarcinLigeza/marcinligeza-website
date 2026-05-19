---
title: "Why I Use C++"
date: 2026-05-19
tags: ["c++", "opinion"]
summary: "A quick overview of why C++ is still my language of choice."
---
C++ gives me control over memory and performance that higher-level languages often hide.
Here's a typical RAII pattern:
{{< code cpp >}}
#include <memory>
void process() {
    auto data = std::make_unique<int[]>(100);
    // use data
} // automatically freed
{{< /code >}}