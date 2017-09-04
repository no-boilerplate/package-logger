# package-logger
Emit logging events from packages.

External packages don't know if `logger` that may exists on the global level is reasonably defined. To solve this `PackageLogger` defines a common subset of logging levels which when invoked emit `log` events. Package's user are then free to listen to these events and pass them to their logger.

Use:

```js
const PackageLogger = require('package-logger');
const logger = PackageLogger.create('my-package'); // We never assign this `logger` to `global` as that would overwrite application's `global.logger` if such exists.

// ...

logger.info('Succeeded marvelously.'); // This will emit `log` event. For users to listen to it logger needs to be exported from the package (see next example)

// Export the logger so that package's users can use it.
modules.exports.logger = logger;
```

Forwarding log events in the application:

```js
const myPackage = require('my-package');
// This now forwards log events from my-package to Winston.
myPackage.logger.on('log', (level, packageName, ...args) => {
    // In this example we put the package name into meta, if such exists.
    const argsWithoutMeta = [...args];
    const meta = argsWithoutMeta.pop();
    if (typeof(meta) === 'object') { // Nah, not really, use lodash!
        if (!meta._packageName) {
            meta._packageName = packageName;
        }
    }
		// This is application's global logger.
    logger.log(level, ...messages, meta);
})
```
