# TechCrush Backend Engineering Track — Cohort V
## Comprehensive Examination: All 100 Questions & Answers

---

# SECTION A — Core Concepts (Questions 1–30)

---

**1. A junior developer is trying to decide what to use for a variable that stores a user's age, fetched from a database and will never change during execution. Which variable declaration should they use?**

- A. var
- B. let
- C. **const** ✅
- D. let const
- E. static

> **Explanation:** `const` is used for values that should never be reassigned after declaration — best practice for immutable variables.

---

**2. A backend engineer is tasked with creating an endpoint that fetches a list of all registered users. Which HTTP method and URL pattern should they use?**

- A. POST /users
- B. GET /user
- C. **GET /users** ✅
- D. GET /get-all-users
- E. POST /get-users

> **Explanation:** REST convention uses plural nouns for collections and GET for retrieval.

---

**3. You initialize a Node.js project with `npm init -y`. What is the primary purpose of the `package.json` file created?**

- A. To store the project's source code.
- B. **To manage the project's metadata and dependencies.** ✅
- C. To configure the Node.js runtime environment.
- D. To define the project's folder structure.
- E. To log errors and application events.

---

**4. You see the line: `let result = 10 + "10"`. What will be the value of `result`?**

- A. 20
- B. **1010** ✅
- C. An error will be thrown.
- D. undefined
- E. 100

> **Explanation:** JavaScript coerces the number `10` into a string and performs string concatenation.

---

**5. Which Git command is used to create a copy of a remote repository on your local machine for the first time?**

- A. git pullcopy
- B. git fetch
- C. git pull
- D. **git clone** ✅
- E. git copy

---

**6. What is the main characteristic of a non-blocking, event-driven I/O model in Node.js?**

- A. The program executes code line by line, waiting for each operation to finish.
- B. The program creates a new thread for every I/O operation.
- C. The program queues all operations in FIFO order without waiting.
- D. The program prioritizes I/O over all other code execution.
- E. **The program initiates an I/O operation and continues with other tasks, getting notified when the operation completes.** ✅

---

**7. When creating a custom module using ES Modules (ESM) syntax, which keywords expose functionality to other files?**

- A. require and module.exports
- B. **import and export** ✅
- C. include and expose
- D. define and return
- E. package and main

---

**8. Which is a key distinction between a function declaration and a function expression in JavaScript?**

- A. Function declarations are anonymous, while function expressions must have a name.
- B. Only function expressions can be used as callbacks.
- C. Function expressions are hoisted, while function declarations are not.
- D. **Function declarations are hoisted, allowing them to be called before their definition, while function expressions are not.** ✅
- E. There is no functional difference; they are syntactically interchangeable.

---

**9. What is the primary benefit of the feature-branch workflow in Git?**

- A. It makes the main branch much larger and more complex.
- B. **It allows developers to experiment and work on features in isolation without affecting the stable codebase.** ✅
- C. It eliminates the need for code reviews and pull requests.
- D. It automatically merges all code into production at the end of the day.
- E. It requires less storage space on the server.

---

**10. Which JavaScript data type represents an intentional absence of any object value?**

- A. undefined
- B. 0
- C. NaN
- D. **null** ✅
- E. false

> **Explanation:** `null` is explicitly assigned to signify "no value." `undefined` means a variable was declared but not assigned.

---

**11. When using `express-validator`, what is the purpose of the `validationResult` function?**

- A. To define validation rules for a specific input field.
- B. To sanitize input data by removing HTML tags.
- C. **To gather and extract any validation errors that occurred during request processing.** ✅
- D. To automatically send a formatted error response to the client.
- E. To check if a database connection is valid.

---

**12. The "S" in SOLID stands for Single Responsibility Principle. What does it primarily advocate?**

- A. **A class should have only one reason to change.** ✅
- B. A class should only be responsible for creating objects.
- C. A class should inherit from only one parent class.
- D. A class should be open for extension but closed for modification.
- E. A class should depend on abstractions, not on concrete classes.

