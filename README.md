// NM Coaching Pro - Service Worker
// Strategia: cache-first per asset statici, network-first per Supabase

const CACHE_VERSION = 'nm-coaching-v11';
const STATIC_CACHE = `${CACHE_VERSION}-static`;
const RUNTIME_CACHE = `${CACHE_VERSION}-runtime`;

// Asset locali da pre-cachare all'installazione
const PRECACHE_URLS = [
  './',
  './index.html',
  './manifest.json',
  './icon-192.png',
  './icon-512.png'
];

// Domini Supabase: SEMPRE network-first (dati live)
const SUPABASE_HOSTS = ['supabase.co', 'supabase.in'];

// CDN di librerie/font: cache-first (cambiano raramente)
const CDN_HOSTS = [
  'cdnjs.cloudflare.com',
  'cdn.jsdelivr.net',
  'fonts.googleapis.com',
  'fonts.gstatic.com'
];

// Install: pre-cache asset statici
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(STATIC_CACHE)
      .then((cache) => cache.addAll(PRECACHE_URLS))
      .then(() => self.skipWaiting())
      .catch((err) => console.error('[SW] Precache failed:', err))
  );
});

// Activate: pulisci cache vecchie
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys()
      .then((keys) => Promise.all(
        keys
          .filter((key) => !key.startsWith(CACHE_VERSION))
          .map((key) => caches.delete(key))
      ))
      .then(() => self.clients.claim())
  );
});

// Fetch: routing per tipo di risorsa
self.addEventListener('fetch', (event) => {
  const { request } = event;
  const url = new URL(request.url);

  // Solo GET
  if (request.method !== 'GET') return;

  // Supabase: network-first (con fallback cache se offline)
  if (SUPABASE_HOSTS.some((host) => url.hostname.includes(host))) {
    event.respondWith(networkFirst(request));
    return;
  }

  // CDN librerie/font: cache-first
  if (CDN_HOSTS.some((host) => url.hostname.includes(host))) {
    event.respondWith(cacheFirst(request));
    return;
  }

  // Asset locali (stesso origine): cache-first
  if (url.origin === self.location.origin) {
    event.respondWith(cacheFirst(request));
    return;
  }

  // Default: network con fallback cache
  event.respondWith(networkFirst(request));
});

// Strategia: cache-first
async function cacheFirst(request) {
  const cached = await caches.match(request);
  if (cached) return cached;
  try {
    const response = await fetch(request);
    if (response.ok) {
      const cache = await caches.open(RUNTIME_CACHE);
      cache.put(request, response.clone());
    }
    return response;
  } catch (err) {
    return new Response('Offline', { status: 503, statusText: 'Offline' });
  }
}

// Strategia: network-first
async function networkFirst(request) {
  try {
    const response = await fetch(request);
    if (response.ok) {
      const cache = await caches.open(RUNTIME_CACHE);
      cache.put(request, response.clone());
    }
    return response;
  } catch (err) {
    const cached = await caches.match(request);
    if (cached) return cached;
    return new Response('Offline', { status: 503, statusText: 'Offline' });
  }
}
