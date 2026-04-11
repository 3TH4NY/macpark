# MacPark - McMaster University Parking App

Interactive parking map for McMaster University campus.

## Features

- **Interactive Map** - OpenStreetMap with parking lot locations
- **GPS Navigation** - Driving directions to parking lots
- **PWA Ready** - Install on mobile devices
- **Smart Suggestions** - Find the best available spot

## Live Demo

Visit: `https://3th4ny.github.io/macpark/`

## Local Development

```bash
# From repository root
python3 -m http.server 8000
# Open http://localhost:8000
```

## Deployment

This app is deployed via GitHub Pages with automatic deployment on push to `main`.

### Setup

1. Push code to GitHub repository
2. Go to Settings → Pages
3. Set Source to "GitHub Actions"
4. The workflow automatically deploys on push

### Manual Deploy

Go to Actions → "Deploy to GitHub Pages" → "Run workflow"

## Tech Stack

- **Frontend**: Vanilla JS, Leaflet.js, OpenStreetMap
- **Routing**: OSRM API (driving directions)
- **Hosting**: GitHub Pages (free)

## Project Structure

```
macpark/
├── index.html           # Self-contained app (HTML/CSS/JS)
├── .github/workflows/   # GitHub Actions deployment
├── LICENSE
└── README.md
```

## Data

Parking lot information is embedded directly in the app. To update:

1. Edit the `META` object in `index.html`
2. Commit and push changes
3. GitHub Pages auto-deploys

## License

MIT License

## Links

- McMaster Parking: [parking.mcmaster.ca](https://parking.mcmaster.ca)
- OpenStreetMap: [openstreetmap.org](https://openstreetmap.org)
