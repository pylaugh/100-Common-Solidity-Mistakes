# 100 Common Solidity Mistakes

## Basic Security (1-20)
1. **Reentrancy vulnerabilities**  
2. **Using `tx.origin` for authentication**  
3. **Missing access controls**  
4. **Unprotected initialization**  
5. **Unprotected `selfdestruct` function**  
6. **Missing input validation**  
7. **Dependence on `block.timestamp`** (vulnerable to manipulation)  
8. **Front-running vulnerability**  
9. **Missing zero-address checks**  
10. **Unchecked external calls**  
11. **Delegate call vulnerabilities**  
12. **Signature replay attacks**  
13. **Insufficient gas stipend for fallback**  
14. **Missing event emissions for critical actions**  
15. **Improper error handling**  
16. **Unprotected proxy implementations**  
17. **Missing function access modifiers (public/private)**  
18. **Improper use of `require/assert`**  
19. **Missing emergency stop mechanism (circuit breaker)**  
20. **Unprotected state variables**  

## Mathematical Operations (21-40)
21. **Division before multiplication**  
22. **Rounding errors in division**  
23. **Precision loss in calculations**  
24. **Integer overflow/underflow** (for Solidity < 0.8.0)  
25. **Incorrect decimal handling**  
26. **Missing SafeMath (for Solidity < 0.8.0)**  
27. **Improper percentage calculations**  
28. **Incorrect order of operations**  
29. **Missing bounds checking for inputs**  
30. **Floating point approximation errors**  
31. **Incorrect scaling factors**  
32. **Missing minimum/maximum checks**  
33. **Improper fee calculations**  
34. **Incorrect balance updates after operations**  
35. **Missing slippage protection in exchanges**  
36. **Incorrect price calculations**  
37. **Unhandled remainder values in division**  
38. **Missing decimal normalization**  
39. **Incorrect rate calculations in loops or series**  
40. **Improper rounding direction in financial calculations**  

## Gas Optimization (41-60)
41. **Unbounded loops**  
42. **Expensive operations within loops**  
43. **Unnecessary storage usage (vs memory)**  
44. **Inefficient array operations (e.g., array resizing)**  
45. **Unoptimized struct packing**  
46. **Redundant state reads (use memory)**  
47. **Missing memory vs storage optimization**  
48. **Inefficient string operations**  
49. **Unnecessary event emissions**  
50. **Unoptimized function visibility**  
51. **Missing view/pure modifiers**  
52. **Inefficient `require` statements** (long error messages)  
53. **Unnecessary variable initialization**  
54. **Unoptimized mapping usage**  
55. **Missing batching of operations**  
56. **Inefficient error message storage**  
57. **Excessive contract size**  
58. **Unoptimized bytecode**  
59. **Missing gas limit constraints**  
60. **Inefficient function parameters (avoid storage)**  

## Token Standards (61-80)
61. **Incorrect ERC20 implementation**  
62. **Missing SafeERC20 usage**  
63. **Incorrect ERC721 implementation**  
64. **Missing token approval checks**  
65. **Incorrect allowance handling in ERC20**  
66. **Missing transfer event emissions**  
67. **Incorrect decimal handling for token units**  
68. **Missing token recovery mechanisms for lost tokens**  
69. **Incorrect or missing burn mechanisms**  
70. **Missing pause mechanisms for tokens**  
71. **Improper minting controls**  
72. **Missing token validation on transfer/receive**  
73. **Incorrect token transfer implementation**  
74. **Missing token locking mechanisms**  
75. **Incorrect or missing token upgrade mechanisms**  
76. **Missing compliance with token standards**  
77. **Incorrect token metadata storage/format**  
78. **Missing token enumeration for collections**  
79. **Incorrect or missing royalty support (e.g., ERC2981)**  
80. **Missing access control for token administrators**  

## Contract Design (81-100)
81. **Poor error handling and user feedback**  
82. **Missing events for critical state changes**  
83. **Incorrect constructor usage (initializer patterns)**  
84. **Missing initializer modifiers in upgradeable contracts**  
85. **Incorrect inheritance order**  
86. **Missing interface implementations**  
87. **Poor or missing code documentation**  
88. **Lack of NatSpec comments for external functions**  
89. **Incorrect function visibility (public/private)**  
90. **Missing or improper use of function modifiers**  
91. **Incorrect state management (lifecycle issues)**  
92. **Missing upgrade mechanisms (e.g., UUPS)**  
93. **Poor contract architecture (monolithic design)**  
94. **Missing contract documentation for maintenance**  
95. **Improper dependency management**  
96. **Missing version control practices**  
97. **Insufficient testing coverage**  
98. **Lack of security considerations in design**  
99. **Incorrect deployment procedures**  
100. **Missing maintenance procedures for deployed contracts**  

---


Here's a concise breakdown of each mistake into explanations, examples, and learning tips. To keep the response manageable, I'll present them in chunks. Let's start with the **Basic Security** section:

---

### Basic Security (1-20)

#### 1. **Reentrancy Vulnerabilities**
   - **Explanation**: Allows an attacker to repeatedly call a function before previous executions complete, often used to drain funds.
   - **Example**:
     ```solidity
     function withdraw(uint256 amount) external {
         require(balances[msg.sender] >= amount);
         (bool success, ) = msg.sender.call{value: amount}("");
         require(success, "Transfer failed");
         balances[msg.sender] -= amount; // Move this before the external call
     }
     ```
   - **Learn**: Always update internal state before external calls. Use `Checks-Effects-Interactions` pattern.

#### 2. **Using `tx.origin` for Authentication**
   - **Explanation**: `tx.origin` can lead to phishing attacks, as it represents the original transaction sender, allowing attackers to act on behalf of a user.
   - **Example**:
     ```solidity
     require(tx.origin == owner); // Vulnerable
     ```
   - **Learn**: Use `msg.sender` for authentication, as it is specific to the immediate caller.

