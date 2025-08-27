Middleware

1. Fundamentals
1. What is Middleware in Next.js, where in the request–response cycle does it run, and why does it exist compared to SSR and API routes?
2. How does Middleware differ from API routes, getServerSideProps, and Server Components?
3. What is the Edge Runtime in Next.js, how does it differ from the Node.js runtime, and why is Middleware sometimes faster than traditional SSR?


2. File Structure & Configuration
4. How do you create a Middleware file in Next.js, including correct naming conventions and file structure for middleware.ts or middleware.js?
5. How do you configure Middleware for specific routes using the matcher option, and what is the execution order if multiple Middleware files or matchers exist?


3. Core Features
6. What is the NextResponse object, and how can it be used to: redirect requests, rewrite URLs, and set custom headers?
7. What’s the difference between NextResponse.redirect() and NextResponse.rewrite()?
8. How do you read and set cookies inside Middleware, and how can you modify request or response headers for analytics, caching, or security (including adding CSP, CORS, HSTS, etc.)?


4. Authentication & Authorization
9. How do you protect routes using Middleware for authentication and role-based access control (RBAC)?
10. How do you handle JWT tokens or session cookies in Middleware, and what are best practices for redirecting unauthenticated users?


5. Advanced Use Cases
11. How can Middleware be used for localization, geolocation-based routing, A/B testing or feature flags, rate limiting, IP blocking, and logging requests without slowing responses?


6. Performance & Deployment
12. What are the performance implications of running Middleware at the edge, how does it interact with caching and CDNs on Vercel, what are the code size limits, and how can performance bottlenecks be avoided?


7. Testing & Debugging
13. How do you test Middleware locally (including simulating the Edge Runtime), handle errors in production, debug async operations, and avoid common mistakes?


8. Integration & Best Practices
14. How do you combine Middleware with API routes, chain multiple Middleware functions for complex logic, make Middleware reusable across projects, and in what real-world scenarios does Middleware outperform SSR or API routes?


=============================================================================================================================================================

Fundamentals
1. What is Middleware in Next.js, where in the request–response cycle does it run?
	What Is Middleware in Next.js?
		In Next.js, Middleware is a function that runs on the server before a request is completed (before a request is processed by your route handler) (e.g. page, API route, or sever component), allowing you to inspect, modify, and handle the request before it reaches a page or an API route.

	Where Does It Run in the Request–Response Cycle?
	*******
		Client Request
     			↓
		Middleware (Edge Runtime) (CDN)
		        ↓
		Route Handler (Page, API, etc.)
		        ↓
		Response Sent to Client
	*******  Diagram *******
		The typical request-response cycle with Middleware looks like this:
			Step 1: Client makes a request.
				User sends a request to your application (e.g., to /dashboard).
			Step 2: Middleware runs at the edge before your app code.
				This is where you can inspect the request, check cookies, read headers, or decide whether to rewrite/redirect it.
				(where your Middleware function runs.)
			Step 3: Middleware can either:
				Pass the request along unchanged to your app, or
				Redirect/Rewrite it to a different route, or
				Return a custom response entirely.
			The final response is sent back to the user