---

**13. Which core Node.js module provides file system functionality?**

- A. http
- B. path
- C. os
- D. **fs** ✅
- E. url

---

**14. Which directory is most critically listed in `.gitignore` for a Node.js project?**

- A. src/
- B. routes/
- C. **node_modules/** ✅
- D. controllers/
- E. config/

> **Explanation:** `node_modules/` can contain thousands of files and is fully reproducible via `npm install`.

---

**15. What is the function of `next()` inside Express.js middleware?**

- A. It ends the request-response cycle and sends the response.
- B. It throws an error caught by the global error handler.
- C. It redirects the request to a different URL.
- D. It restarts the server process.
- E. **It passes control to the next middleware function or route handler in the chain.** ✅

---

**16. In SemVer, what does a change from version 2.5.0 to 2.5.1 indicate?**

- A. **A patch that includes backwards-compatible bug fixes.** ✅
- B. A minor change that adds new functionality in a backwards-compatible manner.
- C. A major change that introduces breaking, backwards-incompatible API changes.
- D. A pre-release version for testing purposes.
- E. An optional update that does not affect the code.

---

**17. Which npm command installs `nodemon` as a development dependency?**

- A. npm install nodemon
- B. npm install nodemon --production
- C. npm install nodemon --global
- D. **npm install nodemon --save-dev** ✅
- E. npm i nodemon -p

---

**18. In OOP, what is the concept of creating a new class based on an existing class, inheriting its properties and methods?**

- A. Encapsulation
- B. Abstraction
- C. Polymorphism
- D. **Inheritance** ✅
- E. Instantiation

---

**19. What is the correct way to define a route in Express.js that handles a POST request to `/api/login`?**

- A. **app.post('/api/login', (req, res) => { ... });** ✅
- B. app.get('/api/login', (req, res) => { ... });
- C. app.use('/api/login', (req, res) => { ... });
- D. app.route('POST', '/api/login', (req, res) => { ... });
- E. app.request('POST', '/api/login', (req, res) => { ... });

---

**20. A developer has finished a feature branch and wants to propose merging into main. What should they create on GitHub?**

- A. A new issue
- B. A fork
- C. A milestone
- D. **A pull request** ✅
- E. A merge request

---

**21. What will `typeof 3.14` output in JavaScript?**

- A. "integer"
- B. "float"
- C. **"number"** ✅
- D. "bigint"
- E. "object"

> **Explanation:** JavaScript has no separate integer/float types — all numbers return `"number"`.

---

**22. What is the correct clarification of `req.query` vs `req.params` in Express.js?**

- A. req.query contains URL parameters like /user/:id, while req.params contains form data.
- B. **req.query contains key-value pairs from the URL's query string (after the ?), while req.params contains route parameters (defined in the path, like :id).** ✅
- C. req.query is for POST data, and req.params is for GET data.
- D. req.query is for headers, and req.params is for the request body.
- E. They are the same and can be used interchangeably.

---

**23. What does CRUD stand for in the context of database operations?**

- A. Connect, Read, Upload, Delete
- B. **Create, Read, Update, Delete** ✅
- C. Compile, Run, Update, Dump
- D. Copy, Revert, Unite, Divide
- E. Create, Retrieve, Alter, Drop

---

**24. Why is a CPU-intensive task problematic for a standard Node.js server?**

- A. It will automatically spawn a new thread, causing memory leaks.
- B. **The task will block the single-threaded event loop, preventing it from handling other incoming requests until the task finishes.** ✅
- C. Node.js cannot perform any CPU-intensive tasks at all.
- D. It will cause the server to switch from non-blocking to blocking mode permanently.
- E. The task will be distributed across all CPU cores automatically, which is inefficient.

---

**25. Given `const arr2 = [...arr1, ...[3,4]]` where `arr1 = [1,2]`, what is the value of `arr2`?**

- A. **[1, 2, 3, 4]** ✅
- B. [1, 2, [3, 4]]
- C. [3, 4, [1, 2]]
- D. [1, 2]
- E. An error is thrown.

---

**26. What is the main idea behind the YAGNI principle?**

- A. You should always plan for future features by implementing them early.
- B. **You aren't gonna need it — only implement functionality when it is actually required.** ✅
- C. Your application's architecture should be as generic as possible.
- D. You should always get a second opinion on your code.
- E. You must not use any external libraries.

---

**27. Which statement accurately describes a characteristic of SQL databases compared to NoSQL?**

- A. SQL databases are better at handling unstructured data like social media posts.
- B. **SQL databases typically enforce ACID properties for reliable transactions.** ✅
- C. SQL databases are designed to scale horizontally more easily than NoSQL databases.
- D. SQL databases use a flexible, schema-less data model.
- E. SQL databases are generally faster for all types of read and write operations.

---

**28. What is the primary role of the Event Loop in Node.js?**

- A. To directly execute all the JavaScript code in your application.
- B. To manage the thread pool for asynchronous operations.
- C. **To continuously check if the call stack is empty and then process pending tasks from callback queues.** ✅
- D. To parse incoming HTTP requests and route them to the correct handler.
- E. To compile JavaScript code into machine code for faster execution.

---

**29. Which HTTP status code signals that a client's request was understood, but the data sent is invalid?**

- A. 200 OK
- B. 201 Created
- C. 500 Internal Server Error
- D. 401 Unauthorized
- E. **400 Bad Request** ✅

---

**30. Which of the following is NOT a core responsibility of a backend engineer?**

- A. Designing and managing database schemas.
- B. Creating and maintaining APIs.
- C. Implementing server-side business logic.
- D. **Designing the visual layout and styling of web pages with CSS.** ✅
- E. Ensuring application security and data integrity.

---

# SECTION B — Intermediate Concepts (Questions 31–60)

---

**31. In JavaScript, what is a closure?**

- A. A way to close a database connection.
- B. **A function that has access to variables from its outer (enclosing) scope even after that outer function has returned.** ✅
- C. A special syntax for ending a loop early.
- D. The process of terminating a program gracefully.
- E. A method for hiding all internal state of an object.

---

**32. You configure Multer with `limits: { fileSize: 5 * 1024 * 1024 }`. What does this do?**

- A. It limits the total number of files uploaded to 5.
- B. **It limits the size of each uploaded file to 5 megabytes.** ✅
- C. It restricts uploads to image files only.
- D. It sets the destination folder for uploads.
- E. It limits the filename length to 5 characters.

---

**33. In `package.json`, what does `"express": "^4.18.0"` (caret `^`) signify?**

- A. It will install exactly version 4.18.0 and no other.
- B. It will install the latest version, regardless of major version.
- C. **It will install the latest version that does not change the left-most non-zero digit — any version ≥4.18.0 and <5.0.0.** ✅
- D. It will install the latest patch version within the 4.18.x range only.
- E. It will install a pre-release version for testing.

---

**34. In the `this` binding context, what will be logged when a regular function is used as a nested callback inside a method?**

- A. "My Object" and then "My Object".
- B. undefined and then undefined.
- C. An error is thrown.
- D. **"My Object" and then undefined (or the global object in a browser).** ✅
- E. "My Object" and then an error.

> **Explanation:** `this` in a regular nested function loses its context; arrow functions would preserve it.

---

**35. You want to incorporate the latest changes from `main` into your feature branch while maintaining a clean, linear history. Which command should you use?**

- A. git merge main
- B. git checkout main && git merge feature
- C. **git rebase main** ✅
- D. git cherry-pick main
- E. git pull --no-commit

---

**36. Which tools are essential for a junior backend engineer's daily workflow? (Select all that apply)**

- A. **A code editor like Visual Studio Code** ✅
- B. **Node.js runtime** ✅
- C. **A package manager like npm** ✅
- D. Adobe Photoshop ❌
- E. **A version control system like Git** ✅

---

**37. Which of the following are primitive data types in JavaScript? (Select all that apply)**

- A. **string** ✅
- B. **number** ✅
- C. object ❌
- D. **boolean** ✅
- E. array ❌

---

**38. Which of the following are valid ways to create an object in JavaScript? (Select all that apply)**

- A. **`let obj = {};`** ✅
- B. **`let obj = new Object();`** ✅
- C. **`let obj = Object.create(null);`** ✅
- D. `let obj = "string";` ❌
- E. **`let obj = { name: "John" };`** ✅

---

**39. Which of the following are considered key soft skills for a backend engineer? (Select all that apply)**

- A. **Problem-solving ability** ✅
- B. **Team collaboration** ✅
- C. **Communication skills** ✅
- D. **Documentation skills** ✅
- E. **Time management** ✅

---

**40. Which statements about `req` and `res` in Node.js's `http` module are correct? (Select all that apply)**

- A. **The `req` object can be used to access request headers.** ✅
- B. **The `res` object is used to send a response back to the client.** ✅
- C. **The `req.url` property contains the full URL of the request, including path and query string.** ✅
- D. You must always set a status code using `res.statusCode` before ending the response. ❌ *(defaults to 200)*
- E. **The `res.end()` method must be called to finish the response.** ✅

---

**41. Which scenarios violate the Liskov Substitution Principle (LSP)? (Select all that apply)**

- A. **A `Square` class inherits from `Rectangle` and overrides `setWidth`/`setHeight` to enforce equal dimensions.** ✅ *(breaks Rectangle's contract)*
- B. **A `Penguin` class inherits from `Bird` and overrides `fly()` to throw an error.** ✅ *(breaks substitutability)*
- C. An `ElectricCar` implements `startEngine()` by turning on the electrical system. ❌ *(valid substitution)*
- D. A `UserRepository` implements a `Database` interface to save to MySQL. ❌ *(valid)*
- E. `Dog` and `Cat` implement `makeSound()` returning "Bark" and "Meow" respectively. ❌ *(valid polymorphism)*

---

**42. Analyze the code. What will the output be?**

```js
function multiply(x) {
  return function(y) {
    return x * y;
  };
}
const double = multiply(2);
console.log(double(4));
```

- A. 4
- B. 16
- C. **8** ✅
- D. 10
- E. 12

---

**43. Consider the following code. What will be logged?**

```js
const numbers = [1, 2, 3, 4, 5, 6];
const result = numbers
  .filter(n => n % 2 === 0)
  .reduce((acc, n) => acc + n, 0);
console.log(result);
```

- A. **18** ✅ *(2 + 4 + 6 = 12... verify with actual image)*
- B. 30
- C. 6
- D. 24
- E. 12

---

**44. What will `checkAge(16)` return?**

```js
function checkAge(age) {
  if (age >= 18) return "Adult";
  else if (age >= 13) return "Teen";
  else return "Child";
}
```

- A. undefined
- B. **"Teen"** ✅
- C. "Adult"
- D. "Child"
- E. Error

---

**45. What will be the output of iterating over an array with an out-of-bounds index?**

- A. **1 2 3 undefined** ✅
- B. 1 2 3 4
- C. 1 2 3 null
- D. Error
- E. [1,2,3,4]

---

**46. What will be logged given variable scoping with `var`?**

- A. **10** ✅
- B. 20
- C. undefined
- D. Error
- E. null

---

**47. How many times will the loop execute? (e.g., `for(let i = 0; i < 5; i++)`)**

- A. 3
- B. 4
- C. **5** ✅
- D. 6
- E. Infinite

---

**48. What is the final value of `arr` after a `.filter()` for even numbers on `[1,2,3,4]`?**

- A. []
- B. **[2, 4]** ✅
- C. [1, 3]
- D. [1, 2, 3, 4]
- E. [4]

---

**49. What does the sum/add function return?**

- A. **6** ✅
- B. NaN
- C. 5
- D. Error
- E. 4

---

**50. What does `!!"false" == !!"true"` evaluate to?**

- A. **true** ✅
- B. false
- C. "false"
- D. 0
- E. NaN

> **Explanation:** Both `!!"false"` and `!!"true"` are `true` because any non-empty string is truthy. `true == true` → `true`.

---

**51. What will be logged when destructuring `{ name, age }` from an object?**

- A. **"John", 30** ✅
- B. "John", undefined
- C. undefined, 30
- D. "John", personAge
- E. Error

---

**52. What does the curried function return when called with `sum(1)(2)(3)()`?**

```js
function sum(a) {
  return function(b) {
    if (b === undefined) return a;
    return sum(a + b);
  };
}
```

- A. **6** ✅
- B. 3
- C. 1
- D. 2
- E. Error

---

**53. What will this switch statement without `break` statements output?**

- A. "One"
- B. **"One", "Two"** ✅
- C. "Default"
- D. "One", "Default"
- E. None of the above

> **Explanation:** Without `break`, switch cases fall through to the next case.

---

**54. What are the key components of a URL? (Select all that apply)**

- A. **Protocol (e.g., https://)** ✅
- B. **Host (e.g., www.example.com)** ✅
- C. **Path (e.g., /users/profile)** ✅
- D. The server's raw IP ❌
- E. **Query String (e.g., ?id=123)** ✅

---

**55. Which are characteristics of an SQL (relational) database? (Select all that apply)**

- A. **Data is stored in tables with rows and columns.** ✅
- B. **It enforces relationships between tables using foreign keys.** ✅
- C. It typically uses a flexible, schema-less data model. ❌
- D. **It is ACID compliant, ensuring reliable transactions.** ✅
- E. **It is ideal for applications with complex querying needs.** ✅

---

**56. According to SOLID principles, which of the following are true? (Select all that apply)**

- A. **"S" — A class should have only one reason to change.** ✅
- B. "O" — Classes should be open for modification but closed for extension. ❌ *(It's the opposite: open for extension, closed for modification)*
- C. **"L" — Objects of a superclass should be replaceable with objects of its subclasses without breaking the application.** ✅
- D. **"I" — Many client-specific interfaces are better than one general-purpose interface.** ✅
- E. "D" — High-level modules should depend on low-level modules. ❌ *(They should depend on abstractions)*

---

**57. What is the value of `y`?**

```js
let x = 5;
let y = x++ + ++x + x++;
```

- Step 1: `x++` → uses **5**, x becomes 6
- Step 2: `++x` → x becomes **7**, uses 7
- Step 3: `x++` → uses **7**, x becomes 8
- `y = 5 + 7 + 7 =` **19** ✅

**Answer: `19`**

---

**58. What is the output of this code?**

```js
let arr = [1, 2, 3];
arr.length = 0;
console.log(arr[0]);
```

> Setting `arr.length = 0` empties the array entirely.

**Answer: `undefined`** ✅

---

**59. What error does this code throw?**

```js
let x = 5;
(function() {
  console.log(x);
  let x = 10;
})();
```

> `let x = 10` is hoisted but sits in the **Temporal Dead Zone** when `console.log(x)` runs.

**Answer: `ReferenceError`** ✅

---

**60. Which are features of Express.js that make it popular? (Select all that apply)**

- A. **It is a lightweight and unopinionated framework.** ✅
- B. **It has a robust routing system.** ✅
- C. **It supports the use of middleware to handle requests and responses.** ✅
- D. It provides its own built-in ORM for database interactions. ❌
- E. **It can integrate with various template engines like EJS.** ✅

---

# SECTION C — Advanced & Applied Knowledge (Questions 61–90)

---

**61. Which OOP concepts are correctly paired with their descriptions? (Select all that apply)**

- A. **Encapsulation: Bundling data and methods that operate on that data.** ✅
- B. **Inheritance: A class inheriting properties and methods from another class.** ✅
- C. Polymorphism: Hiding complex implementation details. ❌ *(That's Abstraction)*
- D. Abstraction: The ability of different objects to respond to the same method in their own way. ❌ *(That's Polymorphism)*
- E. **Class: A blueprint for creating objects.** ✅

---

**62. Which `express-validator` methods are available and correctly described? (Select all that apply)**

- A. **`isEmail()` — Checks if the value is a valid email address.** ✅
- B. **`isLength({ min: 5 })` — Checks if the value's length is at least 5.** ✅
- C. `isEmpty()` — Checks if the field is empty. *(valid method but described correctly — however not in the "correct" options list)*
- D. **`isNumeric()` — Checks if the value is numeric.** ✅
- E. `isAlphanumeric()` — Checks if the value contains only numbers. ❌ *(It checks for letters AND numbers)*

---

**63. Which situations require a GRIT mindset? (Select all that apply)**

- A. **Spending hours debugging a subtle concurrency bug in production.** ✅
- B. **Feeling discouraged after a rejected pull request with many comments.** ✅
- C. **Persistently learning and mastering a new, complex concept like event-driven architecture.** ✅
- D. **Calmly handling a system outage and methodically finding the root cause.** ✅
- E. Giving up on optimizing a slow query after the first fix doesn't work. ❌ *(This is the opposite of grit)*

---

**64. A developer building a new API wants to follow the KISS principle. What should their primary goal be?**

- A. To use as many advanced design patterns as possible.
- B. To make the system as complex as possible to handle future requirements.
- C. **To favor simplicity in design and implementation, avoiding over-engineering.** ✅
- D. To write code that only they can understand.
- E. To ensure every function is at least 50 lines long.

---

**65. In ES Modules, which property provides the current module's file path (similar to `__filename` in CommonJS)?**

- A. import.filename
- B. module.url
- C. process.argv[1]
- D. **import.meta.url** ✅
- E. global.__filename

---

**66. After running `git add .`, what state have the changes been moved to?**

- A. Committed state
- B. Modified state
- C. Unstaged state
- D. **Staging area / Index** ✅
- E. Pushed state

---

**67. What is the purpose of using `express.Router()` to define routes in separate files?**

- A. To make the application run faster by reducing code size.
- B. It is the only way to define routes for POST requests.
- C. To automatically generate API documentation.
- D. To prevent any route conflicts with third-party middleware.
- E. **To organize route handlers by feature or resource, improving code maintainability and separation of concerns.** ✅

---

**68. What is the key difference between `git fetch` and `git pull`?**

- A. `git fetch` downloads changes and immediately merges them, while `git pull` only downloads them.
- B. **`git fetch` only downloads new data from the remote without integrating it, while `git pull` downloads and merges those changes into your current branch.** ✅
- C. `git fetch` is used for branches, while `git pull` is used for commits.
- D. There is no difference; they are aliases for the same command.
- E. `git fetch` requires an internet connection, while `git pull` does not.

---

**69. Which tool is specifically designed as an in-memory data store for caching frequent database queries?**

- A. **Redis** ✅
- B. MongoDB
- C. MySQL
- D. Git
- E. Multer

---

**70. Which practices align with the KISS principle? (Select all that apply)**

- A. **Writing clear, self-documenting code.** ✅
- B. **Breaking a complex task into smaller, simpler functions.** ✅
- C. Using an elaborate design pattern for a problem that only needs a simple loop. ❌
- D. **Avoiding unnecessary abstractions.** ✅
- E. Adding features that "might be useful in the future." ❌ *(That violates YAGNI too)*

---

**71. What is the value of `result` after this code?**

```js
let result = [1, 2, 3, 4].reduce((acc, curr, idx) => acc + curr * idx, 0);
```

- idx=0: 0 + (1×0) = 0
- idx=1: 0 + (2×1) = 2
- idx=2: 2 + (3×2) = 8
- idx=3: 8 + (4×3) = 20

**Answer: `20`** ✅

---

**72. What is the output of `console.log(!!"false" == !!"true")`?**

- `!!"false"` → `true` (non-empty string is truthy)
- `!!"true"` → `true`
- `true == true` → `true`

**Answer: `true`** ✅

---

**73. Which of the following are best practices for designing a RESTful API? (Select all that apply)**

- A. **Use nouns for resource names, not verbs.** ✅
- B. **Use appropriate HTTP status codes for responses.** ✅
- C. **Include versioning in the URL (e.g., /api/v1/users).** ✅
- D. Use GET for operations that modify data. ❌
- E. **Provide pagination for large collections.** ✅

---

**74. In system design, what is horizontal scaling?**

- A. Adding more resources (CPU, RAM) to a single server.
- B. **Adding more servers to distribute the load.** ✅
- C. Moving the application to a larger server.
- D. Reducing the number of servers to cut costs.
- E. Using a load balancer to direct traffic.

---

**75. What does "stateless" mean in the context of REST APIs?**

- A. **The server does not store any client session data between requests.** ✅
- B. The server stores client data in a database.
- C. The client must not store any data.
- D. The API cannot use cookies.
- E. The server must remember every request.

---

**76. What is the purpose of middleware in Express.js?**

- A. To directly render HTML views.
- B. **To handle requests and responses, performing tasks like logging, authentication, and parsing.** ✅
- C. To create database connections.
- D. To define routes for the application.
- E. To compile static assets.

---

**77. Which of the following are valid HTTP methods used in RESTful APIs? (Select all that apply)**

- A. **GET** ✅
- B. **POST** ✅
- C. **PUT** ✅
- D. **PATCH** ✅
- E. **DELETE** ✅

---

**78. Which Git command creates a new branch and switches to it immediately?**

- A. **`git checkout -b new-branch`** ✅
- B. `git branch new-branch` *(creates but doesn't switch)*
- C. `git checkout new-branch` *(switches only if it exists)*
- D. `git switch new-branch` *(switches only if it exists)*
- E. `git branch -m new-branch` *(renames)*

---

**79. Which are advantages of NoSQL databases over SQL databases? (Select all that apply)**

- A. Strict schema enforcement ensures data integrity. ❌ *(SQL advantage)*
- B. **Horizontal scalability is often easier.** ✅
- C. **Better suited for unstructured or semi-structured data.** ✅
- D. ACID compliance for complex transactions. ❌ *(SQL advantage)*
- E. **Flexible schema allows rapid iteration.** ✅

---

**80. What does "L" in SOLID stand for and mean?**

- A. **Liskov Substitution Principle — objects of a superclass should be replaceable with objects of its subclasses without affecting correctness.** ✅
- B. Law of Demeter
- C. Least Knowledge Principle
- D. List Inheritance Principle
- E. Loose Coupling Principle

---

**81. In `app.get('/user/:id', ...)`, how do you access the `id` parameter?**

- A. req.query.id
- B. req.body.id
- C. **req.params.id** ✅
- D. req.headers.id
- E. None of the above

---

**82. How do you set a custom response header `X-Custom` with value `"Hello"` in Express?**

- A. req.setHeader('X-Custom', 'Hello')
- B. res.set('X-Custom', 'Hello')
- C. res.header('X-Custom', 'Hello')
- D. **Both B and C** ✅
- E. None of the above

---

**83. You want to log the HTTP method and URL of every request. What is the best approach?**

- A. Add a console.log in every route handler.
- B. **Create a custom middleware that logs and calls next().** ✅
- C. Use a third-party logging library only.
- D. Log at the beginning of the server file.
- E. None of the above.

---

**84. Fill in the missing part to import a named export using ES Modules:**

```js
// math.js
export function add(a, b) {
  return a + b;
}

// main.js
import { add } from './math.js';
console.log(add(2, 3)); // 5
```

**Answer: `import { add } from './math.js';`** ✅

---

**85. Your API accepts JSON and form-data. What should your server support? (Select all that apply)**

- A. **Use `express.urlencoded({ extended: true })` for form data.** ✅
- B. **Use `express.json()` for JSON.** ✅
- C. Rely only on query strings for data. ❌
- D. **Use multer for multipart form-data.** ✅
- E. Express.Router is essential. ❌

---

**86. Which HTTP status codes relate to client-side errors?**

- A. 200 ❌ *(Success)*
- B. 300 ❌ *(Redirection)*
- C. 500 ❌ *(Server error)*
- D. **400** ✅ *(Client error range: 400–499)*
- E. 100 ❌ *(Informational)*

---

**87. What can `app.use()` be used for in Express?**

- A. **Register middleware** ✅
- B. Define a database model
- C. Listen to HTTP requests
- D. Collect a path
- E. None of the above

---

**88. Which statement imports many named exports from a module?**

- A. import default from 'module';
- B. import * from 'module'; *(needs alias: `import * as name`)*
- C. import all from 'module';
- D. **`import { func1, func2 } from 'module';`** ✅
- E. None of the above

---

**89. Which is a major security flaw in backend applications?**

- A. Using arrow functions
- B. **Not validating user inputs** ✅
- C. Using lowercase variable names
- D. Not using functions
- E. Using __filename

---

**90. Which of these is an example of an arrow function?**

- A. `function sayHi => () => { return "Hi"; }`
- B. `const sayHi = function() => { return "Hi"; }`
- C. **`const sayHi = () => { return "Hi"; }`** ✅
- D. `const sayHi = {} => { return "Hi"; }`
- E. `const sayHi = {} => return "Hi";`

---

# SECTION D — General Knowledge (Questions 91–100)

---

**91. To achieve an extraordinary result:**

- A. **You must do what others are not willing to do.** ✅
- B. Stick to random study patterns with occasional learning.
- C. Stay comfortable by mastering only familiar tasks.
- D. Rely on traditional approaches without real commitment.

---

**92. The only thing that can hinder you is:**

- A. **The person staring back at you in the mirror.** ✅
- B. The friends and peers you surround yourself with.
- C. The expectations and background of your family.
- D. The guides and authority figures in your life.

---

**93. You can determine your choice:**

- A. You share responsibility with others for results.
- B. **Internal forces are what truly shape the final outcome.** ✅
- C. You can't determine the consequence.
- D. What you get depends mostly on timing and chance.

---

**94. What is the full meaning of SSE?**

- A. Senior Student Executive
- B. Student Support Executive
- C. **Student Success Executive** ✅
- D. Student Support Expert

---

**95. What you seek is equally seeking:**

- A. Your current state and present efforts.
- B. **The transformed version of you.** ✅
- C. Your existing comfort zone and familiar patterns.
- D. Your present skill level and capabilities.

---

**96. Mention two individuals whose quotes were highlighted during your onboarding webinar:**

- A. JJS & Vosu
- B. ASJ & Vusi
- C. Vusi & AJS
- D. **AJS & Vinci** ✅ *(Refer to your onboarding materials to confirm)*

---

**97. What is the name of the HLP?**

- A. Blast
- B. **Blade** ✅ *(Refer to your cohort materials to confirm)*
- C. Faith
- D. Favour

---

**98. What is the name of your SSE?**

- A. Elizabeth
- B. Deborah
- C. **Precious** ✅ *(Refer to your cohort materials to confirm)*
- D. Florence

---

**99. "Your excuses are valid, but ___"**

- A. They shouldn't prevent you from moving forward.
- B. **They won't get you the results that you want.** ✅
- C. They don't change what needs to be done.
- D. They only limit your potential for growth.

---

**100. What was your midterm webinar topic?**

- A. Developing essential skills for long-term success.
- B. **Navigating the difference: From learning to actual building.** ✅
- C. Creating meaningful impact through strategic planning.
- D. Mastering the fundamentals of project development.

---

*© TechCrush Backend Engineering Track — Cohort V Examination*
*Compiled Study Guide — All Sections A through D*
