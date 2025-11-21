# Alyx Steadman Therapy PLLC - Income Manager

A single-page web application for managing weekly income, business expenses, and tax savings for self-employed therapists.

## Features

- **Weekly Income Tracking**: Enter weekly income and automatically calculate splits for business expenses, tax savings, and personal paycheck
- **Tax Management**: Track quarterly estimated payments and year-end tax projections
- **Expense Budgeting**: Upload monthly expense budgets via CSV and track bill due dates
- **Local Data Storage**: All data stored locally in your browser - completely private and offline-capable
- **Data Export**: Export data to CSV for your CPA or backup as JSON
- **Transparent Calculations**: All tax calculations are auditable with detailed breakdowns

## Installation

1. Simply open `alyx-income-manager.html` in any modern web browser (Chrome, Firefox, Edge, Safari)
2. No installation, no server, no internet required after the initial load
3. Bookmark the file for easy access

## Getting Started

### First-Time Setup

When you first open the app, you'll be guided through a setup wizard:

1. **Tax Information**: Enter your 2024 total tax (from Form 1040, Line 24)
2. **Expense Budget**: Upload your monthly expense budget CSV (optional - you can do this later)
3. **Bill Due Dates**: Configure when each bill is due each month
4. **Starting Balances**: Enter your current business checking and tax savings balances

### Creating Your Expense Budget

Use the included `expense-budget-template.csv` file or download it from within the app:

1. Open the template in Excel, Google Sheets, or any spreadsheet program
2. Edit the expense names and monthly amounts
3. Save as CSV
4. Upload in the Expenses screen

**CSV Format:**
```csv
Expense,Jan,Feb,Mar,Apr,May,Jun,Jul,Aug,Sep,Oct,Nov,Dec
SimplePractice,87,87,87,87,87,87,87,87,87,87,87,87
Rent,410,410,410,410,410,410,410,410,410,410,410,410
Phone,123,123,123,123,123,123,123,123,123,123,123,123
```

## How to Use

### Weekly Income Entry (Dashboard)

1. Go to the Dashboard
2. Enter your weekly income in the input field
3. Click "Calculate My Split"
4. Review the breakdown:
   - **Business Expenses**: Amount to reserve for upcoming bills
   - **Tax Savings**: 30% saved for quarterly and year-end taxes
   - **Paycheck**: Amount safe to pay yourself
5. Click "Record This Week" to save

### Viewing History

1. Go to History screen
2. See all recorded weeks with income splits
3. Export to CSV for record-keeping

### Managing Taxes

1. Go to Taxes screen
2. View quarterly payment schedule and status
3. Mark payments as paid when you make them
4. Review year-end tax projection based on current pace
5. See detailed tax calculation breakdowns

### Managing Expenses

1. Go to Expenses screen
2. Upload or update your monthly budget CSV
3. Configure bill due dates (day of month each bill is due)
4. View current business checking balance
5. Add new bills as needed

### Settings

1. Go to Settings screen
2. Adjust tax settings (2024 tax total, standard deduction, tax savings rate)
3. Update business settings
4. Export data for your CPA
5. Backup your data (download JSON file)
6. Import previous backups

## Data Backup & Export

### Regular Backups

It's recommended to backup your data monthly:

1. Go to Settings
2. Click "Backup Data (JSON)"
3. Save the file to a safe location (cloud storage, external drive)

### Export for CPA

1. Go to Settings
2. Click "Export All Data (CSV)"
3. Send this file to your CPA at tax time

### Importing a Backup

1. Go to Settings
2. Click "Import Backup"
3. Select your previously saved JSON backup file

## Tax Calculations Explained

### Self-Employment Tax

```
Net Income × 0.9235 = SE Tax Base
SE Tax Base × 0.153 = SE Tax (15.3%)
SE Tax ÷ 2 = SE Tax Deduction
```

### Income Tax

```
Net Income - SE Tax Deduction = Adjusted Income
Adjusted Income - Standard Deduction = Taxable Income
Taxable Income × Tax Brackets = Income Tax
```