2. How does Middleware differ from API routes, getServerSideProps, and Server Components?
	This is where people often get confused because all four (Middleware, API routes, getServerSideProps, Server Components) touch the request–response pipeline but at different stages and with different capabilities. (Middleware, API routes, getServerSideProps, and Server Components all serve different purposes in Next.js)

	Middleware vs API Routes vs getServerSideProps vs Server Components
	Middleware
		When it runs: 
			Before a request reaches your app’s logic.

		Runtime: 
			Edge Runtime (Web APIs, no Node.js APIs).

		Capabilities:
			Inspect & modify request headers, cookies, URL.
			Redirect or rewrite requests before they hit the route.
			Run globally across multiple routes.

		Limitations:
			No access to request body (e.g., JSON form POST).
			No heavy computation (size limits + must be fast).

		Best for: Auth checks, A/B testing, geo-based routing, localization, IP blocking, lightweight security.

	API Routes
		When it runs: 
			When you hit an endpoint like /api/user.

		Runtime: 
			Node.js server (or Edge if explicitly configured).

		Capabilities:
			Full access to request + response body.
			Can connect to databases, run heavy logic, perform CRUD operations.
			Great for server-side business logic.

		Limitations:
			Adds latency (round trip to server).
			Scoped only to API endpoints (not global).

		Best for: REST/GraphQL APIs, handling POST/PUT/DELETE requests, data fetching from client.

	getServerSideProps
		When it runs: 
			At request time during page rendering.

		Runtime:
			Node.js server.

		Capabilities:
			Fetch data securely (DB calls, APIs).
			Pass props directly to a React page.

		Limitations:
			Runs on every page request, so it can be slower.
			Only works in the Pages Router (deprecated in App Router).

		Best for: Preloading data for a page before rendering, especially when SEO matters.
		(In App Router, getServerSideProps is replaced by Server Components + fetch())

	Server Components (App Router)
		When it runs:
			At render time when React renders your tree.

		Runtime:
			Node.js (or Edge if configured).

		Capabilities:
			Fetch data directly in the component using async/await.
			Run heavy logic without shipping JS to client.

		Composable — works inside your React tree.

		Limitations:
			Runs per render, not globally.
			Can’t intercept requests like Middleware does.

		Best for: Data-fetching and rendering UI in the App Router.

	Comparison Table
		Feature / Aspect					Middleware									API Routes									getServerSideProps				Server Components
		Purpose								Intercept & modify requests early			Handle backend logic (CRUD, etc.)			Fetch data for SSR				Render UI on the server
		Runs When							Before routing								On request to /api/*						On page request					During rendering of a page/component
		Runtime								Edge (fast, limited)						Node.js (full access)						Node.js							Node.js or Edge
		Access to Request Body				No											Yes											Yes								No
		Access to Headers/Cookies			Yes											Yes											Yes								Yes
		Can Redirect						Yes (NextResponse)							Yes (manually via response)					Yes (redirect in props)			No (handled outside)
		Can Modify Response					Yes (headers, rewrites)						Yes											Yes								Yes (HTML output)
		Use Case Examples					Auth, geo-routing, A/B testing				Form handling, DB queries, APIs				Dynamic SSR pages				UI logic with server-side data
		Performance							Very fast (runs at edge)					Slower (server-side)						Slower (server-side)			Fast (streamed, no client JS)
	

	Think of it this way:
		Middleware 			= traffic cop at the highway entrance.
		API Routes 			= dedicated service windows.
		getServerSideProps 	= waiter fetching fresh ingredients for each dish.
		Server Components 	= chef cooking with those ingredients right in the kitchen.

3. What is the Edge Runtime in Next.js, how does it differ from the Node.js runtime, and why is Middleware sometimes faster than traditional SSR?
	The Edge Runtime is a lightweight, secure, and globally distributed execution Javascript environment designed to run code close to the user — at CDN edge locations — rather than on centralized servers.
	Key Characteristics
		Runs on V8 isolates (like Cloudflare Workers or Vercel Edge Functions)
		Cold starts are minimal — near-instant execution.
		Limited APIs: no access to Node.js core modules (e.g., fs, net, child_process). (It is designed for speed and low latency, not for heavy computation.)
		Designed for fast, stateless operations like redirects, header manipulation, and auth checks.
		In Next.js, Middleware and Edge Functions run in this runtime.

	Edge Runtime vs Node.js Runtime
		Aspect				Edge Runtime																			Node.js Runtime
		Location			Runs on CDN/edge locations worldwide													Runs in centralized server environment
		APIs available		Limited to Web APIs (e.g., fetch, Request, Response, crypto.subtle)						Full Node.js APIs (fs, net, http, process access, etc.)
		Startup time		Near-instant (no cold start)															Higher cold start time
		Use cases			Lightweight request interception, personalization, redirects, caching, auth checks		Heavy logic, DB queries, file system access, business logic
		Code limits			Strict (around 1 MB compressed bundle)													Larger (tens of MBs)
		Persistence			Stateless, no file system, no long-running connections									Can hold connections, run background jobs
		Execution Model		Isolated, event-driven																	Full-featured, long-lived processes
		Latency				Low (closer to user)																	Higher (depends on server location)
		Memory & CPU		Limited																					More powerful

	Why Is Middleware Sometimes Faster Than Traditional SSR?
		Runs at the Edge
			Middleware executes before the request travels all the way to your origin server.
			Example: If a user is in India, the Middleware logic runs in a Vercel edge node in India, not in a US-based server.

			Traditional SSR (Server Side Rendering) runs on your origin server, which might be in a different region or even a different continent, leading to higher latency.
		
		Low-latency tasks
			Middleware doesn’t render HTML — it just modifies requests/responses.
			Perfect for redirects, rewrites, auth checks that don’t need heavy computation.

		No HTML Rendering
			SSR must fetch data, render HTML, and send it to the client — more compute. (Traditional SSR (getServerSideProps) fetches data, renders HTML, and returns it.)
			
			Middleware can short-circuit and send a response (redirect, 403, cached asset) without rendering a page.

		Instant cold starts
			Edge Runtime has no cold start penalty.
			
			Node.js SSR may spin up slower under load.

	Summary
		Edge Runtime is optimized for speed, proximity, and lightweight logic.
		Node.js Runtime is better for complex backend tasks and full server-side rendering.
		Middleware is faster because it runs at the edge, avoids rendering, and executes early in the request lifecycle.


File Structure & Configuration
4. How do you create a Middleware file in Next.js, including correct naming conventions and file structure for middleware.ts or middleware.js?
	How to Create a Middleware File in Next.js?
		File Name & Location
			The middleware file must be named exactly:
				middleware.ts (TypeScript)
				middleware.js (JavaScript)
			Place it in the root of your project (same level as pages/ or app/).

			my-app/
			├─ app/               # App Router
			├─ pages/             # Pages Router
			├─ public/
			├─ middleware.ts      # Middleware lives here
			└─ next.config.js

			You cannot put it inside pages/ or app/ — Next.js won’t detect it there.

		Basic Example
			// middleware.ts
			import { NextResponse } from 'next/server'
			import type { NextRequest } from 'next/server'
			export function middleware(request: NextRequest) {
				// Example: Block access to /dashboard if not logged in
				const isLoggedIn = request.cookies.get('token')?.value
				if (!isLoggedIn && request.nextUrl.pathname.startsWith('/dashboard')) {
					return NextResponse.redirect(new URL('/login', request.url))
				}
				// Allow request to continue
				return NextResponse.next()
			}
		
		Controlling Which Routes Middleware Runs On
			By default, Middleware runs on all routes.
			You can scope it with the matcher config:
			// Only run on /dashboard and /profile
			export const config = {
				matcher: ['/dashboard/:path*', '/profile/:path*'],
			}

			What Does matcher Do?
			'/dashboard/:path*' matches:
				/dashboard
				/dashboard/settings
				/dashboard/profile/edit
			[Same for /profile]

		Naming Conventions Recap
			Rule							Description
			File name						Must be middleware.ts or middleware.js (another file name is not allowed such as _middleware.ts or _middleware.js or middlewares.ts or middlewares.js etc.)
			Location						Must be in the root directory of your project
			No nested middleware			You cannot place middeware inside app/, pages/, or any other subfolder (middleware is must on root directory).
			Single middleware file			Only one middleware file is allowed per project (if you need multiple middewares, you must combine them inside the single file or import helper functions).

		In short:
			File = middleware.ts or middleware.js
			Location = project root
			Export a middleware() function
			Use NextResponse to rewrite, redirect, or continue requests
			Control scope with config.matcher

5. How do you configure Middleware for specific routes using the matcher option, and what is the execution order if multiple Middleware files or matchers exist?
	How to Configure Middleware for Specific Routes Using matcher?
		By default, Middleware runs on every request. To restrict it, you export a config object with a matcher property from your middleware.ts file:

		Basic Syntax
			// middleware.ts
			import { NextResponse } from 'next/server'
			import type { NextRequest } from 'next/server'
			export function middleware(request: NextRequest) {
				return NextResponse.next()
			}
			export const config = {
				matcher: ['/dashboard/:path*', '/profile/:path*'], 		// only these routes
			}

		Matcher syntax
			Pattern														Matches..
			/about														Only /about
			/dashboard/:path*											/dashboard, /dashboard/settings, etc.
			/api/:function*												Any API route under /api
			/:locale(en|fr)/:path*										Locale-based routing like /en/home, /fr/blog
			/((?!api|_next/static|_next/image|favicon.icc).*)			All routes except excluded ones

		Important exclusions
			Next.js automatically excludes:
				/_next/static/* (build assets)
				/_next/image/* (image optimizer)
				/favicon.ico
		If you write your own regex matcher, you must account for these exclusions manually.

		Execution Order: What If Multiple Matchers Exist?
			How does Next.js handle multiple matchers in middleware?
				In Next.js, you can only have a single middleware.ts file, but you can define multiple matchers inside its config. When a request comes in, the middleware checks the matchers in the order they are defined. If the request path matches one or more patterns, the middleware executes once for that request, and you can branch your logic inside the function to handle each route differently. This ensures a predictable execution flow: matcher filtering happens first, then the middleware runs before any route handlers, API routes, or server components.
				
				Example 1:
					export const config = {
						matcher: [
							'/dashboard/:path*',
							'/profile/:path*',
							'/admin/:path*',
							'/((?!api|_next|favicon.ico).*)',
						],
					};
				
				Explanation:
					- '**/dashboard/:path***' is matcher 1 — it matches /dashboard, /dashboard/settings, /dashboard/anything/else, etc.
					- '**/profile/:path***' is matcher 2 — it matches /profile, /profile/edit, /profile/view/123, and so on.
					And the others follow the same logic:
					- '**/admin/:path***' is matcher 3
					- '**/((?!api|_next|favicon.ico).*)**' is matcher 4 — this one’s a bit fancy: it uses a regular expression to exclude internal Next.js routes and static files

				Example 2:
					export const config = {
						matcher: ["/((?!_next/static|favicon.ico).*)", "/admin/:path*"],
					};

				Explanation:
					First matcher (/((?!_next/static|favicon.ico).*)) → applies to all routes except static files.
					Second matcher (/admin/:path*) → more specific, also included.
					Middleware is still executed once per request; order of evaluation inside the function decides which block executes.
				 
				when you define multiple matchers within a single middleware.ts file, they are evaluated independently for each incoming request. The execution order isn’t sequential or prioritized — instead, the middleware runs if any matcher in the array matches the request path. This means all matchers act as parallel filters, and the middleware logic executes once per request, regardless of how many matchers match. Since Next.js supports only one middleware file per project, the focus is on crafting precise matcher patterns to control where and when the middleware activates.

			How to Make Middleware Code Manageable and Reusable in Next.js?
				In Next.js, only one middleware.ts (or middleware.js) file is allowed in the project root, and it has a size limit of 1 MB (roughly 1000–2000 lines of code). Managing all your logic in a single large file quickly becomes messy and hard to maintain.
				The easiest solution is to split your logic into multiple files and then import them into the main middleware.ts file.

				Example structure:
					middleware/
					├── auth.ts   # For authentication checks
					├── admin.ts  # For admin-specific logic
					└── other.ts  # For other middleware logic

				Your main middleware.ts file in the project root would look like this:
					import authMiddleware from "./middleware/auth";
					import adminMiddleware from "./middleware/admin";
					export function middleware(request: Request) {
						// Run different middleware logic
						const authResult = authMiddleware(request);
						const adminResult = adminMiddleware(request);
						// Combine logic or return one of them
						return authResult || adminResult;
					}

				Benefits of this approach:
					Manageability → You don’t have to deal with one massive file; instead, each responsibility is isolated.
					Reusability → For example, middleware/auth.ts can be reused in other projects or extended later.
					Clean structure → Your logic is organized and easier to test, debug, and maintain.

				Splitting into multiple files doesn’t reduce the final bundle size (it still counts toward the 1 MB Edge Runtime limit), but it greatly improves maintainability and reusability.

