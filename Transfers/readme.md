import re
from dateparser.search import search_dates

def has_any_date(text: str) -> bool:
    text = text.strip().lower()

    # 1. Regex patterns for fiscal year and year ranges
    fy_pattern = r'\b(?:fy\s*)?\d{4}[-–/]\d{2,4}\b'       # FY 2023-24 or 2024–2025
    qtr_pattern = r'\bq[1-4]\b'                           # Q1, Q2, etc.

    # 2. Manual handling for expressions like "next quarter", "this quarter"
    relative_quarter_keywords = [
        "next quarter", "this quarter", "last quarter",
        "current quarter", "previous quarter"
    ]

    # 3. Regex or keyword match
    if re.search(fy_pattern, text) or re.search(qtr_pattern, text):
        return True
    if any(phrase in text for phrase in relative_quarter_keywords):
        return True

    # 4. Natural language date parsing
    parsed = search_dates(text, settings={'PREFER_DATES_FROM': 'future'})
    if parsed:
        return True

    return False
