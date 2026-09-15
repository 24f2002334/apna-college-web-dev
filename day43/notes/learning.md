```bash

 Express.js Middleware — Complete Notes
Day 43 | Apna College Web Dev
1. What is Middleware?
Definition: Middleware is an intermediary — a function that sits between the request and the response.

text
Client → Request → [ Middleware ] → Response → Client
In Express: Middleware functions run after the server receives the request and before the response is sent to the client.

Real-life analogy: Think of a security guard at a building entrance. Before you reach your destination (route handler), you pass through the guard (middleware) who checks your ID, logs your entry, etc.

2. What Can Middleware Do?
Task	Example
✅ Execute any code	Logging, validation
✅ Modify req and res objects	req.user = user, req.time = Date.now()
✅ End the request-response cycle	res.send("bye")
✅ Call the next middleware	next()
Golden Rule: Every middleware must either:

Send a response (res.send, res.json, etc.) → cycle ends

Call next() → pass control forward

⚠️ If it does neither → the request hangs forever.

3. Types of Middleware
Type	How it's used	Scope
Application-level	app.use(fn)	Entire app
Route-level	app.get("/path", fn, handler)	Specific route only
Router-level	router.use(fn)	Specific Router instance
Built-in	express.static(), express.urlencoded()	Built into Express
Third-party	body-parser, morgan, cors	Installed via npm
Error-handling	app.use((err, req, res, next) => {})	Catches errors (4 args)
4. Built-in Middleware
body-parser
Parses data inside the request body (JSON, form data) and converts it into a readable JS object so you can use req.body.

js
app.use(express.json());           // for JSON data
app.use(express.urlencoded({ extended: true })); // for form data
express.static
Serves static files (CSS, JS, images) from the backend to the frontend.

js
app.use(express.static("public"));
express.urlencoded
Makes form data accessible via req.body.

5. Middleware Syntax & Chaining
Basic Syntax
js
app.use((req, res, next) => {
  console.log("I am middleware");
  next(); // move to next
});
Chaining (multiple middleware)
js
app.use((req, res, next) => {
  console.log("1st middleware");
  next();
});

app.use((req, res, next) => {
  console.log("2nd middleware");
  next();
});

app.get("/", (req, res) => {
  res.send("Hi, I am root.");
});
Order of execution for ANY request:

text
1st middleware → 2nd middleware → route handler
⚠️ Critical: Middleware runs in the order it's defined. Even for wrong/nonexistent routes, global middleware still runs.

6. next() — Deep Dive
js
app.use((req, res, next) => {
  console.log("Before next");
  next();
  console.log("After next"); // ⚠️ THIS STILL RUNS!
});
Important: next() does not return — it just signals Express to continue. Code after next() will execute unless you return.

Best Practice: Use return next()
js
app.use((req, res, next) => {
  console.log("Before");
  return next();          // ✅ stops here
  console.log("Never runs");
});
Rule of thumb:

If sending response → return res.send(...)

If calling next → return next()

7. Middleware as an Array (Reusability)
js
function logUrl(req, res, next) {
  console.log("URL:", req.originalUrl);
  next();
}

function logMethod(req, res, next) {
  console.log("Method:", req.method);
  next();
}

const logStuff = [logUrl, logMethod];

app.get("/user/:id", logStuff, (req, res) => {
  res.send("User Info");
});
Why? Keeps code DRY — reuse the same middleware stack across routes.

8. Router-Level Middleware
Works the same as app-level, but attached to an express.Router().

js
const router = express.Router();

router.use((req, res, next) => {
  console.log("Router middleware");
  next();
});

router.get("/", (req, res) => res.send("Router Home"));

app.use("/admin", router); // mounts router at /admin
Use case: Modular routing — e.g., /users, /products, /admin each get their own router.

9. Utility Middleware (Logger Example)
A middleware that adds useful functionality, like logging every request.

js
app.use((req, res, next) => {
  req.time = new Date(Date.now()).toString();
  console.log(req.method, req.hostname, req.path, req.time);
  next();
});
Output example:

text
GET localhost /api 2024-01-15T10:30:00.000Z
What it logs:

req.method → GET/POST/PUT/DELETE

req.hostname → domain

req.path → route path

req.time → timestamp

10. Order Matters! (Route vs Middleware)
js
// Case 1: Middleware FIRST
app.use(logger);
app.get("/", handler);
// → logger runs BEFORE handler

// Case 2: Middleware AFTER route
app.get("/", handler);
app.use(logger);
// → handler runs FIRST (if it sends response, logger never runs)
Rule: Express executes functions in top-to-bottom order.

11. Authentication Middleware (Token Example)
The Goal
Protect /api route — only allow access if a valid token is passed as a query string.

The Code
js
const checkToken = (req, res, next) => {
  let { token } = req.query;

  if (token === "giveaccess") {
    return next(); // ✅ valid token → continue
  }

  res.status(403).send("ACCESS DENIED!"); // ❌ invalid
};

app.get("/api", checkToken, (req, res) => {
  res.send("data");
});
How it works
URL	Token	Result
/api?token=giveaccess	✅ valid	next() → "data"
/api?token=wrong	❌ invalid	"ACCESS DENIED!"
/api	❌ missing	"ACCESS DENIED!"
⚠️ Common Bug: Missing return before next(). Without it, res.send("ACCESS DENIED!") runs too → error "Cannot set headers after they are sent".

12. Error Handling
Throwing Errors
js
app.get("/wrong", (req, res) => {
  abcd = abcd; // ReferenceError!
});
Express Default Error Handler
When an error is thrown, Express catches it and sends:

Status 500

Error message + stack trace (in development)

Custom Error Handler (4 arguments)
js
app.use((err, req, res, next) => {
  console.error(err.message);
  res.status(500).send("Something broke!");
});
Key points:

Must have 4 parameters: (err, req, res, next)

Must be defined after all routes

Use throw new Error("msg") in middleware to trigger it

13. Complete Flow Diagram
text
┌─────────────┐
│   Client    │
│  (Browser)  │
└──────┬──────┘
       │ Request
       ▼
┌─────────────────────┐
│  Global Middleware  │  ← Logger, body-parser, cors
│  (app.use)          │
└──────┬──────────────┘
       │ next()
       ▼
┌─────────────────────┐
│  Route Middleware   │  ← Auth check, validation
│  (checkToken)       │
└──────┬──────────────┘
       │ next()
       ▼
┌─────────────────────┐
│   Route Handler     │  ← res.send()
└──────┬──────────────┘
       │ Response
       ▼
┌─────────────┐
│   Client    │
└─────────────┘

       (if error thrown)
              ▼
┌─────────────────────┐
│  Error Handler      │  ← (err, req, res, next)
└─────────────────────┘
14. Common Mistakes & Best Practices
❌ Mistake	✅ Fix
Forgetting next()	Always call next() or send response
Code after next() runs unexpectedly	Use return next()
Missing return before res.send	return res.send()
Error handler defined before routes	Place it after all routes
Error handler with 3 args	Must have 4 args (err, req, res, next)
Using app.use() for route-specific logic	Pass middleware directly to route: app.get("/api", checkToken, handler)
15. Quick Reference Cheatsheet
js
// ─── GLOBAL MIDDLEWARE ───
app.use((req, res, next) => { /* ... */ next(); });

// ─── ROUTE MIDDLEWARE ───
app.get("/path", middleware, (req, res) => { /* ... */ });

// ─── ARRAY MIDDLEWARE ───
app.get("/path", [mw1, mw2], handler);

// ─── ROUTER MIDDLEWARE ───
const router = express.Router();
router.use(mw);
app.use("/prefix", router);

// ─── ERROR HANDLER ───
app.use((err, req, res, next) => {
  res.status(500).send(err.message);
});

// ─── BUILT-IN ───
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static("public"));
16. Key Terms to Remember
Term	Meaning
Middleware	Function between request and response
next()	Pass control to next function
req.query	URL query params (?token=x)
req.body	Request body (needs body-parser)
req.params	Route params (/user/:id)
req.method	HTTP verb (GET/POST/etc.)
app.use()	Register middleware globally
Error handler	4-arg middleware for catching errors
🎯 Summary — 5 Things to Remember
Middleware = middleman between request & response.

Always call next() or send a response — never both.

Order matters — middleware runs top-to-bottom.

return next() prevents code after from running.

Error handler = 4 args, defined last.

```

