# Web Scraping with LLaMA 3

[![Bright Data Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/)

This guide explains how to use LLaMA 3 to convert big HTML to structured, clean, and usable JSON:

- [Why Choose LLaMA 3 for Web Scraping](#why-choose-llama-3-for-web-scraping)
- [System Requirements](#system-requirements)
- [Setting Up Ollama](#setting-up-ollama)
- [Selecting the Right LLaMA Model](#selecting-the-right-llama-model)
- [Downloading and Running the Model](#downloading-and-running-the-model)
- [Building an Amazon Scraper Powered by LLMs](#building-an-amazon-scraper-powered-by-llms)
- [Handling Anti-Bot Protection](#handling-anti-bot-protection)
- [Enhancing and Expanding Your Scraper](#enhancing-and-expanding-your-scraper)

## Why Choose LLaMA 3 for Web Scraping

[Meta's LLaMA 3](https://ai.meta.com/blog/meta-llama-3/) (debuted April 2024) is an open-weight LLM series, scaling from 8B to 405B parameters, fitting a broad array of tasks and hardware setups. The 3.1 through 3.3 updates have further enhanced its capabilities.

Traditional scraping techniques—using [XPath or CSS](https://brightdata.com/blog/web-data/xpath-vs-css-selectors)—are vulnerable to website layout changes. LLaMA 3, however, understands content in a human-like manner, offering intelligent, resilient scraping that stays reliable through updates.

This makes it ideal for:

- Retail giants like Amazon
- Complex data parsing
- Robust, durable scrapers
- Ensuring sensitive data remains in-house

You can learn more about AI-based web scraping in our [earlier guide](https://brightdata.com/blog/web-data/ai-web-scraping).

## System Requirements

Before starting the [LLM-powered scraping](https://brightdata.com/blog/web-data/web-scraping-with-scrapegraphai) project, ensure you have:

- [Python 3](https://www.python.org/downloads/)
- A basic understanding of Python
- One of the following OS setups:
  - macOS 11 Big Sur or later
  - Linux
  - Windows 10 or newer
- Sufficient machine resources (more details below)

## Setting Up Ollama

Ollama streamlines installing, running, and managing large language models locally.

![Ollama installation page](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/ollama-llm-download-installation-page.png)

To get started:

1. Head to the [Ollama official site](https://ollama.com/)
2. Download the version matching your OS
3. **Important**: During setup, you’ll be asked to run a command—hold off until you pick your model.

## Selecting the Right LLaMA Model

Explore [Ollama’s model library](https://ollama.com/library) to find the right version.

If you're on a standard machine, `llama3.1:8b` is an excellent pick—compact, efficient, needing about 4.9 GB of disk space and 6–8 GB of RAM. It runs fine on most modern laptops.

For more powerful hardware, larger versions like `70B` or `405B` unlock superior reasoning and longer context windows—but they’re hardware-hungry.

## Downloading and Running the Model

Pull the LLaMA 3.1 (8B) model with:

```sh
ollama run llama3.1:8b
```

You’ll get an interactive prompt:

```sh
>>> Send a message (/? for help)
```

Test it:

```sh
>>> who are you?
I am LLaMA, *an AI assistant developed by Meta AI...*
```

Once confirmed, start the Ollama server:

```sh
ollama serve
```

This spins up a local server at `http://127.0.0.1:11434/`. Keep this window open.

Visit it in your browser—you should see the message **“Ollama is running.”**

## Building an Amazon Scraper Powered by LLMs

Let's build a scraper that extracts product details from Amazon—one of the most challenging targets due to its [dynamic content](https://brightdata.com/blog/how-tos/scrape-dynamic-websites-python) and strong anti-bot protections.

![Amazon product page](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/amazon-office-chair-product-page-1.png)

We'll extract:

- Title
- Prices
- Discounts
- Ratings and review counts
- Descriptions and features
- Stock status and [ASINs](https://brightdata.com/blog/web-data/how-to-scrape-amazon-asin)

### Smart Multi-Stage Approach

Our LLaMA-powered scraper follows a smart, multi-stage workflow:

1. **Browser Automation** with Selenium
2. **HTML Extraction** from targeted sections
3. **Markdown Conversion** for leaner input
4. **LLM Processing** to structure data
5. **Result Storage** for further analysis

Here’s a visual breakdown of the workflow:

![Workflow diagram](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/llama-web-scraping-workflow-diagram.png)

We'll be using **Python**, but this can be adapted to other languages like [JavaScript](https://brightdata.com/blog/web-data/best-languages-web-scraping).

### Step 1 – Install Required Libraries

First, install the necessary Python libraries:

```sh
pip install requests selenium webdriver-manager markdownify
```

- `requests` – [The best Python HTTP client](https://brightdata.com/blog/web-data/best-python-http-clients) for sending API calls to the LLM service
- `selenium` – Automates the browser, ideal for JavaScript-heavy websites
- `webdriver-manager` – Automatically downloads and manages the correct ChromeDriver version
- `markdownify` – Converts HTML into Markdown

### Step 2 – Initialize the Headless Browser

Set up a [headless browser](https://brightdata.com/blog/proxy-101/what-is-a-headless-browser) using Selenium:

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from webdriver_manager.chrome import ChromeDriverManager

options = Options()
options.add_argument("--headless")

driver = webdriver.Chrome(
    service=Service(ChromeDriverManager().install()),
    options=options
)
```

### Step 3 – Extract the Product HTML

Amazon product details are rendered dynamically and wrapped inside a `<div id="ppd">` container. The script will wait for this section to load, then extract its HTML:

```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

wait = WebDriverWait(driver, 15)
product_container = wait.until(
    EC.presence_of_element_located((By.ID, "ppd"))
)

# Extract the full HTML of the product container
page_html = product_container.get_attribute("outerHTML")
```

This approach:

- Waits for JavaScript-rendered content (like prices and ratings)
- Targets only the relevant product section—ignoring headers, footers, and sidebars

_Check out our complete guide on [how to scrape Amazon product data in Python](https://brightdata.com/blog/how-tos/how-to-scrape-amazon)._

### Step 4 – Convert HTML to Markdown

Amazon pages contain deeply nested HTML that is inefficient for LLMs to process. It's best to remove the excess data by converting this HTML to clean Markdown, thus reducing token count and improving comprehension.

When you run the complete script, two files will be generated: `amazon_page.html` and `amazon_page.md`. Try pasting both into the [Token Calculator Tool](https://token-calculator.net/) to compare their token counts.

The HTML contains around **270,000 tokens**:

![token-calculator-html-tokens](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/token-calculator-html-tokens.png)

The Markdown version contains only **~11,000 tokens**:

![token-calculator-markdown-tokens](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/token-calculator-markdown-tokens.png)

This **96% reduction** leads to:

- **Cost efficiency** – Fewer tokens mean lower API or compute costs
- **Faster processing** – Less input data = quicker LLM responses
- **Improved accuracy** – Cleaner, flatter text helps the model extract structured data more precisely

_Read more on [why AI agents prefer Markdown over HTML](https://hackernoon.com/why-are-the-new-ai-agents-choosing-markdown-over-html)._

Here’s how to do the conversion in Python:

```python
from markdownify import markdownify as md

clean_text = md(page_html, heading_style="ATX")
```

### Step 5 – Create the Data Extraction Prompt

A well-structured prompt is critical for getting consistent, clean JSON output from the LLM. Below is a prompt that instructs the model to return **only** valid JSON in a predefined format:

```python
PROMPT = (
    "You are an expert Amazon product data extractor. Your task is to extract product data from the provided content. "
    "Return ONLY valid JSON with EXACTLY the following fields and formats:\n\n"
    "{\n"
    '  "title": "string – the product title",\n'
    '  "price": number – the current price (numerical value only)",\n'
    '  "original_price": number or null – the original price if available,\n'
    '  "discount": number or null – the discount percentage if available,\n'
    '  "rating": number or null – the average rating (0–5 scale),\n'
    '  "review_count": number or null – total number of reviews,\n'
    '  "description": "string – main product description",\n'
    '  "features": ["string"] – list of bullet point features,\n'
    '  "availability": "string – stock status",\n'
    '  "asin": "string – 10-character Amazon ID"\n'
    "}\n\n"
    "Return ONLY the JSON without any additional text."
)
```

### Step 6 – Call the LLM API

Send the Markdown text to your LLaMA instance via its HTTP API:

```python
import requests
import json

response = requests.post(
    "<http://localhost:11434/api/generate>",
    json={
        "model": "llama3.1:8b",
        "prompt": f"{PROMPT}\n\n{clean_text}",
        "stream": False,
        "format": "json",
        "options": {
            "temperature": 0.1,
            "num_ctx": 12000,
        },
    },
    timeout=250,
)

raw_output = response.json()["response"].strip()
product_data = json.loads(raw_output)
```

Here is what each option does:

- `temperature` – Set to 0.1 for deterministic output (ideal for JSON formatting)
- `num_ctx` – Defines the maximum context length. 12,000 tokens are sufficient for most Amazon product pages
- `stream` – When `False`, the API returns the full response after processing
- `format` – Specifies the output format (JSON)
- `model` – Indicates which LLaMA version to use

Sure! Here’s a cleaner rewrite:

The converted Markdown often has about 11,000 tokens, so set the context window (`num_ctx`) accordingly. Increasing it supports longer inputs but uses more RAM and slows processing. Only raise it if needed or if you have the resources.

### Step 7 – Save the Results

Finally, save the structured product data to a JSON file:

```python
with open("product_data.json", "w", encoding="utf-8") as f:
    json.dump(product_data, f, indent=2, ensure_ascii=False)
```

### Step 8: Execute the Script

To run your scraper, provide an Amazon product URL and call your scraping function:

```python
if __name__ == "__main__":
    url = "<https://www.amazon.com/Black-Office-Chair-Computer-Adjustable/dp/B00FS3VJAO>"

    # Call your function to scrape and extract product data
    scrape_amazon_product(url)
```

### Step 9 – Full Code Example

Below is the complete Python script:

```python
import json
import logging
import time
from typing import Final, Optional, Dict, Any

import requests
from markdownify import markdownify as html_to_md
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait
from webdriver_manager.chrome import ChromeDriverManager

# Configuration constants
LLM_API_CONFIG: Final[Dict[str, Any]] = {
    "endpoint": "<http://localhost:11434/api/generate>",
    "model": "llama3.1:8b",
    "temperature": 0.1,
    "context_window": 12000,
    "stream": False,
    "timeout_seconds": 220,
}

DEFAULT_PRODUCT_DATA: Final[Dict[str, Any]] = {
    "title": "",
    "price": 0.0,
    "original_price": None,
    "discount": None,
    "rating": None,
    "review_count": None,
    "description": "",
    "features": [],
    "availability": "",
    "asin": "",
}

PRODUCT_DATA_EXTRACTION_PROMPT: Final[str] = (
    "You are an expert Amazon product data extractor. Your task is to extract product data from the provided content. "
    "Return ONLY valid JSON with EXACTLY the following fields and formats:\n\n"
    "{\n"
    '  "title": "string - the product title",\n'
    '  "price": number - the current price (numerical value only),\n'
    '  "original_price": number or null - the original price if available,\n'
    '  "discount": number or null - the discount percentage if available,\n'
    '  "rating": number or null - the average rating (0-5 scale),\n'
    '  "review_count": number or null - total number of reviews,\n'
    '  "description": "string - main product description",\n'
    '  "features": ["string"] - list of bullet point features,\n'
    '  "availability": "string - stock status",\n'
    '  "asin": "string - 10-character Amazon ID"\n'
    "}\n\n"
    "Return ONLY the JSON without any additional text."
)

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
    handlers=[logging.StreamHandler()],
)

def initialize_web_driver(headless: bool = True) -> webdriver.Chrome:
    """Initialize and return a configured Chrome WebDriver instance."""
    options = Options()
    if headless:
        options.add_argument("--headless=new")

    service = Service(ChromeDriverManager().install())
    return webdriver.Chrome(service=service, options=options)

def fetch_product_container_html(product_url: str) -> Optional[str]:
    """Retrieve the HTML content of the Amazon product details container."""
    driver = initialize_web_driver()
    try:
        logging.info(f"Accessing product page: {product_url}")
        driver.set_page_load_timeout(15)
        driver.get(product_url)

        # Wait for the product container to appear
        container = WebDriverWait(driver, 5).until(
            EC.presence_of_element_located((By.ID, "ppd"))
        )
        return container.get_attribute("outerHTML")
    except Exception as e:
        logging.error(f"Error retrieving product details: {str(e)}")
        return None
    finally:
        driver.quit()

def extract_product_data_via_llm(markdown_content: str) -> Optional[Dict[str, Any]]:
    """Extract structured product data from markdown text using LLM API."""
    try:
        logging.info("Extracting product data via LLM API...")
        response = requests.post(
            LLM_API_CONFIG["endpoint"],
            json={
                "model": LLM_API_CONFIG["model"],
                "prompt": f"{PRODUCT_DATA_EXTRACTION_PROMPT}\n\n{markdown_content}",
                "format": "json",
                "stream": LLM_API_CONFIG["stream"],
                "options": {
                    "temperature": LLM_API_CONFIG["temperature"],
                    "num_ctx": LLM_API_CONFIG["context_window"],
                },
            },
            timeout=LLM_API_CONFIG["timeout_seconds"],
        )
        response.raise_for_status()

        raw_output = response.json()["response"].strip()
        # Clean JSON output if it's wrapped in markdown code blocks
        if raw_output.startswith(("```json", "```")):
            raw_output = raw_output.split("```")[1].strip()
            if raw_output.startswith("json"):
                raw_output = raw_output[4:].strip()

        return json.loads(raw_output)

    except requests.exceptions.RequestException as e:
        logging.error(f"LLM API request failed: {str(e)}")
        return None
    except json.JSONDecodeError as e:
        logging.error(f"Failed to parse LLM response: {str(e)}")
        return None
    except Exception as e:
        logging.error(f"Unexpected error during data extraction: {str(e)}")
        return None

def scrape_amazon_product(
    product_url: str, output_file: str = "product_data.json"
) -> None:
    """Scrape an Amazon product page and save extracted data along with HTML and Markdown to files."""
    start_time = time.time()
    logging.info(f"Starting scrape for: {product_url}")

    # Step 1: Fetch product page HTML
    product_html = fetch_product_container_html(product_url)
    if not product_html:
        logging.error("Failed to retrieve product page content")
        return

    # Optional: save HTML for debugging
    with open("amazon_product.html", "w", encoding="utf-8") as f:
        f.write(product_html)

    # Step 2: Convert HTML to Markdown
    product_markdown = html_to_md(product_html)

    # Optional: save Markdown for debugging
    with open("amazon_product.md", "w", encoding="utf-8") as f:
        f.write(product_markdown)

    # Step 3: Extract structured data via LLM
    product_data = (
        extract_product_data_via_llm(product_markdown) or DEFAULT_PRODUCT_DATA.copy()
    )

    # Step 4: Save JSON results
    try:
        with open(output_file, "w", encoding="utf-8") as json_file:
            json.dump(product_data, json_file, indent=2, ensure_ascii=False)
        logging.info(f"Successfully saved product data to {output_file}")
    except IOError as e:
        logging.error(f"Failed to save JSON results: {str(e)}")

    elapsed_time = time.time() - start_time
    logging.info(f"Completed in {elapsed_time:.2f} seconds")

if __name__ == "__main__":
    # Example usage
    test_url = (
        "<https://www.amazon.com/Black-Office-Chair-Computer-Adjustable/dp/B00FS3VJAO>"
    )
    scrape_amazon_product(test_url)
```

The script saves the extracted product data to a file named `product_data.json`. The output will be similar to this:

```json
{
    "title": "Home Office Chair Ergonomic Desk Chair Mesh Computer Chair with Lumbar Support Armrest Executive Rolling Swivel Adjustable Mid Back Task Chair for Women Adults, Black",
    "price": 36.98,
    "original_price": 41.46,
    "discount": 11,
    "rating": 4.3,
    "review_count": 58112,
    "description": 'Office chair comes with all hardware and tools, and is easy to assemble in about 10–15 minutes. The high-density sponge cushion offers flexibility and comfort, while the mid-back design and rectangular lumbar support enhance ergonomics. All components are BIFMA certified, supporting up to 250 lbs. The chair includes armrests and an adjustable seat height (17.1"–20.3"). Its ergonomic design ensures a perfect fit for long-term use.',
    "features": [
        "100% mesh material",
        "Quick and easy assembly",
        "High-density comfort seat",
        "BIFMA certified quality",
        "Includes armrests",
        "Ergonomic patented design",
    ],
    "availability": "In Stock",
    "asin": "B00FS3VJAO",
}
```

## Handling Anti-Bot Protection

When running the above [web scraping bot](https://brightdata.com/blog/how-tos/what-is-a-scraping-bot), you’ll likely encounter Amazon’s anti-bot measures, such as CAPTCHA challenges:

![amazon-captcha-anti-bot-challenge](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/amazon-captcha-anti-bot-challenge.png)

While LLaMA 3 handles parsing beautifully, bypassing site protections is still tricky. [Bright Data’s Scraping Browser](https://brightdata.com/products/scraping-browser) provides a powerful solution.

### Why Use Bright Data Scraping Browser

The [Bright Data Scraping Browser](https://brightdata.com/products/scraping-browser) is a headless, cloud-based browser designed for scaling modern web scraping projects. It features built-in proxy infrastructure and advanced unblocking capabilities, and is part of the [Bright Data Unlocker scraping suite](https://docs.brightdata.com/scraping-automation/introduction).

Some of the reasons to choose it are:

- Reliable TLS fingerprints and stealth evasion
- Built-in IP rotation via a [150M+ residential IP proxy network](https://brightdata.com/proxy-types/residential-proxies)
- Automatic CAPTCHA solving
- Reduce infrastructure costs – no cloud setup or maintenance needed
- Native support for Playwright, Puppeteer, and Selenium
- Unlimited scalability for high-volume extraction

Best of all, you can integrate it into your workflow with just a few lines of code.

### Setting Up Scraping Browser

To get started with Scraping Browser:

[Create a Bright Data account](https://brightdata.com/) (new users receive a $5 credit after adding a payment method) and in your dashboard, go to **Proxies & Scraping** and click **Get started**.

![brightdata-scraping-solutions-dashboard](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/brightdata-scraping-solutions-dashboard.png)

Create a new zone (e.g., _test\_browser_) and enable features like _Premium domains_ and [CAPTCHA solver](https://brightdata.com/products/web-unlocker/captcha-solver).

![brightdata-create-scraping-browser-zone](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/brightdata-create-scraping-browser-zone.png)

Next, copy the Selenium URL from your dashboard.

![brightdata-selenium-connection-credentials](https://github.com/luminati-io/llama-3-web-scraping/blob/main/images/brightdata-selenium-connection-credentials.png)

### Modifying Your Code for Scraping Browser

Update your `initialize_web_driver` function to connect via the Scraping Browser:

```python
from selenium.webdriver import Remote
from selenium.webdriver.chrome.options import Options as ChromeOptions
from selenium.webdriver.chromium.remote_connection import ChromiumRemoteConnection

SBR_WEBDRIVER = "<https://username:password@host>:port"

def initialize_web_driver():
    options = ChromeOptions()
    sbr_connection = ChromiumRemoteConnection(SBR_WEBDRIVER, "goog", "chrome")
    driver = Remote(sbr_connection, options=options)
    return driver
```

Your scraper now routes through Bright Data’s infrastructure and handles Amazon and other anti-bot systems with ease.

## Enhancing and Expanding Your Scraper

Here are some of the future improvements you can add:

- Make URL and prompt arguments configurable
- Load credentials securely from a `.env` file
- Support multi-page scraping and [pagination handling](https://brightdata.com/blog/web-data/pagination-web-scraping)
- Expand scraping to [other marketplaces](https://brightdata.com/blog/how-tos/ecommerce-web-scraping-guide)
- Extract data from Google services:
  - [Google Flights](https://github.com/luminati-io/google-flights-api)
  - [Google Search](https://github.com/luminati-io/google-search-api)
  - [Google Trends](https://github.com/luminati-io/google-trends-api)
- Explore different LLM integrations like:
  - [Gemini](https://brightdata.com/blog/web-data/web-scraping-with-gemini)
  - [Perplexity](https://brightdata.com/blog/web-data/web-scraping-with-perplexity)
  - [Crawl4AI and DeepSeek](https://brightdata.com/blog/web-data/crawl4ai-and-deepseek-web-scraping)

## Conclusion

This guide sets you up to create reliable, intelligent scrapers with LLaMA 3.

For ultimate scraping success, combine LLaMA’s reasoning with the infrastructure of tools like [Bright Data’s Scraping Browser](https://brightdata.com/products/scraping-browser).

Ready to level up? [Try Bright Data’s full scraping suite](https://brightdata.com/) for free!
