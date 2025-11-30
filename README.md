# search-algoritm

![npm version](https://img.shields.io/npm/v/search-algoritm.svg)
![npm downloads](https://img.shields.io/npm/dm/search-algoritm.svg)
![license](https://img.shields.io/npm/l/search-algoritm.svg?cacheBust=1)
[![GitHub Repository](https://img.shields.io/badge/GitHub-Repository-blue?logo=github)](https://github.com/FelixLind1/SearchAlgoritm)

A simple Node.js library for **fuzzy searching** through arrays of objects.  
It calculates relevance scores based on how well the query matches the `title` and `description` of each item.

---

## Installation

```bash
npm install search-algoritm
```

---

## Node.js Usage (Server-side) — In-Built Data

```js
const express = require('express');
const path = require('path');
const { searchAlgoritm } = require('search-algoritm');

const app = express();
const PORT = 3000;

// Serve static files
const staticPath = path.join(__dirname, 'Example files');
app.use(express.static(staticPath));

// Example search data directly in server
const searchData = [
  { title: "Apple", description: "A juicy fruit" },
  { title: "Banana", description: "Yellow and sweet" },
  { title: "Orange", description: "Citrus fruit with vitamin C" },
  { title: "Grapes", description: "Small sweet fruits" }
];

// API: performs server-side search
app.get('/api/search', (req, res) => {
  const query = req.query.q || "";
  const results = searchAlgoritm(query, searchData);
  res.json({ query, results });
});

// Start server
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

✅ Fast setup, simple for small datasets.

---

## Node.js Usage (Server-side) — JSON Data

### `data.json`
```json
[
  { "title": "Apple", "description": "A juicy fruit" },
  { "title": "Banana", "description": "Yellow and sweet" },
  { "title": "Orange", "description": "Citrus fruit with vitamin C" },
  { "title": "Grapes", "description": "Small sweet fruits" }
]
```

### `server.js`
```js
const express = require('express');
const path = require('path');
const fs = require('fs');
const { searchAlgoritm } = require('search-algoritm');

const app = express();
const PORT = 3000;

// Serve static files
const staticPath = path.join(__dirname, 'Example files');
app.use(express.static(staticPath));

// Load search data from JSON file
const dataPath = path.join(__dirname, 'data.json');
let searchData = [];
try {
  const rawData = fs.readFileSync(dataPath, 'utf-8');
  searchData = JSON.parse(rawData);
} catch (err) {
  console.error('Error reading JSON file:', err);
}

// API: performs server-side search
app.get('/api/search', (req, res) => {
  const query = req.query.q || "";
  const results = searchAlgoritm(query, searchData);
  res.json({ query, results });
});

// Start server
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

✅ Flexible for larger datasets; update JSON without touching server code.

---

## Frontend Usage (Browser)

> Important: The frontend fetches search results from the server.
> You do **not** import the NPM package directly in the browser.

### `ip-adress.js`
```js
// Example backend IP
const backendIP = 'http://localhost:3000';
export default backendIP;
```

### `search.js`
```js
import backendIP from './ip-adress.js';

async function initSearch() {
  const searchInput = document.getElementById('searchInput');
  const searchBtn = document.getElementById('searchBtn');
  const resultsList = document.getElementById('searchResults');

  async function performSearch() {
    const query = searchInput.value.trim();
    if (!query) {
      resultsList.innerHTML = `<li class="no-result">Please enter a search term</li>`;
      return;
    }

    try {
      const res = await fetch(`${backendIP}/api/search?q=${encodeURIComponent(query)}`);
      const { results } = await res.json();

      resultsList.innerHTML = results.length
        ? results.map(item => `
            <li class="result-item">
              <strong>${item.title}</strong><br>
              <span>${item.description}</span>
            </li>
          `).join('')
        : `<li class="no-result">No matches found</li>`;
    } catch (err) {
      console.error('Error fetching search results:', err);
      resultsList.innerHTML = `<li class="no-result">Could not fetch results</li>`;
    }
  }

  searchInput.addEventListener('keydown', e => { if (e.key === 'Enter') performSearch(); });
  searchBtn.addEventListener('click', performSearch);
}

initSearch();
```

---

## Example HTML

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Search Example</title>
  <link rel="stylesheet" href="./style.css">
</head>
<body>
  <div class="page-wrapper">
    <div class="search-wrapper">
      <input type="text" id="searchInput" class="search-input" placeholder="Search...">
      <button id="searchBtn" class="search-btn">Search</button>
    </div>
    <ul id="searchResults" class="search-results"></ul>
  </div>

  <script type="module" src="./search.js"></script>
</body>
</html>
```

---

## Style (`style.css`)

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

## License

MIT

---

**Made by [Felix Lind](https://github.com/FelixLind1)**