abc 


```bash

aa

=abc

```



 Express.js Middleware — Complete Notes
Day 43 | Apna College Web Dev
1. What is Middleware?
Definition: Middleware is an intermediary — a function that sits between the request and the response.

text
Client → Request → [ Middleware ] → Response → Client
In Express: Middleware functions run after the server receives the request and before the response is sent to the client.

Real-life analogy: Think of a security guard at a building entrance. Before you reach your destination (route handler), you pass through the guard (middleware) who checks your ID, logs your entry, etc.

2. What Can Middleware Do?
Task	Example
✅ Execute any code	Logging, validation
✅ Modify req and res objects	req.user = user, req.time = Date.now()
✅ End the request-response cycle	res.send("bye")
✅ Call the next middleware	next()
Golden Rule: Every middleware must either:

Send a response (res.send, res.json, etc.) → cycle ends

Call next() → pass control forward

⚠️ If it does neither → the request hangs forever.

3. Types of Middleware
Type	How it's used	Scope
Application-level	app.use(fn)	Entire app
Route-level	app.get("/path", fn, handler)	Specific route only
Router-level	router.use(fn)	Specific Router instance
Built-in	express.static(), express.urlencoded()	Built into Express
Third-party	body-parser, morgan, cors	Installed via npm
Error-handling	app.use((err, req, res, next) => {})	Catches errors (4 args)
4. Built-in Middleware
body-parser
Parses data inside the request body (JSON, form data) and converts it into a readable JS object so you can use req.body.

