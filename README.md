# Email Dataset Case Study

Exploratory Data Analysis on personal Gmail data — exported via Google Takeout, parsed from `.mbox`, and analyzed for sending/receiving patterns.

## What it does

1. **Extract** — parses a Gmail `.mbox` export (`mailbox` module) into a CSV with subject, sender, date, recipient, label, and thread ID
2. **Clean** — parses dates, drops rows with missing dates, extracts clean email addresses from the `From` header via regex
3. **Label** — tags each email as `sent` or `inbox` based on the account owner's address
4. **Feature engineer** — converts timezone, derives day-of-week, hour, and fractional year columns
5. **Visualize**
   - Sent vs. received emails: time of day across years (scatter)
   - Combined panel: average emails/day + scatter + average emails/hour
   - Emails per day of week (bar chart)
   - Incoming vs. outgoing fraction per day of week
   - Average emails per hour, broken down by day of week
   - Word cloud of email subjects

## Setup

```bash
pip install pandas numpy matplotlib scipy wordcloud pytz
```

(In Google Colab, `wordcloud` is installed inline via `!pip install`.)

## How to run

1. Export your Gmail data from [Google Takeout](https://takeout.google.com) (Mail only, `.zip` format)
2. Unzip and locate the `.mbox` file under `Mail/`
3. Upload it to your Colab environment (or local working directory)
4. In `email_case_study.ipynb`, update:
   - `mboxfile` — path to your uploaded `.mbox` file
   - `myemail` — your Gmail address (used to label sent vs. received)
5. Run all cells top to bottom

## Notes

- The scatter-plot and combined-panel plotting functions were reconstructed to match the original output style, since the underlying source wasn't fully visible in the source export — verified to run correctly end-to-end, but double-check styling against your original if you have it
- `custom_words` in the word cloud step (`Re`, `Fwd`, `FW`, `RE`) filters out reply/forward prefixes from subject lines — add more stopwords there if your subjects have other noise (e.g. newsletter boilerplate)
