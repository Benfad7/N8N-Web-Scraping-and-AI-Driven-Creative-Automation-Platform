
<img width="1508" height="600" alt="image" src="https://github.com/user-attachments/assets/46c254b5-6efe-42b9-80af-23f147ebdaa8" />




Example of the output - https://drive.google.com/file/d/1yxi8DEDkvV2QyENx1UO_DRMCmjRsnhoX/view?usp=sharing
## How it works:
1. Daily Workflow: Ad Scraping & Feature Extraction
This workflow is the system's data-gathering engine. Its primary job is to find ads on ynet.co.il, analyze new ones, and keep a record of when they were seen.

The process begins with a two-stage fetch to efficiently gather ad data. First, it performs a simple, lightweight HTML request to the Ynet homepage to quickly parse and extract the main article links.

Then, for each of these article URLs, it sends a request to the browserless.io API. This service renders the full page in a real browser environment, executing all JavaScript. This step is crucial for revealing dynamically loaded ad content, especially ads served within iframes from known ad networks, which would be invisible in a simple HTML fetch. The rendered HTML is then meticulously scraped for ad images, which are consolidated into a master list for the current run.

It is important to note a key technical constraint of the n8n environment: it does not natively support browser automation libraries like Selenium or Playwright. Consequently, advanced interactions required by the assignment, such as programmatic scrolling to trigger lazy-loaded images or handling complex pagination, could not be implemented directly within this workflow.

Furthermore, the ads returned by the browserless.io API may differ from those seen on a personal computer due to factors like geo-targeting and ad personalization. For a more robust, future-proof solution, the ideal architecture would be to create a custom API (e.g., using Flask in Python). This API would leverage a library like Selenium to gain full control over the browser, enabling it to handle scrolling and reliably extract all relevant iframe data. The n8n workflow would then call this custom API, abstracting away the complexities of the scraping process.

Comparison & Sorting: The system reads the historical ad database from your Google Sheet. It compares the newly scraped ads against this history to sort them into two groups:

New Ads: Images that have never been seen before.

Existing Ads: Images that are already in the database.

Processing Existing Ads: For ads it has seen before, the system simply updates their last_seen timestamp to the current time, keeping a fresh record of their activity.

Processing New Ads (The Image Analysis Pipeline): This is where the deep analysis happens for each new ad.

Metadata Enrichment: The ad is prepared by adding initial metadata, such as a width and heigth, the source URL, and other details.

AI Vision Analysis: It sends the image to the Google Vision API to perform feature extraction. This pulls out key characteristics like the top 3 dominant colors.

AI Character Analysis: It then sends the image context to the Google Gemini API with a specific prompt to identify any characters present.

Data Merging: The results from the Vision API and Gemini API are merged back into the ad's record.

Final Update: Finally, the workflow combines the updated records for existing ads and the newly analyzed records for new ads. This complete list is then written back to the Google Sheet.

2. Weekly Workflow: Trend Analysis & Creative Generation
This workflow acts as the "intelligence" layer. It uses the data collected daily to find patterns and then acts on them.

Data Ingestion: The process starts by reading the entire ad database from the Google Sheet, including all the features extracted by the daily workflow.

Trend Analysis: It analyzes all the collected data to identify high-performing trends. It specifically looks for:

The most common objects appearing in ads.

The most frequently used color palettes - This code doesn't just find the most common raw hex code. It first identifies the most popular color trend by categorizing hexes into general names (like "blue"), and then selects the hex with the median strength from all the colors in that winning group.

The top characters identified by the AI - This step uses an AI model to analyze all the character descriptions, identifying and ranking the top 3 most frequently appearing characters across the collected ads.

AI-Powered Ad Generation: Using the insights from the trend analysis, the workflow instructs an AI image generation model to create three new ad creatives for Guardio. These new ads are designed to reflect the dominant trends in color, objects, and characters.

Output: The newly generated ad images are then uploaded and saved to a Google Drive, ready for use in marketing campaigns.



## setup:

1. Google Cloud Project Setup:

Create a new Google Cloud project to house all your APIs.
Enable billing for the project to use paid services like the Vision and Vertex AI APIs.

2. Enable Necessary APIs:

In the Google Cloud Console, enable the following APIs:
Google Sheets API
Google Drive API
Cloud Vision API

3. Web Scraping Service (Browserless)

To properly scrape modern, JavaScript-heavy websites like Ynet, a headless browser service is required. This project is configured to use browserless.io.

You will need an API key from this service. You may use the one I have provided. No worries about unexpected charges, as I have not provided any billing information for this key.

4. Prepare Data Storage:

Create a new Google Sheet.
Set up the required headers for the ads database.
run_id	run_ts	page_url	image_url	width	height	placement_hint	ad_host	source	first_seen	last_seen	palette_1	palette_2	palette_3	has_face	objects	text	character
