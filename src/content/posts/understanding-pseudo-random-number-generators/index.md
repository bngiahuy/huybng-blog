---
title: "Unveiling the Magic Behind \"Randomness\": A Deep Dive into Pseudo-Random Number Generators (PRNGs)"
published: 2025-07-09
description: "Explore how computers simulate randomness using Pseudo-Random Number Generators (PRNGs) and learn why they’re crucial in tech applications"
image: "./cover.jpg"
tags: ["random numbers","Programming Blogs","C++"]
category: "Algorithms"
draft: false
lang: ""
---

## Are Computers Truly Random? The Surprising Truth

Have you ever wondered how your computer or smartphone generates "random" numbers for games, simulations, or even security features? It might surprise you to learn that, by default, computers aren't truly random. Instead, they rely on something called **Pseudo-Random Number Generators (PRNGs)**.

Think of it like this: a truly random number would be unpredictable, like a coin flip where you can't possibly guess the outcome. A PRNG, on the other hand, is an algorithm that *looks* random, but is actually completely **deterministic**. This means that given the same starting point, it will always produce the exact same sequence of "random" numbers.

---

## The Core Concept: How PRNGs Fool Us

So, how do PRNGs work their "magic"? At their heart, most PRNGs are mathematical formulas that take a previous number and use it to calculate the next. The most common and simple type is the **Linear Congruential Generator (LCG)**.

The formula for an LCG is:

```plaintext
Xₙ₊₁ = (a * Xₙ + c) % m
```

Where:

-   `X_n+1` is the **next pseudo-random number**.

-   `X_n` is the **current pseudo-random number** (or the state of the generator).

-   `a` is the **multiplier**.

-   `c` is the **increment**.

-   `m` is the **modulus**.

The "magic" here is that the sequence of numbers produced by this formula appears random, passing many statistical tests for randomness. However, because it's based on a fixed formula, if you start with the same initial value, you'll always get the same sequence.

---

## The Key to Control: Understanding the "Seed"

This "initial value" is what we call the **seed**.

-   **No Seed Provided:** When you usually ask a program for a random number without explicitly setting a seed, it often uses a dynamic value like the **current system time** (e.g., milliseconds since a specific date) as its seed. Since the system time is always changing, you get a different "random" sequence each time you run the program, creating the illusion of true randomness.

-   **Seed Provided:** If you explicitly provide a fixed seed, the PRNG will always start from that exact point. Consequently, it will **always generate the exact same sequence of pseudo-random numbers** every single time you run your program with that specific seed.

### Why would we want "random" results to be fixed?

While it seems counterintuitive, controlling the randomness with a seed is incredibly useful for:

-   **Testing and Debugging:** Imagine a game with random enemy spawns. If a bug only appears under certain random conditions, setting a fixed seed allows developers to reliably reproduce those conditions and fix the bug.

-   **Reproducibility in Science and Research:** For simulations or machine learning experiments, researchers need to ensure their results can be replicated. A fixed seed guarantees that the "random" inputs are identical for every run.

-   **Procedural Content Generation:** In game development, seeds can be used to generate vast, "random" game worlds. Players can then share a specific seed to experience the exact same world as others.

---

## Building Our Own PRNG: A C++ Example

Let's dive into some code to see this in action. Below is a basic C++ implementation of an LCG. Notice that we don't use any built-in C++ random libraries; we're building it from scratch!

