# 🔍 Web Scraper Using Chrome DevTools Console (JavaScript Only)

A lightweight, browser-based web scraper that extracts data from [quotes.toscrape.com](https://quotes.toscrape.com) using **pure JavaScript** inside Chrome's DevTools Console — no external tools, no setup, no browser automation!

✅ Outputs data in **JSON**, **CSV**, and **Excel (.xlsx)** formats  
✅ Uses native `fetch()` and DOM parsing  
✅ Follows all pagination automatically

---

## 📸 Demo Screenshots

> 📷 Inspecting elements → 🧠 Writing code → 📥 Exporting data

| HTML Inspection | Query Selection | Console Output |
|-----------------|-----------------|----------------|
| ![HTML](./screenshots/browser_inspect.png) | ![Query](./screenshots/console.png) | ![Console](./screenshots/js_in_console.png) |

---

## 🚀 How It Works

### 1️⃣ Open Chrome DevTools Console

- Go to [https://quotes.toscrape.com](https://quotes.toscrape.com)
- Press `Ctrl+Shift+I` or right-click → **Inspect**
- Navigate to the **Console** tab

### 2️⃣ Paste and Run the Full JavaScript Code

> 🔽 See `scraper.js` in this repo or copy from below 👇

```js
(async function scrapeAndDownloadQuotes() {
  const baseURL = "https://quotes.toscrape.com";
  let currentURL = "/";
  const allQuotes = [];

  while (currentURL) {
    const response = await fetch(baseURL + currentURL);
    const html = await response.text();

    const tempDiv = document.createElement('div');
    tempDiv.innerHTML = html;

    const quoteDivs = tempDiv.querySelectorAll('.quote');
    quoteDivs.forEach(quoteDiv => {
      const quoteText = quoteDiv.querySelector('.text')?.innerText.trim();
      const author = quoteDiv.querySelector('.author')?.innerText.trim();
      const tags = Array.from(quoteDiv.querySelectorAll('.tags .tag')).map(tag => tag.innerText.trim());

      allQuotes.push({
        quote: quoteText,
        author: author,
        tags: tags.join(", ")
      });
    });

    const nextLink = tempDiv.querySelector('.next > a');
    currentURL = nextLink ? nextLink.getAttribute('href') : null;
  }

  // ===== Download JSON =====
  const jsonBlob = new Blob([JSON.stringify(allQuotes, null, 2)], { type: 'application/json' });
  const jsonUrl = URL.createObjectURL(jsonBlob);
  const jsonLink = document.createElement('a');
  jsonLink.href = jsonUrl;
  jsonLink.download = 'quotes.json';
  jsonLink.click();

  // ===== Download CSV =====
  const csvHeader = "quote,author,tags\n";
  const csvRows = allQuotes.map(q => {
    const quote = `"${q.quote.replace(/"/g, '""')}"`;
    const author = `"${q.author.replace(/"/g, '""')}"`;
    const tags = `"${q.tags.replace(/"/g, '""')}"`;
    return `${quote},${author},${tags}`;
  });
  const csvContent = csvHeader + csvRows.join("\n");
  const csvBlob = new Blob([csvContent], { type: 'text/csv' });
  const csvUrl = URL.createObjectURL(csvBlob);
  const csvLink = document.createElement('a');
  csvLink.href = csvUrl;
  csvLink.download = 'quotes.csv';
  csvLink.click();

  // ===== Download Excel (.xlsx) =====
  const script = document.createElement('script');
  script.src = "https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js";
  script.onload = () => {
    const worksheet = XLSX.utils.json_to_sheet(allQuotes);
    const workbook = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(workbook, worksheet, "Quotes");
    const excelBlob = XLSX.write(workbook, { bookType: "xlsx", type: "array" });
    const excelUrl = URL.createObjectURL(new Blob([excelBlob], { type: 'application/octet-stream' }));
    const excelLink = document.createElement('a');
    excelLink.href = excelUrl;
    excelLink.download = 'quotes.xlsx';
    excelLink.click();
  };
  document.body.appendChild(script);
})();
