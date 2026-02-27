````markdown
# search-algoritm

![npm version](https://img.shields.io/npm/v/search-algoritm.svg)
![npm downloads](https://img.shields.io/npm/dm/search-algoritm.svg)
![license](https://img.shields.io/npm/l/search-algoritm.svg)
[![GitHub Repository](https://img.shields.io/badge/GitHub-Repository-blue?logo=github)](https://github.com/FelixLind1/SearchAlgoritm)

A lightweight Node.js library for **fuzzy searching** arrays of objects.  
It calculates relevance scores based on how well a query matches **any string field** in the objects.

**Features:**

- Fuzzy matching using **Levenshtein distance**  
- **Accent-insensitive** searches (`é → e`)  
- Token-based word matching for partial or multi-word queries  
- Multi-word sequence detection  
- Weighted scoring: fields can contribute differently to relevance  
- Stopword filtering for more relevant results  
- Works with **any object structure**, not limited to `title`, `description`, `file`

---

## Installation

```bash
npm install search-algoritm
```

---

## Node.js Usage (Server-side)

This library supports **two server setups** depending on your dataset size and update frequency:

1. **Cached JSON Server** – loads JSON into memory once, reloads if the file changes (fast for large datasets).
2. **Dynamic JSON Server** – reads JSON from disk on each request (simple, slower for large datasets).

---

### 1. Cached JSON Server (`server.js`)

```js
const express = require('express');
const path = require('path');
const fs = require('fs').promises;
const searchAlgoritm = require('search-algoritm'); // default export

const app = express();
const PORT = 3000;
const dataPath = path.join(__dirname, 'data.json');

let searchData = [];

/**
 * Load JSON into memory
 */
const loadData = async () => {
  try {
    const rawData = await fs.readFile(dataPath, 'utf-8');
    searchData = JSON.parse(rawData);
    console.log(`[server] Loaded ${searchData.length} items`);
  } catch (err) {
    console.error('[server] Failed to load data.json:', err);
    searchData = [];
  }
};

// Initial load
loadData();

// Reload cache if file changes
fs.watchFile(dataPath, async () => {
  console.log('[server] data.json changed, reloading cache...');
  await loadData();
});

// Serve static frontend files
app.use(express.static(path.join(__dirname, 'Example files')));

/**
 * Search API endpoint
 */
app.get('/api/search', async (req, res) => {
  const query = (req.query.q || "").trim();
  if (!query) return res.json({ query, results: [] });

  const results = searchAlgoritm(query, searchData);
  res.json({ query, results });
});

/**
 * Start server
 */
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

✅ Fast searches using in-memory cache. Ideal for **medium to large datasets**.

---

### 2. Dynamic JSON Server (`json-server.js`)

```js
const express = require('express');
const path = require('path');
const fs = require('fs').promises;
const searchAlgoritm = require('search-algoritm'); // default export

const app = express();
const PORT = 3000;

// Directory containing the static frontend files
const staticPath = path.join(__dirname, 'Example files');
app.use(express.static(staticPath));

// Path to the JSON dataset used by the search engine
const dataPath = path.join(__dirname, 'data.json');

/**
 * Reads and parses a JSON file asynchronously.
 * If reading or parsing fails, returns an empty array.
 */
const loadJson = async (filePath) => {
  try {
    const rawData = await fs.readFile(filePath, 'utf-8');
    return JSON.parse(rawData);
  } catch (err) {
    console.error('[server] Error loading JSON file:', err);
    return [];
  }
};

/**
 * Search API endpoint
 * Example: GET /api/search?q=example
 */
app.get('/api/search', async (req, res) => {
  const query = req.query.q || "";

  // Load data from disk
  const searchData = await loadJson(dataPath);

  // Execute search
  const results = searchAlgoritm(query, searchData);

  res.json({ query, results });
});

/**
 * Start the Express server
 */
app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});
```

✅ Reads JSON on each request. Best for **small datasets** or frequently changing data.

---

## Best Practices

| Server Type  | Pros                            | Cons                                           | When to Use                                 |
| ------------ | ------------------------------- | ---------------------------------------------- | ------------------------------------------- |
| Cached JSON  | Fast searches, reduces disk I/O | Uses more memory, needs cache reload on change | Medium to large datasets, frequent searches |
| Dynamic JSON | Always fresh data, simple setup | Slower for large datasets                      | Small datasets or frequently changing data  |

**Tip:** For production with large datasets, use the **cached server**. For prototypes or rapidly changing content, use the **dynamic server**.

---

## Frontend Usage (Browser)

Always fetch search results from the server.

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
              ${Object.entries(item).map(([key, value]) => `<strong>${key}:</strong> ${value}<br>`).join('')}
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

> **Note:** The frontend example now dynamically displays **all fields** in the object, reflecting the fact that `searchAlgoritm` searches all string fields, not just `title`, `description`, or `file`.

---

## License

MIT

**Made by [Felix Lind](https://github.com/FelixLind1)**

```