```cpp
#include <iostream>
#include <chrono>   // For generating a dynamic seed based on time

class SimplePRNG {
private:
    unsigned long long current_seed; // Current state of the PRNG (Xn)
    unsigned long long multiplier;   // Multiplier (a)
    unsigned long long increment;    // Increment (c)
    unsigned long long modulus;      // Modulus (m)

public:
    // Constructor to initialize the PRNG with a starting seed
    SimplePRNG(unsigned long long initial_seed) {
        // These constants are carefully chosen to ensure a long period and good distribution
        // They are common and effective values for LCGs (e.g., from ANSI C rand())
        multiplier = 1103515245ULL;
        increment = 12345ULL;
        modulus = 2147483648ULL; // 2^31

        current_seed = initial_seed;
    }

    // Generates the next pseudo-random number in the sequence
    unsigned long long next() {
        current_seed = (multiplier * current_seed + increment) % modulus;
        return current_seed;
    }

    // Allows resetting the PRNG to a new or old seed
    void set_seed(unsigned long long new_seed) {
        current_seed = new_seed;
    }

    // Helper to get a random number within a specific range [min, max]
    int get_random_range(int min_val, int max_val) {
        if (min_val > max_val) { // Swap if range is inverted
            int temp = min_val;
            min_val = max_val;
            max_val = temp;
        }
        long long range_val = static_cast<long long>(max_val) - min_val + 1;
        if (range_val == 0) return min_val;
        return min_val + (static_cast<int>(next() % range_val));
    }
};

int main() {
    std::cout << "--- Test with a Fixed Seed ---" << std::endl;
    // Initialize PRNG with a fixed seed
    SimplePRNG prng1(12345); 

    std::cout << "First 5 random numbers (Seed 12345):" << std::endl;
    for (int i = 0; i < 5; ++i) {
        std::cout << prng1.next() << " ";
    }
    std::cout << std::endl;

    std::cout << "Next 5 random numbers (Seed 12345):" << std::endl;
    for (int i = 0; i < 5; ++i) {
        std::cout << prng1.next() << " ";
    }
    std::cout << std::endl;

    // Reset the seed to the same initial value
    prng1.set_seed(12345); 
    std::cout << "5 random numbers after resetting seed (Seed 12345):" << std::endl;
    for (int i = 0; i < 5; ++i) {
        std::cout << prng1.next() << " ";
    }
    std::cout << std::endl;

    std::cout << "\n--- Test with Different Seeds ---" << std::endl;
    SimplePRNG prng2(67890); // Initialize with a different seed
    std::cout << "First 5 random numbers (Seed 67890):" << std::endl;
    for (int i = 0; i < 5; ++i) {
        std::cout << prng2.next() << " ";
    }
    std::cout << std::endl;

    std::cout << "\n--- Test with a Dynamic Seed (Time-based) ---" << std::endl;
    // Get current system time as a seed
    unsigned long long dynamic_seed = static_cast<unsigned long long>(
        std::chrono::high_resolution_clock::now().time_since_epoch().count()
    );
    SimplePRNG dynamic_prng(dynamic_seed);

    std::cout << "Seed used: " << dynamic_seed << std::endl;
    std::cout << "10 random numbers (will change each run):" << std::endl;
    for (int i = 0; i < 10; ++i) {
        std::cout << dynamic_prng.next() << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

### The Importance of Parameters: Choosing 'a', 'c', and 'm'

You might be wondering about those seemingly arbitrary large numbers used for `multiplier`, `increment`, and `modulus`. While you *can* choose different values, it's not arbitrary. These parameters are crucial for the **quality** of your PRNG:

-   **Period Length:** The chosen parameters (`a`, `c`, `m`) determine how long the sequence of unique numbers will be before it starts repeating. For an LCG, we aim for a "full period" equal to m. This is ensured by following specific mathematical rules (like the **Hull-Dobell Theorem**).

-   **Statistical Quality:** Good parameters ensure the numbers are well-distributed and don't show obvious patterns or correlations, making them *appear* more random.

The values used in our example are well-known for producing an LCG with a long period and reasonable statistical properties, often derived from established standards like ANSI C's `rand()` function.

---

## Conclusion

While LCGs are simple for demonstration, modern applications often require more sophisticated PRNGs. Cryptography, for instance, uses **Cryptographically Secure Pseudo-Random Number Generators (CSPRNGs)**, which are much harder to predict even if you know previous outputs.

Understanding PRNGs is fundamental to appreciating how computer systems handle randomness. It's a fascinating blend of mathematics and practical application, showing how deterministic processes can effectively mimic the unpredictable!
