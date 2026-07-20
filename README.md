# ScraperAPI JavaScript Rendering: How to Scrape Dynamic Pages with Node.js — `render=true` Explained, Plan Comparison, Credit Costs, and Getting Started Without Getting Blocked

You've written the scraper. Axios fetches the URL. Cheerio tries to parse the result. And you get... almost nothing. Or worse, a skeleton of a page that has none of the actual data you care about.

Welcome to the world of JavaScript-rendered websites — where the content you see in your browser was built by scripts that run *after* the initial HTML loads, and most scrapers never even see it.

If you've been searching for how to get **ScraperAPI JavaScript** rendering working, you're in the right place. This guide walks through exactly how it works, when you need it, how much it costs (the credit math is trickier than the pricing page suggests), and how to integrate it into a real Node.js project from scratch.

---

## **Why JavaScript Rendering Is a Problem for Scrapers**

When your browser opens a modern website, a lot happens before you see the page. The server sends some HTML. Then the browser downloads JavaScript bundles, runs them, makes additional API calls, and *then* paints the content you see. That entire process can take one to five seconds — and for a headless HTTP client like Axios, none of it happens. The request fires, the server sends the raw HTML shell, and the client receives it without executing a single line of JavaScript.

The result: you scrape a page and get placeholder divs, loading spinners encoded in HTML, or empty `<body>` tags where the product prices and article headlines are supposed to live.

Dealing with this usually means one of two approaches:

1. **Run your own headless browser** — launch Puppeteer or Playwright, let the browser render the page, and then scrape the rendered output.
2. **Use a rendering API** — delegate that headless browser work to a service and just consume the result.

Option 1 works great until you hit bot detection, IP bans, CAPTCHAs, or need to run 200 parallel requests. Option 2 is why services like ScraperAPI exist.

---

## **How ScraperAPI JavaScript Rendering Actually Works**

ScraperAPI's JavaScript rendering feature is activated with a single parameter: `render=true`. When you include it in your request, instead of fetching the raw HTML from the server and returning it immediately, ScraperAPI spins up a headless Chromium browser instance, navigates to the URL, waits for the JavaScript to execute and the page to settle, and *then* returns the fully rendered HTML.

From your code's perspective, you send one request and get back one response — the same API call structure you'd use for a static page. The heavy lifting (browser pool management, residential proxy rotation, CAPTCHA solving if the page has one) all happens behind the scenes.

The basic API request looks like this:

javascript
import fetch from 'node-fetch';

const API_KEY = 'your_api_key_here';
const targetUrl = 'https://example.com/dynamic-page';

const url = `https://api.scraperapi.com?api_key=${API_KEY}&render=true&url=${encodeURIComponent(targetUrl)}`;

fetch(url)
  .then(response => response.text())
  .then(html => console.log(html))
  .catch(err => console.error(err));


If you're using Axios, which is common in Node.js scraping projects, the integration is just as clean:

javascript
const axios = require('axios');

const API_KEY = 'your_api_key_here';
const targetUrl = 'https://example.com/dynamic-page';

axios('https://api.scraperapi.com/', {
  params: {
    url: targetUrl,
    api_key: API_KEY,
    render: 'true'
  }
})
.then(response => {
  const html = response.data;
  // pass to Cheerio, parse, extract — same as always
  console.log(html);
})
.catch(console.error);


