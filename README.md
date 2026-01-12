# POC_Simulation-
Entity Resolution and Data analysis and QC
## Part 1: Entity Resolution

The goal of this step was to resolve each input company to a real-world entity using the candidate matches provided by the client’s database. The process included the following steps:

## Part 1: Entity Resolution

The goal of this step was to resolve each input company to a real-world entity using the candidate matches provided by the client’s database. The process included the following steps:

1. **Data Simplification**  
    Created two auxiliary columns: `input_simplified` and `db_name`, removing spaces, punctuation, slashes, and converting all text to uppercase to standardize the names.  

2. **Name Matching**  
    Compared simplified names using substring search; assigned `1` if there was a match, `0` otherwise.  

3. **Country and City Matching**  
    Country: exact match → `Yes` / `No`  
    City: substring match to account for administrative variants (e.g., 'Augsburg municipality' vs 'Augsburg') → `Yes` / `No`  

4. **Match Level Determination**  
    High: name_match = 1 AND country = Yes AND city = Yes  
    Medium: name_match = 1 AND (country = Yes OR city = Yes)  
    Low: all other cases  
    Formula:  
     ```excel
     =IF(AND(W7=1;X7="Yes";Y7="Yes");"High";IF(AND(W7=1;OR(X7="Yes";Y7="Yes"));"Medium";"Low"))
     ```

5. **Chosen ID Assignment and Match Decision**  
    High match → `Chosen_ID` assigned directly from Veridion ID  
    Medium match → further checked city/country variants; valid matches assigned ID, flagged as `Match Found`  
    Low match or country mismatch → automatically `No Match`  
    Match Found / No Match column formula:  
     ```excel
     =IF(AA2<>"";"Match Found";"No Match")
     ```

6. **Duplicate Handling**  
    Duplicate `Chosen_ID`s were reviewed; only correct matches were kept.  
    After removing duplicates, the final counts were: 318 `Match Found` (and 333 before removing)  and 2801 `No Match`.

7. **Summary**  
    The majority of High match candidates were automatically resolved.  
    Medium matches required additional checks for city variations.  
   Low matches were excluded to prevent incorrect or unreliable matches.
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
- All decisions were made following clear, consistent rules to allow scaling the process to larger datasets.  
- The workflow ensures that only validated IDs are used for further analysis in Part 2 of the POC.  
- Medium matches may require additional manual review for ambiguous city variations.

7. **Summary**  
    The majority of High match candidates were automatically resolved.  
    Medium matches required additional checks for city variations.  
    Low matches were excluded to prevent incorrect matches.
    Administrative city variants were handled as soft matches to minimize manual review.

