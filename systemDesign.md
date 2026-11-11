# Basics

- never rush to implement
- always talk to interviewer about the requirement and if u are understanding in the right direction
- practice 2-3 system design question by yourself on some online platform which make it comfortable to use (draw.io)

# Frontend System Design

- HLD and LLD, 2 type of system design

![HLD LLD image](./img/design-1.png)
![HLD LLD image](./img/design-1-1.png)

HLD have Requirement gathering, Scoping and Tech choices where as in LLD we discuss about Component Architecture and Data API and protocols and implementations.

## FSD components

### Architectural Pattern

- Monolithic frontend - when all project component reside in one tree
- Micro frontend - when for each specific task like dine-in area, swimming pool area etc. are treated as 1 specific component and the main benefit is we can design them with different technology and integrate them.
  - Different ways fir integrations are - Iframe, web components, module federation, micro apps/ route based.

### Communication Protocol

- Long pooling - keep sending request to server and server reply for each request.
- websocket - 2 way commmunication but no need to send request always.
- server sent event - 1 way communication, once client sent the request now server will only reply - like email subscribe, msg notification etc.

### Availability

- When if we are offline we want to keep user enggaged then this helps. Mainly Service worker helps to achieve this.

A `Service Worker` is a JavaScript script that runs in the background, separately from the web page, and acts as a proxy between the browser and the network.

**A Service Worker is a background JavaScript worker that intercepts network requests and enables caching, offline functionality, push notifications, and background synchronization.**

Key responsibilities:

1. Caching: Caches resources for faster loading.
2. Offline support: Allows applications to work without an internet connection.
3. Push notifications: Receives notifications even when the website isn't open.
4. Background sync: Syncs data when the internet connection is restored.

`It has no direct access to the DOM and communicates with the page through messages.`

### Accessibility

- Keyboard accessibility
- screen reader
- color/contrast

### Consistency

- CSS properties
- Polyfills in JS
- now we use Design system like AntUI, fluent ui, material ui etc which itself manages different browwser version adaptability.

### Credibility and Trust(SEO)

- On-page
  - Title
  - description
  - meta
  - performance
  - content
- Off-page
  - Backlinks 
  - ads

### Logging and Monitoring

- Error Logging
- User tracking
- User activity
- Feature usage
- Infra/ Capacity monitoring

### Database/ Caching

- HTTP caching
- in-memory caching
- apollo caching
- state management(redux, context, zustand)
- local storage
- session storage
- cookies
- IndexedDB

### Security

- Authetication/ Authorization
- CSP - Content security policy
- CORS
- Man in the middle

### Performance and Optimization

- Assets Optimization
- Delivery Option
- Build assets
- SSR (server side rendering)
- service worker
- web vitals
- Percieved performance (visible to user)

### Testing

- unit testing - jest,mocha,chai
- integration testing - cypress
- end to end testing (E2E) - Playwright