# Architect changelog

---
Also see:
- [Architect Functions changelog](https://github.com/architect/functions/blob/master/changelog.md)
---

## [6.0.20] 2019-10-19

### Added

- Integrates new `@architect/create` module for bootstrapping projects and initializing project files
- Added ability to specify project name and install path, e.g. `arc init ./foo` creates a dir named `foo` in your current dir, and creates a new Arc project named `foo` in there


### Changed

- Updated dependencies (which are also now using `@architect/create`)


### Fixed

- Fixes issue when `@tables` definition includes `stream true`; thanks @gr2m!
- Fixes and `arc help <command>` command
- Runtime flag now works for project creation: `runtime`, `--runtime`, or `-r` + `node`, `js`, `python`, `py`, `ruby`, `rb` initializes with Node, Python, or Ruby
- Removed some unnecessary dependencies

---

## [6.0.19] 2019-10-17

### Fixed

- Fixes failed deploys if file type is unknown by common mime-type database; resolves #56, thanks @mikemaccana!
- Fixes paths for deployment of assets on Windows; resolves #58, thanks @mikemaccana!

---

## [6.0.18] 2019-10-15

### Added

- Infrequently prints version update notifications to CLI

---

## [6.0.15 - 6.0.17] 2019-10-15

### Added

- Added update notifier to help ensure folks are running the (hopefully) least buggy, most stable, most secure version of Architect


### Changed

- Updated dependencies


### Fixed

- Fixes deployment issue if `get /` is not specified in `@http`; resolves @package#27, /ht @grahamb and @jeffreyfate!


### Changed

- Improves error states for missing static configs, 404s, etc. when using `@http` and/or `@static` with `arc.http.proxy` or without defining `get /`

---

## [6.0.15] 2019-10-14

### Changed

- Legacy WebSockets paths on the filesystem are now formally deprecated
  - Your default three WebSockets paths should be: `src/ws/default`, `src/ws/connect`, `src/ws/disconnect`
  - If you're using legacy WebSockets paths (either `src/ws/ws-default` or `src/ws/ws-$default`), simply remove `ws-[$]` and you should be all set!


### Fixed

- Fixed issue when emitting to WebSockets with Arc Functions (`arc.ws.send`); resolves #48, thanks @andybee + @bvkimball!
- Fixed issue where `sandbox` may not have correctly resolved some custom WebSocket actions
- Fixed HTTP request with `body` and no `Content-Type` header; resolves #102, thanks @andybee!
- Fixed issue where killed subprocesses would not trigger timeouts; resolves #30, /ht @mikemaccana
- Fixed issue where functions with legacy runtimes may not have been fully hydrated

---

## [6.0.14] 2019-10-11

### Added

- Added support for `@static fingerprint true` in root spa / proxy requests
  - This enables Architect projects to deliver fully fingerprinted static assets while also ensuring that each file is appropriately cached by clients and network infra
  - Also includes support for build-free calls between your fingerprinted static assets
    - Example: in `public/index.html`, use the following syntax to automatically replace the local / human-friendly filename reference to the deployed fingerprinted filename:
    - `${arc.static('image.png')}` will be automatically replaced by `image-a1c3e5.png`
    - Or `${STATIC('image.png')}` (which is the same thing, but shoutier)
    - Note: although those look like JS template literal placeholders, they're intended to live inside non-executed, static files within `public/` (or `@static folder foo`)


### Changed

- Updated dependencies
- Ensures html / json are published to S3 with anti-cache headers


### Fixed

- Restores static asset pruning (`arc deploy --prune` or `arc deploy --static --prune`)
- Fixes issues with parsing certain properties in `arc.json`, thanks @mikemaccana!
- Fixed issue that may prevent `repl` from working in more recent versions
- Add anti-caching headers to `sandbox` 404 response
- Fixes root spa / proxy requests when Architect and/or Sandbox are globally installed; resolves #92 /ht @grahamb

---

## [6.0.13] 2019-10-02

### Fixed

- Fixed issue where with `@static fingerprint true` enabled, the `static.json` file would not be copied into deployed functions' shared dirs; thanks @dawnerd + @jgallen23!
- Removed unnecessary second-order dependency, which should lighten up the Architect install size a bit


### Changed

- Updated dependencies

---

## [6.0.11 - 6.0.12] 2019-09-29

### Added

- `sandbox` Auto-hydration received a bunch of nice upgrades:
  - Auto-hydration now detects changes to the state of your installed Node dependencies, and rehydrates if necessary; for example:
    - You're working on a project, and a teammate updates a dependency in `get /foo` from version `1.0.0` to `1.1.0`
    - Upon your next git pull, `sandbox` will detect the dependency update in `get /foo` and automatically install version `1.1.0` for you
  - Auto-hydration now has a rate limit of one change every 500ms to prevent recursive or aggressive file updates
  - Auto-hydration now has `@static folder` support
  - Auto-hydration now only hydrates the shared files necessary
    - For example: if you change a file in `src/views`, it will only update your `@views` functions, and not attempt to rehydrate all your project's functions with `src/shared`
  - Events now have a timestamp and improved formatting
- Beta: `sandbox` init script support!
  - `sandbox` will now run the init script of your choosing upon startup after all subsystems have started up:
    - `scripts/sandbox-startup.js` - a CommonJS module, receives your parsed Arc project as a parameter, supports async/await
    - `scripts/sandbox-startup.py` - a Python script
    - `scripts/sandbox-startup.rb` - a Ruby script
- Startup auto-hydration now hydrates `src/views` and `src/shared`
- Added support for `@static folder` to static asset `fingerprint`ing


### Changed

- Improvements to the conditions under which the HTTP server starts, shuts down, and restarts; fixes `sandbox` #65
- Improved `sandbox` async error copy (displayed when execution does not complete)
- Proxied requests in `sandbox` now sends a proper `req.resource`, which can resolve some SPA bugs, esp when used with newer Arc Functions
- `sandbox` now respects and errors on invalid response params for proper Architect 6 compatibility; fixes `sandbox` #49
- Updates `sandbox` to Dynalite to `3.0.0`, thanks @mhart!
- Better 404 / file missing handling in `sandbox` when using `http.proxy` (or loading assets without `@http get /` specified)
- `hydrate --update` now properly inventories its update operations, avoiding superfluous work


### Fixed

- Fixes correct inventory paths for `src/ws/*`, which should in turn fix WebSocket function hydration, thanks @mikemaccana!
- When auto-hydrating functions upon startup, `sandbox` no longer hydrates `src/views` and `src/shared` with each function
- Fixed issue where `hydrate`'s shared file copying may have leaked across Lambda executions in rare circumstances
- Fixed undefined message in `hydrate` init
- Improved `hydrate` error bubbling
- Formatting and line breaks in `hydrate` printer return should now more closely (or exactly) match console output
- Fixed issue where in certain circumstances `get /` wouldn't reload `sandbox` after a change to the project manifest
- Minor fix where if you specified a `SESSION_TABLE_NAME` env var outside of `.arc-env`, `sandbox` won't clobber it
- Fixed caching headers for various `sandbox` error states (async, timeout, etc.) to ensure your browser won't accidentally cache an error response
- Fixes issue where binary assets delivered via `sandbox` / root may not be properly encoded
- Fixes empty mock `context` object encoding

---

## [6.0.10] 2019-09-11

### Changed

- Running `hydrate` now properly inventories its update operations, avoiding superfluous work there as well
- Results returned by `hydrate` are now symmetrical with what's printed
- `@http` functions are now provisioned with the `ARC_HTTP` env var, which is set to `aws_proxy`


### Fixed

- Fixed `env` to ensure env vars were populated out of the correct region
- `sandbox.close` will no longer throw an error if project doesn't use `@http` or `@ws`
- Fixed reliability of `hydrate` and other Architect operations printing in CI containers and other non-TTY environments

---

## [6.0.9] 2019-09-09

### Changed

- Running `hydrate` now properly inventories its install operations, avoiding superfluous work


### Fixed

- Fixed issue where `sandbox` would hang if POST requests were sent without a body
- Fixed `logs` to ensure log data is read out of the correct region
- Fixed S3 permissions to enable direct asset uploading

---

## [6.0.8] 2019-09-06

### Fixed

- Fixes case where user-defined region may not be respected
- Fixes provisioning `@scheduled` functions

---

## [6.0.7] 2019-09-05

### Changed

- Add clearer bucket region instructions during init

---

## [6.0.6] 2019-08-30

### Changed

- Internal change; improve error handling states

---

## [6.0.5] 2019-08-28

### Added

- Added initialization confirmation messages


### Changed

- Improved boilerplate project files written during initialization
- Updated boilerplate .arc file initialized
- Patched vendored proxy bundle to 3.3.7
- Updated deps

---

## [6.0.4] 2019-08-27

### Changed

- Updated deps

---

## [6.0.3] 2019-08-24

### Fixed

- Internal error handling change


### Changed

- Updated deps

---

## [6.0.1 - 6.0.2] 2019-08-22

### Fixed

- Fixes issue with auto-hydration of new dependencies during `sandbox` startup
- Fixes issue with `arc deploy --static` throwing an unnecessary error after uploading files, fixes #427
- Fixes minor copy issue in `arc deploy` reporting the incorrect static asset folder when there are no files to deploy

---

## [6.0.0] 2019-08-20

### Changed

- Many things! Updates to this changelog forthcoming shortly. Arc 6.0 is a breaking change, please refer to https://arc.codes and upgrade mindfully!

---

## [5.9.24] 2019-08-05

### Fixed

- Resolves issue where static assets aren't loading from `_static/` in `sandbox`, fixes #416

---

## [5.9.23] 2019-07-23

### Fixed

- Fixed issue preventing Ruby functions from properly executing in `sandbox`
- Fixed issue prevent Python functions from properly executing in Windows `sandbox`
- Fix broken characters in Windows `sandbox` console
- Fixes super obscure bug where certain shared files may not be included in a single function deploy

### Changed

- `sandbox` context now passes an empty object (to be mocked soon!) to all runtimes
  - This deprecates the legacy AWS implementation of `context` (since retired in production) passed to `sandbox` Node functions

---

## [5.9.21-22] 2019-07-17

### Added

- Adds auto-hydration to new functions without restarting `sandbox`


### Fixed

- Fixes issue with auto-hydration on `sandbox` startup
- Fixes potential issue in static asset deploys in Windows

---

## [5.9.20] 2019-07-15

### Added

- Expanded support for static asset fingerprinting! If you've enabled fingerprinting (`@static fingerprint true`):
  - `sandbox` will regenerate your `public/static.json` file on startup
  - And whenever making any changes to your `public/` dir, `sandbox` auto-refresh will automatically regenerate `public/static.json` and re-hydrate your shared files with the latest version


### Fixed

- `sandbox` auto-refresh now detects file deletions from `src/shared` and `src/views`

---

## [5.9.19] 2019-07-12

### Added

- Adds PYTHONPATH to `sandbox` Lambda invocation for `/vendor` modules

### Fixed

- Fixes `sandbox` crashing when `get /` and other functions aren't defined in `.arc` or present in the filesystem, but are requested by a client
- Prevents startup of http server if `@http` isn't defined in `.arc`
- Improves `sandbox` support in Windows

---

## [5.9.18] 2019-07-12

### Fixed

- Fixes WebSocket provisioning issue

---

## [5.9.17] 2019-07-12

### Changed

- Disables delta resource creation during deployments
  - This functionality is better served by more reliable and deterministic resource creation via the forthcoming Architect 6 release

---

## [5.9.16] 2019-07-12

### Changed

- Reverts previous breaking change on WebSockets, default directories that get created are now once again `ws-connect`, `ws-default`, and `ws-disconnect`

---

## [5.9.15] 2019-06-26

### Added

- Auto-refresh! `sandbox` now keeps an eye out for the following changes to your project:
  - Edits to your Architect project manifest will mount or unmount HTTP routes without having to restart `sandbox`
  - Changes to `src/shared` and `src/views` will automatically rehydrate your functions' shared code
  - More to come!


### Changed

- Prettied up `sandbox` initialization printing a bit

---

## [5.9.14] 2019-06-25

### Added

- Auto-hydration!
  - Say goodbye to running `npx hydrate` before starting new projects, cloning existing projects, or pulling down new functions
  - On startup, any functions missing dependencies on the local filesystem will now be auto-hydrated
  - Supported runtimes and dependency manifests: Node + `package.json` (requires npm >= 5.7), Python + `requirements.txt` (calls pip3), and Ruby + Gemfile (calls bundle)

---

## [5.9.10-13] 2019-06-24

### Fixed

- Ensures `sandbox` starts in the cases of:
  - No local AWS credentials file (e.g `~/.aws/credentials`)
  - The local AWS credentials file is present, but is missing the requested profile name
  - Fixes #382, 391
- Fixed issue with `hydrate` caused by an errant merge

---

## [5.9.9] 2019-06-23

### Changed

- Projects that use WebSockets (`@ws`) in their `.arc` file will need to be cautious about upgrading
  - The default directories that get created are now `ws-$connect`, `ws-$default`, and `ws-$disconnect`; it is recommended that you run `npx create` and copy your code from `ws-connect` to `ws-$connect`, `ws-default` to `ws-$default`, and `ws-disconnect` to `ws-$disconnect` and then delete the old directories

### Fixes

- `@ws` directive now works correctly with the `npx inventory` command set
- Fixes `sandbox` wrapper when use as async module, adds `sandbox` test

---

## [5.9.8] - 2019-06-19

### Changed

- Deployments of static assets now follow symlinks in the `public/` directory

---

## [5.9.5-7] - 2019-06-18

### Changed

- The `sandbox` workflow is now its own module!
  - No functionality changes to `sandbox` in this release
  - A few minor improvements to `sandbox` console messages during startup, and drying up port assignment logic
  - You can find the `sandbox` module at https://github.com/architect/sandbox


### Fixed

- Some Arc-supported runtimes defined in `.arc-config` files (such as `nodejs8.10`) will no longer cause `sandbox` to crash

---

## [5.9.4] - 2019-06-15

### Added

- Added ability to disable Architect managing a given function's environment variables
  - Add an `@arc` pragma to your function's `.arc-config` file, and pass it the `env false` flag


---

## [5.9.3] - 2019-05-29

### Fixed

- Corrects URI encoding when accessing local static assets in _static, fixes #390
- Warns users of static deployments of files approaching the payload limit of Lambda, fixes #387
- Fixes static deploy errors of unknown file types


---

## [5.9.0 - 5.9.2] - 2019-05-23

### Added

- CloudFormation support! 🚀
  - `npx package` will export the current `.arc` file to `sam.json` and print further instructions for deploying
  - Currently only `@http`, `@static` and `@tables` pragmas are supported; you can track the other pragmas dev (or submit a PR!) here at #386
  - Any env vars in `.arc-env` are automatically applied to the CloudFormation stack
  - `.arc-config` settings are also fully supported
  - Unfortunately CF currently has a bug with binary media types which we're tracking here https://github.com/awslabs/serverless-application-model/issues/561
- Static asset fingerprinting beta!
  - Add `fingerprinting true` to your `@static` pragma to enable fingerprinting
  - Add `ignore` followed by a two-space indented list to ignore certain files from `public/`
  - [More information here](https://arc.codes/reference/static)
- Added new flag for pruning old static assets: `npx deploy [--static] --prune`
- Added ability to completely disable shared folder copying into functions
  - Add an `@arc` pragma to your function's `.arc-config` file, and pass it the `shared false` flag
- Ruby and Python local runtime support
  - `.arc-config` with `runtime` of either `ruby2.5` or `python3.7` works on localhost (make sure you have python and ruby installed!)


### Fixed

- Hydration (and other things that depend on hydrator operations) should now work more reliably on non-UNIXy machines


---

## [5.8.7] - 2019-05-21

### Fixed

- Fixes lack of `console.[warn|error|trace]` output in sandbox console


## [5.8.5 - 5.8.6] - 2019-05-15

### Changed

- Default Lambda runtime is now Node 10.x (from Node 8.10)
- Added simple rate limits to `npx config` and `npx config --apply`
- Updated dependencies


---

## [5.8.0 - 5.8.4] - 2019-05-08

### Added

- Adds `npx inventory nuke -f` for deleting DynamoDB tables and S3 buckets (even if they have contents)
  - Also adds aliases: `npx inventory -nf`, `npx inventory -fn`
- Scheduled function state enabled / disabled flag in `.arc-config` with `npx config`
- Concurrency `0-1` flag in `.arc-config` with `npx config` (applies to all function types)
- Support for custom routes in WebSocket API Gateways
  - `@ws` now accepts routes defined in `.arc`
  - To use these custom routes, the client message must contain an `action` key that is the name of the route


### Fixed

- DynamoDB tables and indexes now enqueue during create; fixes #268
- Sync queue visibility to function timeout; fixes #204
- Verifies queue resources deleted with `npx inventory nuke`; fixes #132
- Enhanced cron() and rate() validatation; fixes #148
- Now `npx inventory nuke` destroys Lambdas and CloudWatch Events rules
- Enhanced error reporting for bad creds or bad arcfiles (closes #339, #364)


### Changed
- New GitHub name! Find us at: [github.com/architect](https://github.com/architect)
  - If you're already developing for Architect projects, don't forget to update your git remotes, e.g.: `git remote set-url origin https://github.com/architect/architect.git && git remote -v`
  - Special thanks to @pug132 for the name (and a big hat-tip to @mikemaccana)!
- scheduled functions always putRule on `npx create` (fixes #326)


---

## [5.7.0] - 2019-04-17

### Added

- Support for AWS layers! /ht @julianduque
  - You may now specify `layer $layerARN` in the `@aws` section of your project manifest, or `.arc-config` files (see: [config](https://arc.codes/reference/arc-config)!
- Support for additional runtimes: `python3.7` and `ruby2.5`!
  - The list also includes: `nodejs8.10`, `python3.6`, `go1.x`, `dotnetcore2.1`, and `java8`
  - Specify `runtime $yourRuntime` in the `@aws` section of your project manifest, or `.arc-config` files
  - Of course, you can also add the runtime of your choosing by way of layer support
- Support for `application/binary-octet` & `multipart/form-data` requests, fixes #353
  - Requests with those two `Content-Type` headers will produce a `base64` object in the `body` object, like so: `body: {base64: 'R28gR2lhbnRzIQ=='}`
  - Empty request bodies will still produce an empty object (e.g. `body: {}`)


### Fixed

- `config [--apply]` now has an easier to read layout, improved diffing, and instructions on how to apply changes
- Added resource creation error handling and validation to `deploy`
- Invalid runtime settings emit a friendly warning about defaulting to `node8.10`


### Changed

- `get /` no longer required by `@http`
- Updated dependencies (most notably `@architect/parser` to support `runtime` and `layers` settings)
- Resource creation error handling moved into common utils (internal)


---


## [5.6.3] - 2019-04-08

### Fixed

- Fixes WebSocket support in `sandbox`; fixes #328 /ht @rschweizer


### Changed

- Cleans up http invocation for doc content-types in `sandbox`
- Slightly better rate limit error message


---


## [5.6.1] - 2019-04-06

### Added

- Adds `req.httpMethod` and `req.queryStringParameters`


### Removed

- Removes deprecated code paths


---


## [5.6.0 - 5.6.1] - 2019-04-04

### Added

- Enables both text and binary file transit in newly provisioned Arc apps
- Adds the same capability to `sandbox`
- This is NOT a breaking update, however if you'd like your existing app to serve binary assets, you'll need to re-create your API (or hang tight until we release our forthcoming API migration tool)
- Adds `req.httpMethod` and `req.queryStringParameters`
- Removes deprecated code paths

---


## [5.5.10] - 2019-03-15

### Added

- Cache-control header support to `sandbox`
  - Default for local dev environment is not to send any `cache-control` header
  - If specified, `cache-control` passes through


---

## [5.5.9] - 2019-03-13

### Added

- Cache-control header support; if not specified, defaults to:
  - HTML + JSON: `no-cache, no-store, must-revalidate, max-age=0, s-maxage=0`
  - Everything else: `max-age=86400`
  - This change only applies to Architect apps provisioned from this version forward

### Fixed

- Default `content-type` response of `application/json` is now `application/json; charset=utf-8;`

### Changed

- Updated dependencies

---

## [5.5.6 - 5.5.8] - 2019-02-28

### Fixed

- Now properly deletes the entire CloudWatch log group when nuking; fixes #311 /ht @mikeal
- Fixes `sandbox` to work properly when `content-type` has charset assignment; #303, #305 /ht @hada-unlimited
- Fixes issue with `audit` breaking on `@ws` functions; #311 /ht @mikeal
- `sandbox` now accepts `statusCode`, in addition to `status` and `code`; fixes #323


---

## [5.5.5] - 2019-02-22

### Fixed

- Fixes ordering logs by last event /ht @mikeal


---

## [5.5.2 - 5.5.4] - 2019-02-20

### Fixed

- `logs` forces descending order
- Query params no longer trigger index.html override for `sandbox`
- Adds support for `text/tsx` in `/public`


---

## [5.5.1] - 2019-02-08

### Changed

- Updated dependencies

### Removed

- `test:watch` script and `tape-watch` devdep (due to multiple dependencies with CVEs)


---

## [5.5.0] - 2019-02-03

SPA support: mount S3 on the `/` of API Gateway

### Added

- `ANY /{proxy+}` will now route any 'not found' to the HTTP Lambda `src/http/get-index`
  - Note: proxy+ routes have a slightly different request payload that includes `request.requestContext` not available to regular HTTP Lambdas (so you can use this to test a 404 condition)
- `ANY /_static/{proxy+}` will proxy to S3 buckets defined by `@static` (if you want to skip proxying through Lambda)
- Companion library `@architect/functions` also gained `arc.proxy.public` superpower for proxying all requests NOT defined by `.arc` to the `@static` S3 buckets

[For more about single page apps with Architect see there docs here.](https://arc.codes/guides/spa)


### Changed

- If `@http` is defined, then `get /` must also be defined
- `npx sandbox` now mounts `/public` on `http://localhost:3333/_static` to match the deployed API Gateway S3 proxy


---

## [5.0.6] - 2019-01-25

### Added

- Improvements to build plugins system (did you know Architect has a build plugins system?)
  - Now supports NPM scoping (e.g. `@architect/arc-plugin-node-prune`)
  - Build preparation order now runs pre-deploy build plugins last (after dependency hydration)
  - Published a companion beta / demo plugin: [`@architect/arc-plugin-node-prune`](https://www.npmjs.com/package/@architect/arc-plugin-node-prune), clean the cruft out of your `node_modules`!


### Fixed

- Fixes `create` breaking if `@ws` is not present in `.arc`, #276


---

## [5.0.4] - 2019-01-24

### Added

- `repl` now respects environments, allowing you to connect to your remote databases with the `NODE_ENV` environment variable


### Fixed

- `repl` was being clobbered by `@architect/data`'s own implementation; fixed in `architect/arc-data` #12, dependency updated


### Changed

- Updated dependencies


---

## [5.0.2] - 2019-01-22

### Added

- Support for `@ws` in `.arc` for generating WebSocket Lambdas and API Gateway endpoints
- Support for WebSocket Lambdas in the local sandbox
- WebSocket Lambdas support in `inventory`, which in turn powers most other workflows


### Removed

- Support for `@slack` as the more generic `@http` Lambdas support that use case better


### Fixed

- Fixes `@indexes` creation bug
- Runs `npm i` during `hydrate --update`, resolving a long-standing NPM issue where package-lock files may fall out of sync
- Fixed: on Windows, S3 assets will be correctly created relative to their location beneath `public` rather than their full paths.

### Changed

- Updated dependencies


---

## [4.5.6] - 2019-01-14


### Fixed

- Fixes callback in _create-code task, fixes #263


### Changed

- Improved error handling in NPM operations


---

## [4.5.5] - 2019-01-11


### Fixed

- In Windows, NPM no longer fails with `undefined`, fixes #261


---

## [4.5.4] - 2019-01-10


### Changed

- Improved status reporting during `deploy` and `create`


### Fixed

- Fixes cases where hydration was crashing single function deployments, and deployments that execute `create` for missing resources


---

## [4.5.1] - 2019-01-08


### Changed

- `hydrate` refactor and API
  - NPM operations are now queued and concurrently processed (env var `ARC_MAX_NPM`, defaults to 10) /ht @grncdr
  - Vastly improved NPM error handling and related deployment reliability (fixes #141 + #151)
  - Now hydrates (and updates) deterministically from the current .arc manifest, as opposed to globbing
  - All NPM operations now use `npm ci` for more consistent behavior across environments
  - Updates `deploy` to use `hydrate` API, and `sandbox` to use new shared code copier module
- DynamoDB tables now use on-demand/pay-per-request billing mode, mitigating the need for capacity planning /ht @alexdilley
- `inventory` now supplies its own Arc project data, allowing it to be called as needed without relying on `util/init`


### Fixed

- Freshly created Functions are now properly hydrated with shared code (if available and appropriate; fixes #241)
- Running `create` on already existing projects now runs orders of magnitute faster
- `.arc` file is now reliably copied into each Function's `node_modules/@architect/shared` even if you don't use `src/shared` (needed by `@architect` deps)
- `deploy` now respects `--delete` flag when deploying the whole project
- Improved progress reporting in `CI` mode, and for `hydrate`, `create`, and `deploy`


### Added

- New command: `hydrate --shared [--update]` - hydrates and/or updates `src/shared` and `src/views` (if available)
- Added test run watcher script /ht @filmaj


---

## [4.4.12] - 2018-12-23


### Removed

The following folders are no longer required nor autogenerated:

- `src/shared`
- `src/views`
- `/public`

Functionality remains unchanged: Contents of `src/shared` are synced to `node_modules/@architect/shared` in all lambdas whenever deploying or using the sandbox. Contents of `src/views` are synced to all HTTP GET lambdas `node_modules/@architect/views`. Public is synced to S3 buckets.


---

## [4.4.11] - 2018-12-19


### Added

- Updates `sandbox`, adding minor performance tweaks now, and setting up for future enhancements
  - Updated `sandbox` to asynchronously read files when invoking a Lambdas
  - Updates runtime handling in `sandbox` to make it easier to add additional runtimes
  - Each runtime now lives in its own function, which also enables process forking later on


---

## [4.4.10] - 2018-12-18


### Added

- New logo (added to readme)! /ht @amberdawn
- `hydrate` and `deploy` now install dependencies in `src/shared` and `src/views` (#240)
- `QUIET` boolean env var suppresses init header (fixes #238)
  - Helpful for piping data to disk, e.g. setting up new users on an Architect project with `QUIET=1 npx env > .arc-env`
- `app.arc` app filename supported as a non-dotfile alternative to `.arc` (#239)


### Changed

- Static asset deploys now exclude default `public/readme.md` file
- Improved `hydrate` progress and completion confirmation
- Adds Architect version back into the init header


### Fixed

- `hydrate` was not properly globbing (and thus, not hydrating) `src/shared` contents


---

## [4.4.9] - 2018-12-16 (merge commit, no changes)


---

## [4.4.8] - 2018-12-12


### Added

- To help accommodate `sandbox` calling out to remote databases, `SANDBOX_TIMEOUT` env var allows you specify in seconds how long `sandbox` should wait for all child processes to complete
  - `SANDBOX_TIMEOUT` is overridden by any directory-specific `.arc-config` files


### Changed

- Default `sandbox` timeout is now symmetrical with Architect's default Lambda timeout time of 5 seconds


---

## [4.4.7] - 2018-12-11


### Added

- Form-encoded POST values can now be sent enclosed in single quotes /ht @herschel666


### Changed

- Skip logging when `@static` isn't deployed
- Additional hardening of `sandbox` handling of JSON responses emitted from `@architect/functions`
- Preliminary / prototype commits for outputting CloudFormation from .arc
- Preliminary / prototype commits in the direction of adding arbitrary header support /ht @mweagle


### Fixed

- Issue where `arc.http` JSON responses were crashing `sandbox` due to an encoding mismatch
- Issue where `ARC_LOCAL` env var wasn't being properly respected by `sandbox` events


---

## [4.4.5] - 2018-12-7


### Added

- Local `@queues` now available in `sandbox`
- Big ups to @grncdr for this feature!


### Fixed

- `hydrate` wasn't updated to use the new progress indicator, and would fail when used – no longer!


---

## [4.4.4] - 2018-12-1


### Added

- `deploy` operations now read local and remote last-modified times, and will skip files whose times don't differ, thereby speeding up deploy operations of static files
- Big ups to @filmaj for this release, too!


---

## [4.4.3] - 2018-11-30


### Added

- `npx deploy static`: deploys only `@static` assets (as found in the `public/` folder)
  - Accepts `staging | --staging | -s` and `production | --production | -p` flags
- `npx deploy static --delete`: deploys static assets, and deletes remote S3 files not present locally in `public/`
  - Can be used with `staging` and `production` flags
- Big ups to @filmaj for this release!


---

## [4.4.0] - 2018-11-27


### Added

- Large refactor of `deploy` workflow to improve stability and reliability
- `deploy` now identifies missing project resources during deploy operations
  - Instead of failing / throwing errors, `deploy` now completes its first pass deployment
  - Then, once completed, `deploy` creates any resources missing from the deployment (and deploys them)


### Changed

- `npx inventory --nuke` now destroys `@static` resources (S3 buckets) and `@events` resources (SNS Topics).
- New projects will no longer create `arc-sessions` tables (for use with `@architect/functions`) by default, and are now explicitly opt-in
- Replaces `progress` module with a lighter weight, more readily cross-platform homegrown solution


---

## [4.3.14] - 2018-11-26


### Changed

- `logs` command results now sorted chronologically


### Fixed

- Fixes console leaking of large responses in `sandbox`


---

## [4.3.13] - 2018-11-24


### Fixed

- `sandbox` was broken in the JWE changeover
- Fixes some broken tests


### Removed

- Unnecessary session table test stubs


---

## [4.3.12] - 2018-11-23


### Changed

- New default for Architect sessions is based on JWE
- DynamoDB sessions are still available, but [now opt-in](https://arc.codes/guides/sessions)
- `sandbox` now matches the 6MB request payload limit of Lambda /ht @herschel666


---

## [4.3.10] - 2018-11-19


### Added

- dotfiles are now included in Lambda deployments


---

## [4.3.9] - 2018-11-15


### Changed

- Architect parser now accepts multiple spaces between http verb and path in `@http` functions


### Fixed

- `sandbox` now properly pretty prints paths


---

## [4.3.8] - 2018-11-14


### Added

- Additional S3 deploy tests


---

## [4.3.7] - 2018-11-13


### Changed

- `sandbox` clears async function timeout if execution is faster than specified timeout


---

## [4.3.6] - 2018-11-12


### Added

- Better async error handling (and more helpful error text) in sandbox
- Trap and present friendly error when async functions don't return a value in sandbox
- `sandbox` returns a promise if no callback is specified
- Adds repl


---

## [4.3.1] - 2018-11-12


### Fixed

- Error with `/public` folder in sandbox


---

## [4.3.0] - 2018-11-12


### Added

- `arc.sandbox.start` now accepts a regular node errback as the last arg


---

## [4.2.2] - 2018-11-12


### Added

- This changelog!


### Fixed

- Removed generated test coverage files from NPM package bundle
- Inline help typos


---

## [4.2.1] - 2018-11-09


### Added

- `CI` env boolean for disabling `deploy` progress indicator in CI


---

## [4.2.0] - 2018-08-12


## Added

- CORS support for `sandbox`


## Fixed

- CORS bug in HTTP functions
- Issue with `sandbox` not properly passing callbacks if used as a module


---

## [4.1.3] - 2018-11-6


### Added

- Massive tests refactor, shout out to @filmaj!
  - Added tests for many important workflows
  - Code coverage now tracked with Istanbul
- Architect's CI is now public via TravisCI


---

## [4.1.2] - 2018-10-31


### Changed

- Roughed in resource flagging (not yet ready, though)


### Fixed

- Restored accidentally removed `readme.md`


---

## [4.1.1] - 2018-10-31


### Added

- `logs` workflow!
- Command line flags for various workflows:
  - `audit`: `apply`, `--apply`, `-a`
  - `config`: `apply`, `--apply`, `-a`
  - `create`: `local`, `--local`, `-l`
  - `deploy`: `production`, `--production`, `staging`, `--staging`, `public`, `--public`, `/public`, `lambda`, `--lambda`, `lambdas`, `--lambdas`, `functions`, `--functions`
  - `dns`: `nuke`, `--nuke`, `-n`, `route53`, `--route53`, `-r`
  - `env`: `verify`, `--verify`, `-v`, `remove`, `--remove`, `-r`
  - `hydrate`: `update`, `--update`, `-u`
  - `inventory`: `nuke`, `--nuke`, `-n`, `--nuke=tables`, `verify`, `--verify`, `-v`
  - `sandbox`: `production`, `--production`, `-p`, `staging`, `--staging`, `-s`, `testing`, `--testing`, `-t`
- Additional path validation for leading and trailing special characters


### Changed

- Refactored `deploy` command to support additional command line flags, and surgical deploys of just `/public`, or just Lambdas
- Copy fixes for deployment workflow
- Made boilerplate HTTP functions a bit more minimal
- Improved `sandbox` testing


---

## [4.1] - 2018-10-24


### Added

- Automagical `src/views` folder: copies contents into all `HTTP GET` functions' `node_modules/@architect/views`
- `@views` pragma, overrides bulk `src/views` copy, and only copies into specified functions
- More information on `src/views` and `@views` [can be found here](https://blog.begin.com/serverless-front-end-patterns-with-architect-views-cf4748aa1ec7)
- Adds ability to use the following special characters in static URL parts: `-` (dash), `.` (period), `_` underscore
- New HTTP function validation logic:
  - HTTP functions must begin and end with a letter or number
  - Cannot create URL params that contain special chars (except leading `:`, of course)
- Adds command-line flags:
  - `npx create`:
    - `local`, `--local`, `-l`
  - `npx deploy`:
    - `production`, `--production`, `-p`
    - `staging`, `--staging`, `-s`
- New [Examples repo](https://github.com/architect/examples)


### Changed
- Some light boilerplate code cleanup
- [#168](https://github.com/architect/architect/issues/168) Fixed issue where Architect parser was missing `@http` support in JSON + YAML manifests
- [#164](https://github.com/architect/architect/issues/164) Fixed issue in Windows where Architect would try to copy files over itself


---

## [4.0] - 2018-10-20


### Added

- `@http` pragma, now the default way to create Lambda functions for the web
  - Supports fully dynamic Content-Type, Status Code
  - Supports all HTTP methods
  - CORS support with a boolean flag
- `public` folder, now the default way to sync static assets to S3
- `JSON` & `YAML` support for the `.arc` manifest
- Per-Lambda function configuration support with `.arc-config` files
- Per-Lambda function IAM roles support with `role.json` files


### Changed

- Simpler package name (`npm i @architect/architect`)
- New GitHub name ([https://github.com/architect/architect](https://github.com/architect/architect))
- Smarter rate-limiting for deployments of large (50+ function) projects
- Complete docs revamp with new sample projects at [arc.codes](https://arc.codes)
- Fix for obscure bug where `server.close`causes a TypeError
- Readme file cleanup


### Removed

- `.static` folder has been deprecated in favor of the new `public` folder
- Statically bound Content-Type web functions (i.e. `@html`, `@css`) are deprecated
  - `sandbox` will no longer bootstrap these kinds of functions
  - `create` will no longer make these kinds of functions
  - However, `deploy` still supports deploying these legacy functions


---

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
