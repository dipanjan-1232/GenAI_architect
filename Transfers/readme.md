import re
import dateparser
from dateparser.search import search_dates

def has_any_date(text: str) -> bool:
    text = text.strip()

    # Check for fiscal year or quarter patterns
    fy_or_qtr_pattern = re.search(r'\b(?:fy\s*)?\d{4}[-–/]\d{2,4}\b', text, re.IGNORECASE)
    qtr_pattern = re.search(r'\bq[1-4]\b', text, re.IGNORECASE)

    if fy_or_qtr_pattern or qtr_pattern:
        return True

    # Check for natural language date
    parsed = search_dates(text, settings={'PREFER_DATES_FROM': 'future'})
    if parsed:
        return True

    return False
