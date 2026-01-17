# Salesforce Performance Scorecard

A lightweight web application that displays Salesforce report data as a visual performance scorecard with targets and achievement tracking.

## Features

- **OAuth Authentication** - Secure login with Salesforce
- **Owner Filtering** - Dropdown to filter metrics by opportunity owner (filtered to "VC AEs" role)
- **Current Month Data** - Automatically shows only current month's data
- **Visual Progress Bars** - Color-coded status indicators (green/yellow/red)
- **Direct Report Links** - Click through to Salesforce reports
- **Real-time Data** - Fetches live data from Salesforce Reports API

## Current Configuration

### Reports Tracked
1. **New Leads Created Per Month**
   - Report ID: `00OTP000008LLJO2A4`
   - Target: 29
   - Type: Count

2. **Applications Submitted Per Month**
   - Report ID: `00OTP000008Vesb2AC`
   - Target: 17
   - Type: Count

3. **Activations Per Month**
   - Report ID: `00OTP000008Vfwk2AC`
   - Target: 13
   - Type: Count

### Owner Filter
- Shows only users with **"VC AEs"** in their role name
- Automatically populated from Salesforce

### Performance Thresholds
- **Green (On Track)**: 90% or above of target
- **Yellow (At Risk)**: 70-89% of target
- **Red (Behind)**: Below 70% of target

## Setup Instructions

### Prerequisites
- Salesforce account with API access
- Python 3 installed (for local server)
- Web browser (Chrome, Firefox, Safari, or Edge)

### One-Time Salesforce Setup

#### 1. Create Connected App in Salesforce

1. Log into Salesforce → **Setup**
2. Search for **"App Manager"** in Quick Find
3. Click **New External Client App** (or **New Connected App**)
4. Fill in basic information:
   - **External Client App Name**: `Performance Scorecard`
   - **API Name**: `Performance_Scorecard`
   - **Contact Email**: Your email
   - **Distribution State**: Local

5. In **OAuth Settings** section:
   - **Callback URL**: `http://localhost:8080/salesforce-scorecard-oauth.html`
   - **Selected OAuth Scopes**: Add `Full access (full)`