#### 3. **Missing Access Controls**
   - **Explanation**: Functions without access control allow anyone to call them, potentially causing harm.
   - **Example**:
     ```solidity
     function setConfig(uint256 value) public { config = value; } // Anyone can call this
     ```
   - **Learn**: Implement access control modifiers (e.g., `onlyOwner`) on sensitive functions.

#### 4. **Unprotected Initialization**
   - **Explanation**: Without initialization checks, a contract may be initialized by anyone, especially in proxies.
   - **Example**:
     ```solidity
     function initialize(uint256 _value) public { value = _value; }
     ```
   - **Learn**: Include `initializer` modifier to restrict re-initialization.

#### 5. **Unprotected `selfdestruct`**
   - **Explanation**: Allows anyone to destroy the contract and claim its funds if not protected.
   - **Example**:
     ```solidity
     function destroy() public { selfdestruct(payable(msg.sender)); } // No protection
     ```
   - **Learn**: Always protect `selfdestruct` with `onlyOwner` or similar access control.

#### 6. **Missing Input Validation**
   - **Explanation**: Functions may enter invalid states without input validation.
   - **Example**:
     ```solidity
     function setPrice(uint256 _price) external { price = _price; } // No validation
     ```
   - **Learn**: Always validate inputs, e.g., `require(_price > 0, "Price must be positive");`.

#### 7. **Dependence on `block.timestamp`**
   - **Explanation**: Miners can manipulate timestamps, impacting time-based logic.
   - **Example**:
     ```solidity
     if (block.timestamp == deadline) { /* risky */ }
     ```
   - **Learn**: Use comparisons like `>=` rather than exact equality with `block.timestamp`.

#### 8. **Front-running Vulnerability**
   - **Explanation**: Allows attackers to profit by manipulating transaction order (e.g., bidding higher gas).
   - **Example**:
     ```solidity
     function buy() public payable { price = msg.value; }
     ```
   - **Learn**: Use commit-reveal schemes or limit orders to mitigate front-running risks.

#### 9. **Missing Zero-Address Checks**
   - **Explanation**: Transfers or assignments to address(0) result in permanent loss.
   - **Example**:
     ```solidity
     function setOwner(address newOwner) public { owner = newOwner; } // No zero-check
     ```
   - **Learn**: Add checks like `require(newOwner != address(0), "Invalid address");`.

#### 10. **Unchecked External Calls**
   - **Explanation**: Failure in external calls may go unnoticed, causing contract logic to proceed incorrectly.
   - **Example**:
     ```solidity
     token.transfer(to, amount); // No check on transfer success
     ```
   - **Learn**: Always use `require(token.transfer(to, amount), "Transfer failed");`.

#### 11. **Delegate Call Vulnerabilities**
   - **Explanation**: Allows caller contract to execute code in the context of another contract, often without control.
   - **Example**:
     ```solidity
     other.delegatecall(abi.encodeWithSignature("someFunction()")); // Unsafe
     ```
   - **Learn**: Use delegatecall carefully; only with trusted, well-understood contracts.

#### 12. **Signature Replay Attacks**
   - **Explanation**: Reuse of a valid signature allows repeated, unintended transactions.
   - **Example**:
     ```solidity
     function execute(bytes memory signature) public { /* no nonce check */ }
     ```
   - **Learn**: Include nonces or expiration times in signatures to prevent reuse.

#### 13. **Insufficient Gas Stipend**
   - **Explanation**: Limited gas in `send` or `transfer` can prevent contract functionality.
   - **Example**:
     ```solidity
     msg.sender.transfer(amount); // Only sends 2300 gas
     ```
   - **Learn**: Prefer `call` with gas control over `transfer` and `send`.

#### 14. **Missing Event Emissions for Critical Actions**
   - **Explanation**: Events help external monitoring and auditing of significant state changes.
   - **Example**:
     ```solidity
     function updateBalance(uint256 amount) external { balance = amount; } // No event emitted
     ```
   - **Learn**: Emit events for changes to critical contract state.

#### 15. **Improper Error Handling**
   - **Explanation**: Lack of `require`, `revert`, or `assert` for critical checks can cause silent failures.
   - **Example**:
     ```solidity
     token.transfer(to, amount); // Does not check success
     ```
   - **Learn**: Use `require` statements to handle all external calls and critical conditions.

#### 16. **Unprotected Proxy Implementations**
   - **Explanation**: Proxy contracts may allow unapproved parties to upgrade the logic contract.
   - **Example**:
     ```solidity
     function upgradeTo(address newImplementation) public { /* No access control */ }
     ```
   - **Learn**: Secure proxies by limiting upgrade functions to admins or owners.

#### 17. **Missing Function Access Modifiers**
   - **Explanation**: Without access modifiers, functions default to public visibility.
   - **Example**:
     ```solidity
     function configure(uint256 value) { config = value; } // Implicitly public
     ```
   - **Learn**: Explicitly set visibility modifiers (public/private) on all functions.

#### 18. **Improper Use of `require/assert`**
   - **Explanation**: `assert` should only be used for critical invariants, while `require` handles input conditions.
   - **Example**:
     ```solidity
     assert(input > 0); // Use require for conditions, assert for invariants
     ```
   - **Learn**: Use `require` for inputs, `assert` for critical invariants that should never fail.

#### 19. **Missing Emergency Stop Mechanism**
   - **Explanation**: No way to stop contract operations in case of vulnerabilities.
   - **Example**:
     ```solidity
     // No pause mechanism
     ```
   - **Learn**: Implement a pause function using `onlyOwner` and a `stopped` variable to stop risky operations.

