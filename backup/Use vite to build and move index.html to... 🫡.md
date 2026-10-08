- package.json
```
"scripts": {
    "build": "vite build && mv ../think/public/static/wxmp/index.html ../think/view/wxmp/home_index.html", // Move to specified location
}
```
- vite.config.js
```
base: '/static/wxmp', // Change static resource paths

build: {
    outDir: '../think/public/static/wxmp', // Packaging location
    minify: true, // Whether to compress
}
```