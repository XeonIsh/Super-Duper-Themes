Google Ads Performance Max (PMax) Search Themes Report Script

This Google Ads Script automates the extraction of "Search Themes" (also known as "Search Categories" or "Insights") from your Performance Max campaigns and exports this data to a Google Sheet. It provides valuable insights into the types of queries driving traffic to your PMax campaigns, along with key performance metrics.
🚀 Purpose

Performance Max campaigns are highly automated, and detailed search term data can be limited in the standard Google Ads UI. This script leverages the Google Ads API to retrieve the available "Search Theme" insights, helping advertisers understand broad user intent and campaign performance at a thematic level.
✨ Features

    Extracts "Search Theme" (Category Label) data for your PMax campaigns.

    Provides granular metrics per theme: Clicks, Impressions, and Click-Through Rate (CTR).

    Includes Conversions and Conversion Value directly attributed to each specific Search Theme.

    Includes the Total Campaign Cost for the entire PMax campaign, providing financial context for the themes.

    Allows configuration of a lookback window for data extraction.

    Includes a minimum impressions filter to focus on more significant themes.

    Optionally filters data for specific PMax campaigns by name.

    Automates data export to a specified Google Sheet.

⚠️ Important Limitations

Due to Google Ads API limitations for the campaign_search_term_insight report on Performance Max campaigns:

    Cost Data per Theme is NOT available directly: This script cannot pull cost directly attributed to individual search themes. Therefore, the "Campaign Cost (Total)" column will show the total cost for the entire PMax campaign for the selected date range, repeated for each theme of that campaign.

    Not All Search Terms Exposed: Just like in the Google Ads UI, Google does not expose every single search term. Lower-volume or privacy-sensitive queries are often grouped into an "Other search terms" category or simply not provided individually. This script specifically focuses on the higher-level "Search Themes."

🛠️ How to Use (Installation)

Follow these steps to set up and run the script in your Google Ads account:
1. Create a Google Sheet

    Go to Google Sheets and create a new, blank spreadsheet.

    Copy the entire URL of this new spreadsheet from your browser's address bar. You will need this for the script configuration.

2. Add the Script to Google Ads

    Log in to your Google Ads account.

    Navigate to TOOLS AND SETTINGS (the wrench icon in the top right).

    Under BULK ACTIONS, click on SCRIPTS.

    Click the blue plus (+) button to create a new script.

    Select "New script" or "Blank script".

    Copy the entire script code provided below and paste it into the script editor, replacing any default content.

3. Configure the Script

At the top of the script code, you'll find the // --- Configuration --- section. You must edit these lines:

    const SPREADSHEET_URL = 'YOUR_GOOGLE_SHEET_URL_HERE';

        Replace 'YOUR_GOOGLE_SHEET_URL_HERE' with the Google Sheet URL you copied in Step 1. Ensure the URL is enclosed within the single quotes.

    const LOOKBACK_DAYS = 30;

        Adjust this number to set the date range for your report (e.g., 7 for the last 7 days, 60 for the last 60 days).

    const MIN_IMPRESSIONS = 1;

        This sets a minimum impressions threshold for search themes to be included in the report. Themes with fewer impressions will be excluded. Adjust as needed (e.g., 10, 50).

    const CAMPAIGN_NAME_CONTAINS = '';

        (Optional) If you want to limit the report to specific PMax campaigns, enter a part of their name (e.g., 'Brand PMax'). If left empty (''), the script will process all enabled Performance Max campaigns in the account.

4. Authorize and Run

    Give your script a descriptive name (e.g., "PMax Search Themes Report").

    Click "Authorize" if prompted. You'll need to grant the script permission to access your Google Ads data and your Google Sheets.

    Click "Preview" first to test the script without making actual changes to your sheet. Check the "Logs" section for any errors.

    If the preview is successful, click "Run" to execute the script and populate your Google Sheet.

5. Schedule (Optional)

To keep your report updated automatically:

    Go back to the Scripts page in Google Ads.

    Find your script and click on the Frequency column.

    Set a schedule (e.g., Daily, Weekly) for the script to run.

🤝 Contribution

Feel free to fork this repository, suggest improvements, or submit pull requests.
