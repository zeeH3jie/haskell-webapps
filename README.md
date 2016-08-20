# Motivation

* Too much choice with respect to libraries when building a typical Postgres-backed webapp. Most people don't know enough about advantages/disadvantages to make the right choice.
  * Web server + routing + cookies + headers
  * Templating (primarily HTML templates)
  * DB access (Postgres only)
  * DB migrations
  * Redis for caching expensive things -- eg. expensive DB queries or JSON generation
  * Authentication
  * Authorization
  * Audit logs
  * Logging
  * Validating incoming user-input
  * Dealing with static assets (i.e. images, JS, CSS) - during development and during production. Integrating non-Haskell tech into the static asset toolchain (eg. Coffeescript, Typescript, LessCSS, Sass, asset combining and minification)
  * Automated testing - unit testing, controller/functional testing, integration tests using Selenium (at least)
  * Anything else?
* I believe most of these things are *possible* in Haskell and its rich library ecosystem. However, making all of this work is *not as easy as it ought to be.* This is because the *idiomatic* and *will-work for-80%-of-the-use-cases-with-20%-effort* way of dealing with these things is not documented properly in one place. This gets more complicated due to the library fragmentation. Libraries make very different decision choices and take **very different** approaches for solving the same problem. Transliterating the idiomatic way from one library to the other may not result in the most pragmatic codebase.
* While there are great tutorials (either provided by the project maintainers themselves or in various blog posts), my experience is that most tutorials walk you through the most basic scenarios. I wasn't able to *easily* find answers for most real-life scenarios. Asking on various forums like IRC (#haskell), [/r/haskell](https://www.reddit.com/r/haskell), or StackOverflow, while helpful (the Haskell community is the most helpful and knowledgable community that I have come across) has the following problems:
  * Most forums are designed for Q&A format. Not long-form opinion or long-form code-sharing. Even explaining the problem that I'm facing takes a long-form question, which most people don't have time to read. Questions that don't have a narrow focus or a one correct answer are actively discouraged on StackOverflow nowadays.
  * Discussions quickly migrate from pragmatic, getting-things-done zone to philosophical debates.
  * My personal experience is that most people in the Haskell community haven't written large-scale webapps. Or, the ones that have written aren't active on the channels that I hang-out on. In fact, apart from FP Compllete Web IDE, I don't yet know of large-scale modern webapps written in Haskell. Even if there are commercial outfits doing that, there's no way for one to look at their code and learn webapp patterns in Haskell.
* I'm scratching my own itch. I'm running a SaaS company, [Vacation Labs](https://www.vacationlabs.com), which has a Rails+AngularJS codebase that has grown to [250,000+ LoC](http://www.vacationlabs.com/we-are-hiring/software-engineer/) over the past 4 years. We're experiencing the disadvantages of using a dynamically typed language on a very large code-base. While automated tests help (unit tests & controller tests), they don't give enough correctness guarantees that something like Haskell can give. I wanted to quickly evaluate Haskell for our use-case, but going through the steep learning curve (functors, monads, laziness, purity, Reader, monad transformers, etc.) has taken a lot of calendar-days (I don't have too much free time these days). I'm not left with enough time to evaluate multiple libraries to pick the best. Therefore, I want to crowd-source the effort, generating a valuable community resource in the process.

# The Plan

* Spec out a typical Postgres-backed web-app which covers all the points mentioned in the "Motivation" section. 
  * Design the DB schema to include the following commonly occuring webapp requirements:
    * one:one, one:many, and many:many associations
    * created-at and updated-at housekeeping columns
    * unified audit-log table with "before" and "after" columns implemented as HSTOREs
    * using JSONB and Postgres-arrays in the DB schema
  * Design a JSON API which covers all domain-level operations, which a hypothetical "dumb" UI may consume. Why a "dumb" UI? Because IMO pushing domain-logic to the UI layer isn't a good idea. Generally, UIs are harder to test, and in today's multi-screen/multi-device world you'll end up implementing multiple UIs for the same app. Thus, forcin you to implement the domain-logic multiple times -- for your single-page-app (SPA) in Javascript, your Android app, your iOS app, your Windows app, and your desktop app.
  * Design a Bootstrap-3 based UI (with HTML & LessCSS) with enough HTML-level duplication to test-out all use-cases of a templating library.
* However, it won't make sense to implement the entire app (UI **and** API) for every possible combination of libraries. Therefore, implement the spec in three distinct phases. The assumption is that the phases are loosely coupled and the library choices for one phase do not impact the other phase significantly. (This might be not be true in the cases of frameworks like Yesod, but then again, this is not supposed to be a scientifically controlled experiment).
  * Phase 1: Domain-level API + DB-access + Redis-caching + validations (common to both UI approaches)
  * Phase 2.1: JS-powered SPA (single page app)
  * Phase 2.2: Server-powered HTML UI
  * Phase 3: Testing (common to both UI approaches)
  * Phase 4: Deployment (common to both UI approaches)
  * Phase 5: Supporting the app in production - logging, debugging, and troubleshooting

## Phase 1: Domain-level API + DB-access + Validations

1. Low-level DB functions, DB<=>Haskell mapping, and house-keeping columns (createdAt, updatedAt)
  * Opaleye
  * Groundhog 
  * Persistent
  * Postgresql-ORM
2. Domain-level API with validations (which takes care of fetching records that may have DB-associations)
3. Audit logging

## Phase 2.1: JS-powered SPA

1. Domain-API mentioned in the section above will be reused
2. Write JSON-based API to access the domain API
  * Servant + Aeson
  * Yesod + Aeson
  * Snap + Aeson
3. Write SPA in Haskell-powered technologies
  * GHCJS + Reflex-FRP
  * Any other?
4. Redis caching for JSON responses from the API

## Phase 2.2: Server-powered HTML UI

1. Domain-API mentioned in the section above will be reused
2. Convert HTML into templates, and wire up the UI as per the spec:
  * Shakespearean templates
  * Lucid
  * Blaze
  * Heist
3. Redis caching for the HTML responses from the server

## Phase 3: Testing

1. Unit tests for domain-level API
  * Quicktest
  * Hspec
  * Anything else?
2. Integration/browser tests using Selenium
  * Which library?

## Phase 4: Deplyoment

TODO - Which libraries are used for deploying Haskell webapps?

1. Combining and minifying static assets
2. Various cache-busting techniques available for ensuring stale assets are not fetched by browsers
3. How to rollback a faulty deployment?

## Phase 5: Supporting the app in production

1. Implement specialized app-level logging
2. Tweak web-server request/response logging to log in a customized format. 
  * Scrub out sensitive data (like passwords or API secrets) in the request/response logs.
3. How does the app log runtimes error in the hopefully rare instances of when they occur? Should one always deploy with profiling turned on? If not, how does one get stacktraces to help debug production errors?
4. What if you have to hotpatch a code-fix? (like how it's down in the Common Lisp world)?
5. Can one connect to a running Haskell intance and inspect its state?
6. Configure a standard tool for remote error logging (eg. Airbrake, Scoutapp, New Relic, etc)
7. Configure a standard tool for performance monitoring (eg. Skylight, Scoutapp, New Relic, etc)

# The Spec

TODO