js
app.use(express.json());           // for JSON data
app.use(express.urlencoded({ extended: true })); // for form data
express.static
Serves static files (CSS, JS, images) from the backend to the frontend.

js
app.use(express.static("public"));
express.urlencoded
Makes form data accessible via req.body.

5. Middleware Syntax & Chaining
Basic Syntax
js
app.use((req, res, next) => {
  console.log("I am middleware");
  next(); // move to next
});
Chaining (multiple middleware)
js
app.use((req, res, next) => {
  console.log("1st middleware");
  next();
});

app.use((req, res, next) => {
  console.log("2nd middleware");
  next();
});

app.get("/", (req, res) => {
  res.send("Hi, I am root.");
});
Order of execution for ANY request:

text
1st middleware → 2nd middleware → route handler
⚠️ Critical: Middleware runs in the order it's defined. Even for wrong/nonexistent routes, global middleware still runs.

6. next() — Deep Dive
js
app.use((req, res, next) => {
  console.log("Before next");
  next();
  console.log("After next"); // ⚠️ THIS STILL RUNS!
});
Important: next() does not return — it just signals Express to continue. Code after next() will execute unless you return.

Best Practice: Use return next()
js
app.use((req, res, next) => {
  console.log("Before");
  return next();          // ✅ stops here
  console.log("Never runs");
});
Rule of thumb:

If sending response → return res.send(...)

If calling next → return next()

7. Middleware as an Array (Reusability)
js
function logUrl(req, res, next) {
  console.log("URL:", req.originalUrl);
  next();
}

function logMethod(req, res, next) {
  console.log("Method:", req.method);
  next();
}

const logStuff = [logUrl, logMethod];

app.get("/user/:id", logStuff, (req, res) => {
  res.send("User Info");
});
Why? Keeps code DRY — reuse the same middleware stack across routes.

8. Router-Level Middleware
Works the same as app-level, but attached to an express.Router().

js
const router = express.Router();

router.use((req, res, next) => {
  console.log("Router middleware");
  next();
});

router.get("/", (req, res) => res.send("Router Home"));

app.use("/admin", router); // mounts router at /admin
Use case: Modular routing — e.g., /users, /products, /admin each get their own router.

9. Utility Middleware (Logger Example)
A middleware that adds useful functionality, like logging every request.

js
app.use((req, res, next) => {
  req.time = new Date(Date.now()).toString();
  console.log(req.method, req.hostname, req.path, req.time);
  next();
});
Output example:

text
GET localhost /api 2024-01-15T10:30:00.000Z
What it logs:

req.method → GET/POST/PUT/DELETE

req.hostname → domain

req.path → route path

req.time → timestamp

