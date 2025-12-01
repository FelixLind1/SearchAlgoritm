````markdown
# search-algoritm

![npm version](https://img.shields.io/npm/v/search-algoritm.svg)
![npm downloads](https://img.shields.io/npm/dm/search-algoritm.svg)
![license](https://img.shields.io/npm/l/search-algoritm.svg?cacheBust=1)
[![GitHub Repository](https://img.shields.io/badge/GitHub-Repository-blue?logo=github)](https://github.com/FelixLind1/SearchAlgoritm)

A simple Node.js library for **fuzzy searching** through arrays of objects.  
It calculates relevance scores based on how well the query matches the `title` and `description` of each item.  
**Supports Levenshtein distance for fuzzy matching.**

---

## Installation

```bash
npm install search-algoritm
````

---

## Node.js Usage (Server-side)

This package supports **two server setups** depending on your data source:

1. **Cached JSON Server** – loads JSON once into memory and optionally reloads on file changes (fast for frequent searches).
2. **Dynamic JSON Server** – reads JSON from disk on every request (simpler but slower for large datasets).

---

### 1. Cached JSON Server (`server.js`)

```js
const express = require('express');
const path = require('path');
const fs = require('fs').promises;
const { searchAlgoritm } = require('search-algoritm');

const app = express();
const PORT = 3000;
const dataPath = path.join(__dirname, 'data.json');

let searchData = [];

const loadData = async () => {
  try {
    const rawData = await fs.readFile(dataPath, 'utf-8');
    searchData = JSON.parse(rawData);
    console.log(`[server] Loaded ${searchData.length} items`);
  } catch (err) {
    console.error('[server] Failed to load data.json:', err);
  }
};

loadData();
fs.watchFile(dataPath, async () => {
  console.log('[server] data.json changed, reloading cache...');
  await loadData();
});

app.use(express.static(path.join(__dirname, 'Example files')));

app.get('/api/search', async (req, res) => {
  const query = (req.query.q || "").trim();
  if (!query) return res.json({ query, results: [] });

  const results = searchAlgoritm(query, searchData);
  res.json({ query, results });
});

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

✅ Fast searches using in-memory cache. Best for **medium to large datasets** where performance matters.

---

### 2. Dynamic JSON Server (`json-server.js`)

```js
const express = require('express');
const path = require('path');
const fs = require('fs').promises;
const { searchAlgoritm } = require('search-algoritm');

const app = express();
const PORT = 3000;

app.use(express.static(path.join(__dirname, 'Example files')));
const dataPath = path.join(__dirname, 'data.json');

const loadJson = async (filePath) => {
  try {
    const rawData = await fs.readFile(filePath, 'utf-8');
    return JSON.parse(rawData);
  } catch (err) {
    console.error('Error reading JSON file:', err);
    return [];
  }
};

app.get('/api/search', async (req, res) => {
  const query = req.query.q || "";
  const searchData = await loadJson(dataPath);
  const results = searchAlgoritm(query, searchData);
  res.json({ query, results });
});

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

✅ Reads JSON on each request. Best for **small datasets** or when the data changes frequently and caching is not desired.

---

## Best Practices

| Server Type  | Pros                                  | Cons                                                     | When to Use                                 |
| ------------ | ------------------------------------- | -------------------------------------------------------- | ------------------------------------------- |
| Cached JSON  | Fast searches, reduces disk I/O       | Slightly more memory usage, needs cache reload on change | Medium to large datasets, frequent searches |
| Dynamic JSON | Always reads fresh data, simple setup | Slower for large datasets                                | Small datasets or frequently changing data  |

**Tip:** For production with large datasets, use the **cached server**. For quick prototypes or dynamic content that changes often, use the **dynamic server**.

---

## Frontend Usage (Browser)

> Always fetch search results from the server.

### `ip-adress.js`

```js
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

## Example HTML & CSS

See previous sections for **HTML structure** and **style.css**.

---

## License

MIT

**Made by [Felix Lind](https://github.com/FelixLind1)**

```
