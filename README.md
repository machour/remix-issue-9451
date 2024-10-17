# Reproduce case for Remix's issue 9451

[#9451](#https://github.com/remix-run/remix/issues/9451)


## The problem

When running `npm run build`, two different tailwind.css files are generated between client & server builds. 

This causes a hydration issue and a major FOUC (me) when the server layout is replaced with the client layout.

## See it by yourself

```diff
🍺  ~/my-remix-app (master)*$ npm run build

> build
> remix vite:build

vite v5.1.0 building for production...
✓ 87 modules transformed.
build/client/assets/root-legacy-XzIZwh6f.js            1.77 kB │ gzip:  1.01 kB
build/client/assets/entry.client-legacy-BTxn8Qvm.js    3.72 kB │ gzip:  1.51 kB
build/client/assets/_index-legacy-Cx1xQ0eu.js          5.13 kB │ gzip:  2.11 kB
build/client/assets/jsx-runtime-legacy-DYl_mBzd.js     7.89 kB │ gzip:  3.01 kB
build/client/assets/polyfills-legacy-CaO-zBZS.js      92.73 kB │ gzip: 36.65 kB
build/client/assets/components-legacy-DsvU9e92.js    245.55 kB │ gzip: 78.19 kB
build/client/.vite/manifest.json                2.38 kB │ gzip:  0.48 kB
build/client/assets/root-B_SY1GJM.css           0.00 kB │ gzip:  0.02 kB
+build/client/assets/tailwind-CJADGt21.css       6.81 kB │ gzip:  2.04 kB
build/client/assets/root-CsgH_M0G.js            1.69 kB │ gzip:  0.98 kB
build/client/assets/entry.client-Bfvj2nMc.js    3.63 kB │ gzip:  1.48 kB
build/client/assets/_index-SDKXM1vf.js          5.16 kB │ gzip:  2.12 kB
build/client/assets/jsx-runtime-BlSqMCxk.js     8.09 kB │ gzip:  3.04 kB
build/client/assets/components-DnSrVFvj.js    247.54 kB │ gzip: 79.88 kB
✓ built in 3.81s
vite v5.1.0 building SSR bundle for production...
✓ 7 modules transformed.
build/server/.vite/manifest.json                0.34 kB
build/server/assets/server-build-B_SY1GJM.css   0.00 kB
+build/server/assets/tailwind-BgBy1aS6.css       6.80 kB
build/server/index.js                          12.71 kB

✓ 1 asset moved from Remix server build to client assets.
build/client/assets/tailwind-BgBy1aS6.css

✓ built in 36ms
```

This happens for me as I'm using the "@vitejs/plugin-legacy" plugin to handle older devices. As soon as I introduce transparency in a color, Remix (or vite?) starts generating different colors between server and client builds:

```diff
--- build/client/assets/tailwind-BgBy1aS6.css	2024-10-17 14:34:42
+++ build/client/assets/tailwind-CJADGt21.css	2024-10-17 14:34:35
@@ -385,7 +385,7 @@
   border-color: rgb(229 231 235 / var(--tw-border-opacity));
 }
 .bg-red-900\/10 {
-  background-color: #7f1d1d1a;
+  background-color: rgba(127, 29, 29, 0.1);
 }
 .stroke-gray-600 {
   stroke: #4b5563;
 ```

 Which in turns leads to the described issue.

## Relevant files
- vite.config.ts: Where the plugin is loaded
- root.css: Where the `?url` import is used, and its `<body>` tag where a transparent color is applied.
