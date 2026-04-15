Task 1 — Classify and Handle PII Fields

------------------------------------------------------------------------------------------
Fields                      Type            Action              Justification
------------------------------------------------------------------------------------------
full_name               Direct PII            Drop          directly identifies person
email                   Direct PII            Drop          sensitive and unique
date_of_birth           Indirect PII          Mask          can identify with other data
zip_code                Indirect PII          Mask          reveals location
job_title               Indirect PII          Keep          useful for analysis
diagnosis_notes         Sensitive             Clean         contains private health info


Task 2 — Ethical / TOS Violations

Violation 1: Hardcoded API Key
    Problem:

    The API key is written directly in the code.

    This is not safe because:
        Anyone who sees the code can misuse the key
        It can lead to security issues

    Fix:
        """
        import os
        import requests

        API_URL = "https://healthstats-api.example.com/records"
        API_KEY = os.getenv("API_KEY")  # secure way
        """

Violation 2: Bulk Data Scraping Without Limits

    The script is making continuous requests for 100 pages without any delay.
    This can:
        Overload the server
        Break the API provider’s usage rules

    Fix:
        """
        import time
        import requests

        records = []

        for page in range(1, 101):
            response = requests.get(API_URL, params={"page": page, "key": API_KEY})
            
            if response.status_code != 200:
                break
            
            data = response.json()
            records.extend(data.get("results", []))
            
            time.sleep(1)  # respect rate limits
        """

Violation 3: Storing Raw PII Data Permanently

    The script stores raw personal and sensitive data directly in the database.
    This can:
        Expose private user information

    Fix:
        """
        def clean_record(record):
        return {
            "age": 2026 - int(record["date_of_birth"][:4]),
            "zip_code": record["zip_code"][:3],
            "job_title": record["job_title"],
            "diagnosis_notes": record["diagnosis_notes"]
        }

    cleaned_records = [clean_record(r) for r in records]

    save_to_database(cleaned_records)
        """