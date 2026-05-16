# Frequently Asked Questions (FAQ)

## General Questions

### Q1: Do I need coding experience to follow this project?
**A:** Basic Python knowledge helps but isn't required. Most code is provided and explained step-by-step. The learning focus is on **data engineering concepts**, not Python syntax.

### Q2: How long does the complete project take?
**A:** 
- **Quick walkthrough:** 2-3 hours
- **Full understanding:** 1 day
- **With Power BI reports:** 1-2 days

### Q3: Can I run this outside Microsoft Fabric?
**A:** The core concepts (Medallion Architecture) apply everywhere, but you'd need to adapt for:
- Apache Spark (Databricks)
- AWS (Glue, Athena)
- Google Cloud (BigQuery, Dataproc)

### Q4: Do I need Power BI to complete this?
**A:** No. Power BI is only for visualization (optional). The core data engineering works without it.

### Q5: Is there a cost associated with this project?
**A:** Yes, Microsoft Fabric uses cloud resources that may incur charges. Costs depend on:
- Your organization's Fabric capacity
- Amount of data processed
- Duration of execution

Check with your organization's billing team.

---

## Technical Questions

### Q6: What's the difference between Bronze, Silver, and Gold?

**Bronze (Raw):**
- Original data from API
- No transformation
- Used for: Audit trail, debugging

**Silver (Cleaned):**
- Structured and cleaned
- Standardized schema
- Used for: Data quality checks, internal analytics

**Gold (Business-Ready):**
- Enriched with business logic
- Pre-calculated metrics
- Used for: Dashboards, reports, ML models

### Q7: Why do we need all three layers?
**A:** They solve different problems:
- **Bronze** → Trust (we have original data)
- **Silver** → Quality (data is standardized)
- **Gold** → Performance (optimized for queries)

Without this separation, you'd have to re-implement transformations in every project.

### Q8: Can I skip a layer?
**A:** Technically yes, but we don't recommend it. Each layer teaches important concepts:
- Bronze → API integration & data ingestion
- Silver → PySpark transformations & schema design
- Gold → Business logic & data enrichment

For teaching, going through all three is valuable.

### Q9: Why use reverse geocoding in the Gold layer?
**A:** Reverse geocoding (lat/long → country) adds business value:
- Enables geographic analysis
- Supports regulatory reporting
- Filters data by region
- Improves data interpretation

It's a practical example of enrichment that real data pipelines do.