3. Core Features
6. What is the NextResponse object, and how can it be used to: redirect requests, rewrite URLs, and set custom headers?
	What is the NextResponse object?
		The NextResponse object is the powerhouse of control inside Next.js middleware. It’s what you use to respond to, redirect, rewrite, or modify an incoming request before it reaches your route handler.
		NextResponse is a utility provided by Next.js that lets you:
		- Continue the request (NextResponse.next())
		- Redirect the user (NextResponse.redirect(url))
		- Rewrite the request path (NextResponse.rewrite(url))
		- Modify headers or cookies on the response
		It’s part of the next/server module and is designed to work within the Edge Runtime.

			Why do we need it?
				Since Middleware runs before the request reaches your route handler or page, NextResponse acts as your “traffic controller.” It controls whether the request:
				1. Proceeds normally,
				2. Gets redirected,
				3. Or gets handled differently (e.g., blocked, authenticated, rewritten).

	How can it be used to: redirect requests, rewrite URLs, and set custom headers?
		1. Redirect Requests
			Use NextResponse.redirect() to send users to a different URL. Perfect for auth guards, onboarding flows, or geo-based redirects.

			import { NextResponse } from 'next/server'
			import type { NextRequest } from 'next/server'
			export function middleware(req: NextRequest) {
			const isLoggedIn = req.cookies.get('token')?.value
				if (!isLoggedIn && req.nextUrl.pathname.startsWith('/dashboard')) {
					return NextResponse.redirect(new URL('/login', req.url))
				}
				return NextResponse.next()
			}

		2. Rewrite URLs
			Use NextResponse.rewrite() to internally reroute a request without changing the URL in the browser. Great for A/B testing, feature flags, or serving content from a different path.

			export function middleware(req: NextRequest) {
  				if (req.nextUrl.pathname === '/blog') {
					return NextResponse.rewrite(new URL('/blog/index.html', req.url))
  				}
				return NextResponse.next()
			}
		
		3. Set Custom Headers
			Use NextResponse.next() and then modify the response headers. Useful for security headers, caching, or passing metadata downstream.

			export function middleware(req: NextRequest) {
				const response = NextResponse.next()
				response.headers.set('x-gs-powered-by', 'Next.js Middleware')
				response.headers.set('x-feature-flag', 'beta-dashboard')
				return response
			}

			You can also read headers from the request:
			const userAgent = req.headers.get('user-agent')
		
		Combine All Three
			You can mix and match these in a single middleware flow:
			
			export function middleware(req: NextRequest) {
				if (req.nextUrl.pathname === '/old-route') {
					return NextResponse.redirect(new URL('/new-route', req.url))
				}

				if (req.nextUrl.pathname === '/preview') {
					return NextResponse.rewrite(new URL('/preview/index.html', req.url))
				}

				const response = NextResponse.next()
				response.headers.set('x-gs-debug', 'active')
				return response
			}
	
	Common Methods
		Method									Purpose																			Example
		NextResponse.next()						Allows the request to continue normally											return NextResponse.next()
		NextResponse.redirect()					Redirects to a different URL													return NextResponse.redirect(new URL('/login', request.url))
		NextResponse.rewrite()					Internally rewrites the request path (URL in browser doesn’t change)			return NextResponse.rewrite(new URL('/new', request.url))
		response.headers.set()					Sets custom headers on the response												response.headers.set('X-Auth', 'true')
		response.headers.append()				Appends a value to an existing header											response.headers.append('Set-Cookie', 'theme=dark')
		response.cookies.set()					Sets cookies on the response													response.cookies.set('user', 'GS')
		response.cookies.delete()				Deletes a cookie from the response												response.cookies.delete('user')
		response.cookies.get()					Reads a specific cookie															const user = response.cookies.get('user')
		NextResponse.json()						Returns a JSON response (useful in middleware APIs)								return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
		NextResponse.redirect(URL, status?)		Redirect with status (default 307)												return NextResponse.redirect(new URL('/', request.url), 308)
		NextResponse.error()					Returns a generic error response (500)											return NextResponse.error()

	Why It Matters
		- NextResponse gives you low-level control over how requests are handled.
		- It’s optimized for the Edge Runtime, so it’s fast and lightweight.
		- It’s the only way to manipulate requests/responses inside middleware — you can’t use res.writeHead() or res.end() like in Node.js.

