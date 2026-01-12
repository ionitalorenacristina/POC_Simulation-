# POC_Simulation - Entity Resolution and Data Analysis & QC

## Part 1: Entity Resolution

The goal of this step was to resolve each input company to a real-world entity using the candidate matches provided by the client’s database. The process included the following steps:

## Part 1: Entity Resolution

The goal of this step was to resolve each input company to a real-world entity using the candidate matches provided by the client’s database. The process included the following steps:

1. **Data Simplification**  
    Created two auxiliary columns: `input_simplified` and `db_name`, removing spaces, punctuation, slashes, and converting all text to uppercase to standardize the company names.  

2. **Name Matching**  
    Compared simplified names using substring search; assigned `1` if there was a match, `0` otherwise.  

3. **Country and City Matching**  
    Country: exact match → `Yes` / `No`  
    City: substring match to account for administrative variants (e.g. 'Augsburg municipality' vs 'Augsburg') = `Yes` / `No`  

4. **Match Level Determination**  
    High: name_match = 1 AND country = Yes AND city = Yes  
    Medium: name_match = 1 AND (country = Yes OR city = Yes)  
    Low: all other cases  
    Formula:  
     ```excel
     =IF(AND(W7=1;X7="Yes";Y7="Yes");"High";IF(AND(W7=1;OR(X7="Yes";Y7="Yes"));"Medium";"Low"))
     ```

5. **Chosen ID Assignment and Match Decision**  
    High match = `Chosen_ID` assigned directly from Veridion ID  
    Medium match = further checked city/country variants; valid matches assigned ID, flagged as `Match Found`  
    Low match or country mismatch → automatically `No Match`  
    Match Found / No Match column formula:  
     ```excel
     =IF(AA2<>"";"Match Found";"No Match")
     ```

6. **Duplicate Handling**  
    Duplicate `Chosen_ID`s were reviewed; only correct matches were kept.  
    After removing duplicates, the final counts were: 318 `Match Found` (and 333 before removing) and 2801 `No Match`.

7. **Summary**  
    The majority of High match candidates were automatically resolved.  
    Medium matches required additional checks for city variations.  
    Low matches were excluded to prevent incorrect matches.
    Administrative city variants were handled as soft matches to minimize manual review.

---

| Column Name       | Purpose / Description                                               | Example Formula |
|------------------|--------------------------------------------------------------------|----------------|
| `input_simplified` | Standardized input company name for comparison                     | `=UPPER(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(B7;" ";"");"/";"");".";"");"-";""))` |
| `db_name`          | Standardized database company name for comparison                  | `=UPPER(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(K7;" ";"");"/";"");".";"");"-";""))` |
| `name_match`       | Checks if input and db_name partially match                        | `=IF(OR(ISNUMBER(SEARCH(U7;V7));ISNUMBER(SEARCH(V7;U7)));1;0)` |
| `country_match`    | Checks if country matches exactly                                    | `=IF(UPPER(D7)=UPPER(O7);"Yes";"No")` |
| `city_match`       | Checks if city partially matches (accounts for administrative variants) | `=IF(ISNUMBER(SEARCH(UPPER(F7);UPPER(Q7)));"Yes";"No")` |
| `match_level`      | Determines Low / Medium / High match based on name/country/city    | `=IF(AND(W7=1;X7="Yes";Y7="Yes");"High";IF(AND(W7=1;OR(X7="Yes";Y7="Yes"));"Medium";"Low"))` |
| `Chosen_ID`        | ID assigned for valid matches                                        | `=IF(Z7="High";J7;"")` (High) / `=IF(OR(ISNUMBER(SEARCH(F316;Q316));ISNUMBER(SEARCH(Q316;F316)));J316;"")` (Medium) |
| `Match Found / No Match` | Final decision for entity resolution                           | `=IF(AA2<>"";"Match Found";"No Match")` |

---

**Notes:**  
 Medium matches may require additional manual review for ambiguous city variations.
 I only use validated IDs for the next part of the analysis.

## Part 2: Data analysis and QC

All rows with a Chosen_ID already have a verified Veridion ID, country and city, since these were validated during the entity resolution process. For the remaining rows with no match, we will focus on analyzing them for potential patterns or inconsistencies.
During the QC process, I observed missing values in the dataset for rows without a Chosen_ID (No Match):

- 254 records from the database had no country specified compared to 53 records from the input.  
- 429 records from the database had no city specified compared to 494 records from the input.  
- All missing values occur only within the No Match records.  

During data quality checks, several anomalies were observed across all records in the dataset, regardless of whether a Chosen_ID was assigned:

- `revenue` contained text entries instead of numeric values in some records.  
- `employee_count` had unexpected values such as links or non-numeric text.  
- `num_locations` contained text entries that appear to be domain names or other non-numeric information.  
- `latitude` and `longitude` also contained text entries instead of numeric coordinates.  
- Some street numbers were incorrectly interpreted as dates by Excel (e.g., "5-2" = "5-Feb").  
- Missing or incomplete values were observed in many fields, including postal codes, emails, website URLs, and other company attributes.  
- Text fields like company names and descriptions occasionally contained special characters or were missing so these were noted but not corrected.  

