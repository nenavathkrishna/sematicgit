# SalesPOC — Sample Power BI Semantic Model (model-only, Git-ready)

This is a tiny, self-contained **semantic model** (no report) for the Git version-control POC.
It is provided in the "split" text format so you can commit it, diff it, branch it and merge it.

## What's inside

```
SalesPOC/
├─ README.md
├─ .gitignore
├─ Model.bim                         ← single-file TMSL (fallback, guaranteed to open in Tabular Editor 2)
└─ SalesPOC.SemanticModel/           ← the Fabric/PBIP semantic-model item (commit THIS)
   ├─ .platform
   ├─ definition.pbism
   └─ definition/
      ├─ database.tmdl
      ├─ model.tmdl
      ├─ relationships.tmdl
      └─ tables/
         ├─ FactSales.tmdl
         ├─ DimProduct.tmdl
         └─ DimDate.tmdl
```

## The model

- **FactSales** — Date, ProductID, Amount (6 sample rows, inline via Power Query `#table`).
  Measures: `Total Sales`, `Avg Sale`, `Sales YTD`.
- **DimProduct** — ProductID, Product, Category (3 rows).
- **DimDate** — 120 dates from 2026-01-01, with Year / MonthNo / MonthName (marked as a date table).
- **Relationships** — FactSales[ProductID] → DimProduct[ProductID]; FactSales[Date] → DimDate[Date].

All data is generated inside the model (Power Query), so there is **no external data source** and
the whole model is a few KB of text — the point of the POC.

## Open / edit it

Pick whichever tool you have:

- **Tabular Editor 2 (free):** File ▸ Open ▸ File ▸ `Model.bim`.
  To regenerate the split TMDL files from it: File ▸ Save to TMDL Folder.
- **Tabular Editor 3 / Fabric:** open the `SalesPOC.SemanticModel/definition` TMDL folder directly.
- **Power BI Desktop:** Desktop opens *projects* (report+model). To view this model-only item in
  Desktop, either connect a new report to it after you deploy it (below), or open `Model.bim` in
  Tabular Editor and Deploy.

## Deploy it to make it a live semantic model

- **Tabular Editor:** Model ▸ Deploy… → point at a Power BI Premium/Fabric workspace XMLA endpoint
  (`powerbi://api.powerbi.com/v1.0/myorg/<WorkspaceName>`) → deploy. It appears as a semantic model.
- **Fabric Git integration:** commit this folder to a repo, connect the workspace to that repo/branch
  (Workspace settings ▸ Git integration), then **Update** the workspace — the semantic model is created
  from the TMDL.

## Put it under Git (run in the `SalesPOC` folder)

```bash
git init
git add .
git commit -m "POC: SalesPOC sample semantic model (TMDL)"
git branch -M main
git remote add origin https://github.com/<you>/<repo>.git   # or your Azure DevOps repo
git push -u origin main
```

## Branch + merge POC

```bash
git checkout -b feature/add-margin
```
Edit `SalesPOC.SemanticModel/definition/tables/FactSales.tmdl` and add a measure under the others:
```
	measure 'Total Margin' = SUMX(FactSales, FactSales[Amount] * 0.3)
		formatString: \$#,##0
		lineageTag: 00000000-0000-0000-0000-000000000001
```
```bash
git commit -am "Add Total Margin measure"
git push -u origin feature/add-margin
# open a Pull Request and merge to main
```
Edit a **different** file on another branch (e.g. add a `Category Group` column in `DimProduct.tmdl`)
and both merge automatically. Make two branches edit the **same** measure line to see a normal Git
conflict you resolve by hand — that's the whole lesson: different objects merge on their own, the
same object needs a human.

> Note: `lineageTag` values just need to be unique GUIDs. When you add objects by hand, give them a
> fresh GUID (or let Tabular Editor/Desktop assign them on first save).
