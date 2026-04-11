# MacPark - McMaster University Parking App

Real-time parking navigation for McMaster University campus.

## Features

- 🗺️ **Interactive Map** - OpenStreetMap with parking lot locations
- 📍 **GPS Navigation** - Walking directions to parking lots
- ⏱️ **Real-Time Availability** - Live parking spot counts
- 📱 **PWA Ready** - Install on mobile devices
- 🎯 **Smart Suggestions** - Find the best available spot

## Architecture

```
macpark/
├── frontend/          # Static HTML/CSS/JS (GitHub Pages)
│   ├── index.html
│   ├── css/
│   └── js/
└── backend/           # Express API (Railway/Render)
    ├── server.js
    ├── routes/
    └── parking.json
```

## Local Development

### Backend API

```bash
cd backend
npm install
npm start
# API runs on http://localhost:3001
```

### Frontend

```bash
# From repository root
python3 -m http.server 8000
# Open http://localhost:8000/frontend/
```

## Deployment

### Option 1: GitHub Pages + Railway (Recommended)

**Frontend (GitHub Pages):**
1. Push code to GitHub repository
2. Go to Settings → Pages
3. Select source: `main` branch, `/frontend` folder
4. Access at: `https://username.github.io/macpark`

**Backend (Railway):**
1. Go to [railway.app](https://railway.app)
2. Connect GitHub repository
3. Select `backend/` as root directory
4. Add environment variable: `PORT=3001`
5. Railway provides URL like: `https://macpark-api.railway.app`

**Update API endpoint in frontend:**
Change `http://localhost:3001` to your Railway URL in `frontend/js/app.js`

### Option 2: GitHub Pages + Render

**Backend (Render):**
1. Go to [render.com](https://render.com)
2. Create new Web Service
3. Connect GitHub repository
4. Settings:
   - Root Directory: `backend`
   - Build Command: `npm install`
   - Start Command: `npm start`
5. Free tier available (spins down on idle)

### Option 3: GitHub Pages + Fly.io

**Backend (Fly.io):**
1. Install: `curl -L https://fly.io/install.sh | sh`
2. Login: `fly auth login`
3. Launch: `fly launch` (in backend directory)
4. Deploy: `fly deploy`

## API Endpoints

- `GET /api/lots` - All parking lots with availability
- `GET /api/lots/:id` - Single lot details
- `POST /api/lots/:id/update` - Update availability (admin)
- `GET /health` - Health check

## Environment Variables

Backend:
- `PORT` - Server port (default: 3001)

Frontend:
- Update API URL in `js/app.js` for production

## Tech Stack

- **Frontend**: Vanilla JS, Leaflet.js, OpenStreetMap
- **Backend**: Node.js, Express
- **Database**: JSON file (SQLite alternative)
- **Routing**: OSRM API (walking directions)

## Cost

- Frontend (GitHub Pages): **Free**
- Backend (Railway/Render): **Free tier**
- External APIs (OSRM, OSM): **Free**

**Total: $0/month**

## License

MIT License

## Contributing

1. Fork repository
2. Create feature branch
3. Submit pull request

## Support

- GitHub Issues: [macpark/issues](https://github.com/yourusername/macpark/issues)
- McMaster Parking: [parking.mcmaster.ca](https://parking.mcmaster.ca)
