# Deploying the Gold semantic models to Fabric via Git

These 10 `*.SemanticModel` folders are the **Fabric/Git-importable** form of the
single-file `SM_<domain>.tmdl` models. Each folder is a complete semantic-model
item definition:

```
SM_<domain>.SemanticModel/
├── .platform                 # item metadata: type=SemanticModel, displayName, logicalId
├── definition.pbism          # version 4.0  -> TMDL format
└── definition/
    ├── database.tmdl         # compatibilityLevel 1604
    ├── model.tmdl            # model header + `ref table` ordering
    ├── expressions.tmdl      # GoldSqlEndpoint (Direct Lake connection)
    ├── relationships.tmdl    # all relationships
    └── tables/<table>.tmdl   # one file per table
```

Descriptions were converted from the `description:` property to canonical `///`
triple-slash comments (what the TMDL serializer expects). The `//` header
comments from the single-file version were removed (not valid TMDL syntax).

> A single `.tmdl` file cannot be imported by Git — Fabric imports the **folder**.
> This is why we repackage.

## Prerequisites
- Fabric workspace on a **Fabric capacity** (F SKU / Premium / trial) — Direct Lake requires it.
- The **Gold** warehouse deployed in the same workspace, so the Direct Lake SQL
  endpoint in `expressions.tmdl` resolves. (It is already bound to the real endpoint.)
- A Git provider supported by Fabric: **Azure DevOps (Azure Repos)** or **GitHub**.
  For Azure DevOps the repo must be in the same Entra tenant as the workspace.
- You are a **workspace Admin** (Member can commit but Admin is needed to connect).

## Step 1 — Put the folders in a Git repo
1. Create a repo (Azure DevOps project + repo, or a GitHub repo) and clone it locally.
2. Copy all 10 `SM_<domain>.SemanticModel` folders into the repo. Put them under a
   single directory you will point Fabric at — e.g. the repo root or `/fabric`.
3. Commit and push to a branch (e.g. `main`).

## Step 2 — Connect the workspace to Git
1. In the Fabric workspace: **Workspace settings → Git integration**.
2. Pick the provider, sign in / authorize.
3. Select **Organization / Project / Repository / Branch**, and set **Git folder**
   to the directory that contains the `.SemanticModel` folders.
4. Click **Connect and sync**.

## Step 3 — Import (sync) into the workspace
1. Fabric compares repo vs workspace and lists the 10 models as **incoming changes**.
2. Open the **Source control** panel → **Update all**. Fabric creates each
   semantic model item from its folder definition.
3. Confirm all 10 semantic models now appear in the workspace.

## Step 4 — Bind the connection & validate
1. Open each model → **Settings → Data source credentials / cloud connection** →
   sign in (Entra/OAuth) so Direct Lake can read from the Gold SQL endpoint.
2. Open the model in **web modeling** ("Open data model") to verify tables,
   relationships, measures, and descriptions.
3. Use **Explore this data** or build a report to smoke-test.

## Where "Fabric web modeling" fits
Web modeling (the browser "Open data model" editor, including its in-service
**TMDL view**) **edits an existing** model — add measures, relationships, tweak
properties. It does **not** create a model from an uploaded `.tmdl`. So Git (or the
REST API) is the *import* mechanism; web modeling is the *edit/verify* surface
after the item exists. Once imported, you can paste further TMDL changes into the
service TMDL view.

## If a sync errors
Fabric shows a per-item error. Common causes:
- `.platform` `type` casing wrong (must be exactly `SemanticModel`).
- Invalid TMDL (a stray property / indentation) — fix the file, commit, Update all.
- Direct Lake source not authenticated → set the cloud connection (Step 4).

## Alternative (lowest-risk) path
Deploy once with **Tabular Editor** over the workspace **XMLA endpoint**, then turn
on Git — Fabric serializes the perfectly-valid folder into the repo for you. Use
that if a hand-authored folder ever fails to sync.
