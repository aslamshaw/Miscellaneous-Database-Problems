# 🧮 Print Prime Numbers Up to N using Stored Procedure

## 📌 Problem Statement

Design a MySQL stored procedure named `PrintPrime(IN n INT)` that outputs all **prime numbers** from `2` to `n` as a **single string**, with each number separated by an ampersand (`&`). The output should be formatted like:

2&3&5&7&11&13&...


### Constraints
- The implementation must be within a MySQL stored procedure.
- Intermediate outputs are not allowed; only return the final string.
- Efficient use of loops and logical checks is encouraged.
- Prime numbers should be computed dynamically for any valid `n` (up to at least `1000`).
- Use of temporary tables is allowed.

---

## 🚀 Solution Approaches

### Method 1: Naive Trial Division (Nested Loops)

#### SQL Query:
```sql
DELIMITER //
CREATE PROCEDURE PrintPrime(IN n INT)
BEGIN
    DECLARE i INT;
    DECLARE j INT;
    DECLARE is_prime BOOLEAN;
    CREATE TEMPORARY TABLE IF NOT EXISTS temp_prime(prime INT);
    TRUNCATE TABLE temp_prime;
    SET i = 2;
    WHILE i <= n DO
        SET is_prime = TRUE;
        SET j = 2;
        LOOP_CHECK: WHILE j <= FLOOR(POW(i, 0.5)) DO
            IF i % j = 0 THEN
                SET is_prime = FALSE;
                LEAVE LOOP_CHECK;
            END IF;
            SET j = j + 1;
        END WHILE;
        IF is_prime THEN
            INSERT INTO temp_prime(prime) VALUES (i);
        END IF;
        SET i = i + 1;
    END WHILE;
    SELECT GROUP_CONCAT(prime SEPARATOR '&') FROM temp_prime;
END //
DELIMITER ;
CALL PrintPrime(1000);
DROP PROCEDURE PrintPrime;
```

#### 🔧 How It Works
This method checks each number `i` from `2` to `n` to determine whether it's a prime by:
- Iterating through possible divisors `j` from `2` to `√i`
- If any `j` divides `i`, `i` is not a prime
- A flag `is_prime` tracks whether `i` is prime after the inner loop
- Prime numbers are collected into a temporary table and later returned as a concatenated string

#### ✅ Advantages
- Simple and intuitive
- Good for understanding the fundamentals of prime checking

#### ⚠️ Limitations
- Inefficient for large `n` due to repeated checks for every number
- Redundant computations — every number is checked independently, even if known to be composite from earlier steps

---

### Method 2: Sieve of Eratosthenes

#### SQL Query:
```sql
DELIMITER //
CREATE PROCEDURE PrintPrime(IN n INT)
BEGIN
    DECLARE i INT;
    DECLARE j INT;
    SET SQL_SAFE_UPDATES = 0;
    CREATE TEMPORARY TABLE IF NOT EXISTS prime_table(num INT, is_prime BOOL);
    TRUNCATE TABLE prime_table;
    SET i = 0;
    WHILE i <= n DO
        INSERT INTO prime_table VALUES (i, TRUE);
        SET i = i + 1;
    END WHILE;
    UPDATE prime_table SET is_prime = FALSE WHERE num IN (0, 1);
    SET i = 2;
    WHILE i <= FLOOR(POW(n, 0.5)) DO
        IF (SELECT is_prime FROM prime_table WHERE num = i) THEN
            SET j = i * i;
            WHILE j <= n DO
                UPDATE prime_table SET is_prime = FALSE WHERE num = j;
                SET j = j + i;
            END WHILE;
        END IF;
        SET i = i + 1;
    END WHILE;
    SELECT GROUP_CONCAT(num SEPARATOR '&') FROM prime_table WHERE is_prime = 1;
END //
DELIMITER ;
CALL PrintPrime(1000);
DROP PROCEDURE PrintPrime;
```

#### 🔧 How It Works
This is an optimized algorithm that avoids redundant checks by marking multiples of primes:
1. Initialize a list of numbers from `0` to `n`, assuming all are prime
2. Set `0` and `1` as non-prime
3. Start from `i = 2`, and eliminate all multiples of `i` (starting from `i²`)
4. Continue this process up to `√n`
5. The remaining marked values are prime

#### ✅ Advantages
- Highly efficient for generating all primes up to `n`
- Avoids unnecessary re-checks by marking multiples early
- Reduces time complexity compared to nested loop approach

#### ⚠️ Considerations
- Requires proper setup of temporary storage (e.g. prime table)
- Slightly more complex logic compared to naive method
- May require `SQL_SAFE_UPDATES` to be disabled depending on environment

---

## 📊 Summary Comparison

| Feature                | Method 1: Trial Division | Method 2: Sieve of Eratosthenes |
|------------------------|--------------------------|----------------------------------|
| Time Complexity        | O(n√n)                   | O(n log log n)                   |
| Space Complexity       | O(n) (for result table)  | O(n) (tracking primes)           |
| Simplicity             | ✅ Easy to implement     | ⚠️ Slightly more complex         |
| Performance (Large n)  | ❌ Slower                | ✅ Much faster                   |
| Redundant Checks       | ✅ Present               | ❌ Eliminated                    |

---

## ✅ Conclusion

- Use **Method 1** for small values of `n` or if you're learning prime logic from scratch.
- Use **Method 2 (Sieve of Eratosthenes)** for better performance and scalability, especially when handling large ranges of numbers.

Both methods demonstrate valid logical strategies in MySQL using stored procedures, but their trade-offs are important based on context.
