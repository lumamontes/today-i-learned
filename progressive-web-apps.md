# progressive-web-apps


Did you ever go into a website and see a prompt asking you to install the app? That's a progressive web app.
Today I learned about progressive web apps (PWAs) and how we can use them to provide a offline-first experience for users. PWAs can be installed on the user's device, similar to native apps. They offer offline capabilities, push notifications, and a responsive design, making them a great choice for modern web development.

What do we need to build a PWA?
1. ** Web app manifest**: This is a file that at minimum provides info for the browser to install the app, like app name and icon, and also other infos like theme color, background color.
2. **Service worker**: So I was kinda of confused on this one at first, but doing a demo I understood better: it basically acts as a proxy server that sits between web apps, the browser, and the network. It's a JavaScript file that runs in the background and can intercept and modify navigation and requests, caching resources to enable offline access and improve performance. It only works in https or secure contexts.

Links:

- [MDN Web Docs - Progressive Web Apps](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps)
- [Google Developers - Progressive Web Apps](https://developers.google.com/web/progressive-web-apps)