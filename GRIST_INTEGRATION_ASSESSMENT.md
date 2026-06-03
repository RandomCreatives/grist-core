# Grist Integration Assessment for DigitalMehandis_V5.0

## 1. Executive Summary
Grist is highly suitable for integration into the **DigitalMehandis_V5.0 (EthioQS)** project. It provides a professional, browser-based "Excel-like" experience that satisfies the requirement for a seamless quantity surveying (QS) workbench without needing external software like MS Excel.

## 2. Key Integration Components

### A. Infrastructure (Docker Compose)
Add Grist as a service to your existing `docker-compose.yml`. Grist can share the same PostgreSQL instance for its management database.

```yaml
  grist:
    image: gristlabs/grist:latest
    ports:
      - "8484:8484"
    environment:
      - APP_HOME_URL=http://localhost:8484
      - GRIST_SINGLE_ORG=docs
      - GRIST_SUPPORT_ANON=true
      - TYPEORM_TYPE=postgres
      - TYPEORM_HOST=db
      - TYPEORM_DATABASE=grist_home
      - TYPEORM_USERNAME=ethioqs
      - TYPEORM_PASSWORD=ethioqs_secret
    volumes:
      - ./grist_data:/persist
```

### B. Backend Bridge (FastAPI)
Create a service in FastAPI to automate Grist via its REST API. When a user starts a project, the backend will:
1. Create a new Grist document by copying a pre-defined **QS Template**.
2. Store the resulting `docId` in the project's PostgreSQL record.
3. Push measurements from the PDF viewer directly into the Grist document using the `POST /api/docs/{docId}/tables/{tableId}/records` endpoint.

### C. Frontend Embedding (Next.js)
Embed the Grist workbook directly into your Project Dashboard using an iframe. This keeps the user within your application while providing full spreadsheet functionality.

```tsx
<iframe
  src={`http://localhost:8484/doc/${gristDocId}`}
  className="w-full h-[800px] border-none"
  allow="clipboard-read; clipboard-write"
/>
```

## 3. Recommended Workflow
1. **Template Design:** Create a "Master QS Template" in Grist with your preferred Ethiopian MoUDC columns, formulas, and pre-loaded Material Rates.
2. **Data Extraction:** When a user takes a measurement in your PDF tool, click "Send to BOQ".
3. **Automated Entry:** FastAPI sends this data to the Grist API, and the user sees the row appear instantly in the embedded spreadsheet.
4. **Finalization:** The user completes any manual adjustments in the spreadsheet. Grist handles all the math (subtotals, VAT, etc.) automatically.

## 4. Conclusion
Grist is perfectly aligned with the "Integrated Workbench" goal. It leverages your existing tech stack and provides the powerful calculation engine your users expect from Excel, fully in the browser.
