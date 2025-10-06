# ⚙️ YouTube Focus — Backend (Node.js + Express)

This Node.js + Express backend fetches YouTube playlist data using the YouTube Data API v3.  
It keeps your API key on the server (never exposed to the frontend) and serves a simple JSON endpoint for the extension.

---

## Tech stack

- Node.js (v16+ recommended)
- Express
- axios (or fetch)
- dotenv
- cors
- Hosting: Render (recommended for easy deploy)

---

## Prerequisites

- Node.js & npm installed locally
- A YouTube Data API key (create via Google Cloud Console: enable YouTube Data API v3 → create API key)
- A GitHub account (for Render deployment)

---

## Folder structure

youtube-focus-backend/
├── server.js
├── package.json
├── package-lock.json
├── .env # local only — DO NOT COMMIT
├── .gitignore
└── README.md


---

## server.js (what you should have — example)

> If your `server.js` uses `import` syntax you must set `"type": "module"` in `package.json`.
> Alternatively, use `require(...)` CommonJS style.

```js
import express from "express";
import axios from "axios";
import cors from "cors";
import dotenv from "dotenv";

dotenv.config();
const app = express();
app.use(cors());
app.use(express.json());

const API_KEY = process.env.YOUTUBE_API_KEY;

// health
app.get("/", (req, res) => res.send("YouTube Focus Backend is running"));

// playlist endpoint
app.get("/api/playlist", async (req, res) => {
  const playlistId = req.query.id;
  if (!playlistId) return res.status(400).json({ error: "Missing playlist ID" });

  try {
    const response = await axios.get("https://www.googleapis.com/youtube/v3/playlistItems", {
      params: {
        part: "snippet",
        maxResults: 50,
        playlistId,
        key: API_KEY,
      },
    });

    const videos = (response.data.items || []).map(item => ({
      title: item.snippet.title,
      videoId: item.snippet.resourceId.videoId,
      thumbnail: item.snippet.thumbnails?.medium?.url || item.snippet.thumbnails?.default?.url,
    }));

    res.json({ videos });
  } catch (err) {
    console.error(err.response?.data || err.message);
    res.status(500).json({ error: "Failed to fetch playlist" });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`✅ Server running on port ${PORT}`));
```

## Create .env in project root:
```
YOUTUBE_API_KEY=YOUR_YOUTUBE_DATA_API_KEY_HERE
PORT=3000
```
##(IMP) Add .env to .gitignore (never commit your key).
```
node_modules/
.env
npm-debug.log*
```
## Run locally (development)
```
cd path/to/youtube-focus-backend
npm install
```
# create .env with YOUTUBE_API_KEY
```
node server.js
```
# or use nodemon
```
npx nodemon server.js
```
## Visit: http://localhost:3000/api/playlist?id=PLAYLIST_ID to test.

## Security & privacy notes

Keep YOUTUBE_API_KEY secret; never include it in frontend code.

For public/shared deployments consider implementing rate limiting or simple auth to avoid abuse of your API key


##License
MIT License © 2025