7. What’s the difference between NextResponse.redirect() and NextResponse.rewrite()?
	NextResponse.redirect()
		What it does: Redirects the user to a different URL.
		Effect: The browser’s URL changes (client sees the new URL).
		Use case: Redirecting unauthenticated users to /login, or redirecting old routes to new ones.

		Example:
			import { NextResponse } from "next/server";
			export function middleware(req) {
			const isLoggedIn = false;
			if (!isLoggedIn) {
				return NextResponse.redirect(new URL("/login", req.url));
			}
			return NextResponse.next();
			}

		If a user visits /dashboard, they get redirected to /login, and the browser shows /login.

	NextResponse.rewrite()
		What it does: Internally serves a different page while keeping the original URL in the browser.
		Effect: The browser’s URL does not change.
		Use case: Serving different content for the same route, A/B testing, localization, or proxying routes.

		Example:
			import { NextResponse } from "next/server";
			export function middleware(req) {
			const url = req.nextUrl;
			if (url.pathname === "/about") {
				return NextResponse.rewrite(new URL("/info", req.url));
			}
			return NextResponse.next();
			}

		If a user visits /about, they see the content from /info, but the browser still shows /about.

		Comparision Table
		Feature							redirect()													rewrite()
		User-visible URL change			Yes — browser URL changes									No — browser URL stays the same
		Client round-trip				Yes — triggers a full client-side redirect					No — handled internally at the edge
		Use case						Auth redirects, onboarding flows, external links			Localization, A/B testing, internal routing
		Performance Impact				Slightly higher — involves network round-trip				Lower — stays within edge runtime
		SEO implications				Can affect crawl/indexing									Transparent to crawlers — no redirect status code

		Key Difference
			redirect() → Changes the user’s URL (client-side redirect).
			rewrite() → Keeps the same URL but serves different content (server-side rewrite).

