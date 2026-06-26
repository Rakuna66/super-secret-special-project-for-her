# 💖 A Playlist For You

A personal music player — a birthday gift built with love.

---

## How to set up (for the creator)

### 1. Add your songs
Drop your MP3 files into the `songs/` folder.

### 2. Add your photos
Drop your photos (JPG or PNG) into the `photos/` folder.

### 3. Edit the song list
Open `index.html` and find the `songs` array near the top of the `<script>` tag.
Fill in each entry:

```js
{
  title:  "Song Name",
  artist: "Artist",
  src:    "songs/yourfile.mp3",   // filename in songs/
  photo:  "photos/yourphoto.jpg", // filename in photos/ — or "" for emoji
  emoji:  "💖",                   // shown if no photo
  why:    "Your personal note here..."
}
```

Copy and paste the block to add more songs.

### 4. Push to GitHub

```bash
git init
git add .
git commit -m "a playlist for you 💖"
git remote add origin https://github.com/YOURUSERNAME/YOURREPO.git
git push -u origin main
```

### 5. Enable GitHub Pages
- Go to your repo → **Settings** → **Pages**
- Under *Branch*, select `main` and click **Save**
- Your site will be live at: `https://YOURUSERNAME.github.io/YOURREPO/`

### 6. Send her the link 💌
She opens it in Safari on her iPhone and it just works!

---

> *"The best gift is the one made with your own hands."*
