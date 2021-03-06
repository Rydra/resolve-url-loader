# test-my-cli

A test helper for e2e testing of cli applications in Tape

> EXPERIMENTAL: api is in flux, stability is debatable, support is
> discretionary

This packages use used in conjunction with `Tape` or similar test
framework.

**Why to use**

To test a shell program, such as running **Webpack CLI** on an actual
file-system.

**Why _not_ to use**

If you are developing a loader or plugin for Webpack this might be a bit
excessive.

This is immature. The API is in flux, stability is debatable, support is
discretionary.

**When to use**

If your Webpack loader or plugin requires file system access then you
_need_ to have full `fs`. Any you need to be testing on mulitple OS,
including Windows.

**Disclaimer**

The author uses this package to test a Webpack loader in MacOS, linux,
Windows. The author is biased toward functional programming.

This package has not (yet) trashed the author's filesystem. But there are
is use of `mkdir -p` and `rm -rf`. Make no assumptions and proceed with
caution, at your own risk.

## Usage

### Install

```
npm install --dev test-my-cli
```

or

```
yarn add --dev test-my-cli
```

**Tape**

The author favours functional programming style.

For this we use `blue-tape` and `promise-compose`.

```
npm install --dev blue-tape promise-compose outdent
```

### Init

You will need `init()` at the start of your test. This creates an object
that is passed through a promise chain.

We recommend using `promise-compose` to compose a sequence where
`init()` is the first element.

```javascript
const {platform} = require('process');
const sequence = require('promise-compose');
const {init} = require('test-my-cli');

const appendPath = (...paths) =>
  paths.join((platform === 'win32') ? ';' : ':');

test('myTest', sequence(
  init({
    directory: ['/my/absolute/path", 'tmp', `myTest-${Date.now()}`],
    ttl: '1s',
    debug: false,
    env: {merge: {PATH: appendPath}}
  }),
  ...
));
```

* **directory**

  You need to specify 3 path segments in an Array `[base, temp, test]`.

  > **Important**
  >
  > You need to _think_ about the these paths. There is an _intentional_ syntax
  > error in the example above to avoid copy paste.
  >
  > Obviously you should avoid syntax errors in your configuration.
  
  **base** The base directory must already exist. It is where we will create
  and remove the temp directioy.
  
  **temp** A temporary directory that is usually shared by all tests. It will
  be removed when we detect all tests have finished (see `ttl`).
  
  **test** The name of this test. Something with `basename(__dirname)` or
  `basename(__filename)` may work for you.

* **ttl**

  This determins how long the `temp` directory will remain after the test is
  idle. Specify any value recognised by [ms](https://www.npmjs.com/package/ms).

  Commencement of operation will reset a watchdog timer. The exeptions is
  `exec()` during which the watchdog is reset every `50ms`.
  
  When the watchdog expires this test is considered complete and the `test`
  directory is removed.
  
  Multiple tests may share the `temp` directory, but when all such tests are
  considered complete it will be removed.
  
  > **pro tip**
  > 
  > For debugging, set `ttl: false`. This ensures the temp directories will
  > remain for this test and you can inspect the file system yourself.

* **debug**

  Setting this `true` will put all operations into a mode where they
  `console.log` verbose information. This can be useful it you are unsure
  what operations are being performed but can be _very_ verbose.
  
  Alternatively you can set an operation to debug individually, for example
  `fs: {debug: true}`.
  
  To customise logging set `debug` to your own function.

* **(...each function)**

  Each of the remaining functions can be configured in `init()`.

  For example, it is recommended that `env()` is configured to append
  `PATH` to previously defined values using `appendPath`.

  Refer to the implementation of each operation for further detail.

### Test

**TODO** - For now refer to the dependent projects to see how people are using this.