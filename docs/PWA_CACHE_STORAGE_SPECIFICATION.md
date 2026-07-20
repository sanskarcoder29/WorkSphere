# PWA Service Worker Cache Storage Specification

## Overview

This document outlines the Service Worker caching strategies for the WorkSphere Progressive Web App (PWA). It details Workbox routing, cache size limitations, offline fallbacks, and specific handling for Mobile Safari storage quotas.

---

## 1. Workbox Routing & Strategies

We utilize Google Workbox to manage our service worker routing and caching logic to ensure reliable offline performance and fast load times.

- **Static Assets (CSS, JS, Fonts):** Uses the `CacheFirst` strategy. These assets are versioned during the build process, making them safe to cache long-term.
- **Images & Avatars:** Uses the `CacheFirst` strategy with strict capacity limits to prevent storage bloat.
- **API Requests (Data fetching):** Uses the `StaleWhileRevalidate` strategy to show cached data instantly while refreshing the cache in the background.

## 2. Cache Size Capping & LRU Eviction

To prevent the PWA from consuming too much device storage, we enforce strict size limits using the Workbox `ExpirationPlugin`. Workbox automatically applies a Least Recently Used (LRU) eviction policy when these limits are reached.

- **Image Cache (`worksphere-images`):** Capped at `50` entries. Max age: `30 days`.
- **API Data Cache (`worksphere-api`):** Capped at `100` entries. Max age: `24 hours`.
- **Static Assets (`worksphere-static`):** Managed automatically by Workbox Precache (purges old revisions on updates).

## 3. Offline Page Fallback

When a user loses network connectivity and navigates to a route that is not currently stored in the cache, the Service Worker intercepts the failed network request and serves a pre-cached offline fallback screen.

- **Offline Route:** `/offline.html` (Precached during the initial Service Worker `install` event).
- **Implementation:** Handled via a global `setCatchHandler` in Workbox that triggers specifically for `request.destination === 'document'`.

## 4. Mobile Safari (iOS) Storage Quota Rules

iOS Safari employs aggressive and opaque cache eviction policies compared to Chrome/Android, requiring specific architectural considerations.

- **Storage Limits:** Safari typically limits total PWA storage to roughly 50MB for isolated web apps, though this scales dynamically based on available device disk space.
- **Silent Eviction:** If the iOS device experiences storage pressure, Safari will silently purge the Service Worker Cache API data without firing any warning events to the application.
- **Mitigation Strategy:**
  1.  Keep the `worksphere-static` cache as small as possible (under 10MB).
  2.  Rely on IndexedDB for critical, user-generated offline data, as Safari tends to prioritize Cache API data for eviction before touching IndexedDB.