8. How do you read and set cookies inside Middleware, and how can you modify request or response headers for analytics, caching, or security (including adding CSP, CORS, HSTS, etc.)?
	1. Reading & Setting Cookies in Middleware
		In Next.js Middleware, you use the cookies API available on the NextRequest and NextResponse objects.

		Read Cookies
		import { NextResponse } from "next/server";
		import type { NextRequest } from "next/server";
		export function middleware(req: NextRequest) {
			// Reading cookies from the request
			const token = req.cookies.get("token")?.value;
			if (!token) {
				// If no token, redirect to login
				return NextResponse.redirect(new URL("/login", req.url));
			}

			return NextResponse.next();
		}

		Set Cookies
			import { NextResponse } from "next/server";
			import type { NextRequest } from "next/server";
			export function middleware(req: NextRequest) {
				const res = NextResponse.next();
				// Set a cookie
				res.cookies.set("user", "GS", {
					httpOnly: true,
					secure: true,
					path: "/",
					sameSite: "strict",
				});
				return res;
			}

		You can also delete cookies:
			res.cookies.delete("token");

		Cookies are great for auth tokens, user preferences, A/B testing buckets, and session hints — but remember, you can’t access the request body in middleware, so cookies are your best bet for lightweight state.
	
	2. Modifying Request & Response Headers
		In Next.js, cookies are primarily accessed through the request headers, but you can also read and write cookies in the response—especially in middleware, route handlers, and server actions.
		You can add, remove, or update headers on the request or response for analytics, caching, or security.

		Adding Custom Headers (Analytics / Debugging)

			export function middleware(req: NextRequest) {
				const res = NextResponse.next();
				// Example: custom analytics header
				response.headers.set('X-Request-ID', crypto.randomUUID());
				response.headers.set('X-Geo-Region', request.geo?.region || 'unknown');
			return res;
			}


		Setting Security Headers (CSP, HSTS, CORS, etc.)
			
			export function middleware(req: NextRequest) {
				const res = NextResponse.next();
				// Content Security Policy
				res.headers.set(
					"Content-Security-Policy",
					"default-src 'self'; script-src 'self' https://trustedscripts.example.com"
				);
				// Strict Transport Security (HSTS)
				res.headers.set("Strict-Transport-Security", "max-age=63072000; includeSubDomains; preload");
				// Cross-Origin Resource Sharing (CORS)
				res.headers.set("Access-Control-Allow-Origin", "*");
				res.headers.set("Access-Control-Allow-Methods", "GET, POST, OPTIONS");
				res.headers.set("Access-Control-Allow-Headers", "Content-Type, Authorization");
				// Prevent clickjacking
				res.headers.set("X-Frame-Options", "DENY");
				// Prevent MIME-sniffing
				res.headers.set("X-Content-Type-Options", "nosniff");
				return res;
			}



		Common Security Headers in Middleware

		Header												Purpose																												Example Value																						Usage in Middleware
		Content-Security-Policy (CSP)						Controls what resources (scripts, styles, images, etc.) the browser can load, preventing XSS attacks.				default-src 'self'; script-src 'self' https://cdn.example.com; object-src 'none'					res.headers.set("Content-Security-Policy", "default-src 'self'; script-src 'self' https://cdn.example.com; object-src 'none'")
		Strict-Transport-Security (HSTS)					Forces browsers to use HTTPS only, preventing protocol downgrade attacks.											max-age=63072000; includeSubDomains; preload														res.headers.set("Strict-Transport-Security", "max-age=63072000; includeSubDomains; preload")
		Access-Control-Allow-Origin (CORS)					Defines which origins can access resources, preventing unauthorized cross-origin requests.							* (allow all) or https://myapp.com (restrict)														res.headers.set("Access-Control-Allow-Origin", "https://myapp.com")
		X-Frame-Options										Prevents your site from being loaded in an <iframe>, blocking clickjacking attacks.									DENY or SAMEORIGIN																					res.headers.set("X-Frame-Options", "DENY")
		X-Content-Type-Options								Prevents MIME type sniffing, ensuring files are only interpreted as their declared type.							nosniff																								res.headers.set("X-Content-Type-Options", "nosniff")
		Referrer-Policy										Controls how much referrer information (URL) is shared with requests.												strict-origin-when-cross-origin																		res.headers.set("Referrer-Policy", "strict-origin-when-cross-origin")
		Permissions-Policy (previously Feature-Policy)		Controls access to browser features like camera, microphone, geolocation.											geolocation=(), camera=(), microphone=()															res.headers.set("Permissions-Policy", "geolocation=(), camera=(), microphone=()")
		Cache-Control										Defines caching behavior to improve performance and security.														public, max-age=31536000, immutable																	res.headers.set("Cache-Control", "public, max-age=31536000, immutable")


		Example: Adding All Security Headers in Middleware
			
			import { NextResponse } from "next/server";
			import type { NextRequest } from "next/server";
			export function middleware(req: NextRequest) {
				const res = NextResponse.next();
				res.headers.set(
					"Content-Security-Policy",
					"default-src 'self'; script-src 'self' https://cdn.example.com; object-src 'none'"
				);
				res.headers.set("Strict-Transport-Security", "max-age=63072000; includeSubDomains; preload");
				res.headers.set("Access-Control-Allow-Origin", "https://myapp.com");
				res.headers.set("X-Frame-Options", "DENY");
				res.headers.set("X-Content-Type-Options", "nosniff");
				res.headers.set("Referrer-Policy", "strict-origin-when-cross-origin");
				res.headers.set("Permissions-Policy", "geolocation=(), camera=(), microphone=()");
				res.headers.set("Cache-Control", "public, max-age=31536000, immutable");
				return res;
			}

