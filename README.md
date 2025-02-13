# streams-web

A Flask web application for browsing and streaming a local collection of MP4 videos. It serves a browser interface, lists and searches videos with pagination, issues in-memory access tokens for selected videos, supports byte-range playback, and provides direct downloads. The repository also includes a separate helper script for downloading videos from VK.

## Web application

The Flask service reads video files from `./videos` and access keys from `keys.txt` in its working directory. Put one authorized key on each line of `keys.txt` and place MP4 files in the video directory. The root route serves the UI from `static/`.

Authenticated endpoints can list videos, count matching videos, and request a private-video token. The `/video/<filename>` route supports HTTP byte ranges so browser players can seek within a video. The `/video:download/<filename>` route sends a file as an attachment. Token state is held in process memory and is lost when the service restarts.

## Run locally

Install Flask in a virtual environment, create the video directory and key file, then launch the server:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install Flask
mkdir -p videos
printf '%s\n' 'replace-with-a-private-key' > keys.txt
python3 main.py
```

Replace the example key with a private value and do not commit `keys.txt`, uploaded videos, or private media. The application expects to be started from the repository directory because its video and key paths are relative to the current working directory.

## API behavior

- `GET /videos?offset=0&limit=10&query=...` lists matching MP4 files and their creation times.
- `GET /videos/count?query=...` returns a count for the optional name filter.
- `POST /generate-token` creates a token for an existing video.
- `GET /private-video?token=...` returns metadata for a token issued by this process.
- `GET /video/<filename>` streams the file and handles a byte-range request.
- `GET /video:download/<filename>` downloads a file as an attachment.

Send the configured access key in the `Authorization` header for protected endpoints. Review `main.py` and the browser code for exact JSON payloads.

## VK download helper

`vk_download.py` calls the VK video API and downloads available videos, preferring the highest listed MP4 quality. It can use `yt-dlp` when a direct quality URL is unavailable. Configure its access token, group ID, and destination folder before running it. The script depends on `requests` and `tqdm`, with `yt-dlp` optional.

## Deployment considerations

The service is a small self-hosted project. Before exposing it outside a trusted network, add HTTPS, validate filenames and range requests for your threat model, configure a stable media directory, and put the application behind an appropriate production server. Keep access keys and media private.
