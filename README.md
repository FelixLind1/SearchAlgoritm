
---

# SearchAlgoritm

![npm version](https://img.shields.io/npm/v/SearchAlgoritm.svg)
![npm downloads](https://img.shields.io/npm/dm/SearchAlgoritm.svg)
![license](https://img.shields.io/npm/l/SearchAlgoritm.svg?cacheBust=1)

A simple Node.js library for **fuzzy searching** through arrays of objects.
It calculates relevance scores based on how well the query matches the `title` and `description` of each item.

---

## Installation

```bash
npm install SearchAlgoritm
```

---

## Node.js Usage

```js
const { searchAlgoritm } = require('SearchAlgoritm');

// Example data
const dataList = [
  { title: "Apple", description: "A juicy fruit" },
  { title: "Banana", description: "Yellow and sweet" },
];

// Search for a query
const results = searchAlgoritm("apple", dataList);

console.log(results);
// Returns matched items sorted by relevance score
```

---

## Frontend Usage (Browser)

> Important: The browser cannot import directly from a package name in `node_modules` unless you use a bundler.
> If **not using a bundler**, import via a **relative path** to the ESM file.

### Folder structure

```
project/
├─ index.html
├─ css/
│  └─ style.css
├─ js/
│  ├─ search.js
│  ├─ ip-adress.js
│  └─ node_modules/
│      └─ SearchAlgoritm/src/searchAlgoritm.js
```

---

### `index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Search Example</title>
  <link rel="stylesheet" href="./css/style.css">
</head>
<body>

  <div class="page-wrapper">
    <div class="search-wrapper">
      <input type="text" id="searchInput" class="search-input" placeholder="Search...">
      <button id="searchBtn" class="search-btn">Search</button>
    </div>
    <ul id="searchResults" class="search-results"></ul>
  </div>

  <script type="module" src="./js/search.js"></script>
</body>
</html>
```
---

### `css/style.css`

```css
body {
  font-family: Arial, sans-serif;
  background: #0f0f0f;
  color: #e4e4e4;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  margin: 0;
}

.page-wrapper {
  max-width: 500px;
  width: 100%;
}

.search-wrapper {
  display: flex;
  gap: 8px;
  margin-bottom: 12px;
}

.search-input {
  flex: 1;
  padding: 10px;
  border-radius: 6px;
  border: 1px solid #444;
  background: #1b1b1b;
  color: #e6e6e6;
}

.search-btn {
  padding: 10px 16px;
  border: none;
  border-radius: 6px;
  background: #3b82f6;
  color: white;
  cursor: pointer;
}

.search-results {
  list-style: none;
  padding: 0;
}

.result-item {
  padding: 10px;
  background: #181818;
  margin-bottom: 6px;
  border-radius: 6px;
}

.no-result {
  color: #f87171;
  padding: 10px;
}
```

---

### `js/ip-adress.js`

```js
// Backend IP address for API requests
const backendIP = 'http://localhost:3000';
export default backendIP;
```

---

### `js/search.js`

```js
// Import the ESM version of searchAlgoritm from node_modules
import { searchAlgoritm } from '../node_modules/SearchAlgoritm/src/searchAlgoritm.js';
import backendIP from './ip-adress.js';

// Main function that initializes the search feature
async function initSearch() {
  try {
    // Fetch search data from the backend
    const res = await fetch(`${backendIP}/api/search`);
    const searchData = await res.json(); // array of objects with title/description

    // Get references to HTML elements
    const searchInput = document.getElementById('searchInput');   
    const searchBtn = document.getElementById('searchBtn');       
    const resultsList = document.getElementById('searchResults'); 

    // Function that runs every time the user performs a search
    function performSearch() {
      const query = searchInput.value; 
      const matches = searchAlgoritm(query, Object.values(searchData));

      resultsList.innerHTML = matches.length
        ? matches.map(item => `
            <li class="result-item">
              <strong>${item.title || ''}</strong><br>
              <span>${item.description || ''}</span>
            </li>
          `).join('')
        : `<li class="no-result">No results found</li>`; 
    }

    searchInput.addEventListener('keydown', e => { if(e.key === 'Enter') performSearch(); });
    searchBtn.addEventListener('click', performSearch);

  } catch (err) {
    console.error("Error fetching search data:", err);
    const resultsList = document.getElementById('searchResults');
    resultsList.innerHTML = `<li class="no-result">Failed to load data</li>`;
  }
}

initSearch();
```

---

### `server.js`

```js
import express from 'express';
import path from 'path';
import { fileURLToPath } from 'url';

const app = express();
const PORT = 3000;
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

// Serve static files from "Example files" folder
app.use(express.static(path.join(__dirname, 'Example files')));

// Example search data
const searchData = [
  { title: "Apple", description: "A juicy fruit" },
  { title: "Banana", description: "Yellow and sweet" },
  { title: "Orange", description: "Citrus fruit with vitamin C" },
  { title: "Grapes", description: "Small sweet fruits" }
];

// API endpoint
app.get('/api/search', (req, res) => {
  res.json(searchData);
});

// Start server
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```