### Total Tax

```
SE Tax + Income Tax = Total Tax Liability
```

### Weekly Tax Savings (Simple Rule)

```
Weekly Income × 30% = Weekly Tax Savings
```

This 30% covers both:
- Self-employment tax (~15.3%)
- Income tax (~12-22% depending on bracket)

## Quarterly Tax Payment Schedule

- **Q1 2025** (Jan 1 - Mar 31): Due April 15, 2025
- **Q2 2025** (Apr 1 - May 31): Due June 15, 2025
- **Q3 2025** (Jun 1 - Aug 31): Due September 15, 2025
- **Q4 2025** (Sep 1 - Dec 31): Due January 15, 2026

## Troubleshooting

### Data Not Saving

- Make sure you're not in "Private/Incognito" browsing mode
- Check that your browser allows localStorage
- Try a different browser if issues persist

### Missing Weeks

- The app will alert you on the Dashboard if you have missing weeks
- Click "Enter Missing Weeks" to catch up on past weeks
- You can edit any week from the History screen

### Calculations Look Wrong

- Click "Show Calculation" buttons to see detailed breakdowns
- Verify your 2024 tax total is entered correctly in Settings
- Check that your expense budget CSV uploaded correctly

### Lost Data

- If you cleared browser data, your local storage was deleted
- Restore from your most recent JSON backup (Settings → Import Backup)
- This is why regular backups are important!

## Technical Details

### Technologies Used

- React 18 (via CDN)
- Tailwind CSS (via CDN)
- PapaParse (CSV parsing)
- Browser localStorage for data persistence

### Browser Compatibility

- Chrome/Edge: Fully supported
- Firefox: Fully supported
- Safari: Fully supported
- Requires JavaScript enabled

### Data Storage

All data is stored in your browser's localStorage under the key `alyxIncomeManager`. No data is sent to any server - everything stays on your computer.

### File Size

The HTML file is approximately 100KB and contains everything needed - no external dependencies besides CDN libraries.

## Privacy & Security

- **100% Local**: All data stored locally in your browser
- **No Server**: No data transmitted anywhere
- **No Tracking**: No analytics or tracking code
- **Portable**: Just backup the HTML file and your JSON backups

## Support & Feedback

This is a custom-built application. Keep the HTML file and your JSON backups safe!

### Common Questions

**Q: Can I use this on multiple computers?**
A: Yes, but you'll need to manually transfer your data using JSON backups.

**Q: What if I miss entering a week?**
A: No problem! The Dashboard will alert you and you can enter missed weeks anytime.

**Q: Can I change the tax savings rate from 30%?**
A: Yes, go to Settings → Tax Settings → Edit Settings and adjust the percentage.

**Q: What if my actual income is very different from projections?**
A: The projection updates automatically as you enter more weeks. It's based on your actual year-to-date average.

**Q: Do I need internet to use this?**
A: Only for the first load (to download CSS/JS from CDNs). After that, it works offline.

## Developer Notes

### Code Structure

The application is built as a single HTML file with:
- All React components inline
- Tailwind CSS via CDN
- Data model in localStorage
- Pure JavaScript for calculations

### Key Functions

- `calculateTotalTax()`: Calculates SE tax and income tax
- `calculateBusinessReserve()`: Determines needed business expense reserve
- `calculateUpcomingBills()`: Finds bills due in next 14 days
- `parseExpenseCSV()`: Parses uploaded expense budget
- `saveToLocalStorage()` / `loadFromLocalStorage()`: Data persistence

### Modifying the App

To modify:
1. Open `alyx-income-manager.html` in a code editor
2. Find the `<script type="text/babel">` section
3. Edit React components or functions
4. Save and refresh in browser

### Tax Bracket Updates

Tax brackets are defined in the `TAX_BRACKETS_2024` constant. Update this array if tax brackets change.

## License

Custom-built for Alyx Steadman Therapy PLLC. All rights reserved.

---

**Version**: 1.0
**Last Updated**: November 2025
**Author**: Built with Claude Code
