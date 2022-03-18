# Function.prototype.once for JavaScript
ECMAScript Stage-0 Proposal. J. S. Choi, 2022.

## Rationale
It is often useful to ensure that callbacks execute only once, no matter how
many times are those callbacks called. To do this, developers frequently use
“once” functions, which wrap around those callbacks and ensure they are called
at most once. This proposal would standardize such a once function in the
language core.

## Description
The Function.prototype.once method would create a new function that calls the
original function at most once, no matter how much the new function is called.
Arguments given in this call are passed to the original function. Any
subsequent calls to the created function would return the result of its first
call.

```js
function f (x) { console.log(x); return x * 2; }

const fOnce = f.once();
fOnce(3); // Prints 3 and returns 6.
fOnce(3); // Does not print anything. Returns 6.
fOnce(2); // Does not print anything. Returns 6.
```

(If subsequent calls somehow occur before the first call finishes executing,
then the subsequent calls are synchronously blocked until the first call
finishes.)

```js
function g (x) { /* Stuff that takes a long time */ }
const gOnce = g.once();
const promise0 = (async () => gOnce())();
const promise1 = (async () => gOnce())();
// The gOnce call in promise1 will return
// only after the gOnce call in promise0 returns.
```

## Real-world examples
The following code was adapted to use this proposal.

From [execa@6.1.0][]:
```js
export function execa (file, args, options) {
  /* … */
  const handlePromise = async () => { /* … */ };
  const handlePromiseOnce = handlePromise.once();
  /* … */
  return mergePromise(spawned, handlePromiseOnce);
});
```

From [glob@7.2.1][]:
```js
function Glob (pattern, options, cb) {
  /* … */
  if (typeof cb === 'function') {
    cb = cb.once();
    this.on('error', cb);
    this.on('end', function (matches) {
      cb(null, matches);
    })
  } /* … */
});
```

From [Meteor@2.6.1][]:
```js
// “Are we running Meteor from a git checkout?”
export const inCheckout = (function () {
  try { /* … */ } catch (e) { console.log(e); }
  return false;
}).once();
```

From [cypress@9.5.2][]:
```js
cy.on('command:retry', (() => { /* … */ }).once());
```

```js
From jitsi-meet 1.0.5913:
this._hangup = (() => {
  sendAnalytics(createToolbarEvent('hangup'));
  /* … */
}).once();
```

From [jitsi-meet 1.0.5913][]:
```js
this._hangup = (() => {
  sendAnalytics(createToolbarEvent('hangup'));
  /* … */
}).once();
```

[execa@6.1.0]: https://github.com/sindresorhus/execa/blob/v6.1.0/index.js
[glob@7.2.1]: https://github.com/isaacs/node-glob/blob/v7.2.1/glob.js
[Meteor@2.6.1]: https://github.com/meteor/meteor/blob/release/METEOR%402.6.1/tools/fs/files.ts
[cypress@9.5.2]: https://github.com/cypress-io/cypress/blob/v9.5.2/packages/driver/cypress/integration/commands/waiting_spec.js
[jitsi-meet 1.0.5913]: https://github.com/jitsi/jitsi-meet/blob/stable/jitsi-meet_7001/react/features/toolbox/components/HangupButton.js