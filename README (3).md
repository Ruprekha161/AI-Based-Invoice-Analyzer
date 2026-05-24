# InvoiceLens AI

An AI-powered invoice analyzer built for the AI for FinTech hackathon. Upload any invoice or receipt, extract data automatically using OCR, categorize expenses with AI, and view a real-time financial dashboard.

---

## Features

- User registration and login with session management
- Invoice upload via file drag and drop, browse, or live camera capture
- OCR extraction using OCR.space free API — reads merchant name, amount, date, and GST from any PDF or image
- AI-based expense categorization using Claude API with a rule-based offline fallback
- Real-time dashboard showing income, expenses, savings, and financial score
- Category breakdown, monthly trends, weekly spending, and daily spending charts
- Duplicate invoice detection and high-spend anomaly alerts
- 30-day spending prediction based on current data
- AI-generated financial insights analyzing your full expense history
- Invoice history with search and category filter
- Profile settings with currency switcher (INR, USD, EUR), income setup, and JSON export
- Fully responsive — works on laptop and mobile with no installation

---

## Project Structure

```
invoicelens/
├── login.html        Auth page — register, login, demo mode
├── dashboard.html    Full application — all five pages in one file
└── README.md
```

The entire application runs in the browser with no backend, no server, and no dependencies to install. Data is stored in browser localStorage.

---

## How It Works

```
User registers or logs in
        |
Uploads an invoice (PDF, PNG, JPG) or captures with camera
        |
File is sent to OCR.space API
        |
Raw text is parsed — merchant, amount, date, tax extracted
        |
Claude AI categorizes the expense into Food, Shopping, Travel, etc.
        |
Invoice is saved to localStorage
        |
Dashboard refreshes — charts, score, and alerts update live
        |
Claude AI generates 4 personalized financial insights
```

---

## Built With

**Frontend**
- HTML5, CSS3, Vanilla JavaScript
- Chart.js 4.x for all charts and visualizations
- Google Fonts — Syne and Space Mono

**OCR**
- OCR.space Free API (https://ocr.space)
- Sends the uploaded file as base64 to the OCR.space endpoint
- Parses the returned text with regex to extract financial fields
- Falls back to filename-based detection if OCR returns no usable text

**AI**
- Anthropic Claude API — claude-sonnet-4-20250514
- Used for expense categorization and generating financial insights
- Rule-based categorization runs offline as fallback

**Storage**
- Browser localStorage — no database required
- All invoices, user session, and preferences persist across page reloads

---

## OCR Integration

The OCR pipeline sends the uploaded file to OCR.space and parses the response.

```javascript
async function runOCR(file) {
  const formData = new FormData();
  formData.append('file', file);
  formData.append('apikey', 'helloworld'); // free OCR.space key
  formData.append('language', 'eng');
  formData.append('isOverlayRequired', false);

  const response = await fetch('https://api.ocr.space/parse/image', {
    method: 'POST',
    body: formData
  });

  const result = await response.json();
  const text = result.ParsedResults?.[0]?.ParsedText || '';

  const amount = text.match(/(?:total|amount)[:\s₹$]*([0-9,]+\.?\d*)/i)?.[1];
  const date   = text.match(/\d{1,4}[-\/]\d{1,2}[-\/]\d{2,4}/)?.[0];
  const tax    = text.match(/(?:gst|tax)[:\s₹$]*([0-9,]+\.?\d*)/i)?.[1];

  return { amount, date, tax, rawText: text };
}
```

---

## AI Categorization

Each saved invoice is classified by Claude into one of nine categories.

```javascript
async function categorize(merchant, amount, description) {
  const response = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 15,
      messages: [{
        role: 'user',
        content: `Categorize this expense into one of: Food, Shopping, Travel,
Medical, Utilities, Entertainment, Subscriptions, Income, Other.
Merchant: ${merchant}, Amount: ${amount}, Notes: ${description}.
Reply with the category name only.`
      }]
    })
  });

  const data = await response.json();
  return data.content?.[0]?.text?.trim() || 'Other';
}
```

---

## Dashboard Metrics

The dashboard recalculates all values from localStorage on every save.

```javascript
function updateDashboard() {
  const invoices = JSON.parse(localStorage.getItem('il_invoices') || '[]');
  const prefs    = JSON.parse(localStorage.getItem('il_prefs') || '{}');
  const income   = parseFloat(prefs.income || 0);

  const expenses    = invoices.filter(i => i.cat !== 'Income');
  const totalExp    = expenses.reduce((s, i) => s + i.amount, 0);
  const totalInc    = invoices.filter(i => i.cat === 'Income')
                              .reduce((s, i) => s + i.amount, 0) + income;
  const savings     = totalInc - totalExp;
  const savingsRate = totalInc > 0 ? ((savings / totalInc) * 100).toFixed(1) : 0;

  document.getElementById('m-inc').textContent  = format(totalInc);
  document.getElementById('m-exp').textContent  = format(totalExp);
  document.getElementById('m-sav').textContent  = format(savings);
  document.getElementById('m-sav-sub').textContent = savingsRate + '% savings rate';
}
```

---

## Financial Score Algorithm

```javascript
function calcScore(totalExp, totalInc, savingsRate, invoices) {
  if (!invoices.length) return null;

  let score = 50;

  if (savingsRate > 30)     score += 20;
  else if (savingsRate > 15) score += 10;
  else if (savingsRate < 0)  score -= 20;

  if (totalInc > 0 && totalExp < totalInc * 0.7) score += 15;
  if (invoices.length > 5) score += 5;

  const anomalies = detectAnomalies(invoices);
  score -= anomalies.length * 5;

  return Math.min(100, Math.max(0, score));
}
```

---

## Expected Output

| Requirement | Status |
|---|---|
| Upload invoices in PDF or image format | Done |
| Extract merchant name, amount, date, tax | Done via OCR.space API |
| Auto-categorize expenses | Done via Claude AI |
| Generate spending insights and summaries | Done |
| Display visual dashboards and charts | Done — 5 chart types |
| AI-generated financial recommendations | Done |
| Category-wise expense breakdown | Done |
| GST and tax summaries | Done |
| Suspicious transaction detection | Done |
| Mobile support | Done |

---