#### 20. **Unprotected State Variables**
   - **Explanation**: Without `private` or `internal`, any address can read state variables.
   - **Example**:
     ```solidity
     uint256 public sensitiveData;
     ```
   - **Learn**: Use `private` for sensitive data unless it needs to be accessible.

---

### Mathematical Operations (21-40)

#### 21. **Division Before Multiplication**
   - **Explanation**: Can lead to unintended zero values due to integer division truncation.
   - **Example**:
     ```solidity
     uint result = (amount / total) * multiplier; // Might truncate before multiplication
     ```
   - **Learn**: Perform multiplications before divisions to avoid truncation errors.

#### 22. **Rounding Errors in Division**
   - **Explanation**: Integer division truncates decimals, potentially causing miscalculations.
   - **Example**:
     ```solidity
     uint result = 10 / 3; // Result is 3, not 3.333...
     ```
   - **Learn**: Use rounding techniques, such as `amount * 100 / divisor` for percentage calculations.

#### 23. **Precision Loss in Calculations**
   - **Explanation**: Solidity lacks floating-point support, so decimal points are truncated.
   - **Example**:
     ```solidity
     uint result = amount * 0.01; // This won't work as expected
     ```
   - **Learn**: Use scaled integers (e.g., `1000` for 1.000) for decimal approximations.

#### 24. **Integer Overflow/Underflow (Pre-0.8.0)**
   - **Explanation**: Arithmetic operations could wrap around in versions before 0.8.0.
   - **Example**:
     ```solidity
     uint result = type(uint).max + 1; // Wraps to 0 in pre-0.8.0
     ```
   - **Learn**: Use SafeMath library or upgrade to Solidity 0.8+, where overflow protection is default.

#### 25. **Incorrect Decimal Handling**
   - **Explanation**: Misinterpretation of token decimals can lead to miscalculations.
   - **Example**:
     ```solidity
     uint tokens = 100; // Assuming token has 18 decimals, this is actually 0.0000000000000001 tokens
     ```
   - **Learn**: Understand and correctly apply the token’s decimal standard in calculations.

#### 26. **Missing SafeMath (Pre-0.8.0)**
   - **Explanation**: SafeMath protects against overflow but isn’t automatically used in pre-0.8.0 versions.
   - **Example**:
     ```solidity
     uint result = a + b; // No overflow protection pre-0.8.0
     ```
   - **Learn**: Use `SafeMath` in pre-0.8.0 contracts to prevent overflow.

#### 27. **Improper Percentage Calculations**
   - **Explanation**: Dividing before multiplying can lead to inaccuracies in percentages.
   - **Example**:
     ```solidity
     uint result = (amount / 100) * percentage; // Might lose precision
     ```
   - **Learn**: Multiply before dividing, e.g., `(amount * percentage) / 100`.

#### 28. **Incorrect Order of Operations**
   - **Explanation**: Misordered operations can yield incorrect results.
   - **Example**:
     ```solidity
     uint result = 1 + 2 * 3; // Result is 7, not 9
     ```
   - **Learn**: Use parentheses to control operation precedence for clarity.

#### 29. **Missing Bounds Checking**
   - **Explanation**: Lack of bounds checking can lead to out-of-range values.
   - **Example**:
     ```solidity
     function buy(uint amount) public { balance[msg.sender] += amount; } // No limit on amount
     ```
   - **Learn**: Use checks like `require(amount > 0 && amount <= maxLimit, "Invalid amount");`.

#### 30. **Floating Point Approximation Errors**
   - **Explanation**: Solidity doesn’t support floating-point numbers, so calculations might approximate.
   - **Example**:
     ```solidity
     uint percentage = (amount * 5) / 100.0; // Not valid in Solidity
     ```
   - **Learn**: Work with integer approximations, scaling by powers of 10 if necessary.

#### 31. **Incorrect Scaling Factors**
   - **Explanation**: Incorrect application of scaling factors can distort values.
   - **Example**:
     ```solidity
     uint scaled = amount / 1e18; // Assumes `amount` is scaled up, but it may not be
     ```
   - **Learn**: Ensure correct factor application based on decimals or desired precision.

#### 32. **Missing Minimum/Maximum Checks**
   - **Explanation**: Unrestricted values can lead to overflow or unintended contract behavior.
   - **Example**:
     ```solidity
     function deposit(uint amount) public { balance += amount; } // No minimum check
     ```
   - **Learn**: Implement checks, e.g., `require(amount >= minAmount && amount <= maxAmount);`.

#### 33. **Improper Fee Calculations**
   - **Explanation**: Rounding errors in fee calculations can result in incorrect payouts.
   - **Example**:
     ```solidity
     uint fee = amount / 100 * 3; // May round incorrectly
     ```
   - **Learn**: Calculate fees carefully by adjusting for rounding, e.g., `(amount * 3) / 100`.

#### 34. **Incorrect Balance Updates**
   - **Explanation**: Balance updates in the wrong order may cause inconsistencies.
   - **Example**:
     ```solidity
     function transfer(address to, uint amount) public { balance[to] += amount; balance[msg.sender] -= amount; }
     ```
   - **Learn**: Always decrease the sender's balance before increasing the receiver's.

#### 35. **Missing Slippage Protection**
   - **Explanation**: Without slippage limits, unexpected price movements may affect trade outcomes.
   - **Example**:
     ```solidity
     function swap(uint amount) external { executeTrade(amount); }
     ```
   - **Learn**: Implement minimum output checks, e.g., `require(output >= minOutput);`.

#### 36. **Incorrect Price Calculations**
   - **Explanation**: Failure to account for decimals and rounding can yield inaccurate pricing.
   - **Example**:
     ```solidity
     uint price = (tokenAmount / ethAmount) * 1e18; // Misses decimal adjustments
     ```
   - **Learn**: Consider decimal places in calculations, adjusting appropriately for each token.