That's it. No Puppeteer to install, no browser process to manage, no Chromium version conflicts. The `render=true` parameter is supported across all of ScraperAPI's request modes — the standard API endpoint, the async endpoint (for high-volume batch jobs), and the proxy mode (for when you want to route an existing tool like Puppeteer through ScraperAPI's infrastructure).

---

## **The `wait_for_selector` Parameter: For Pages That Load Content Slowly**

Sometimes `render=true` alone isn't enough. Some pages use lazy loading, infinite scroll, or delayed API calls that mean important content might not appear until two or three seconds after the initial render. ScraperAPI handles this with a companion parameter: `wait_for_selector`.

Pass a CSS selector, and the API will wait until that element appears in the DOM before returning the response. If the element doesn't appear within a reasonable timeout window, the request returns whatever state the page is in.

javascript
const url = `https://api.scraperapi.com?api_key=${API_KEY}&render=true&wait_for_selector=.product-price&url=${encodeURIComponent(targetUrl)}`;


This is especially useful for e-commerce pages where price data or stock levels load asynchronously, or news sites where the article body renders after an ad network call completes. Using `wait_for_selector` doesn't cost any extra credits — it's included in the base `render=true` cost.

---

## **Understanding the Credit Math Before You Commit to a Plan**

This is the section most people wish they'd read before signing up. The pricing page lists numbers like "100,000 credits" for the entry-level plan, and that sounds like a lot. Whether it actually *is* a lot depends entirely on what you're scraping and which parameters you're using.

A standard request to a plain, unprotected page costs **1 credit**. The moment you add `render=true`, that's **+10 credits per request**. So a basic rendered page costs 11 credits. Scraping Amazon? That's a **5× domain multiplier** plus the 10 for rendering — 15 credits per request, meaning that "100,000 credit" plan gets you roughly 6,600 successful scrapes of Amazon product pages. Scraping Google? The domain multiplier alone is **25×**, before rendering is even added.

Here's the full breakdown:

| Request Type | Credit Cost |
|---|---|
| Standard page (no rendering) | 1 credit |
| E-commerce (Amazon) | 5 credits |
| SERP (Google, Bing) | 25 credits |
| Social (LinkedIn) | 30 credits |
| + `render=true` | +10 credits |
| + `premium=true` | +10 credits |
| + `ultra_premium=true` | +30 credits |
| + `premium=true` + `render=true` combined | 25 credits total |
| + `ultra_premium=true` + `render=true` combined | 75 credits total |
| + Cloudflare / Datadome / PerimeterX bypass | +10 credits |

The one genuinely fair thing here: **you're only charged for successful requests** — specifically 200 and 404 status codes. If ScraperAPI fails to fetch the page, you don't pay. Before running a large job, you can also use the Domain Multiplier tool in the dashboard to look up the exact credit cost for any specific URL, which takes the guesswork out of budgeting.

---

## **Full ScraperAPI Plan Comparison**

All plans include JavaScript rendering, premium proxy rotation, CAPTCHA bypass, custom headers, session support, automatic retries, and unlimited bandwidth. The differences come down to volume, concurrency, and geotargeting scope.

| Plan | Monthly Price | Annual Price | Credits/Month | Concurrent Threads | Geotargeting | Get Started |
|---|---|---|---|---|---|---|
| **Free** | $0 | — | 1,000/mo + 5,000 trial | 5 | — |  [Start Free Trial](https://www.scraperapi.com/?fp_ref=coupons) |
| **Hobby** | $49/mo | $44.10/mo | 100,000 | 20 | US & EU only |  [Get Hobby Plan](https://www.scraperapi.com/?fp_ref=coupons#pricing) |
| **Startup** | $149/mo | $134.10/mo | 1,000,000 | 50 | US & EU only |  [Get Startup Plan](https://www.scraperapi.com/?fp_ref=coupons#pricing) |
| **Business** | $299/mo | $269.10/mo | 3,000,000 | 100 | Global |  [Get Business Plan](https://www.scraperapi.com/?fp_ref=coupons#pricing) |
| **Scaling** ⭐ | $475/mo | $427.50/mo | 5,000,000 | 200 | Global |  [Get Scaling Plan](https://www.scraperapi.com/?fp_ref=coupons#pricing) |
| **Professional** | $975/mo | $877.50/mo | 10,500,000 | 300 | Global |  [Get Professional Plan](https://www.scraperapi.com/?fp_ref=coupons#pricing) |
| **Advanced** | $1,975/mo | $1,777.50/mo | 21,500,000 | 500 | Global |  [Get Advanced Plan](https://www.scraperapi.com/?fp_ref=coupons#pricing) |
| **Enterprise** | Custom | Custom | 22M+ | 500+ | Global |  [Contact Sales](https://www.scraperapi.com/?fp_ref=coupons) |

A few things worth knowing that aren't obvious at first glance:

- **Geotargeting is gated.** Hobby and Startup are limited to US and EU proxies. If your project needs country-level targeting beyond those regions, Business is the minimum entry point.
- **Pay-as-you-go overflow** only kicks in on Scaling and above. On the three lower paid tiers, hitting your credit limit mid-month means upgrading or contacting support — there's no automatic overflow.
- **Analytics history** is capped at 30 days on Hobby and Startup; Business and above get unlimited dashboard history.
- **Credits don't roll over.** Your balance resets at renewal, so it's worth sizing your plan to actual monthly usage rather than buying extra headroom you won't use.
- **Annual billing saves 10%** across all plans, applied automatically at checkout — no code needed.

---

## **Which Plan Makes Sense for Your JavaScript Scraping Project?**

The honest answer is: it depends on your targets and how aggressively you're using rendering. Here's a practical guide.

**Start with the free trial.** New accounts get 1,000 permanent free credits (5 concurrent connections), plus a 7-day trial that boosts you to 5,000 credits with no credit card required. This is genuinely useful for running test requests against your actual target URLs — not a toy example — before committing to anything. Watch the `sa-credit-cost` response header on your test requests; it tells you the exact credit cost for each call.

**Pick Hobby ($49/mo) if** you're working on a side project, personal dashboard, or prototype. If your targets are mostly plain pages without heavy anti-bot protection, 100,000 credits stretches pretty far. If you're hitting rendered pages, run the math first — 100K credits ÷ 11 credits/request = ~9,090 rendered page scrapes per month.

**Pick Startup ($149/mo) if** you're building something with consistent, moderate volume — a small SaaS, an agency running client jobs, or a product that scrapes a few major sites regularly. The jump from 100K to 1M credits is significant, though you're still US/EU only for geotargeting.

**Pick Business ($299/mo) if** you need global geotargeting, unlimited analytics history, or you're running production infrastructure with real dependencies. The 100 concurrent threads also start to matter for parallelized jobs.

**Scaling and above** is for when you've stopped asking "which plan?" and started asking "how do we keep this predictable at volume?" The pay-as-you-go overflow and priority support at Professional level mean you're never hard-stopped mid-project.

👉 [Compare all plans and start your free trial here](https://www.scraperapi.com/?fp_ref=coupons)

---

## **Integrating JavaScript Rendering with Puppeteer**

There's a question that comes up every time someone discovers the `render=true` parameter: if ScraperAPI can handle JavaScript rendering, why would I still use Puppeteer?

The answer is interaction. ScraperAPI's rendering runs the JavaScript necessary for the page to load and stabilize. What it can't do is click buttons, scroll down to trigger lazy loading, fill out forms, or handle multi-step flows. If your scraping target requires any of those actions before the data appears, you need a headless browser you can control — and you can route that browser through ScraperAPI's proxy infrastructure to still get the IP rotation and anti-bot handling.

Here's how to set up Puppeteer with ScraperAPI as the proxy:

javascript
const puppeteer = require('puppeteer');
const cheerio = require('cheerio');

const PROXY_SERVER = 'proxy-server.scraperapi.com';
const PROXY_PORT = '8001';
const PROXY_USERNAME = 'scraperapi';
const PROXY_PASSWORD = 'YOUR_API_KEY';

const scrapedData = [];

(async () => {
  const browser = await puppeteer.launch({
    ignoreHTTPSErrors: true,
    args: [`--proxy-server=http://${PROXY_SERVER}:${PROXY_PORT}`]
  });

  const page = await browser.newPage();
  await page.authenticate({
    username: PROXY_USERNAME,
    password: PROXY_PASSWORD,
  });

  try {
    await page.goto('https://your-target-site.com', { timeout: 180000 });
    
    // Interact with the page here — click, scroll, fill forms
    await page.click('.load-more-button');
    await page.waitForSelector('.product-item');
    
    const bodyHTML = await page.evaluate(() => document.body.innerHTML);
    const $ = cheerio.load(bodyHTML);
    
    $('.product-item').each((index, element) => {
      scrapedData.push({
        title: $(element).find('.product-title').text().trim(),
        price: $(element).find('.product-price').text().trim()
      });
    });

  } catch (err) {
    console.error(err);
  }

  await browser.close();
  console.log(scrapedData);
})();


The same proxy setup works with Playwright, which many developers prefer for its cross-browser support and more modern API:

javascript
const { chromium } = require('playwright');

(async () => {
  const browser = await chromium.launch({
    args: [
      '--proxy-server=http://proxy-server.scraperapi.com:8001',
      '--ignore-certificate-errors'
    ]
  });

  const context = await browser.newContext({
    httpCredentials: {
      username: 'scraperapi',
      password: 'YOUR_API_KEY'
    }
  });

  const page = await context.newPage();
  await page.goto('https://your-target-site.com');
  
  const content = await page.content();
  // parse content with Cheerio...
  
  await browser.close();
})();


When you route Puppeteer or Playwright through ScraperAPI this way, every request goes through rotating residential proxies automatically. You get the interaction capability of a full browser with ScraperAPI handling the IP reputation and anti-bot layer.

---

## **Using the Async Endpoint for High-Volume JavaScript Scraping**

For batch jobs — scraping hundreds or thousands of pages — the synchronous API endpoint has a ceiling. Waiting for a JavaScript-rendered response can take several seconds per request, and managing concurrency in your application code adds complexity.

ScraperAPI's async endpoint is built for this. You post jobs to `https://async.scraperapi.com/jobs`, and they're processed in parallel by ScraperAPI's infrastructure. You poll for results or receive them at a webhook.

javascript
import axios from 'axios';

const response = await axios({
  method: 'POST',
  url: 'https://async.scraperapi.com/jobs',
  headers: { 'Content-Type': 'application/json' },
  data: {
    apiKey: 'YOUR_API_KEY',
    url: 'https://example.com/dynamic-page',
    apiParams: {
      render: 'true'
    }
  }
});

console.log(response.data); // includes a job ID and status URL


The async endpoint is particularly valuable when scraping JavaScript-heavy sites at scale, because it decouples your job submission from the rendering time and lets ScraperAPI's thread pool do the concurrency management for you.

---

## **What Happens When JavaScript Rendering Isn't Enough**

Most dynamic pages respond fine to `render=true`. But some sites use advanced anti-bot protection — Cloudflare's managed challenge, Datadome, PerimeterX — that kicks in even during rendering and blocks the headless browser before the content loads.

ScraperAPI handles these with dedicated bypass mechanisms, triggered automatically when needed. When a bypass is applied, it adds 10 credits to the request cost (on top of the domain multiplier and the rendering cost). You can check which protection a site uses before running jobs using the API Playground in the ScraperAPI dashboard.

For the hardest targets — sites that specifically fingerprint and block datacenter IPs even with JavaScript rendering — the `ultra_premium=true` parameter routes the request through residential proxies with stronger browser fingerprint spoofing. At 30 extra credits per request (75 with rendering combined), it's the nuclear option, reserved for sites where nothing else works.

---

## **What Developers Actually Say**

Across Trustpilot and G2, ScraperAPI consistently sits around 4.4–4.5 out of 5. The recurring praise is consistent: clean documentation, straightforward integration (most developers describe dropping it into existing code as a proxy replacement in under an hour), and responsive support. The most common frustration is the credit multiplier system being less intuitive than the headline plan numbers suggest — specifically, developers running into Amazon or Google scrapes and being surprised when credits disappear faster than expected.

The fix is simple: run a few test requests against your actual targets during the free trial, watch the `sa-credit-cost` header, and do the monthly math before picking a plan. The service itself is reliable; the main way to be surprised is skipping that step.

---

## **Quick Reference: ScraperAPI JavaScript Rendering Cheat Sheet**

| Parameter | What It Does | Extra Credit Cost |
|---|---|---|
| `render=true` | Enables full JS rendering via headless Chromium | +10/request |
| `wait_for_selector=CSS` | Waits for a DOM element before returning response | None |
| `premium=true` | Routes through premium residential proxies | +10/request |
| `ultra_premium=true` | Higher-trust residential proxies, stronger fingerprinting | +30/request |
| `screenshot=true` | Returns a screenshot of the rendered page | +10/request |
| `country_code=us` | Sets geolocation for the request | None |
| `session_number=123` | Maintains session across requests | None |
| `autoparse=true` | Returns structured JSON for supported domains | None |

Everything in the "None" column is included at no extra cost — geotargeting, session management, output format selection, and selector waiting are all free additions on top of whatever your base request costs.

---

## **Getting Started**

The fastest path is the free trial: sign up, grab your API key from the dashboard, and swap it into the code examples above. The 5,000 trial credits give you enough to test against real targets and actually understand your credit consumption before picking a plan.

If you already know your target sites and can estimate a monthly request volume, the credit math in this guide should give you a solid starting point for which tier makes sense. When in doubt, start on Hobby or Startup and monitor usage in the dashboard — upgrading is instant and prorated.

👉 [Start your free ScraperAPI trial — no credit card required](https://www.scraperapi.com/?fp_ref=coupons)