Authentication & Authorization
9. How do you protect routes using Middleware for authentication and role-based access control (RBAC)?
	In Next.js, Middleware is perfect for route protection because it runs before the request is completed, allowing you to inspect cookies, headers, or tokens and decide whether to allow, block, or redirect the request — making authentication and role-based access control (RBAC) one of the most powerful use cases for the Edge Runtime.
	1. Authentication (checking login status)
		You typically store a session token or JWT in cookies. Middleware checks for that token, and if it’s missing/invalid, it redirects the user to a login page.
		// middleware.ts
		import { NextResponse } from "next/server";
		import type { NextRequest } from "next/server";
		export function middleware(req: NextRequest) {
			const token = req.cookies.get("token"); // session/JWT
			if (!token) {
				// Redirect to login if not authenticated
				return NextResponse.redirect(new URL("/login", req.url));
			}
			// Allow request to continue
			return NextResponse.next();
		}
		// Match protected routes only
		export const config = {
			matcher: ["/dashboard/:path*", "/profile/:path*"], 
		};

	2. Role-Based Access Control (RBAC)
	You can store a role in the cookie/session (e.g., "admin", "user", "editor"). Middleware reads it, and checks if the user has the right permission for the route.
		// middleware.ts
		import { NextResponse } from "next/server";
		import type { NextRequest } from "next/server";
		export function middleware(req: NextRequest) {
			const token = req.cookies.get("token"); 
			const role = req.cookies.get("role"); // e.g., "admin" or "user"
			if (!token) {
				return NextResponse.redirect(new URL("/login", req.url));
			}
			// Example RBAC: only admins can access /admin/*
			if (req.nextUrl.pathname.startsWith("/admin") && role !== "admin") {
				return NextResponse.redirect(new URL("/unauthorized", req.url));
			}
			return NextResponse.next();
		}
		export const config = {
			matcher: ["/dashboard/:path*", "/admin/:path*"], 
		};
	Benefits of Middleware-Based RBAC
	- Fast: Runs at the edge, before routing.
	- Secure: Prevents access before hitting route logic.
	- Scalable: Easily extendable to new roles and routes.

