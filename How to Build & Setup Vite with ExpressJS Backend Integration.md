# How to Build & Setup Vite with ExpressJS Backend Integration

I have used vite previously with client side Javascript, and I really liked how easy it is to get up and running with some amazing features that usually took a lot more time to setup.

So when I got an assingment to convert a Vanilla JS Client Side Web application into a Server Side Rendered Application I wanted to utilize the features vite provides in a SSR environment.

## What technologies did I use?

I’ve used the following technologies to create an SSR application:

- [Express](https://expressjs.com) - a minimalistic and simple node.js framework
- [Vite](https://github.com/vitejs/vite) - build tool that leverages the availability of ES Modules in the browser and compile-to-native bundler
- [vite-express](https://github.com/szymmis/vite-express) - [@vitejs](https://github.com/vitejs/vite) integration module for [@expressjs](https://github.com/expressjs/express)
- [Nunjucks](https://mozilla.github.io/nunjucks/) - A rich and powerful templating language for JavaScript.

*(Every other templating engine would also work)*

- [Nodemon](https://nodemon.io) - Simple monitor script for use during development of a Node.js app.

## Setting up the Dev Server

### Prerequisites

To fully understand this article you need the following:

- Node installed on your system.
- yarn or npm package manager installed.
- Working knowledge of JavaScript.

The first time I set this up, I already had a client side app. For educational purposes I’ll try to leave unnesasary documents out of the examples.

For my project `pnpm`  is used as package manager, but this can also be done with

`npm` or `yarn`

```other
$ pnpm create vite
// my setup
✔ Project name: … vite-project
✔ Select a framework: › Vanilla
✔ Select a variant: › JavaScript
```

**Default Folder Structure (Client Side)**

```javascript
./
├── counter.js
├── index.html
├── javascript.svg
├── main.js
├── package.json
├── public
│   └── Logo.svg
└── style.css
```

**My Folder Structure (Server Side)**

```javascript
./
├── public
│   ├── Logo.png
│   └── favicon.ico
├── src
│   ├── assets
│   │   ├── base.css
│   │   ├── main.js // will handle vite asset
│   │   └── main.css
│   └── server
│       ├── views
│       │   └── base.njk
│       │   └── index.njk
│       └── server.js
├── .env
├── .gitignore
├── package.json
├── README.md
└── vite.config.js
```

Like most frontent frameworks I like to use the `./src/` folder for the applications logic, however its recommended to keep the client side and server side logic seperated.

### Configuring Vite for Express Intergration

Most ExpressJS middleware is written in commonjs, To use the Javascript Module Syntax, the following package needs to be installed so vite can convert the syntax.

```other
pnpm add -D @rollup/plugin-commonjs
```

This plugin transforms commonJS to be usable in Javascript Module Syntax

```other
// vite.config.js
import { defineConfig, loadEnv } from 'vite';
import commonjs from '@rollup/plugin-commonjs';

export default defineConfig({
  appType: 'custom',
  base: "./",
  plugins: [commonjs(),],
  publicDir: './public',
  css: {
    devSourcemap: true
  },
  server: {
    port: 3000,
    origin: 'http://127.0.0.1:3000',
  },
  ssr: {
    target: 'node'
  },
	build: {
    outDir: 'build',
    assetsDir: 'assets',
    sourcemap: true,
    minify: true,
		// generate manifest.json in outDir
    manifest: true,
    ssrManifest: true,
    rollupOptions: {
				// overwrite default .html entry
      input: './src/assets/main.js',
    }
  },
  hmr: {
    clientPort: 5173
  },
},({ command, mode }) => {
  // Load env file based on `mode` in the current working directory.
  // Set the third parameter to '' to load all env regardless of the `VITE_` prefix.
  const env = loadEnv(mode, process.cwd(), '')
  return {
    // vite config
    define: {
      __APP_ENV__: env.APP_ENV,
    },
  }
})
```

For development, inject following in the servers HTML template.

based on [**My Folder Structure (Server Side)**](https://www.notion.so/My-Folder-Structure-Server-Side-e2121b0551644249ac19f22c47b0e721?pvs=21) I applied this in `./src/server/views/base.njk`

To effectively handle assets, we have two choices:

1. Configure the server to forward requests for static assets to the Vite server.
2. Set the `server.origin` so that URLs for generated assets are resolved using the URL of the backend server instead of a relative path.

This is necessary to ensure that assets like images load correctly.

### Configuring Express & Vite

Because of the Vite Docs recommendation to checkout the `Awesome Vite` for existing integrations I found out that `Vite-Express` had all the features I was looking for.

```javascript
// src/server/server.js
import express from 'express';
import ViteExpress from 'vite-express';
import path from 'path';
import { fileURLToPath } from 'node:url';
import nunjucks from 'nunjucks';
import expressNunjucks from 'express-nunjucks';

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

const app = express();

const PORT = process.env.PORT || 3000;

// Serve Static Files
	app.use("/assets", express.static(path.join(__dirname, "../assets")));
	app.use("/", express.static(path.join(__dirname, "../../public")));

app.set('view engine', 'njk');
app.set('views', path.join(__dirname, 'views'));

// Set Templating Engine
const njk = expressNunjucks(app, {
  templateDirs: path.join(__dirname, 'views'),
  loader: nunjucks.FileSystemLoader,
});

// render index.njk at /
app.get("/", (req,res,next) => {
	try {
		return res.render('index.njk', {
			title: 'home',
		});
	} catch (err) {
		console.log(err)
		next(err)
	}
});

// The ViteExpress Server

ViteExpress.listen(app, PORT, () => {
  console.log(__dirname)
  console.log(`Server is listening on port ${PORT}...`)
});
```

## Running the Dev Server

**Client Side**

```json
{
  "scripts": {
    "dev": "vite", // start dev server, aliases: `vite dev`, `vite serve`
    "preview": "vite preview" // locally preview production build
  }
}
```

**Server Side**

```json
{
	"scripts": {
	    "dev": "NODE_ENV=development nodemon src/server/server.js -w src/server"
	  }
}
```

## Building Client & Server for Production

With Vite its quite easy to build javascript files and assets. But if u want the client side build and server side build to be in the same outDir, make sure to add `--emptyOutDir` on the second

build run scrip, so it wont get overwritten by the second build.

### Adding Build & Start Script

```json
{
	"scripts": {
	    "start": "NODE_ENV=production node server.js",
	    "dev": "NODE_ENV=development nodemon src/server/server.js -w src/server",
	    "build": "pnpm build:server && pnpm build:client",
	    "build:server": "vite build --ssr src/server/server.js",
	    "build:client": "vite build --emptyOutDir false --ssrManifest",
	  }
}
```

### Conditional Logic

We need to add some conditional logic for the build to run succesfully when its deployed. When the build runs the compiled files won’t be deeply nested, so thats why I’ve added the following conditional

```javascript
if (process.env.NODE_ENV === 'development') {
	app.use("/assets", express.static(path.join(__dirname, "../assets")));
	app.use("/", express.static(path.join(__dirname, "../../public")));
}
if (process.env.NODE_ENV === 'production') {
	app.use('/', express.static('public'));
	app.use('/assets', express.static('assets'));  
	app.get("/sw.js", (req, res) => {
		res.sendFile(path.resolve(__dirname, "public/", "sw.js"));
	});
}
```

Also the ViteExpress server is only needed in development, because the build will produce static assets. The following connditionals handle this.

```javascript
if (process.env.NODE_ENV === 'development') {
	ViteExpress.listen(app, PORT, () => {
	  console.log(__dirname)
	  console.log(`Server is listening on port ${PORT}...`)
	});
}
if (process.env.NODE_ENV === 'production') {
	app.listen(PORT, () => {
		console.log(__dirname);
		console.log(`Server is listening on port ${PORT}...`);
	});
}
```

### ServerJS with conditionals

```javascript
// src/server/server.js
import express from 'express';
import ViteExpress from 'vite-express';
import path from 'path';
import { fileURLToPath } from 'node:url';
import nunjucks from 'nunjucks';
import expressNunjucks from 'express-nunjucks';

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

const app = express();

const PORT = process.env.PORT || 3000;

// Serve Static Files Based on NODE_ENV

if (process.env.NODE_ENV === 'development') {
	app.use("/assets", express.static(path.join(__dirname, "../assets")));
	app.use("/", express.static(path.join(__dirname, "../../public")));
}
if (process.env.NODE_ENV === 'production') {
	app.use('/', express.static('public'));
	app.use('/assets', express.static('assets'));  
}
app.set('view engine', 'njk');
app.set('views', path.join(__dirname, 'views'));

// Set Templating Engine
const njk = expressNunjucks(app, {
  templateDirs: path.join(__dirname, 'views'),
  loader: nunjucks.FileSystemLoader,
});

// Serve index.njk at /
app.get("/", (req,res,next) => {
	try {
		return res.render('index.njk', {
			title: 'home',
		});
	} catch (err) {
		console.log(err)
		next(err)
	}
};
// The Server to Start Based on NODE_ENV
	// Only in development ViteExpress is used.
if (process.env.NODE_ENV === 'development') {
	ViteExpress.listen(app, PORT, () => {
	  console.log(__dirname)
	  console.log(`Server is listening on port ${PORT}...`)
	});
}
if (process.env.NODE_ENV === 'production') {
	app.listen(PORT, () => {
		console.log(__dirname);
		console.log(`Server is listening on port ${PORT}...`);
	});
}
```

## References

- [Vite - Guide - Backend Integration](https://vitejs.dev/guide/backend-integration.html)
- [Vite - Guide - SSR](https://vitejs.dev/guide/ssr.html)
- [Vite-Express](https://github.com/szymmis/vite-express)

### Inspiration

- [https://dev.to/ghoulkingr/overview-of-expressjs-g1b](https://dev.to/ghoulkingr/overview-of-expressjs-g1b)

