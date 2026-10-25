
const CACHE_NAME = 'wotraca-v2-playstore';
const urlsToCache = [
  './',
  './index.html',
  './manifest.json',
  './icons/icon-192x192.png',
  './icons/icon-512x512.png',
  './screenshots/01-accueil-hero.jpg',
  './screenshots/02-formulaire-eligibilite.jpg'
];

self.addEventListener('install', e => {
  e.waitUntil(caches.open(CACHE_NAME).then(c => c.addAll(urlsToCache)));
  self.skipWaiting();
});
self.addEventListener('activate', e => {
  e.waitUntil(caches.keys().then(keys => Promise.all(keys.map(k => { if(k!==CACHE_NAME) return caches.delete(k);}))));
  self.clients.claim();
});
self.addEventListener('fetch', e => {
  // For navigation, serve index.html to keep inside app (no Chrome)
  if (e.request.mode === 'navigate') {
    e.respondWith(
      fetch(e.request).catch(()=> caches.match('./index.html'))
    );
    return;
  }
  e.respondWith(
    caches.match(e.request).then(r => r || fetch(e.request).then(res => {
      // cache images
      if(e.request.url.match(/\.(png|jpg|jpeg|webp)$/)){
        const clone = res.clone();
        caches.open(CACHE_NAME).then(c=> c.put(e.request, clone));
      }
      return res;
    }).catch(()=> caches.match('./index.html')))
  );
});
