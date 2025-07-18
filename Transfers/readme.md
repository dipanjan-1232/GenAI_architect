import re
import dateparser
from datetime import datetime

def extract_date_info(text):
    text = text.strip()
    results = []

    # --- Detect Q[1-4] of FY yyyy-yy ---
    fy_qtr_match = re.search(r'\bq([1-4])\s*(?:of\s*)?(?:fy\s*)?(\d{4})[-–/](\d{2})\b', text, re.IGNORECASE)
    if fy_qtr_match:
        quarter = int(fy_qtr_match.group(1))
        fy_start = int(fy_qtr_match.group(2))
        fy_end_suffix = int(fy_qtr_match.group(3))
        fy_end = fy_start // 100 * 100 + fy_end_suffix

        # Fiscal year starts April -> Q1 = Apr–Jun
        quarter_start_map = {
            1: (fy_start, 4),
            2: (fy_start, 7),
            3: (fy_start, 10),
            4: (fy_end, 1),
        }
        q_year, q_month = quarter_start_map[quarter]
        results.append({
            'type': 'fiscal_quarter',
            'quarter': f'Q{quarter}',
            'fiscal_year': f'{fy_start}-{fy_end_suffix}',
            'start_date': datetime(q_year, q_month, 1)
        })

    # --- Detect FY yyyy-yy or yyyy-yyyy ---
    fy_match = re.search(r'\b(?:fy\s*)?(\d{4})[-–/](\d{2,4})\b', text, re.IGNORECASE)
    if fy_match:
        fy_start = int(fy_match.group(1))
        fy_end_raw = fy_match.group(2)
        fy_end = int(fy_end_raw) if len(fy_end_raw) == 4 else fy_start // 100 * 100 + int(fy_end_raw)

        results.append({
            'type': 'fiscal_year',
            'fiscal_year': f'{fy_start}-{fy_end_raw}',
            'start_date': datetime(fy_start, 4, 1),
            'end_date': datetime(fy_end, 3, 31)
        })

    # --- Use dateparser to detect natural language dates ---
    parsed = dateparser.search.search_dates(
        text,
        settings={
            'PREFER_DATES_FROM': 'future',
            'RELATIVE_BASE': datetime.today(),
            'STRICT_PARSING': True
        }
    )

    if parsed:
        for match_text, parsed_dt in parsed:
            results.append({
                'type': 'natural_date',
                'text': match_text,
                'parsed_date': parsed_dt
            })

    return results

# --- Test Cases ---
examples = [
    "Q4 of FY 2024-25",
    "FY 2023-24",
    "2024-2025",
    "2023–24",
    "June 13 of next month",
    "Friday of last month",
    "Random string",
    "13th June",
    "April 2025",
    "Q2 2025-26"
]

for e in examples:
    print(f"\nInput: {e}")
    result = extract_date_info(e)
    if result:
        for item in result:
            print(f"  ➤ Detected: {item}")
    else:
        print("  ❌ No date/quarter/FY detected.")
