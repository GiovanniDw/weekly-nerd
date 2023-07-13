# Making a Web App, Progressive

## What is A PWA (Progressive Web App)?

> Progressive Web Apps (often shortened to *PWA*) are web apps that use modern web technologies to create application experiences that don’t show many of the classic limitations of the web. In other words, they *progress beyond* normal web apps, and mirror what would normally be expected of natively developed applications.

A typical Progressive Web App (PWA) should include the following essential elements:

1. **App Icon**: A PWA should have its own icon, allowing users to add it to the home screen, app launcher, launchpad, or start menu for easy access.
2. **App Discovery**: PWAs should appear in search results when users search for apps on their devices, ensuring their visibility alongside native apps.
3. **Standalone Window**: When launched, a PWA should open in a separate window that is independent of the browser's user interface. This separation provides users with a dedicated and immersive experience.
4. **Enhanced Integration**: PWAs have the ability to seamlessly integrate with the operating system. allowing them to do things like managing URLs and customizing the title bar. This integration ensures that the PWA fits well within the overall experience of the device.
5. **Offline Functionality**: An important aspect of PWAs is their ability to work offline. Users should be able to access and interact with the app's content and features even without an internet connection.

## What Makes a PWA?

Technically, a web app requires just a few additional components to make it progressive, a ***web app manifest*** and a ***service worker***.

## Web App Manifest

For every PWA, it is recommended to have a single manifest per application, typically stored in the root folder and linked on all HTML pages where the PWA can be installed. The manifest file should have the `.webmanifest` extension and follow the `JSON` syntax, allowing you to name it as something like `app.webmanifest`.

Example `app.webmanifest` file:

```json
{
    "name": "Progressive Web App Name",
    "short_name": "PWA Name",
    "icons": [
        {
            "src": "/android-chrome-192x192.png",
            "sizes": "192x192",
            "type": "image/png"
        },
        {
            "src": "/android-chrome-512x512.png",
            "sizes": "512x512",
            "type": "image/png"
        }
    ],
    "theme_color": "#343434",
    "background_color": "#343434",
    "start_url": "https://progressive.web.app/",
    "display": "standalone"
}
```

Linking to the Manifest from the `HTML`:

```html
<link rel="manifest" href="/app.webmanifest">
```

### Generating Assets

> In order to support most browsers and platforms, you need *more than a dozen pictures*. That's right. And the HTML code is not straightforward. In fact, most code snipets you can find on the Web are *wrong*.

— [https://realfavicongenerator.net/](https://realfavicongenerator.net/)

This tool generates multiple sizes of a logo and the meta data thats needed.

For Example

- iOS App icon
- Android App Icon
- Windows Phone App Icon
- Browser App icon
   - Safari
   - Chrome
   - Edge
   - Firefox
- Favicon

#### How to generate these assets

1. Upload an image file (`.png` or `.svg`)
2. Edit the settings so the logo looks good everywhere.
3. Click `Generate your Favicons and HTML code`
4. Download the generated assets `.zip`
5. Extract the .zip in the root `./` or the `./public` folder

   Example files:

```html
./
├── android-chrome-192x192.png
├── android-chrome-512x512.png
├── apple-touch-icon.png
├── browserconfig.xml
├── favicon-16x16.png
├── favicon-32x32.png
├── favicon.ico
├── mstile-150x150.png
├── safari-pinned-tab.svg
└── app.webmanifest
```

1. Insert the following code in the `<head>` section of your pages:

```html
<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png">
<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png">
<link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png">
<link rel="manifest" href="/app.webmanifest">
<link rel="mask-icon" href="/safari-pinned-tab.svg" color="#5bbad5">
<meta name="msapplication-TileColor" content="#da532c">
<meta name="theme-color" content="#ffffff">
```

## Service Worker

A Service Worker script is seperate from the main script of your application. They run independently and respond to events. This means they won't freeze or slow down the user interface. For example, when a fetch event is called in a service worker, the service worker handles it in the background without affecting the user's experience.

> Service workers are a fundamental part of a PWA. They enable fast loading (regardless of the network), offline access, push notifications, and other capabilities.

A commonly used file name for a service worker is `sw.js` and it should be placed at the root of the project. and registered at the end of the body of the `index.html`

```html
<script>
  if (typeof navigator.serviceWorker !== 'undefined') {
    navigator.serviceWorker.register('sw.js')
  }
</script>
```

### Offline Access

To make a PWA work offline, we need to implement offline capabilities by utilizing a service worker that handles caching.

When service workers are being installed for the first time, they send out an "install" event.

To make sure our app works offline, we can listen for the install event and pre-cache important resources in advance. This way, when you use the app without an internet connection, all the necessary files will already be stored and ready to use.

### `install`

```javascript
const CACHE_NAME = 'Some-Cache-Name';

// an array of assets to pre-cache
const PRECACHE_ASSETS = [
  "/",
  "/assets/main.css",
  "assets/main.js",
  "/Logo.png",
  "/manifest.webmanifest",
  "/favicon.ico",
  "https://fonts.googleapis.com/css2?family=Material+Symbols+Rounded:opsz,wght,FILL,GRAD@20..48,100..700,0..1,-50..200"
];

// A listener for the install event that pre-caches PRECACHE_ASSETS
self.addEventListener('install', event => {
    event.waitUntil((async () => {
        const cache = await caches.open(CACHE_NAME);
        cache.addAll(PRECACHE_ASSETS);
    })());
});
```

Once the installation event is completed, the next step in the service worker's lifecycle is activation. After installation, an activate event occurs.

During activation, we can instruct the service worker to take control of any existing instances of our app. This process is known as "claiming a client".

### `activate`

After the installation event, the next step in the service worker's lifecycle is activation. When the installation is finished, an `activate` event is triggered right away.

```javascript
self.addEventListener('activate', event => {
  event.waitUntil(clients.claim());
});
```

### `fetch`

Once the service worker has pre-cached the necessary assets, we need to add functionality to retrieve and use those assets.

To achieve this, we listen for the `fetch` event, which allows us to intercept and handle requests for the cached assets.

```javascript
self.addEventListener('fetch', event => {
  event.respondWith(async () => {
    const cache = await caches.open(CACHE_NAME);
      // check if cache maches request
      const cacheMatched = await cache.match(event.request);
      // check if the matched cache is not undefined
      if (cacheMatched !== undefined) {
	      // If there is a matched cache return it
          return cacheMatched;
      } else {
        // else, fetch from network
          return fetch(event.request)
      };
  });
});
```

## Conclusion

PWAs provide numerous benefits, including cross-platform compatibility, offline functionality, an app-like experience, faster performance, and improved discoverability. By putting in a little extra effort to develop a PWA, the user experience is greatly improved, resulting in a more engaging and accessible app across different devices.

## Resources

[PWABuilder Suite Documentation](https://docs.pwabuilder.com/)

[Making PWAs installable - Progressive web apps | MDN](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Guides/Making_PWAs_installable)

[Favicon Generator for perfect icons on all browsers](https://realfavicongenerator.net/)

