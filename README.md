All Node syntax passes; the only "supabase" string left in the web app is a comment in api.ts.

The complete stack is built, zipped, and ready to download.

What's inside /mnt/documents/railway-nvr-stack/:

server/ — Node.js API (no Supabase). pg to Postgres, JWT + bcrypt auth against mnvr_schema.users, full route surface (auth, cameras, recordings, events, users, config, streaming, hls, stats, ai), WebSocket realtime at /ws, Swagger at /api/docs.
server/src/routes/hls.js — adaptive HLS: serves cam_<id>/master.m3u8 (or synthesises one from low/mid/high/index.m3u8, or wraps a single stream.m3u8), rewrites every segment URL to be token-authenticated so <video>/hls.js work without custom headers.
server/src/routes/streaming.js — MP4 range streaming + download + thumbnail, jailed under STORAGE_RECORDINGS.
server/sql/mnvr_schema.sql — nvr_core's schema verbatim (source of truth).
server/ai/sidecar.py — optional FastAPI YOLO sidecar; the Node /api/ai/detect proxies to it.
web/ — the React app with every Supabase call replaced. New src/lib/api.ts (fetch + auto-refresh), new AuthContext, rewritten hooks (use-cameras, use-recordings, use-events, use-users, use-system-config, use-dashboard-stats), WebSocket-driven NotificationsDropdown, plus a new HlsPlayer.tsx using hls.js for ABR.
deploy/ — mnvr-api.service, mnvr-ai.service, mnvr-web.nginx.conf.
scripts/install.sh — one-shot installer for fresh Debian/Ubuntu: installs Postgres + Node 20 + nginx, imports the schema, builds the web app, registers systemd units, configures nginx, and creates an admin / Admin@123 user.
README.md — architecture diagram, full API table, manual-dev steps, HLS layout matrix, ops notes.
To deploy on your VM (where mnvrd runs):


sudo cp -r ./railway-nvr-stack /opt/mnvr-api/
sudo bash /opt/mnvr-api/scripts/install.sh
Then point mnvrd's mnvr.conf at the same Postgres (db_path) and same storage_base / hls_base. Visit http://<host>/, log in as admin / Admin@123, change the password.





i Want you to make it a professional looking and working website, see firstly in camera section where there is grid of cameras have options of display as 1*1,2*2,3*3and 4*4, and update the dashboard to make it actually work, the website has lots of placeholder code i want to make it 100% working, the engine or the logic for this is in nvr-core-main, i want you to integrate both of these codebases 100% and there is a section where we are adding image for object detection and all other ai features, remove this and do the inbuilt ai feature detection while playing the videos on the frontend by taking the videos stored in storage/hls and storage/recordings, you have to take the exact path from the db and constantly get via the apis, and there are 2 streams from the nvr core, first for recording and the second for ai features, make sure all the camera grids display the live stream of cameras, you youself figure out how to do, and the segmented videos are also recorded, do any other relevant changes, but please ensure that every single feature on the frontend is correctly linked with backend and the nvr core, test it thorughly and give me the updated code in downloadable format no matter how much is done, just give the code

Honest caveats (so nothing surprises you):

I can't run Postgres / mnvrd / GStreamer here, so I verified Node syntax on every file but did not execute end-to-end against live cameras — that requires your VM.
HLS multi-rendition output requires mnvrd's HLS module to be writing low/mid/high directories. If it currently only writes single stream.m3u8, the Node API automatically wraps it in a synthesised master so the client flow still works (no real ABR until mnvrd produces multiple renditions).
The AI sidecar needs pip install ultralytics (large download); install script tolerates failure and /api/ai/detect will return 502 until it's installed.