#### 37. **Unhandled Remainder Values**
   - **Explanation**: Remainders in division may be discarded, leading to unexpected results.
   - **Example**:
     ```solidity
     uint portion = total / 3; // Discards remainder
     ```
   - **Learn**: Use modular arithmetic or handle remainders explicitly if necessary.

#### 38. **Missing Decimal Normalization**
   - **Explanation**: Unnormalized decimals can lead to unexpected results in calculations.
   - **Example**:
     ```solidity
     uint result = tokenA.amount * tokenB.amount; // May ignore decimal differences
     ```
   - **Learn**: Standardize to a common decimal scale before operations.

#### 39. **Incorrect Rate Calculations**
   - **Explanation**: Misinterpretation of rate units can yield inaccurate results.
   - **Example**:
     ```solidity
     uint rate = 5; // Ambiguous: is this per second, minute, or transaction?
     ```
   - **Learn**: Clearly define rate units, e.g., `ratePerMinute`.

#### 40. **Improper Rounding Direction**
   - **Explanation**: Rounding inconsistencies can impact critical calculations like fees or balances.
   - **Example**:
     ```solidity
     uint amount = total / 3; // May round down, losing fractional amounts
     ```
   - **Learn**: Implement rounding that’s consistent with business logic (e.g., always round up/down).

---

### Gas Optimization (41-60)

#### 41. **Unbounded Loops**
   - **Explanation**: Loops without bounds can become very costly, especially if they operate on large arrays.
   - **Example**:
     ```solidity
     function processArray(uint[] memory array) public {
         // Unbounded loop: if array is too large, this can consume too much gas
         for (uint i = 0; i < array.length; i++) {
             // process each element
         }
     }
     ```
   - **Solution**:
     ```solidity
     function processLimitedArray(uint[] memory array, uint maxIterations) public {
         uint limit = array.length < maxIterations ? array.length : maxIterations;
         for (uint i = 0; i < limit; i++) {
             // process each element, capped by maxIterations
         }
     }
     ```
   - **Learn**: Limit the number of iterations or use fixed-size arrays if possible.

#### 42. **Expensive Operations in Loops**
   - **Explanation**: Complex operations inside loops, like external calls or storage access, increase gas usage.
   - **Example**:
     ```solidity
     function updateAllBalances(address[] memory users) public {
         for (uint i = 0; i < users.length; i++) {
             balances[users[i]] += 1; // multiple storage writes inside loop
         }
     }
     ```
   - **Solution**:
     ```solidity
     function updateBalances(address[] memory users) public {
         uint updatedBalance = 1;
         for (uint i = 0; i < users.length; i++) {
             balances[users[i]] = updatedBalance; // fewer writes by reusing values
         }
     }
     ```
   - **Learn**: Minimize operations inside loops to save on gas costs.

#### 43. **Unnecessary Storage Usage**
   - **Explanation**: Writing to storage repeatedly without changes wastes gas.
   - **Example**:
     ```solidity
     function updateData(uint newValue) public {
         if (data != newValue) { // Avoiding storage write if value is unchanged
             data = newValue;
         }
     }
     ```
   - **Learn**: Check if a storage update is needed before writing to it.

#### 44. **Inefficient Array Operations**
   - **Explanation**: Shifting elements in an array is costly since it requires multiple storage operations.
   - **Example**:
     ```solidity
     function removeFirstElement(uint[] storage arr) public {
         for (uint i = 0; i < arr.length - 1; i++) {
             arr[i] = arr[i + 1];
         }
         arr.pop(); // Removing last element
     }
     ```
   - **Solution**:
     ```solidity
     function markElementAsRemoved(uint index) public {
         // Instead of shifting, mark as removed to avoid storage costs
         arr[index] = 0; // Logical removal
     }
     ```
   - **Learn**: Use alternative strategies like marking elements as inactive.

#### 45. **Unoptimized Struct Packing**
   - **Explanation**: Incorrect struct member ordering results in inefficient storage slot usage.
   - **Example**:
     ```solidity
     struct Data {
         uint256 a;
         uint8 b;
         uint256 c; // Storage uses 3 slots instead of 2 due to ordering
     }
     ```
   - **Solution**:
     ```solidity
     struct OptimizedData {
         uint256 a;
         uint256 c;
         uint8 b; // Packs into 2 storage slots
     }
     ```
   - **Learn**: Arrange struct members by type and size to optimize slot usage.

#### 46. **Redundant State Reads**
   - **Explanation**: Repeated reads from storage variables within a function increase gas usage.
   - **Example**:
     ```solidity
     function updateBalance(uint amount) public {
         if (balances[msg.sender] >= amount) {
             balances[msg.sender] -= amount;
         }
     }
     ```
   - **Solution**:
     ```solidity
     function updateBalance(uint amount) public {
         uint balance = balances[msg.sender];
         if (balance >= amount) {
             balances[msg.sender] = balance - amount;
         }
     }
     ```
   - **Learn**: Read storage values once into local variables when feasible.

#### 47. **Missing Memory vs. Storage Optimization**
   - **Explanation**: Using `storage` unnecessarily in function parameters can increase gas costs.
   - **Example**:
     ```solidity
     function calculateSum(uint[] storage numbers) public view returns (uint sum) {
         for (uint i = 0; i < numbers.length; i++) {
             sum += numbers[i];
         }
     }
     ```
   - **Solution**:
     ```solidity
     function calculateSum(uint[] memory numbers) public pure returns (uint sum) {
         for (uint i = 0; i < numbers.length; i++) {
             sum += numbers[i];
         }
     }
     ```
   - **Learn**: Use `memory` for function parameters when values don’t need to be saved in storage.

