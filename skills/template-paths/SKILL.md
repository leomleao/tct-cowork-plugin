---
description: File paths and naming conventions for TCT quote templates and output documents
---

# Template paths and output conventions

## Tokenised template files
| Document type | Path |
|---|---|
| Quote | C:\Users\LeonardoLeao\OneDrive - The Config Team\Desktop\Clients\Quote template TOKENISED.docx |
| FTS | (to be added) |
| Unit Tests | (to be added) |

## Output root
C:\Users\LeonardoLeao\OneDrive - The Config Team\Desktop\Clients\

## Client folder convention
Each client has a folder named after their ticket prefix (the part before the hyphen).
- TCTRAT-1252 → Clients\TCTRAT\
- TCTLAOR-30  → Clients\TCTLAOR\

## Output folder and filename convention
- Derive the client prefix by taking everything before the first hyphen in TICKET_ID.
- Create a subfolder inside the client folder: [PREFIX]\[TICKET_ID] - [CHANGE_TITLE]\
- File name: [TICKET_ID]_Quote_[YYYY-MM-DD].docx
- Example: Clients\TCTLAOR\TCTLAOR-30 - Implement BAdI for item category\TCTLAOR-30_Quote_2026-03-20.docx

## Fixed values
- Prepared by: Leonardo Leao
- Quote expiry: Issued + 28 days (do not calculate a specific date)

## Reference example (quality and style benchmark)
C:\Users\LeonardoLeao\OneDrive - The Config Team\Desktop\Clients\TCTLAOR-30 DMND0004968.docx