6. Click **Save**, then **Continue**
7. Click **Manage Consumer Details** or go to Settings tab
8. **Copy your Consumer Key** (you'll need this)

#### 2. Configure CORS in Salesforce

1. In Salesforce Setup, search for **"CORS"** in Quick Find
2. Click **CORS** under Security
3. Click **New**
4. Enter Origin URL Pattern: `http://localhost:8080`
5. Click **Save**

## Running the Application

### 1. Start Local Server

Open Terminal and run:

```bash
cd ~/Downloads
python3 -m http.server 8080
```

Keep this terminal window open while using the app.

### 2. Open the Application

Navigate to: `http://localhost:8080/salesforce-scorecard-oauth.html`

### 3. First Time Login

1. Enter your **Consumer Key** from Salesforce
2. Select login URL:
   - Production: `https://login.salesforce.com`
   - Sandbox: `https://test.salesforce.com`
3. Click **Save Configuration**
4. Click **Login with Salesforce**
5. Authorize the app when prompted

Your credentials are saved in browser localStorage, so you won't need to enter them again.

## Making Changes

### Updating Report IDs or Targets

Open `salesforce-scorecard-oauth.html` in a text editor and find this section (around line 417):

```javascript
const reportConfigs = [
    {
        reportId: '00OTP000008LLJO2A4',  // Change this
        metricName: 'New Leads Created Per Month',
        target: 29,  // Change this
        unit: '',
        aggregationField: 'ID',
        aggregationType: 'count'
    },
    // ... more reports
];
```

**What you can change:**
- `reportId`: The Salesforce report ID (from report URL)
- `metricName`: Display name for the metric
- `target`: Your target/goal number
- `unit`: '$' for currency, '%' for percent, '' for none
- `aggregationType`: 'count', 'sum', or 'average'

### Changing Owner Filter

To change which owners appear in the dropdown, find this line (around line 653):

```javascript
const query = "SELECT Id, Name, UserRole.Name FROM User WHERE Id IN (SELECT OwnerId FROM Opportunity) AND UserRole.Name LIKE '%VC AEs%' ORDER BY Name";
```

**Options:**
- Different role: Change `'%VC AEs%'` to `'%Your Role%'`
- Multiple roles: Use `(UserRole.Name LIKE '%Role1%' OR UserRole.Name LIKE '%Role2%')`
- Remove filter: Delete `AND UserRole.Name LIKE '%VC AEs%'`
- Specific names: Replace entire WHERE clause with `AND Name IN ('John Doe', 'Jane Smith')`

### Changing Performance Thresholds

Find this section (around line 443):

```javascript
const thresholds = {
    success: 0.9,  // 90% or above = green
    warning: 0.7   // 70-89% = yellow, below 70% = red
};
```

### Adding a New Report

Copy an existing report block and update all fields:

```javascript
{
    type: 'report',
    metricId: 'yourMetricId',  // Unique identifier for referencing
    reportId: 'YOUR_REPORT_ID',
    metricName: 'Your Metric Name',
    target: 100,
    unit: '',
    aggregationField: 'ID',
    aggregationType: 'count'
},
```

**Important:** Your Salesforce report must be:
- A **summary report** grouped by Opportunity Owner
- Then sub-grouped by a **date field**
- The date field should show month/year (e.g., "January 2026")

### Adding Calculated Metrics

**NEW FEATURE:** You can now create metrics that calculate values from other metrics using formulas!

Calculated metrics derive their values from existing report-based metrics. For example, the **Win Rate** metric calculates: (Activations ÷ New Leads) × 100

#### Configuration Structure

Add to the `metricConfigs` array in the HTML file (around line 572):

```javascript
{
    type: 'calculated',
    metricId: 'winRate',           // Unique identifier
    metricName: 'Win Rate',         // Display name
    formula: {
        operation: 'divide',        // Type of calculation
        numerator: 'activations',   // Reference to activations metric
        denominator: 'newLeads',    // Reference to new leads metric
        multiplyBy: 100             // Optional: multiply result (for %)
    },
    target: 100,                    // Target percentage
    unit: '%'                       // Display unit
}
```

#### Supported Formula Operations

**1. Division (for rates, percentages)**
```javascript
formula: {
    operation: 'divide',
    numerator: 'metricId1',
    denominator: 'metricId2',
    multiplyBy: 100  // Optional: convert to percentage
}
```

**Example: Application Rate**
```javascript
{
    type: 'calculated',
    metricId: 'applicationRate',
    metricName: 'Application Rate',
    formula: {
        operation: 'divide',
        numerator: 'applications',
        denominator: 'newLeads',
        multiplyBy: 100
    },
    target: 58.6,
    unit: '%'
}
```

**2. Multiplication**
```javascript
formula: {
    operation: 'multiply',
    factor1: 'metricId1',
    factor2: 'metricId2'
}
```

**3. Addition (for totals)**
```javascript
formula: {
    operation: 'add',
    operands: ['metricId1', 'metricId2', 'metricId3']
}
```

**4. Subtraction**
```javascript
formula: {
    operation: 'subtract',
    minuend: 'metricId1',
    subtrahend: 'metricId2'
}
```

#### Important Notes

1. **metricId References**: Formula operands must reference the `metricId` of existing metrics
2. **Dependency Order**: Report-based metrics are fetched first, then calculated metrics are computed
3. **Division by Zero**: Returns 0 if denominator is 0 (e.g., 0% win rate when no leads created)
4. **Error Handling**: If a dependency metric fails to load, the calculated metric will show an error
5. **Editable Targets**: Calculated metrics have editable targets just like report-based metrics

#### More Examples

**Activation Rate** = (Activations ÷ Applications) × 100
```javascript
{
    type: 'calculated',
    metricId: 'activationRate',
    metricName: 'Activation Rate',
    formula: {
        operation: 'divide',
        numerator: 'activations',
        denominator: 'applications',
        multiplyBy: 100
    },
    target: 76.5,
    unit: '%'
}
```

**Total Pipeline** = Applications + Activations
```javascript
{
    type: 'calculated',
    metricId: 'totalPipeline',
    metricName: 'Total Pipeline',
    formula: {
        operation: 'add',
        operands: ['applications', 'activations']
    },
    target: 30,
    unit: ''
}
```

## Troubleshooting

### "CORS error" / "blocked by CORS policy"
- Check that you added `http://localhost:8080` to CORS settings in Salesforce
- Make sure you're accessing via `http://localhost:8080`, not `file://`

### "Session expired. Please log in again."
- Your OAuth token expired (happens after a few hours)
- Click **Login with Salesforce** again

### Reports show 0 or wrong numbers
- Open browser console (F12) to see detailed logs
- Check that report is grouped by Owner → Date
- Verify report IDs are correct
- Ensure reports are accessible to your user

### Dropdown shows no owners or wrong owners
- Check the role filter in the code
- Verify users have the correct role in Salesforce
- Check browser console for query errors

### "No matching month grouping found"
- Report might not have current month data yet
- Check that date grouping format is "Month Year" (e.g., "January 2026")
- Verify report has data for the current month

### Calculated Metric Shows Error

**"Missing metric: [metricId]"**
- The formula references a metricId that doesn't exist
- Check that the referenced metricId matches exactly (case-sensitive)
- Verify the dependency metric is defined in metricConfigs

**"Dependency error"**
- One of the metrics used in the formula failed to load
- Check the browser console for errors on the dependency metric
- Fix the underlying report metric first

**Calculated metric shows 0**
- Check browser console (F12) for calculation logs
- If dividing by zero, the result is 0 by design
- Verify the dependency metrics have actual data

### Division by Zero in Calculated Metrics
- By design, division by zero returns 0 (e.g., 0% win rate when no leads)
- Check console logs to see: "Division by zero in formula..."
- This prevents errors when denominators are 0

## File Location

Main file: `/Users/anthonyhwang/Downloads/salesforce-scorecard-oauth.html`

This is a **single HTML file** containing everything. You can:
- Move it anywhere
- Rename it
- Make backup copies
- Edit in any text editor

## Configuration Storage

Your Salesforce credentials are stored in **browser localStorage**:
- Consumer Key
- Access Token
- Instance URL
- User Info

**Important:** These are stored per-browser. If you:
- Clear browser data → You'll need to log in again
- Use a different browser → You'll need to configure again
- Use incognito/private mode → Credentials won't persist

## Security Notes

- Never commit your Consumer Key or Access Token to public repositories
- The app uses OAuth 2.0 User-Agent Flow (implicit flow)
- Tokens are stored in browser localStorage (client-side only)
- For production use, consider using Web Server Flow with a backend

## Current Salesforce Org

**Instance**: `rho.lightning.force.com`

## Support & Maintenance

### Getting Report IDs
1. Open a report in Salesforce
2. Look at the URL: `https://rho.lightning.force.com/REPORT_ID`
3. The report ID is the alphanumeric string (e.g., `00OTP000008LLJO2A4`)

### Updating the App
All changes are made by editing the HTML file. After making changes:
1. Save the file
2. Refresh your browser (Cmd+R or Ctrl+R)
3. Changes take effect immediately

### Backup Recommendations
1. **Local Backup**: Copy the HTML file to a safe location
2. **Cloud Backup**: Upload to Google Drive, Dropbox, or OneDrive
3. **Version Control**: Use Git/GitHub for tracking changes

### Version Control with Git
```bash
cd ~/Downloads
git init
git add salesforce-scorecard-oauth.html salesforce-scorecard-README.md
git commit -m "Initial commit: Salesforce scorecard"
```

## Future Enhancement Ideas

- [ ] Add date range selector (week, month, quarter, year)
- [ ] Export data to CSV
- [ ] Email scheduling/alerts
- [ ] Historical trend charts
- [ ] Mobile responsive improvements
- [ ] Multi-org support
- [ ] Custom styling/themes
- [ ] Additional aggregate types (average, sum)
- [ ] Team leaderboards

## Technical Details

**Technology Stack:**
- HTML/CSS/JavaScript (vanilla, no frameworks)
- Salesforce REST API (Reports API v59.0)
- OAuth 2.0 User-Agent Flow
- Python HTTP server (for local development)

**Browser Compatibility:**
- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

**File Size:** ~30KB (single file)

---

**Last Updated:** January 17, 2026
**Created by:** Claude Code (Anthropic)