#### 48. **Inefficient String Operations**
   - **Explanation**: Concatenating strings directly can be gas-intensive.
   - **Example**:
     ```solidity
     function greet(string memory name) public pure returns (string memory) {
         return string(abi.encodePacked("Hello, ", name));
     }
     ```
   - **Learn**: Minimize string concatenations, or use indexed events to avoid string manipulation in contracts.

#### 49. **Unnecessary Event Emissions**
   - **Explanation**: Logging events uses gas, especially if they contain large or numerous parameters.
   - **Example**:
     ```solidity
     emit DataUpdated(oldValue, newValue); // Emitting multiple values when not essential
     ```
   - **Solution**:
     ```solidity
     if (newValue != oldValue) {
         emit DataUpdated(newValue); // Only emit essential data
     }
     ```
   - **Learn**: Emit only necessary events for tracking significant state changes.

#### 50. **Unoptimized Function Visibility**
   - **Explanation**: Functions default to `public` if visibility isn’t specified, which allows external calls and increases gas.
   - **Example**:
     ```solidity
     function internalFunction() { ... } // Defaults to public if unspecified
     ```
   - **Solution**:
     ```solidity
     function internalFunction() internal { ... } // Specify internal when appropriate
     ```
   - **Learn**: Use `internal` or `private` visibility for functions that don’t require external access.

#### 51. **Missing View/Pure Modifiers**
   - **Explanation**: Functions that only read or don’t interact with storage should use `view` or `pure`.
   - **Example**:
     ```solidity
     function getValue() public returns (uint) { return value; }
     ```
   - **Solution**:
     ```solidity
     function getValue() public view returns (uint) { return value; }
     ```
   - **Learn**: Use `view` for reading-only functions and `pure` for those with no storage interaction.

#### 52. **Inefficient Require Statements**
   - **Explanation**: Long `require` error messages increase bytecode size.
   - **Example**:
     ```solidity
     require(value == expected, "This error message is too long and costly");
     ```
   - **Solution**:
     ```solidity
     require(value == expected, "Invalid"); // Use concise messages
     ```
   - **Learn**: Keep error messages short to reduce deployment costs.

#### 53. **Unnecessary Variable Initialization**
   - **Explanation**: Variables default to zero, so setting `uint x = 0` is redundant.
   - **Example**:
     ```solidity
     uint count = 0; // Redundant initialization
     ```
   - **Solution**:
     ```solidity
     uint count; // Implicitly 0
     ```
   - **Learn**: Avoid redundant initialization for variables.

#### 54. **Unoptimized Mapping Usage**
   - **Explanation**: Large mappings without bounds checking can result in high storage costs.
   - **Example**:
     ```solidity
     mapping(address => uint) public balances; // Unbounded entries
     ```
   - **Solution**:
     ```solidity
     mapping(address => uint) public balances;
     address[] public addresses;
     function addAddress(address _addr) public {
         require(balances[_addr] == 0, "Already added");
         addresses.push(_addr);
     }
     ```
   - **Learn**: Limit mappings or track entries in arrays to manage storage.

#### 55. **Missing Batching Operations**
   - **Explanation**: Repeating similar operations individually is more expensive.
   - **Example**:
     ```solidity
     function processSingleValue(uint value) public { /* process value */ }
     ```
   - **Solution**:
     ```solidity
     function processBatch(uint[] memory values) public {
         for (uint i = 0; i < values.length; i++) {
             // process each value
         }
     }
     ```
   - **Learn**: Batch similar operations in one function to reduce transactions.


#### 56. **Inefficient Error Messages**
   - **Explanation**: Long or redundant error messages consume more gas when `require` or `revert` statements fail.
   - **Example**:
     ```solidity
     require(x > 0, "Value must be greater than zero, provided value is invalid"); // Long error message
     ```
   - **Solution**:
     ```solidity
     require(x > 0, "Invalid"); // Concise message
     ```
   - **Learn**: Shorten error messages for gas efficiency while retaining clarity.

#### 57. **Unnecessary Contract Size**
   - **Explanation**: Large contracts can exceed the Ethereum block size limit, making them costly and difficult to deploy.
   - **Example**:
     ```solidity
     contract LargeContract {
         // A lot of functions, constants, and variables that may not be essential
     }
     ```
   - **Solution**:
     ```solidity
     contract OptimizedContract {
         // Streamlined functions, reusable code, modularization
     }
     ```
   - **Learn**: Refactor to separate logic across multiple contracts or libraries to reduce contract size.

#### 58. **Unoptimized Bytecode**
   - **Explanation**: Complex, redundant logic increases contract size and bytecode, raising deployment costs.
   - **Example**:
     ```solidity
     uint256 constant DAY = 86400;
     function getDayInSeconds() public pure returns (uint) { return 86400; } // Duplicate constant
     ```
   - **Solution**:
     ```solidity
     uint256 constant DAY = 86400; // Store constants to avoid duplication
     function getDayInSeconds() public pure returns (uint) { return DAY; }
     ```
   - **Learn**: Use constants and simplify logic to minimize bytecode size.

#### 59. **Missing Gas Limits**
   - **Explanation**: Lack of defined gas limits for critical operations can make them vulnerable to gas exhaustion.
   - **Example**:
     ```solidity
     address recipient;
     recipient.call{value: 1 ether}(""); // No gas limit defined
     ```
   - **Solution**:
     ```solidity
     recipient.call{value: 1 ether, gas: 2300}(""); // Define a gas limit for safety
     ```
   - **Learn**: Set gas limits on external calls to avoid vulnerabilities from gas exhaustion.

