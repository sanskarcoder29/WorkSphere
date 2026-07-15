# PWA Asset Caching Strategy

This document outlines the service worker caching strategies used in the WorkSphere Progressive Web App (PWA) to ensure offline reliability and optimal performance while handling network fallbacks.

---

## 1. Cache-First Strategy (Static Assets)

**Target:** Local icons, fonts, UI images, and static global stylesheets.
**Reasoning:** These assets rarely change. Serving them directly from the cache immediately improves initial load times, saves bandwidth, and ensures the UI structure remains intact offline.

**Service Worker Implementation Pattern:**
The service worker intercepts the request, checks the asset registry cache first, and only falls back to the network if the asset is missing.

```javascript
// Example: Cache-First for static assets
self.addEventListener("fetch", (event) => {
  const isStaticAsset =
    event.request.destination === "image" ||
    event.request.destination === "font";

  if (isStaticAsset) {
    event.respondWith(
      caches.match(event.request).then((cachedResponse) => {
        return cachedResponse || fetch(event.request);
      }),
    );
  }
});
```

## 2. Network-First Strategy (Dynamic Content & Mapping Tiles)

**Target:** Mapping tiles, API endpoints, and user-specific dynamic data.
**Reasoning:** This data updates frequently or is location-dependent. We must always attempt to show the user the most up-to-date information, falling back to the cached versions only if the network request fails (e.g., when the user loses signal).

**Service Worker Implementation Pattern:**
The service worker attempts to fetch from the network first. If successful, it updates the cache. If the network fails, it intercepts the failure and returns the last known good state from the cache.

```javascript
// Example: Network-First for mapping tiles and API requests
self.addEventListener("fetch", (event) => {
  const isDynamicContent =
    event.request.url.includes("/api/") ||
    event.request.url.includes("/tiles/");

  if (isDynamicContent) {
    event.respondWith(
      fetch(event.request)
        .then((networkResponse) => {
          // Clone the response before caching it
          return caches.open("dynamic-content-cache").then((cache) => {
            cache.put(event.request, networkResponse.clone());
            return networkResponse;
          });
        })
        .catch(() => {
          // Network failed, fallback to cache
          return caches.match(event.request);
        }),
    );
  }
});
```