10. How do you handle JWT tokens or session cookies in Middleware, and what are best practices for redirecting unauthenticated users?
	In Next.js Middleware, you typically handle authentication using JWT tokens or session cookies. The process starts with reading the cookie or header from the incoming request (using req.cookies.get() or req.headers.get()). For JWTs, you decode and verify the signature to ensure the token is valid and not expired. For session cookies, you check whether the cookie exists and, if needed, validate it against your session store.
		If the request is unauthenticated (missing, invalid, or expired token/session), the best practice is to:
		Redirect users to your login page using NextResponse.redirect().
		Preserve the intended destination (e.g., using a redirectTo query param) so they can be sent back after logging in.
		Keep sensitive routes protected while still allowing public routes (like /login, /signup, /about) to bypass the check.
	This approach ensures your app remains secure, user-friendly, and scalable at the edge.

	This how you achieve this...

	Handling JWTs and session cookies in Next.js middleware is all about early interception, lightweight validation, and smart redirection — all at the edge, before your app even starts rendering.
	Handling JWT Tokens or Session Cookies in Middleware

	Step 1: Read the Cookie
	Middleware runs in the Edge Runtime, so you use request.cookies.get():
		const token = request.cookies.get('auth-token')?.value;

	You can also use headers if your token is passed via Authorization, but cookies are more common for web apps.

	Step 2: Decode or Verify the Token
	You can decode the JWT to extract user info (like role or ID). But remember: Edge Runtime has limited support for Node libraries, so use lightweight or edge-compatible JWT parsers.
		import { jwtVerify } from 'jose'; // Edge-compatible
		const secret = new TextEncoder().encode(process.env.JWT_SECRET);
		const { payload } = await jwtVerify(token, secret);

	Avoid using heavy libraries like jsonwebtoken unless you're sure they're edge-safe.

	Step 3: Check Authentication & Role
	Use the decoded payload to enforce access control:
		if (!payload || !payload.role) {
			return NextResponse.redirect(new URL('/login', request.url));
		}

		if (request.nextUrl.pathname.startsWith('/admin') && payload.role !== 'admin') {
			return NextResponse.redirect(new URL('/unauthorized', request.url));
		}

	Best Practices for Redirecting Unauthenticated Users
	1. Use NextResponse.redirect()
	This sends a 302 redirect to the client:
		return NextResponse.redirect(new URL('/login', request.url));

	2. Preserve the Original Path
	So users can return after login:
		const loginUrl = new URL('/login', request.url);
		loginUrl.searchParams.set('redirect', request.nextUrl.pathname);
		return NextResponse.redirect(loginUrl);

	3. Scope Middleware with matcher
	Only run middleware on protected routes:
	export const config = {
		matcher: ['/dashboard/:path*', '/admin/:path*'],
	};

	4. Avoid Redirect Loops
	Make sure /login and /unauthorized are excluded from middleware logic.
	Bonus: Modular Auth Middleware
	Split logic into reusable functions:
	export async function verifyAuth(request: NextRequest) {
		const token = request.cookies.get('auth-token')?.value;
		if (!token) return false;
		try {
			const { payload } = await jwtVerify(token, secret);
			return payload;
		} catch {
			return false;
		}
	}

	Then use it in your main middleware.ts:
		const user = await verifyAuth(request);
		if (!user) return redirectToLogin(request);


