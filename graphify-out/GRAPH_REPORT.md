# Graph Report - .  (2026-04-14)

## Corpus Check
- Corpus is ~213 words - fits in a single context window. You may not need a graph.

## Summary
- 19 nodes · 21 edges · 5 communities detected
- Extraction: 71% EXTRACTED · 29% INFERRED · 0% AMBIGUOUS · INFERRED: 6 edges (avg confidence: 0.87)
- Token cost: 0 input · 0 output

## Community Hubs (Navigation)
- [[_COMMUNITY_Core Application|Core Application]]
- [[_COMMUNITY_Frontend & Map|Frontend & Map]]
- [[_COMMUNITY_Deployment & CICD|Deployment & CI/CD]]
- [[_COMMUNITY_External Resources|External Resources]]
- [[_COMMUNITY_GPS & Routing|GPS & Routing]]

## God Nodes (most connected - your core abstractions)
1. `MacPark - McMaster University Parking App` - 11 edges
2. `Interactive Map Feature` - 3 edges
3. `GitHub Pages Hosting` - 3 edges
4. `GitHub Actions Auto-Deployment` - 3 edges
5. `MacPark README Documentation` - 2 edges
6. `GPS Navigation Feature` - 2 edges
7. `Leaflet.js Mapping Library` - 2 edges
8. `OpenStreetMap` - 2 edges
9. `Vanilla JavaScript` - 2 edges
10. `Manual Deployment Process` - 2 edges

## Surprising Connections (you probably didn't know these)
- `MacPark - McMaster University Parking App` --uses_deployment--> `GitHub Actions Auto-Deployment`  [INFERRED]
  README.md → README.md  _Bridges community 0 → community 2_
- `MacPark - McMaster University Parking App` --implements--> `Interactive Map Feature`  [EXTRACTED]
  README.md → README.md  _Bridges community 0 → community 1_
- `MacPark - McMaster University Parking App` --implements--> `GPS Navigation Feature`  [EXTRACTED]
  README.md → README.md  _Bridges community 0 → community 4_
- `Interactive Map Feature` --uses--> `OpenStreetMap`  [EXTRACTED]
  README.md → README.md  _Bridges community 1 → community 3_

## Hyperedges (group relationships)
- **Frontend Technology Stack** — tech_vanilla_js, tech_leaflet_js, tech_openstreetmap [INFERRED 0.90]
- **GitHub Pages Deployment Pipeline** — project_workflows, deployment_github_actions, tech_github_pages [INFERRED 0.85]

## Communities

### Community 0 - "Core Application"
Cohesion: 0.33
Nodes (6): META Object Parking Data, Progressive Web App Ready, Smart Suggestions Feature, MIT License, MacPark - McMaster University Parking App, Python HTTP Server

### Community 1 - "Frontend & Map"
Cohesion: 0.5
Nodes (4): Interactive Map Feature, index.html Self-Contained App, Leaflet.js Mapping Library, Vanilla JavaScript

### Community 2 - "Deployment & CI/CD"
Cohesion: 0.5
Nodes (4): GitHub Actions Auto-Deployment, Manual Deployment Process, GitHub Actions Workflows, GitHub Pages Hosting

### Community 3 - "External Resources"
Cohesion: 0.67
Nodes (3): McMaster Official Parking, MacPark README Documentation, OpenStreetMap

### Community 4 - "GPS & Routing"
Cohesion: 1.0
Nodes (2): GPS Navigation Feature, OSRM API

## Knowledge Gaps
- **8 isolated node(s):** `Progressive Web App Ready`, `Smart Suggestions Feature`, `OSRM API`, `Python HTTP Server`, `GitHub Actions Workflows` (+3 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **Thin community `GPS & Routing`** (2 nodes): `GPS Navigation Feature`, `OSRM API`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `MacPark - McMaster University Parking App` connect `Core Application` to `Frontend & Map`, `Deployment & CI/CD`, `GPS & Routing`?**
  _High betweenness centrality (0.850) - this node is a cross-community bridge._
- **Why does `Interactive Map Feature` connect `Frontend & Map` to `Core Application`, `External Resources`?**
  _High betweenness centrality (0.373) - this node is a cross-community bridge._
- **Why does `OpenStreetMap` connect `External Resources` to `Frontend & Map`?**
  _High betweenness centrality (0.209) - this node is a cross-community bridge._
- **Are the 2 inferred relationships involving `MacPark - McMaster University Parking App` (e.g. with `GitHub Actions Auto-Deployment` and `Manual Deployment Process`) actually correct?**
  _`MacPark - McMaster University Parking App` has 2 INFERRED edges - model-reasoned connections that need verification._
- **Are the 2 inferred relationships involving `GitHub Actions Auto-Deployment` (e.g. with `MacPark - McMaster University Parking App` and `GitHub Actions Workflows`) actually correct?**
  _`GitHub Actions Auto-Deployment` has 2 INFERRED edges - model-reasoned connections that need verification._
- **What connects `Progressive Web App Ready`, `Smart Suggestions Feature`, `OSRM API` to the rest of the system?**
  _8 weakly-connected nodes found - possible documentation gaps or missing edges._