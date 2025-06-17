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
1. Download the Script

    Go to the GitHub repository: Super-Duper-Themes.

    Locate the file named SearchThemesScript.js (or similar, depending on the exact naming in the repository).

    Download or copy the entire content of this script file.

2. Create a Google Sheet

    Go to Google Sheets and create a new, blank spreadsheet.

    Copy the entire URL of this new spreadsheet from your browser's address bar. You will need this for the script configuration.

3. Add the Script to Google Ads

    Log in to your Google Ads account.

    Navigate to TOOLS AND SETTINGS (the wrench icon in the top right).

    Under BULK ACTIONS, click on SCRIPTS.

    Click the blue plus (+) button to create a new script.

    Select "New script" or "Blank script".

    Copy the entire script code provided below and paste it into the script editor, replacing any default content.

4. Configure the Script

At the top of the script code, you'll find the // --- Configuration --- section. You must edit these lines:

    const SPREADSHEET_URL = 'YOUR_GOOGLE_SHEET_URL_HERE';

        Replace 'YOUR_GOOGLE_SHEET_URL_HERE' with the Google Sheet URL you copied in Step 2. Ensure the URL is enclosed within the single quotes.

    const LOOKBACK_DAYS = 30;

        Adjust this number to set the date range for your report (e.g., 7 for the last 7 days, 60 for the last 60 days).

    const MIN_IMPRESSIONS = 1;

        This sets a minimum impressions threshold for search themes to be included in the report. Themes with fewer impressions will be excluded. Adjust as needed (e.g., 10, 50).

    const CAMPAIGN_NAME_CONTAINS = '';

        (Optional) If you want to limit the report to specific PMax campaigns, enter a part of their name (e.g., 'Brand PMax'). If left empty (''), the script will process all enabled Performance Max campaigns in the account.

5. Authorize and Run

    Give your script a descriptive name (e.g., "PMax Search Themes Report").

    Click "Authorize" if prompted. You'll need to grant the script permission to access your Google Ads data and your Google Sheets.

    Click "Preview" first to test the script without making actual changes to your sheet. Check the "Logs" section for any errors.

    If the preview is successful, click "Run" to execute the script and populate your Google Sheet.

6. Schedule (Optional)

To keep your report updated automatically:

    Go back to the Scripts page in Google Ads.

    Find your script and click on the Frequency column.

    Set a schedule (e.g., Daily, Weekly) for the script to run.

💻 Script Code

// --- Configuration ---
const SPREADSHEET_URL = 'YOUR_GOOGLE_SHEET_URL_HERE'; // **REQUIRED**: Replace with the URL of your Google Sheet
const LOOKBACK_DAYS = 30; // Number of days to look back for data
const MIN_IMPRESSIONS = 1; // Minimum impressions for a search theme to be included (adjust as needed)
const CAMPAIGN_NAME_CONTAINS = ''; // Optional: Filter by campaign name (e.g., 'PMax - My Campaign'). Leave empty for all PMax campaigns.

