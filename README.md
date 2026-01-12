# POC_Simulation-
Entity Resolution and Data analysis and QC
## Part 1: Entity Resolution

The goal of this step was to resolve each input company to a real-world entity using the candidate matches provided by the client’s database. The process included the following steps:

1. **Data Simplification**  
   - Created two auxiliary columns: `input_simplified` and `db_name`, removing spaces, punctuation, slashes, and converting all text to uppercase to standardize the names.

2. **Name Matching**  
   - Compared simplified names using substring search; assigned `1` if there was a match, `0` otherwise.

3. **Country and City Matching**  
   - Country: exact match → `Yes` / `No`  
   - City: substring match to account for administrative variants (e.g., 'Augsburg municipality' vs 'Augsburg') → `Yes` / `No`

4. **Match Level Determination**  
   - High: name_match = 1 AND country = Yes AND city = Yes  
   - Medium: name_match = 1 AND (country = Yes OR city = Yes)  
   - Low: all other cases

5. **Chosen ID Assignment and Match Decision**  
   - High match → `Chosen_ID` assigned directly from Veridion ID  
   - Medium match → further checked city/country variants; valid matches assigned ID, flagged as `Match Found`  
   - Low match or country mismatch → automatically `No Match`

6. **Duplicate Handling**  
   - Duplicate `Chosen_ID`s were reviewed; only correct matches were kept. After removing duplicates, the final counts were: 333 `Match Found` and 2801 `No Match`.

7. **Summary**  
   - The majority of High match candidates were automatically resolved.  
   - Medium matches required additional checks for city variations.  
   - Low matches were discarded to avoid false positives.  
   - Administrative city variants were handled as soft matches to minimize manual review.

**Notes:**  
- All decisions were made following clear, consistent rules to allow scaling the process to larger datasets.  
- The workflow ensures that only validated IDs are used for further analysis in Part 2 of the POC.
