---
trigger: always_on
---

In this project, the official election results published by the Election Commission of Thailand (ECT) are primarily distributed as raw PDF documents organized by province and constituency.
Therefore, the objective of this project is to design a practical workflow that converts unstructured election documents into structured datasets, analyzes the data, and presents insights from the data via an analytical dashboard.
Because of the large volume of election data, the analysis will focus on selected provinces or constituencies as representative case studies.
The data is publicly available on the ECT website and stored as collections of PDF documents organized by province and electoral constituency.

The raw election data is published in several official document forms, representing different voting methods and ballot types:

• ส.ส.5/16 : Advance voting (in-district) — Constituency vote  
 • ส.ส.5/16 (บช) : Advance voting (in-district) — Party-list vote

• ส.ส.5/17 : Advance voting (out-of-district) — Constituency vote  
 • ส.ส.5/17 (บช) : Advance voting (out-of-district) — Party-list vote

• ส.ส.5/18 : Election day voting — Constituency vote  
 • ส.ส.5/18 (บช) : Election day voting — Party-list vote

Note: Forms 5/16 and 5/17 are at the constituency level, while Form 5/18 is at the individual polling station level.

The project should be a fully functional, end-to-end pipeline that demonstrates practical applications or yields insightful findings. The project must include at least the following 4 components:
Data Criteria: You must use the Election 2026 data (ECT ) and select ONE constituency with at least 250 polling stations (In this case we select อุทัยธานี เขต2) for OCR and analysis (You are required to perform OCR on the entire selected constituency).
If you are unfamiliar with constituencies, please refer to this link.
You can check the number of polling stations for each constituency here. This information may be helpful for your planning.
Moreover, your project will be more compelling if you incorporate data from additional sources (e.g., previous elections data) to enrich your analysis and insights.

Component 1: Data Preparation (Extraction / Cleaning / Validation):
This component includes dataset curation, data extraction (PDF → structured text), data cleaning, and validation. It ensures the correctness of the data and handles potential errors introduced during preprocessing.

Component 2: Data Science Analysis:
This component applies analytical methods to explore patterns, trends, and insights from the prepared dataset.

Component 3: BI Dashboard
Component 4: The outcome of your analysis (insights)