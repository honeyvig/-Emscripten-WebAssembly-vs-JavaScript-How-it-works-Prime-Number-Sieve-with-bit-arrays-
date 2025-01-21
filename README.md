# -Emscripten-WebAssembly-vs-JavaScript-How-it-works-Prime-Number-Sieve-with-bit-arrays
To compare Emscripten WebAssembly (WASM) and JavaScript with a focus on performance, let's create a Prime Number Sieve using both approaches. We will implement the Sieve of Eratosthenes algorithm for finding prime numbers up to a given limit and explore the difference in performance between WebAssembly (compiled from C/C++) and JavaScript.

Here are the steps for both implementations:
1. Prime Number Sieve in JavaScript

This is a typical implementation of the Sieve of Eratosthenes using bit arrays in JavaScript.

// Prime Number Sieve in JavaScript using Bit Arrays
function sieveOfEratosthenes(limit) {
    const sieve = new Uint8Array(Math.floor(limit / 8) + 1); // bit array for optimization
    const primes = [];

    // 0 and 1 are not prime
    for (let num = 2; num <= limit; num++) {
        const byte = Math.floor(num / 8);
        const bit = num % 8;
        
        // Check if the number is already marked as non-prime
        if ((sieve[byte] & (1 << bit)) === 0) {
            primes.push(num);
            // Mark multiples of the number as non-prime
            for (let multiple = num * num; multiple <= limit; multiple += num) {
                const byte = Math.floor(multiple / 8);
                const bit = multiple % 8;
                sieve[byte] |= 1 << bit;
            }
        }
    }
    return primes;
}

// Example usage
const limit = 1000000;
console.time('JavaScript Prime Sieve');
const primesJS = sieveOfEratosthenes(limit);
console.timeEnd('JavaScript Prime Sieve');

Explanation:

    We use a bit array (Uint8Array) for efficient memory usage, marking numbers as prime or not prime.
    The outer loop iterates through numbers and marks their multiples as non-prime.

2. Prime Number Sieve in C++ (Emscripten WebAssembly)

The following C++ code demonstrates the Sieve of Eratosthenes algorithm using bit arrays, which will be compiled into WebAssembly using Emscripten.

#include <iostream>
#include <vector>
#include <emscripten.h>

extern "C" {

// Prime Number Sieve using Bit Array in C++
std::vector<int> sieveOfEratosthenes(int limit) {
    std::vector<bool> sieve(limit + 1, false); // false means prime
    std::vector<int> primes;

    for (int num = 2; num <= limit; num++) {
        if (!sieve[num]) {
            primes.push_back(num);
            // Mark multiples of num as not prime
            for (int multiple = num * num; multiple <= limit; multiple += num) {
                sieve[multiple] = true;
            }
        }
    }
    return primes;
}

// Expose C++ function to JavaScript via WebAssembly
EMSCRIPTEN_KEEPALIVE
std::vector<int> getPrimes(int limit) {
    return sieveOfEratosthenes(limit);
}

}

Explanation:

    We use a vector to represent the sieve array. It is initialized as false (indicating primes).
    The sieveOfEratosthenes function implements the Sieve of Eratosthenes algorithm.
    The EMSCRIPTEN_KEEPALIVE macro ensures that the function is available for JavaScript to call after compiling the C++ code to WebAssembly.

3. Setting Up Emscripten

    Install Emscripten: If you don't already have it, install Emscripten.

    Compile the C++ Code to WebAssembly: Using Emscripten, compile the C++ code to WebAssembly:

emcc prime_sieve.cpp -o prime_sieve.js -s WASM=1 -s EXPORTED_FUNCTIONS="['_getPrimes']" -s EXTRA_EXPORTED_RUNTIME_METHODS="['ccall', 'cwrap']"

This will produce prime_sieve.js (JavaScript glue code) and prime_sieve.wasm (WebAssembly binary).

Serve the Files: Serve the files using a local HTTP server (e.g., Python's simple HTTP server).

    python3 -m http.server

4. JavaScript Integration with WebAssembly

Now, we need to interact with the WebAssembly module from JavaScript.

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Prime Number Sieve with WebAssembly</title>
    <script src="prime_sieve.js"></script>
</head>
<body>
    <h1>Prime Number Sieve</h1>
    <script>
        // Wait for the WebAssembly module to load
        Module.onRuntimeInitialized = function () {
            const limit = 1000000;

            // Call the WebAssembly function to get primes
            console.time('WebAssembly Prime Sieve');
            const primesWASM = Module._getPrimes(limit);
            console.timeEnd('WebAssembly Prime Sieve');

            console.log('Number of primes found:', primesWASM.length);
        }
    </script>
</body>
</html>

Explanation:

    This HTML file loads the prime_sieve.js JavaScript file (which is the WebAssembly glue code).
    Once the WebAssembly module is initialized, it calls the getPrimes function, which is exposed to JavaScript.
    The time taken to run the sieve algorithm in WebAssembly is logged.

5. Comparison: JavaScript vs WebAssembly

    JavaScript: The sieve runs directly in the browser using JavaScript. While JavaScript is interpreted, it is highly flexible but often slower than WebAssembly, especially for CPU-intensive tasks like prime number generation.
    WebAssembly: The C++ code compiled to WebAssembly is executed at near-native speed in the browser. This is generally much faster for computationally heavy algorithms like the Sieve of Eratosthenes, as WebAssembly is optimized for performance.

6. Performance Measurement

In both the JavaScript and WebAssembly examples, we use console.time and console.timeEnd to measure how long each implementation takes to compute primes up to a specified limit (1,000,000 in this case).
Expected Outcome:

    JavaScript Implementation: Will likely take longer to execute, especially as the limit increases.
    WebAssembly Implementation: Will likely perform significantly better in terms of execution time because WebAssembly code runs at near-native speed.

Conclusion:

This example shows the difference in performance between WebAssembly (compiled from C++) and JavaScript. By using Emscripten, we can compile C++ code into WebAssembly, allowing us to take advantage of the near-native performance improvements for computationally expensive tasks like prime number sieves.

In most cases, WebAssembly will outperform JavaScript in tasks that require heavy computation, making it a powerful tool for optimizing performance in the browser, especially for CPU-bound algorithms.