Advanced Use Cases
11. How can Middleware be used for localization, geolocation-based routing, A/B testing or feature flags, rate limiting, IP blocking, and logging requests without slowing responses?
	How can Middleware be used for localization?
		What Is Localization?
		Localization is the process of adapting your application’s content, layout, and behavior to match the language, region, and cultural preferences of the user. It’s not just translation.
		It includes:
		- Translating text into different languages (e.g., English, Hindi, French)
		- Formatting dates, currencies, and numbers based on locale
		- Serving region-specific content (e.g., Indian promotions vs. US offers)
		- Adjusting layout for right-to-left (RTL) languages like Arabic or Hebrew

		How Middleware Helps with Localization in Next.js
		Middleware runs before routing, which makes it perfect for detecting and handling localization logic early in the request lifecycle.
		Common Use Cases:
		- Detect user locale from cookies, headers, or IP
		- Redirect to locale-specific routes (e.g., /en, /hi, /fr)
		- Rewrite URLs internally to serve localized content
		- Set locale cookies for future requests

		Example: Locale Detection & Redirect
			import { NextResponse } from 'next/server';
			import type { NextRequest } from 'next/server';
			export function middleware(request: NextRequest) {
				const pathname = request.nextUrl.pathname;

				// Skip static files and API routes
				if (pathname.startsWith('/api') || pathname.includes('.')) {
					return NextResponse.next();
				}

				// Check if locale is already in the URL
				const locales = ['en', 'hi', 'fr'];
				const hasLocale = locales.some((locale) => pathname.startsWith(`/${locale}`));
				if (hasLocale) return NextResponse.next();

				// Detect locale from cookie or header
				const cookieLocale = request.cookies.get('locale')?.value;
				const headerLocale = request.headers.get('Accept-Language')?.split(',')[0].slice(0, 2);
				const rawLocale = cookieLocale || headerLocale;
				const detectedLocale = locales.includes(rawLocale || '') ? rawLocale : 'en';

				// Redirect to locale-prefixed route
				const normalizedPath = pathname === '/' ? '' : pathname;
				return NextResponse.redirect(new URL(`/${detectedLocale}${normalizedPath}`, request.url));	
			}

			export const config = {
				matcher: ['/((?!api|_next|favicon.ico).*)'],
			};

		Best Practices
		- Use cookies to persist user locale across sessions
		- Fallback to browser language if no cookie is set
		- Avoid redirect loops by checking if locale is already present
		- Use matcher to exclude static assets and API routes

		In short:
		Middleware makes localization seamless by detecting language preferences and redirecting users to the right localized route, ensuring a smoother global user experience.

	How can Middleware handle geolocation-based routing (sreving content by user's country/region)?
		What Is Geolocation-Based Routing?
			In Next.js, Middleware can handle geolocation-based routing by using the request.geo object, which provides country, region, city, and latitude/longitude (only available on Vercel Edge). With this, you can decide which region-specific content or server to serve.
			It’s the practice of detecting a user’s physical location (usually by IP) and then:
				- Redirecting them to a region-specific version of your site
				- Serving localized content (e.g., /in, /us, /uk)
				- Applying country-specific logic (currency, language, legal disclaimers, etc.)

		How Middleware Enables This in Next.js
			In Next.js Middleware, geolocation-based routing works by detecting the user’s location (usually through request headers like x-vercel-ip-country, x-vercel-ip-city, etc., automatically provided when deploying on Vercel Edge). Based on that information, you can decide which version of your site or content to serve.

		Example: Redirect Based on Country
			// middleware.ts
				import { NextResponse } from "next/server";
				import type { NextRequest } from "next/server";

				const supportedRegions = new Set(['in', 'us', 'uk']);
				const fallbackRegion = 'us';

				export function middleware(request: NextRequest) {
				const { pathname } = request.nextUrl;

				// Skip static files and API routes
				if (pathname.startsWith('/api') || pathname.includes('.')) {
					return NextResponse.next();
				}

				// Check if region is already in the path
				const hasRegion = Array.from(supportedRegions).some(region =>
					pathname.startsWith(`/${region}`)
				);
				if (hasRegion) return NextResponse.next();

				// Get region from geo or fallback
				let region = request.geo?.country?.toLowerCase() || fallbackRegion;

				// Validate region
				if (!supportedRegions.has(region)) {
					region = fallbackRegion;
				}

				// Rewrite to region-prefixed path
				const newPath = pathname === '/' ? `/${region}` : `/${region}${pathname}`;
					return NextResponse.rewrite(new URL(newPath, request.url));
				}

				export const config = {
					matcher: ['/((?!api|_next|favicon.ico).*)'],
				};


		NextResponse.rewrite() lets you serve region-specific content without changing the URL.
		NextResponse.redirect() can be used if you want to explicitly send users to a region-specific path.


		Common Use Cases
			Redirect Indian users to /in
			Serve EU users from an EU data center for GDPR compliance
			Route Asian users to faster servers

		Best Practices
			Always fallback gracefully if request.geo is undefined (e.g., in local dev or unsupported platforms).
			Prevent redirect loops by checking if the region is already in the path.
			Use cookies to persist the region choice if a user overrides it manually.
			Combine with localization for full geo-language routing (e.g., /in/hi, /us/en).
			Always set a fallback region (like US) so all users get content even without geo headers.


	How can Middleware enable A/B testing or feature flags for experiments?

