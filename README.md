# electron-asar-hot-updater-v1
[![NPM](https://nodei.co/npm/electron-asar-hot-updater-v1.png)](https://nodei.co/npm/electron-asar-hot-updater-v1/)
## What it is
> A NodeJs module for Electron, that handles app.asar updates. Reconstruction of `electron-asar-updater`

## How it works (Read this first)
* EAU (Electron Asar Updater) was built upon _Electron Application Updater_ to handle the process of updating the app.asar file inside an Electron app ; **it simply replaces the app.asar file (at /resources/) with the new one called "update.asar"!**
* The check for "updates" must by triggered by the application. **EAU doesn't make any kind of periodic checks on its own**.
* EAU talks to an API (let's call it so) to tell it if there is a new update.
    * The API receives a request from EAU with the client's **current version of the application (must be specified inside the application package.json file)**.
    * The API then responds with the new update, ... or simply *false* to abort.
    * If there's an update available the API should respond with the *source* for this update **update.asar** file.
    * EAU then downloads the .asar file, deletes the old app.asar and renames the update.asar to app.asar.

## But why ? (use cases)
* If you think these are too complicated to implement:
https://www.npmjs.com/package/electron-updater
http://electron.atom.io/docs/v0.33.0/api/auto-updater/
* If you don't think it's reasonable to update the hole .app or .exe file (up to 100MB) when you're only changing one file (usually 40MB).
* If you want to see `progress` when updating.
* If you want to `check` the version on the `server side` or on the `client side`.
* If you want to use `zip` to compress files, make your ASAR file smaller.

## Add New Function and Fix Issues

- Fix App Paths
- Error Fix: Cannot find module `'F:\Path\package.json'`
- Fix log file path
- Add New Function `Compare Versions`
- Update Method: Check Update available

---

## Installation

```bash
$ npm install --save electron-asar-hot-updater-v1
$ npm install --save electron-asar-hot-updater-v1@version
```

Now, inside the *main.js* file, call it like this:

```js
const { app, dialog } = require('electron');
const EAU = require('electron-asar-hot-updater-v1');

app.on('ready', function () {
  // Initiate the module
  EAU.init({
    'api': 'request.json', // The API EAU will talk to
    'server': false // Where to check. true: server side, false: client side, default: true.
  });

  EAU.check(function (error, last, body) {
    if (error) {
      if (error === 'no_update_available') { return false; }
      console.error(error)
      return false
    } else {
      if (body !== undefined) {
        const options = {
          type: 'info',
          title: 'A new version available',
          message: "v" + body.version,
          detail: "A new version available. Click the button below to download the latest version.",
          buttons: ["Update New Version"],
          cancelId: 1
        }
        dialog.showMessageBox(options, function (index) {
          if (index === 0) {
            EAU.progress(function (state) {
                // The state is an object that looks like this:
                /** First state: Response */
                // {
                //   "time":{
                //     "elapsed":0.077,
                //     "remaining":null
                //   },
                //   "speed":null,
                //   "percent":0.000275581753514208,
                //   "size":{
                //     "total":36990838,
                //     "transferred":10194
                //   }
                // }

                /** Last state: Response */

                // {
                //   "time":{
                //   "elapsed":63.327,
                //   "remaining":0.249
                //   },
                //   "speed":581835.2045730889,
                //   "percent":0.996081191780516,
                //   "size":{
                //     "total":36990838,
                //     "transferred":36845878
                //   }
                // }
            })

            EAU.download(function (error) {
                if (error) {
                    dialog.showErrorBox('info', error)
                    return false
                }
                dialog.showErrorBox('info', 'App updated successfully! Restart it please.')
            })
          }
        }
      }
    }
  })
})
```

API Request Response

```
{
    "name": "app",
    "version": "0.0.1",
    "asar": "http://127.0.0.1:8083/update.asar",
    "info": "1.fix bug\n2.feat..."
}
```

If you use a zip file, the plug-in will unzip the file after downloading it, which will make your update file smaller, but you must make sure that `update.asar` is at the root of the zip package:
```
── update.zip
   └── update.asar
```

## License

[yansenlei](https://github.com/yansenlei/electron-asar-hot-updater)