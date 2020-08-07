# Predisposizione ambiente

Creazione progetto React. L'opzione *--use-npm* serve a forzare l'uso di npm anche nel caso in cui sia installato yarn, poichè quest'ultimo in alcuni casi si blocca e impedisce l'installazione. L'opzione *--ignore-existing* serve a forzare il download dell'ultima versione di create-react-app anzichè usare quella globale eventualmente presente, che tra l'altro dovrebbe essere rimossa in quanto non più supportata.

    npx --ignore-existing create-react-app myapp --use-npm
    cd myapp

Installazione electron. La variabile d'ambiente serve a evitare litigi con le porcherie che l'antivirus aziendale mette in mezzo ai certificati SSL mentre npm cerca di scaricare i binari di electron.

    NODE_TLS_REJECT_UNAUTHORIZED=0 npm i -D electron

Installazione dipendenze dev:

    npm i -D electron-builder wait-on concurrently

Dipendenze NON dev:

    npm i electron-is-dev

Modifica *package.json* per aggiunta sezione *build* (usata da electron-builder), *homepage* (deve essere uguale a ./ per il funzionamento di create-react-app) e *main* per specificare il file principale del progetto (che andremo a creare nel passaggio successivo).

```json
"homepage": "./",

"build": {
    "appId": "myappid",
    "directories": {
        "buildResources": "assets"
    },
    "win": {
    }
},

"main": "public/electron.js",
```

Creazione del file principale per l'avvio dell'applicazione electron (*public/electron.js*).

```javascript
const electron = require('electron')
const app = electron.app
const path = require('path')
const isDev = require('electron-is-dev')
const BrowserWindow = electron.BrowserWindow

let mainWindow

function createWindow() {
  mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true,
    },
  })

  mainWindow.loadURL(
    isDev
      ? 'http://localhost:3000'
      : `file://${path.join(__dirname, '../build/index.html')}`,
  )

  mainWindow.on('closed', () => {
    mainWindow = null
  })
}

app.on('ready', createWindow)

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', () => {
  if (mainWindow === null) {
    createWindow()
  }
})
```

Predisposizione script avvio e build electron:

```json
"ebuild": "npm run build && electron-builder",
"dev": "concurrently \"npm start\" \"wait-on http://localhost:3000 && electron .\""
```

Predisposizione file *.env*:

```
BROWSER=none
```
