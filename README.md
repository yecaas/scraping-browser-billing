# Scraping Browser Buying Guide: Do You Really Need One, How Per-GB and Per-Hour Billing Differ, and Where ScraperAPI Fits (Free Tier, Credit Costs, Plan Picks)

Most teams shopping for a scraping browser are about to pay for something they don't need. Before renting a hosted Chrome, open the Network tab on the target site and look for a JSON endpoint behind the page; a fair share of "JavaScript-heavy" sites serve their content that way, and a plain HTTP request gets it without a browser anywhere in the loop. When a browser genuinely is the only option, the invoice shape depends on the billing unit, and per GB, per hour and per credit produce very different monthly bills.

ScraperAPI doesn't sell a hosted cloud browser. It sells a scraping API that renders JavaScript, rotates proxies and gets through bot-protected sites, priced per API credit. The free plan gives you 1,000 credits a month at 5 concurrent connections, new accounts get 5,000 credits for the first 7 days, and paid plans carry a 7-day no-questions-asked refund.

> **Quick pick:** start on the free tier and measure how many credits your real targets burn before paying for anything. The free plan's 1,000 credits and 5 concurrent connections are enough to run your hardest URLs a few hundred times and see whether the API route covers them.
> 👉 [Start on ScraperAPI's free tier](https://www.scraperapi.com/?fp_ref=coupons)

## The full ScraperAPI plan lineup

ScraperAPI runs a free plan plus seven paid tiers. The free plan's numbers are public: 1,000 API credits per month, a maximum of 5 concurrent connections, and 5,000 credits during your first 7 days for larger-scale testing. Paid tiers raise the credit allowance and the concurrency ceiling; the current figure for each is on the pricing page.

What separates the tiers in day-to-day use is what happens when your credits run out mid-month.

| Plan | Auto-upgrade to next tier | Pay-as-you-go with monthly cap | Price | Sign up |
| --- | --- | --- | --- | --- |
| Free | No | No | $0 (1,000 credits/mo, 5 concurrent) | [Start free](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | Yes, opt-in | No | Set on the pricing page | [Choose Hobby](https://www.scraperapi.com/?fp_ref=coupons) |
| Startup | Yes, opt-in | No | Set on the pricing page | [Choose Startup](https://www.scraperapi.com/?fp_ref=coupons) |
| Business | Yes, opt-in | No | Set on the pricing page | [Choose Business](https://www.scraperapi.com/?fp_ref=coupons) |
| Scaling | Yes, opt-in | Yes | Set on the pricing page | [Choose Scaling](https://www.scraperapi.com/?fp_ref=coupons) |
| Professional | Yes, opt-in | Yes | Set on the pricing page | [Choose Professional](https://www.scraperapi.com/?fp_ref=coupons) |
| Advanced | No | Yes, limits changed via sales | Set on the pricing page | [Talk to sales about Advanced](https://www.scraperapi.com/?fp_ref=coupons) |
| Enterprise | No | Yes, limits changed via sales | Custom | [Talk to sales about Enterprise](https://www.scraperapi.com/?fp_ref=coupons) |

Two details from the docs matter more than the tier names. Without auto-upgrade or pay-as-you-go, your scraping simply pauses until credits renew. Annual plans hand you the whole year's credits upfront and let you spend them on any schedule, so a project that runs hot for six weeks and quiet for ten months doesn't waste an annual allowance.

## Do you need a scraping browser at all?

No, in a surprising number of cases. The test isn't "does the page use JavaScript" but "can I reach the data with a plain request", and those are different questions.

Scrapy's own developer-tools documentation lays out the trap clearly. Chrome DevTools shows you a live DOM after JavaScript has run and after the browser has cleaned up the markup. Firefox is known for inserting `<tbody>` elements into tables. Scrapy doesn't modify the HTML it receives, so an XPath copied from the inspector with `<tbody>` in it returns nothing, and you conclude the site needs a browser when the real problem was your selector.

Three checks, in order:

1. **Look for the JSON endpoint.** On a demo site that loads quotes as you scroll, the page body doesn't contain the quotes at all; they arrive from a separate request that returns `application/json`. Parse the response as JSON and follow the `has_next` flag for pagination. No browser.
2. **Disable JavaScript and re-inspect.** With JS off in DevTools settings, the DOM you see is much closer to what your scraper receives. Build XPaths from relative selectors (`id`, `class`, `contains(@href, 'image')`) rather than full absolute paths, and drop `<tbody>` unless you know why it's there.
3. **Reproduce the real request.** If the endpoint needs specific headers or cookies, right-click the request in the Network panel, copy it as cURL, and convert it with Scrapy's `Request.from_curl()` or `curl_to_request_kwargs()`. The latter ignores unknown cURL options by default and emits a warning instead of failing.

You need a browser when all three fail: the data is assembled only after client-side execution, the flow requires clicks, scrolling or form submissions, or auth depends on JavaScript running. Bright Data's documentation draws the same line, saying its Browser API is for multi-step flows, form submission and JS-heavy sites, while its README FAQ notes that a plain proxy is enough when you don't need rendering or interaction.

## What a scraping browser costs: per GB, per hour, per credit

Three products in this category, three billing units. The unit decides which kind of site makes your bill explode.

| Provider | Product | Billing unit | Published rates |
| --- | --- | --- | --- |
| Bright Data | Browser API (still labelled Scraping Browser in its public GitHub README) | GB of traffic | $8.40/GB pay-as-you-go, $7.14/GB Growth, $6.30/GB Business, custom Enterprise |
| Evomi | Scraping Browser | Hours of browsing or credits | $24.99/mo for 125 hours or 60,000 credits; $129.99/mo for 729 hours or 350,000 credits; $249.99/mo for 1,562 hours or 750,000 credits |
| ScraperAPI | Scraping API with JavaScript rendering | API credits | 1 credit per standard page; 5 for Amazon; 25 for Google and Bing; 30 for LinkedIn; +10 per request when Cloudflare, Datadome or PerimeterX is bypassed |

Read those units against your own target list before comparing headline numbers.

Per-GB billing charges for bytes, so image-heavy category pages, video portals and endless-scroll feeds inflate the bill even when you only wanted a price field. Per-hour billing charges for wall-clock time, so a session that sits idle waiting on a queue, or a retry loop that sleeps, costs the same as one doing work. Per-credit billing with per-domain multipliers makes cost predictable per URL: a thousand Amazon product pages is 5,000 credits, and you can work out the monthly total on paper. The multipliers also mean credits get expensive on search engine results and LinkedIn, where a thousand requests can cost 25,000 to 30,000 credits.

Bright Data's figures come from its public GitHub README, which carries no date, and Bright Data's own docs now call the product Browser API rather than Scraping Browser, so treat those rates as indicative rather than as its live pricing page. Evomi's prices include asterisks whose footnotes weren't readable in the page text, so don't assume they cover tax or annual discounts.

## ScraperAPI's credit math, concurrency and hard stops

Credits are the whole cost model, so learn the multipliers before you size a plan. A standard page is 1 credit. Amazon is 5. Google and Bing, including subdomains, are 25. LinkedIn is 30. Sites behind Cloudflare, Datadome or PerimeterX add 10 credits per request when ScraperAPI bypasses them. Two tools keep this from becoming guesswork: the Domain Cost Estimator in the dashboard prices any URL before you send traffic at it, and the `max_cost` parameter caps what a single request is allowed to spend.

Concurrency behaves in a way worth knowing in advance. Exceed your plan's limit and you get a 429 telling you to bring concurrency back within range. There's no surcharge for the rejected request, and no queue: the request is turned away rather than held. If your scraper fires 20 parallel workers on a 5-connection plan, you'll see errors that look like blocking but are actually your own parallelism.

For hard-stop avoidance, the lower tiers lean on auto-upgrade, which moves you to the next plan when credits run out, usually at a better price per credit. Scaling, Professional, Advanced and Enterprise can instead switch on pay-as-you-go at a fixed per-credit rate, bounded by a monthly spending cap you set. The cap is a limit, not a prepayment, and your bill won't pass it unless you change the setting.

That combination is the reason to check the pricing page against your own credit estimate rather than picking a tier by feel: at 25 credits per search results page, a plan that looks generous for e-commerce is thin for SEO monitoring.
👉 [Check the current per-credit pricing before committing to a volume](https://www.scraperapi.com/?fp_ref=coupons)

## Using it with Puppeteer and Playwright

Your existing scripts mostly survive, but the two frameworks need different wiring, and getting this wrong wastes an afternoon.

Playwright works by pointing the page at the ScraperAPI endpoint, which wraps the target URL and carries your key as a query parameter, with `render=true` set for JS-heavy targets. Send Playwright's proxy settings at the proxy port instead and it fails: Playwright's proxy configuration expects Basic Auth or IP authentication, while ScraperAPI passes the key as a query string. ScraperAPI's Playwright guide states this plainly and recommends the endpoint method.

Puppeteer goes the other way. Its guide recommends the proxy port, because routing `page.goto` at the API endpoint breaks relative assets and links: Puppeteer resolves `/static/main.css` against the API host instead of the site's host, and you end up with only the first HTML response, which defeats the point of a headless browser.

Both paths give you proxy rotation, geo-targeting and rendering in one request, so the API covers the browser cases most teams actually have. What it doesn't give you is a persistent remote browser session you can drive interactively, which is the thing Bright Data's Chrome DevTools integration and CDP-based CAPTCHA calls are for. If your workflow depends on inspecting a live session mid-run, that's a hosted-browser feature, not an API feature.
👉 [Run the Playwright setup against your own targets with the free credits](https://www.scraperapi.com/?fp_ref=coupons)

## Trials, refunds and cancellation

New accounts get 5,000 API credits for the first 7 days, which is enough to run a real workload rather than a smoke test. After that the free plan continues at 1,000 credits a month with 5 concurrent connections, so you can keep a low-volume job running indefinitely without paying.

Paid plans carry a 7-day no-questions-asked refund: contact support and they process it. You can cancel from the dashboard at any time and you won't be charged for cancelling. The refund window and the trial window are both 7 days, so the sensible sequence is to sign up, test your ugliest target inside that week, and decide before it closes rather than after.
👉 [Open an account and test your hardest target inside the 7-day window](https://www.scraperapi.com/?fp_ref=coupons)

## FAQ

**Can I keep my existing Puppeteer or Playwright code?**
Yes. Playwright goes through the endpoint URL with `render=true`; Puppeteer goes through the proxy port. Keep your key in an environment variable rather than inline in the script.

**Does it handle CAPTCHAs?**
Sites behind Cloudflare, Datadome or PerimeterX add 10 credits per request when ScraperAPI bypasses them, and the bypass is part of the request rather than a separate solver you integrate.

**What happens when I hit my concurrency limit?**
A 429 response, no extra charge, no queue. Lower your worker count or move up a tier.

**What if I run out of credits before renewal?**
Scraping pauses unless you've turned on auto-upgrade (Hobby through Professional) or pay-as-you-go (Scaling and above). Annual plans can also be renewed early for a fresh allocation.

## Which one to pick

The decision comes down to what your targets actually need.

If the data sits behind a JSON endpoint or plain HTML, you need neither a scraping browser nor an expensive plan: a free ScraperAPI account at 1,000 credits a month will likely cover it, and the cost is zero until you outgrow it.

If the pages need JavaScript execution and the sites fight back with Cloudflare or Datadome, the API with `render=true` covers it at 1 credit plus 10 for the bypass. Estimate your monthly page count, multiply by the per-domain credit cost, and pick the smallest tier whose allowance clears it with room for retries.

If you need to drive a live browser session, click through multi-step flows that depend on real user timing, or debug a session interactively through Chrome DevTools, that's hosted-browser territory, and Bright Data's Browser API or Evomi's per-hour plans are built for it. Budget by the unit that matches your traffic: bytes if your pages are heavy with media, hours if your sessions idle.

Two things to confirm before you pay anywhere: your own credit-per-1,000-pages number, measured on real targets rather than a demo site, and whether your hardest site needs interaction at all. Get those two answers and the plan choice stops being a guess.
👉 [Price your workload on ScraperAPI's free tier first](https://www.scraperapi.com/?fp_ref=coupons)
