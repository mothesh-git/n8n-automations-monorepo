# n8n Automations Monorepo

A collection of production-ready n8n workflows for lead generation, outreach, and business automation — ready to import and deploy.

---

## Workflows

| Workflow | Description | Trigger | Key Integrations |
|----------|-------------|---------|-----------------|
| [Lead Generation — Google Forms](./workflows/lead-gen-google-forms/) | Captures form leads, logs to a sheet, and sends a welcome email automatically | Google Sheets (poll) | Google Sheets · Gmail |

---

## Repository Structure

```
n8n-automations-monorepo/
└── workflows/
    └── lead-gen-google-forms/
        ├── lead-gen-google-forms.json   ← Import this into n8n
        └── README.md                    ← Setup & customisation guide
```

---

## Getting Started

### Prerequisites
- A self-hosted or cloud [n8n](https://n8n.io) instance (v1.0+)
- OAuth2 credentials configured for the services each workflow uses

### Import a Workflow
1. Open your n8n instance and go to **Workflows → Import from file**.
2. Select the `.json` file from the relevant workflow folder.
3. Follow the setup checklist in that workflow's `README.md` to wire up credentials and IDs.
4. **Activate** the workflow.

---

## Adding a New Workflow

1. Create a new folder under `workflows/` named after your workflow (use kebab-case).
2. Export your workflow from n8n as JSON and place it in the folder.
3. Add a `README.md` following the same structure as existing workflow docs.
4. Add a row to the **Workflows** table above.

---

## Contributing

Pull requests are welcome. Please keep each workflow self-contained in its own folder and include a clear README with setup steps.

---