#### 60. **Inefficient Function Parameters**
   - **Explanation**: Passing storage arrays or structs as function parameters can be more gas-intensive than necessary.
   - **Example**:
     ```solidity
     function process(uint[] storage data) internal { /* operation */ }
     ```
   - **Solution**:
     ```solidity
     function process(uint[] memory data) internal pure { /* operation */ }
     ```
   - **Learn**: Use `memory` for function parameters whenever possible to reduce gas usage.

---

### Token Standards (61-80)

#### 61. **Incorrect ERC20 Implementation**
   - **Explanation**: An incomplete or incorrect ERC20 implementation can break token functionality or security expectations.
   - **Example**:
     ```solidity
     function transfer(address to, uint256 amount) external { /* missing balance checks */ }
     ```
   - **Solution**:
     ```solidity
     function transfer(address to, uint256 amount) external returns (bool) {
         require(balanceOf[msg.sender] >= amount, "Insufficient balance");
         balanceOf[msg.sender] -= amount;
         balanceOf[to] += amount;
         emit Transfer(msg.sender, to, amount);
         return true;
     }
     ```
   - **Learn**: Always follow the ERC20 standard precisely and implement all required functions.

#### 62. **Missing SafeERC20 Usage**
   - **Explanation**: Direct calls to `ERC20.transfer` or `ERC20.transferFrom` can fail without reason, leading to unhandled errors.
   - **Example**:
     ```solidity
     token.transfer(to, amount); // May fail without a revert reason
     ```
   - **Solution**:
     ```solidity
     import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
     using SafeERC20 for IERC20;
     
     token.safeTransfer(to, amount);
     ```
   - **Learn**: Use OpenZeppelin’s `SafeERC20` for safer token transfers.

#### 63. **Incorrect ERC721 Implementation**
   - **Explanation**: Implementing custom ERC721 functions incorrectly can lead to NFTs that don’t follow expected standards.
   - **Example**:
     ```solidity
     function transferNFT(address to, uint256 tokenId) public { /* incorrect implementation */ }
     ```
   - **Solution**:
     ```solidity
     function transferFrom(address from, address to, uint256 tokenId) public override {
         require(ownerOf(tokenId) == from, "Not owner");
         _transfer(from, to, tokenId);
     }
     ```
   - **Learn**: Always follow the ERC721 standard exactly.

#### 64. **Missing Token Approval Checks**
   - **Explanation**: Missing approval checks can lead to unauthorized transfers.
   - **Example**:
     ```solidity
     function transferFrom(address from, address to, uint256 amount) external {
         balanceOf[from] -= amount;
         balanceOf[to] += amount;
     }
     ```
   - **Solution**:
     ```solidity
     function transferFrom(address from, address to, uint256 amount) external {
         require(allowance[from][msg.sender] >= amount, "Not approved");
         balanceOf[from] -= amount;
         balanceOf[to] += amount;
     }
     ```
   - **Learn**: Implement approval checks properly to ensure secure transfers.

#### 65. **Incorrect Allowance Handling**
   - **Explanation**: Not following allowance reduction expectations can lead to incorrect balance updates.
   - **Example**:
     ```solidity
     function transferFrom(address from, address to, uint256 amount) public {
         allowance[from][msg.sender] -= amount;
         balanceOf[to] += amount;
     }
     ```
   - **Solution**:
     ```solidity
     function transferFrom(address from, address to, uint256 amount) public {
         require(balanceOf[from] >= amount, "Insufficient balance");
         require(allowance[from][msg.sender] >= amount, "Allowance exceeded");
         allowance[from][msg.sender] -= amount;
         balanceOf[from] -= amount;
         balanceOf[to] += amount;
     }
     ```
   - **Learn**: Always validate both balance and allowance in `transferFrom`.

#### 66. **Missing Transfer Event Emissions**
   - **Explanation**: Failing to emit transfer events prevents proper tracking of token transfers.
   - **Example**:
     ```solidity
     balanceOf[to] += amount; // Missing Transfer event
     ```
   - **Solution**:
     ```solidity
     emit Transfer(msg.sender, to, amount);
     ```
   - **Learn**: Emit `Transfer` events after each balance update.

#### 67. **Incorrect Decimals Handling**
   - **Explanation**: Incorrect handling of token decimals can lead to calculation inaccuracies.
   - **Example**:
     ```solidity
     uint256 balanceInETH = balanceInToken / 1e18; // Wrong for tokens with non-18 decimals
     ```
   - **Solution**:
     ```solidity
     uint256 balanceInETH = balanceInToken / (10 ** decimals);
     ```
   - **Learn**: Use token’s decimal value correctly in calculations.

#### 68. **Missing Token Recovery Mechanisms**
   - **Explanation**: Tokens sent by mistake to contracts without recovery mechanisms can become inaccessible.
   - **Example**:
     ```solidity
     // No method to recover accidentally sent tokens
     ```
   - **Solution**:
     ```solidity
     function recoverTokens(IERC20 token, uint256 amount) external onlyOwner {
         token.safeTransfer(msg.sender, amount);
     }
     ```
   - **Learn**: Include a function to allow owner recovery of ERC20 tokens sent by mistake.

#### 69. **Incorrect Burn Mechanisms**
   - **Explanation**: Incorrect burn functionality can lead to unexpected behaviors and supply miscalculations.
   - **Example**:
     ```solidity
     function burn(uint256 amount) public {
         totalSupply -= amount;
     }
     ```
   - **Solution**:
     ```solidity
     function burn(uint256 amount) public {
         balanceOf[msg.sender] -= amount;
         totalSupply -= amount;
         emit Transfer(msg.sender, address(0), amount);
     }
     ```
   - **Learn**: Update both balances and total supply, and emit a `Transfer` event to address `0`.