// --- Main Function ---
function main() {
  const spreadsheet = SpreadsheetApp.openByUrl(SPREADSHEET_URL);

  // --- Setup PMax Search Themes Sheet ---
  let searchThemesSheet = spreadsheet.getSheetByName('PMax Search Themes');
  if (!searchThemesSheet) {
    searchThemesSheet = spreadsheet.insertSheet('PMax Search Themes');
  } else {
    searchThemesSheet.clearContents();
  }

  // Headers for the sheet
  // Conversions & Conv. Value are per Theme. Cost is Campaign Total.
  const headers = [
    'Campaign Name',
    'Search Theme (Category Label)',
    'Clicks (Theme)',
    'Impressions (Theme)',
    'CTR (Theme)',
    'Conversions (Theme)',
    'Conversion Value (Theme)',
    'Campaign Cost (Total)'
  ];
  searchThemesSheet.appendRow(headers);

  const endDate = new Date();
  const startDate = new Date();
  startDate.setDate(endDate.getDate() - LOOKBACK_DAYS);

  const formattedStartDate = Utilities.formatDate(startDate, AdsApp.currentAccount().getTimeZone(), 'yyyyMMdd');
  const formattedEndDate = Utilities.formatDate(endDate, AdsApp.currentAccount().getTimeZone(), 'yyyyMMdd');

  Logger.log(`Fetching PMax Search Themes from ${formattedStartDate} to ${formattedEndDate}`);

  // Query to get Performance Max campaigns and their overall cost (since cost is not available per theme)
  let campaignCostQuery = `
    SELECT
      campaign.id,
      campaign.name,
      metrics.cost_micros
    FROM
      campaign
    WHERE
      campaign.advertising_channel_type = 'PERFORMANCE_MAX'
      AND campaign.status = 'ENABLED'
      AND segments.date BETWEEN '${formattedStartDate}' AND '${formattedEndDate}'
  `;

  if (CAMPAIGN_NAME_CONTAINS) {
    campaignCostQuery += ` AND campaign.name CONTAINS '${CAMPAIGN_NAME_CONTAINS}'`;
  }

  const campaignCostIterator = AdsApp.report(campaignCostQuery).rows();
  const campaignCostMap = new Map(); // Store campaign-level cost for quick lookup

  // First pass: Populate campaignCostMap with total campaign costs
  while (campaignCostIterator.hasNext()) {
    const campaignRow = campaignCostIterator.next();
    const campaignId = campaignRow['campaign.id'];
    const campaignName = campaignRow['campaign.name'];
    const campaignCost = campaignRow['metrics.cost_micros'] / 1000000;

    campaignCostMap.set(campaignId, {
      name: campaignName,
      cost: campaignCost
    });
  }

  // Second pass: Get search theme insights with Clicks, Impressions, Conversions, Conversion Value
  // We re-use the campaignCostQuery to iterate through relevant PMax campaigns again.
  const pmaxCampaignsIterator = AdsApp.report(campaignCostQuery).rows();

  while (pmaxCampaignsIterator.hasNext()) {
    const campaignRow = pmaxCampaignsIterator.next();
    const campaignId = campaignRow['campaign.id'];
    const campaignName = campaignRow['campaign.name']; // Use campaign name from this iteration

    const campaignCostData = campaignCostMap.get(campaignId);

    if (!campaignCostData) {
      Logger.log(`Warning: No campaign cost data found for campaign ID: ${campaignId}. Skipping.`);
      continue;
    }

    Logger.log(`Processing PMax Campaign: ${campaignName} (ID: ${campaignId}) for Search Themes.`);

    // Query to get search term insights (categories/themes) for the campaign.
    // This now includes clicks, impressions, conversions, and conversion_value per theme.
    const themeQuery = `
      SELECT
        campaign_search_term_insight.campaign_id,
        campaign_search_term_insight.category_label,
        metrics.clicks,
        metrics.impressions,
        metrics.conversions,
        metrics.conversions_value
      FROM
        campaign_search_term_insight
      WHERE
        segments.date BETWEEN '${formattedStartDate}' AND '${formattedEndDate}'
        AND campaign_search_term_insight.campaign_id = ${campaignId}
        AND metrics.impressions >= ${MIN_IMPRESSIONS}
      ORDER BY
        metrics.impressions DESC
    `;

    const themeIterator = AdsApp.report(themeQuery).rows();

    while (themeIterator.hasNext()) {
      const themeRow = themeIterator.next();
      const themeLabel = themeRow['campaign_search_term_insight.category_label'];
      const themeClicks = themeRow['metrics.clicks'];
      const themeImpressions = themeRow['metrics.impressions'];
      const themeConversions = themeRow['metrics.conversions'];
      const themeConversionValue = themeRow['metrics.conversions_value'];

      const themeCtr = themeImpressions > 0 ? (themeClicks / themeImpressions * 100).toFixed(2) + '%' : '0.00%';

      searchThemesSheet.appendRow([
        campaignCostData.name, // Campaign Name (from the cost map data)
        themeLabel,
        themeClicks,
        themeImpressions,
        themeCtr,
        themeConversions.toFixed(1),      // Conversions specific to this theme
        themeConversionValue.toFixed(2),  // Conversion Value specific to this theme
        campaignCostData.cost.toFixed(2)  // Total Campaign Cost for context
      ]);
    }
  }

  Logger.log('Script finished. PMax Search Themes exported to Google Sheet.');
}

🤝 Contribution

Feel free to fork this repository, suggest improvements, or submit pull requests.
