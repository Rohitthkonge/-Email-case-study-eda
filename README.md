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

## How each graph is plotted

### 1. Sent vs. Received — time of day across years (scatter)
![Sent vs received scatter](images/1_sent_vs_received.png)
*Sample output on synthetic data — your real inbox will show your own patterns.*
- X-axis = `year` (fractional, e.g. `2025.45`), Y-axis = `timeofday` (hour as a decimal, e.g. `14.5` = 2:30 PM)
- `df.plot.scatter` puts one dot per email at (year sent, time of day sent)
- Two side-by-side subplots — one filtered to `sent`, one to `inbox` — so you can visually compare when you send vs. when you receive
- Vertical "bands" mean you (or people emailing you) tend to be active at consistent hours; a dense bottom-right cluster means recent months are busier

### 2. Average emails per day / per hour (combined panel)
![Combined panel](images/2_avg_per_day_hour_panel.png)
Built with `matplotlib.gridspec` to place three related charts sharing axes:
- **Top strip** — histogram of `year`, weighted so each bar reads as "emails per day" instead of a raw count. The trick: `weights = 1 / (dt * 365.25)` per email, so summing bars gives a rate, not a total
- **Center scatter** — same year vs. time-of-day scatter as above, but sharing its X-axis with the top strip and Y-axis with the side strip, so all three stay aligned when you zoom/pan
- **Right strip** — same idea as the top strip but histogramming `timeofday` instead of `year`, plotted horizontally (`orientation="horizontal"`) so hours line up with the scatter's Y-axis
- This is the same layout style used in astronomy "light curve" plots — one main scatter with marginal histograms

### 3. Emails per day of week (bar chart)
![Emails per day of week](images/3_emails_per_dayofweek.png)
- `df["dayofweek"].value_counts()` just counts rows per day name (`Monday`, `Tuesday`, …)
- Plotted directly with `.plot(kind="bar")` — no weighting, just raw totals over the whole date range

### 4. Incoming vs. outgoing fraction per day of week
![Incoming vs outgoing](images/4_incoming_vs_outgoing.png)
- `pd.crosstab(dayofweek, label, normalize="columns")` — counts sent vs. inbox emails per day, then normalizes **each column** to sum to 1
- Normalizing matters because you likely send far fewer emails than you receive; comparing raw counts would make "sent" bars invisible. Normalizing shows *the shape* of each pattern (e.g. "Wednesday is your busiest sending day") on the same scale
- Grouped bar chart, two bars per day (Outgoing vs Incoming)

### 5. Average emails per hour, per day of week (line chart)
![Hourly by weekday](images/5_hourly_by_weekday.png)
- For each day-of-week group, `plot_number_perhour_per_year()` builds a weighted histogram of `timeofday` (same rate-normalization idea as chart #2), then:
  - Takes the bin centers and heights
  - Smooths them with a Gaussian filter (`scipy.ndimage.gaussian_filter`) to remove jagged histogram edges
  - Fits a cubic interpolation (`scipy.interpolate.interp1d`) through the smoothed points to get a continuous curve
  - Plots that curve instead of a bar histogram, so 7 overlapping days are readable as lines instead of 7 overlapping bar charts
- X-axis relabeled from raw hour numbers (0–24) to 12-hour clock labels (`12 AM`, `03 AM`, …)

### 6. Word cloud of email subjects
![Word cloud](images/6_wordcloud.png)
- All subject lines are joined into one long string
- `WordCloud` tokenizes that string, removes stopwords (English filler words + custom ones like `Re`, `Fwd`) and common newsletter senders, then sizes each remaining word by how often it appears
- `collocations=False` stops it from pairing up common two-word phrases, so word size reflects individual word frequency only

## Notes

- The scatter-plot and combined-panel plotting functions were reconstructed to match the original output style, since the underlying source wasn't fully visible in the source export — verified to run correctly end-to-end, but double-check styling against your original if you have it
- `custom_words` in the word cloud step (`Re`, `Fwd`, `FW`, `RE`) filters out reply/forward prefixes from subject lines — add more stopwords there if your subjects have other noise (e.g. newsletter boilerplate)