#### 70. **Missing Pause Mechanisms**
   - **Explanation**: Without a pause mechanism, you cannot temporarily disable critical functions during emergencies.
   - **Example**:
     ```solidity
     function transfer(address to, uint256 amount) public { /* No pause functionality */ }
     ```
   - **Solution**:
     ```solidity
     bool public paused;
     
     function pause() public onlyOwner { paused = true; }
     function unpause() public onlyOwner { paused = false; }
     
     function transfer(address to, uint256 amount) public {
         require(!paused, "Transfers are paused");
         // transfer logic
     }
     ```
   - **Learn**: Use `paused` state checks in critical functions.

#### 71. **Incorrect Minting Controls**
   - **Explanation**: Lack of restricted access to minting can lead to an uncontrolled increase in token supply.
   - **Example**:
     ```solidity
     function mint(uint256 amount) public { /* Open to anyone */ }
     ```
   - **Solution**:
     ```solidity
     function mint(uint256 amount) public onlyOwner {
         totalSupply += amount;
         balanceOf[msg.sender] += amount;
     }
     ```
   - **Learn**: Limit minting access to authorized roles, typically `onlyOwner`.

#### 72. **Missing Token Validation**
   - **Explanation**: Not validating token transfers can allow invalid addresses or operations.
   - **Example**:
     ```solidity
     function transfer(address to, uint256 amount) public { /* No address checks */ }
     ```
   - **Solution**:
     ```solidity
     function transfer(address to, uint256 amount) public {
         require(to != address(0), "Invalid recipient");
         balanceOf[msg.sender] -= amount;
         balanceOf[to] += amount;
         emit Transfer(msg.sender, to, amount);
     }
     ```
   - **Learn**: Validate recipient addresses to avoid loss of tokens to `address(0)`.

#### 73. **Incorrect Token Transfers**
   - **Explanation**: Incorrect token transfer implementations may lead to lost tokens or contract failures.
   - **Example**:
     ```solidity
     function transfer(address to, uint256 amount) public {
         balanceOf[to] += amount;
     }
     ```
   - **Solution**:
     ```solidity
     function transfer(address to, uint256 amount) public {
         require(balanceOf[msg.sender] >= amount, "Insufficient balance");
         balanceOf[msg.sender] -= amount;
         balanceOf[to] += amount;
         emit Transfer(msg.sender, to, amount);
     }
     ```
   - **Learn**: Always decrease the sender’s balance before increasing the recipient’s to prevent overflows.

#### 74. **Missing Token Locks**
   - **Explanation**: Without token locks, tokens may be transferred before certain conditions are met (e.g., vesting periods).
   - **Example**:
     ```solidity
     // Transfer function without token lock implementation
     ```
   - **Solution**:
     ```solidity
     mapping(address => uint256) public lockedUntil;

     function transfer(address to, uint256 amount) public {
         require(block.timestamp > lockedUntil[msg.sender], "Tokens are locked");
         // transfer logic
     }
     ```
   - **Learn**: Implement time or condition-based token locks to secure transfers until certain conditions are met.

#### 75. **Incorrect Token Upgrades**
   - **Explanation**: Token upgrades without preserving balances or approvals can disrupt user assets.
   - **Solution**:
     ```solidity
     function upgradeToken(address newToken) public onlyOwner {
         require(address(newToken) != address(0), "Invalid token address");
         newToken.migrateFrom(address(this));
     }
     ```
   - **Learn**: Test upgrades on testnets and maintain a well-documented upgrade path.

#### 76. **Missing Token Standards Compliance**
   - **Explanation**: Non-compliance with token standards like ERC20 or ERC721 causes unexpected behavior in wallets and exchanges.
   - **Solution**:
     ```solidity
     function name() public view override returns (string memory) { return _name; }
     function symbol() public view override returns (string memory) { return _symbol; }
     ```
   - **Learn**: Follow all functions specified in token standards.

#### 77. **Incorrect Token Metadata**
   - **Explanation**: Incorrect metadata functions can confuse users about token details like `name`, `symbol`, and `decimals`.
   - **Solution**:
     ```solidity
     string public constant name = "MyToken";
     string public constant symbol = "MTK";
     uint8 public constant decimals = 18;
     ```
   - **Learn**: Define metadata clearly for compatibility with wallets and applications.

#### 78. **Missing Token Enumeration**
   - **Explanation**: Omitting token enumeration in ERC721 tokens prevents access to a list of all owned NFTs.
   - **Solution**:
     ```solidity
     mapping(address => uint256[]) private _ownedTokens;
     // Add functions to fetch owned tokens
     ```
   - **Learn**: Implement enumeration for easier token management.

#### 79. **Incorrect Token Royalties**
   - **Explanation**: Not setting correct royalty values can cause overcharging or undercharging in NFT transactions.
   - **Solution**:
     ```solidity
     function royaltyInfo(uint256 tokenId, uint256 salePrice) external view returns (address, uint256) {
         return (creator, salePrice / 10); // 10% royalty
     }
     ```
   - **Learn**: Ensure royalties are calculated correctly to match terms.

#### 80. **Missing Token Access Controls**
   - **Explanation**: Without access controls, anyone may execute critical token functions.
   - **Solution**:
     ```solidity
     modifier onlyOwner() { require(msg.sender == owner, "Not authorized"); _; }
     function mint(address to, uint256 amount) public onlyOwner { /* Minting logic */ }
     ```
   - **Learn**: Implement `onlyOwner` or `onlyAdmin` modifiers for sensitive token functions.

---

### Contract Design (81-100)

#### 81. **Poor Error Handling**
   - **Explanation**: Failing to handle errors properly can cause user confusion and contract instability.
   - **Solution**:
     ```solidity
     function withdraw(uint256 amount) public {
         require(amount <= balance[msg.sender], "Insufficient funds");
         // Withdraw logic
     }
     ```
   - **Learn**: Use meaningful `require` statements for clear error handling.