10. Order Matters! (Route vs Middleware)
js
// Case 1: Middleware FIRST
app.use(logger);
app.get("/", handler);
// → logger runs BEFORE handler

// Case 2: Middleware AFTER route
app.get("/", handler);
app.use(logger);
// → handler runs FIRST (if it sends response, logger never runs)
Rule: Express executes functions in top-to-bottom order.

11. Authentication Middleware (Token Example)
The Goal
Protect /api route — only allow access if a valid token is passed as a query string.

The Code
js
const checkToken = (req, res, next) => {
  let { token } = req.query;

  if (token === "giveaccess") {
    return next(); // ✅ valid token → continue
  }

  res.status(403).send("ACCESS DENIED!"); // ❌ invalid
};

app.get("/api", checkToken, (req, res) => {
  res.send("data");
});
How it works
URL	Token	Result
/api?token=giveaccess	✅ valid	next() → "data"
/api?token=wrong	❌ invalid	"ACCESS DENIED!"
/api	❌ missing	"ACCESS DENIED!"
⚠️ Common Bug: Missing return before next(). Without it, res.send("ACCESS DENIED!") runs too → error "Cannot set headers after they are sent".

12. Error Handling
Throwing Errors
js
app.get("/wrong", (req, res) => {
  abcd = abcd; // ReferenceError!
});
Express Default Error Handler
When an error is thrown, Express catches it and sends:

Status 500

Error message + stack trace (in development)

Custom Error Handler (4 arguments)
js
app.use((err, req, res, next) => {
  console.error(err.message);
  res.status(500).send("Something broke!");
});
Key points:

Must have 4 parameters: (err, req, res, next)

Must be defined after all routes

Use throw new Error("msg") in middleware to trigger it

13. Complete Flow Diagram
text
┌─────────────┐
│   Client    │
│  (Browser)  │
└──────┬──────┘
       │ Request
       ▼
┌─────────────────────┐
│  Global Middleware  │  ← Logger, body-parser, cors
│  (app.use)          │
└──────┬──────────────┘
       │ next()
       ▼
┌─────────────────────┐
│  Route Middleware   │  ← Auth check, validation
│  (checkToken)       │
└──────┬──────────────┘
       │ next()
       ▼
┌─────────────────────┐
│   Route Handler     │  ← res.send()
└──────┬──────────────┘
       │ Response
       ▼
┌─────────────┐
│   Client    │
└─────────────┘

       (if error thrown)
              ▼
┌─────────────────────┐
│  Error Handler      │  ← (err, req, res, next)
└─────────────────────┘
14. Common Mistakes & Best Practices
❌ Mistake	✅ Fix
Forgetting next()	Always call next() or send response
Code after next() runs unexpectedly	Use return next()
Missing return before res.send	return res.send()
Error handler defined before routes	Place it after all routes
Error handler with 3 args	Must have 4 args (err, req, res, next)
Using app.use() for route-specific logic	Pass middleware directly to route: app.get("/api", checkToken, handler)
15. Quick Reference Cheatsheet
js
// ─── GLOBAL MIDDLEWARE ───
app.use((req, res, next) => { /* ... */ next(); });

// ─── ROUTE MIDDLEWARE ───
app.get("/path", middleware, (req, res) => { /* ... */ });

// ─── ARRAY MIDDLEWARE ───
app.get("/path", [mw1, mw2], handler);

// ─── ROUTER MIDDLEWARE ───
const router = express.Router();
router.use(mw);
app.use("/prefix", router);

// ─── ERROR HANDLER ───
app.use((err, req, res, next) => {
  res.status(500).send(err.message);
});

// ─── BUILT-IN ───
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static("public"));
16. Key Terms to Remember
Term	Meaning
Middleware	Function between request and response
next()	Pass control to next function
req.query	URL query params (?token=x)
req.body	Request body (needs body-parser)
req.params	Route params (/user/:id)
req.method	HTTP verb (GET/POST/etc.)
app.use()	Register middleware globally
Error handler	4-arg middleware for catching errors
🎯 Summary — 5 Things to Remember
Middleware = middleman between request & response.

Always call next() or send a response — never both.

Order matters — middleware runs top-to-bottom.

return next() prevents code after from running.

Error handler = 4 args, defined last.