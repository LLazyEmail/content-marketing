# Images for LinkedIn Posts

This folder holds visuals for LinkedIn posts. It mirrors the Dropbox folder `/13 social-media`, with the same lettered subfolders (`A -` through `K -`, plus `EyesOnUkraine`).

Photos (JPG/JPEG, plus one WebP) already live here. Large videos do not: GitHub cannot take the full MP4 set, which is 1.58 GiB. Those files stay in Dropbox.

## Videos

See **[VIDEOS.md](VIDEOS.md)** for the complete MP4 inventory: Dropbox path, size, modified time, and a share link for every file. A few smaller videos are also in this folder and remain tracked in git.

New `.mp4` and `.mov` files are listed in the repo `.gitignore` so they are not added by accident. Git LFS is not enabled.

## How to use images in a post

1. Keep clear filenames, or use a slug that matches the post: `01-bakhmut-road-vehicle.jpg`.
2. Reference them in the post markdown:

   ```markdown
   ![Description](images/01-bakhmut-road-vehicle.jpg)
   ```

3. Prefer the copies in this folder for stills. For video, use the Dropbox share links in [VIDEOS.md](VIDEOS.md).
4. Keep the original Dropbox files until the GitHub versions are confirmed.

A longer filename convention is in [naming-convention.md](naming-convention.md).