#### 82. **Missing Events**
   - **Explanation**: Without events, it’s hard to track contract state changes on the blockchain.
   - **Solution**:
     ```solidity
     event FundsWithdrawn(address indexed user, uint256 amount);
     ```
   - **Learn**: Emit events for all significant state changes.

#### 83. **Incorrect Constructor Usage**
   - **Explanation**: Failing to properly initialize state variables in constructors can lead to inconsistent contract states.
   - **Solution**:
     ```solidity
     constructor(uint256 _initialSupply) {
         totalSupply = _initialSupply;
     }
     ```
   - **Learn**: Use constructors for initial setup only and ensure state variables are set.

#### 84. **Missing Initializer Modifiers**
   - **Explanation**: Missing `initializer` modifiers in upgradeable contracts can cause initialization functions to be executed multiple times.
   - **Solution**:
     ```solidity
     bool private initialized;
     modifier initializer() { require(!initialized, "Already initialized"); _; initialized = true; }
     ```
   - **Learn**: Use `initializer` modifiers to protect against re-initialization.

#### 85. **Incorrect Inheritance Order**
   - **Explanation**: Incorrect inheritance order can override functions unintentionally.
   - **Solution**:
     ```solidity
     contract FinalContract is BaseContract, ModifierContract { /* ... */ }
     ```
   - **Learn**: Carefully order base contracts to ensure intended function visibility.

#### 86. **Missing Interface Implementations**
   - **Explanation**: Not fully implementing an interface’s functions can make contracts incompatible with expected standards.
   - **Solution**:
     ```solidity
     interface MyInterface { function myFunction() external; }
     contract MyContract is MyInterface { function myFunction() external override { /* ... */ } }
     ```
   - **Learn**: Implement all interface functions to ensure compliance.

#### 87. **Poor Code Documentation**
   - **Explanation**: Lack of documentation makes the code harder to understand and maintain.
   - **Solution**:
     ```solidity
     /// @notice Transfers tokens to an address
     /// @param to The recipient address
     /// @param amount The amount to transfer
     function transfer(address to, uint256 amount) public { /* ... */ }
     ```
   - **Learn**: Use comments and Natspec annotations to explain complex logic.

#### 88. **Missing Natspec Comments**
   - **Explanation**: Missing Natspec comments reduces clarity for tools and developers using the code.
   - **Solution**:
     ```solidity
     /// @dev Internal function to update balances
     function _updateBalance(address user, uint256 amount) internal { /* ... */ }
     ```
   - **Learn**: Natspec comments improve readability and API documentation.

#### 89. **Incorrect Function Visibility**
   - **Explanation**: Incorrect visibility can expose functions that should be private or internal.
   - **Solution**:
     ```solidity
     function _internalFunction() internal { /* ... */ }
     ```
   - **Learn**: Use appropriate visibility (`public`, `private`, `internal`, `external`) for each function.

#### 90. **Missing Function Modifiers**
   - **Explanation**: Lack of function modifiers can lead to repeated code and complex logic.
   - **Solution**:
     ```solidity
     modifier onlyOwner() { require(msg.sender == owner, "Not authorized"); _; }
     ```
   - **Learn**: Create reusable modifiers to simplify and secure code.

#### 91. **Incorrect State Management**
   - **Explanation**: Not managing state changes correctly can lead to unexpected behaviors.
   - **Solution**:
     ```solidity
     enum State { Active, Inactive }
     State public state;

     function deactivate() public onlyOwner {
         require(state == State.Active, "Already inactive");
         state = State.Inactive;
     }
     ```
   - **Learn**: Use `enum`s and state transitions where applicable.

#### 92. **Missing Upgrade Mechanisms**
   - **Explanation**: Without upgrade mechanisms, a contract can’t evolve over time.
   - **Solution**: Use proxy patterns to make contracts upgradeable.
   - **Learn**: Consider an upgradeable architecture if updates will be needed.

#### 93. **Poor Contract Architecture**
   - **Explanation**: Overly complex or tightly coupled contracts make maintenance difficult.
   - **Solution**: Break down large contracts into modular components.
   - **Learn**: Adopt modular design for reusability and simplicity.

#### 94. **Missing Contract Documentation**
   - **Explanation**: Lack of documentation makes it hard for others to understand and use the contract.
   - **Solution**: Write a README explaining contract functionality.
   - **Learn**: Always provide documentation for public use.

#### 95. **Incorrect Dependency Management**
   - **Explanation**: Mismanaging dependencies can lead to outdated or

 insecure code.
   - **Solution**: Regularly update libraries and dependencies.
   - **Learn**: Use tools like `npm` and `yarn` to manage dependencies.

#### 96. **Missing Version Control**
   - **Explanation**: Version control lets developers track changes and revert to stable versions.
   - **Solution**: Use Git or a similar tool.
   - **Learn**: Version control is essential for collaborative development.

#### 97. **Poor Testing Coverage**
   - **Explanation**: Insufficient testing can lead to bugs in production.
   - **Solution**: Write tests for all critical functions.
   - **Learn**: Use tools like Truffle or Hardhat for testing.

#### 98. **Missing Security Considerations**
   - **Explanation**: Not considering security in development can lead to vulnerabilities.
   - **Solution**: Conduct security reviews and use auditing tools.
   - **Learn**: Prioritize security from the start.

#### 99. **Incorrect Deployment Procedures**
   - **Explanation**: Incorrect deployments can waste gas and lead to unoptimized contracts.
   - **Solution**: Optimize gas before deployment and use correct addresses.
   - **Learn**: Test deployments on testnets before mainnet.

#### 100. **Missing Maintenance Procedures**
   - **Explanation**: Not planning for maintenance can lead to unsustainable contracts.
   - **Solution**: Allocate maintenance budgets and resources.
   - **Learn**: Plan for ongoing support and updates.

---