### Q10: What if the USGS API goes down?
**A:** The API has 99.9%+ uptime historically. If it fails:
1. Check [USGS Status Page](https://earthquake.usgs.gov/)
2. Try a different date range
3. Add error handling to your code:
```python
if response.status_code != 200:
    print(f"API error: {response.status_code}")
```

---

## Troubleshooting

### Q11: I'm getting "FileNotFoundError" - what do I do?

**Most common causes:**

1. **Bronze layer didn't run successfully**
   - Check the output for errors
   - Verify API returned data (check status_code = 200)

2. **Wrong variable name**
   - Ensure `start_date` matches between notebooks
   - Example: If Bronze used "2024-01-01", Silver must too

3. **Lakehouse path issue**
   - Verify correct lakehouse is attached
   - Check the file exists in `Files/` folder

**Solution:** Run Bronze layer again, verify success messages, then try Silver.

### Q12: I'm getting "Cannot CREATE TABLE" error - what now?

**This means:** The table already exists from a previous run.

**Solutions:**
```python
# Option 1: Append (add new data to existing table)
df.write.mode('append').saveAsTable('earthquake_events_silver')

# Option 2: Overwrite (replace with new data)
df.write.mode('overwrite').saveAsTable('earthquake_events_silver')

# Option 3: Drop and recreate
spark.sql('DROP TABLE IF EXISTS earthquake_events_silver')
df.write.saveAsTable('earthquake_events_silver')
```

We use `mode='append'` by default because it's safer.

### Q13: Reverse geocoding is very slow - why?

**Reverse geocoder** looks up country codes from coordinates. With many records:
- First lookup is slow (loads database)
- Subsequent lookups are faster

**Speed improvements:**
```python
# Use smaller date ranges
start_date = "2024-01-01"
end_date = "2024-01-01"  # Single day instead of month

# Or filter for significant earthquakes only
df = df.filter(col("mag") > 5.0)

# Cache the lookup function results
from pyspark.sql.functions import broadcast
# Advanced optimization - ask instructor for details
```

### Q14: I'm getting "Lakehouse not connected" error

**This means:** The notebook can't find the Lakehouse.

**Fix it:**
1. Click notebook name (top of notebook)
2. Look for "Lakehouse" section
3. If empty, click **Add lakehouse**
4. Select `earthquake_data`
5. Click **Add**

Now try running again.

### Q15: The API returns no data - what's wrong?

**Common reasons:**

1. **Date too recent** - USGS has a 1-3 hour delay
   ```python
   # Use past dates, not today
   start_date = "2024-01-01"  # ✓ Good
   end_date = "2024-01-02"
   
   # Don't use current date
   start_date = "2024-05-16"  # ✗ Might not have data yet
   ```

2. **Date range too small** - No earthquakes that day
   ```python
   # Use larger range
   start_date = "2024-01-01"
   end_date = "2024-01-31"
   ```

3. **API down** - Check [USGS Status](https://earthquake.usgs.gov/)

**Check what data is available:**
```python
# Test the API
url = "https://earthquake.usgs.gov/fdsnws/event/1/query"
params = {
    "format": "geojson",
    "starttime": "2024-01-01",
    "endtime": "2024-01-02",
    "minmagnitude": 4.5
}
response = requests.get(url, params=params)
print(f"Status: {response.status_code}")
print(f"Records: {len(response.json()['features'])}")
```

---

## Learning Questions

### Q16: How does this relate to real-world data engineering?

**This project mirrors real pipelines:**
- ✓ Fetches from APIs (like real data sources)
- ✓ Implements Medallion Architecture (industry standard)
- ✓ Uses PySpark (most data engineers use Spark)
- ✓ Builds for analytics (actual use case)

**Where it's simplified:**
- ✗ Real pipelines run on schedules (Data Factory)
- ✗ Real pipelines have data quality checks
- ✗ Real pipelines have error handling
- ✗ Real pipelines serve millions of records

But the fundamentals are identical.

### Q17: What skills does this teach?
1. **Data Ingestion** - Pulling data from APIs
2. **Data Transformation** - Using PySpark to reshape data
3. **Schema Design** - Creating meaningful structure
4. **Data Enrichment** - Adding business value
5. **Pipeline Design** - Multi-layer architecture
6. **Analytics** - Preparing for visualization

These are core data engineering skills.

### Q18: Can I use this data for my own analysis?

**For educational use:** Yes, absolutely!

**For publishing:** Check USGS terms:
- [USGS Data Policy](https://www.usgs.gov/faqs/what-are-usage-restrictions-usgs-scientific-data)
- Generally, USGS data is public domain
- Always cite your source

### Q19: How do I extend this project?

**Easy extensions:**
1. Add more filters (magnitude, location)
2. Create additional analysis metrics
3. Build more Power BI visuals
4. Schedule with Data Factory
5. Add data quality checks

**Advanced extensions:**
1. ML models to predict earthquake patterns
2. Real-time streaming from USGS
3. Integration with external datasets (population, infrastructure)

### Q20: What's the best way to learn from this?

**Recommended approach:**
1. **Run the code first** - See it work
2. **Read the explanations** - Understand why
3. **Modify values** - Change dates, thresholds
4. **Break things** - See what errors occur
5. **Fix issues** - Learn troubleshooting
6. **Extend functionality** - Build on it

**Learning mindset:**
- Don't just copy-paste
- Understand each transformation
- Ask "Why is this done this way?"
- Try variations and see results

---

## Need More Help?

1. **For technical issues** → See [Troubleshooting](README.md#troubleshooting) in README
2. **For API questions** → See [API Documentation](README.md#api-documentation) 
3. **For concepts** → See [Architecture Overview](README.md#architecture-overview)
4. **For setup** → See [Step-by-Step Guide](README.md#step-by-step-setup-guide)

---

**Last Updated:** May 2026

**Still have questions?** Your instructor or the Microsoft Fabric community can help!
