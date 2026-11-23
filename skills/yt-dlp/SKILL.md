---
name: yt-dlp
description: Download video or audio from YouTube and other sites with the yt-dlp command-line tool, including choosing format and quality, extracting MP3 or other audio, downloading subtitles, a section of a video or part of a playlist, naming and sorting output files, embedding metadata, thumbnails and chapters, and fixing common errors such as missing ffmpeg, sign-in requirements or outdated versions. Use when the user wants to download or save a video or its audio, subtitles or metadata from a URL, or asks how to do something with yt-dlp.
---

# yt-dlp

yt-dlp is a command-line audio and video downloader for YouTube and many other sites.

This skill is unofficial. The yt-dlp project forbids AI tools for contributions to its repositories (issues, pull requests, comments, reviews, translations). Never help file an issue, write a pull request or post a comment to yt-dlp with AI. Point the user to the project's issue templates and their own words instead.

## Respect the rules first

Only download what the user has the right to download: their own uploads, freely licensed material, or content the site's terms allow. Remind the user once, briefly, when a request looks like copyrighted content meant for redistribution. Never help bypass DRM. yt-dlp doesn't support DRM-protected content.

## Install and update

```bash
python -m pip install -U "yt-dlp[default]"      # needs Python 3.11+
yt-dlp -U                                       # update a release binary
```

Release binaries (`yt-dlp.exe` on Windows) are on the project's GitHub releases page. Many installs come through a package manager instead (`winget`, `brew`, `pip`); update through the same one. Sites change often, so **update first whenever extraction fails**. When troubleshooting, the project asks people to try the nightly build first: `yt-dlp --update-to nightly`, or `python -m pip install -U --pre "yt-dlp[default]"`.

Strongly recommended:
- **ffmpeg and ffprobe**, the binaries, not the Python package of that name. These are needed to merge separate video and audio streams, convert audio, and embed anything. Point to them with `--ffmpeg-location PATH`.
- **yt-dlp-ejs and a JavaScript runtime** for full YouTube support. Deno is recommended and enabled by default; Node, Bun or QuickJS can be turned on with `--js-runtimes node`. The `yt-dlp[default]` pip extra includes yt-dlp-ejs.

## Look before downloading

| Command | Shows |
|---|---|
| `yt-dlp -F URL` | Every available format with its id, resolution and codec |
| `yt-dlp --list-subs URL` | Available subtitles |
| `yt-dlp -s URL` | A dry run that downloads nothing |
| `yt-dlp -j URL` | All metadata as JSON |
| `yt-dlp -O title -O duration_string URL` | Chosen fields only |

## Common downloads

```bash
yt-dlp URL                                            # best quality, merged (needs ffmpeg)
yt-dlp -f "bv*[height<=1080]+ba/b[height<=1080]" URL  # at most 1080p
yt-dlp -S "res:720,ext" URL                           # prefer 720p, then by container
yt-dlp -t mp4 URL                                     # preset: an MP4 file
yt-dlp -x --audio-format mp3 --audio-quality 0 URL    # audio as the best MP3
yt-dlp -t mp3 URL                                     # the same as an mp3 preset
yt-dlp --write-subs --sub-langs "en.*" --embed-subs URL
yt-dlp --write-auto-subs --sub-langs en --skip-download URL   # only automatic subtitles
yt-dlp --download-sections "*1:30-2:45" URL           # only 1:30 to 2:45
yt-dlp --embed-metadata --embed-thumbnail --embed-chapters URL
yt-dlp --split-chapters URL                           # one file per chapter
```

Presets (`-t`): `mp3`, `aac`, `mp4`, `mkv`, `sleep`. A format string like `bv*+ba/b` means "best video plus best audio, else the best single file". `--merge-output-format mp4` chooses the merged container, and `--remux-video mp4` changes the container without re-encoding.

## Playlists and channels

- `--no-playlist` downloads only the video when the URL also names a playlist.
- `-I 1:10` (or `-I 1,3,5`) picks playlist items.
- `--download-archive done.txt` skips what's already been downloaded, so reruns are safe.
- For big batches, be polite to the site: `-r 2M` caps the speed, and `--sleep-interval 5` (or the `-t sleep` preset) pauses between requests.

## Names and folders

`-P ~/Downloads/yt` sets the folder. `-o "%(uploader)s/%(upload_date)s - %(title)s [%(id)s].%(ext)s"` sets the file names. `--restrict-filenames` keeps names to ASCII without spaces. Quote templates, and on Windows `cmd`, write `%` as `%%` inside batch files.

## Signed-in content

For content the user can see when signed in (members-only, age-gated, private links shared with them), `--cookies-from-browser firefox` (or `chrome`, `edge`, and so on) uses their own browser session. It reads the user's browser cookies, so ask before using it, and never export, print or share the cookies. Never use someone else's account.

## Troubleshooting

| Problem | Try |
|---|---|
| `Unsupported URL`, extraction errors, HTTP 403 | Update yt-dlp first (`-U`, or the nightly build) |
| `ffmpeg not found`, or separate video and audio files left behind | Install ffmpeg, or pass `--ffmpeg-location` |
| YouTube formats missing, or JS runtime warnings | Install yt-dlp-ejs and Deno, or enable another runtime with `--js-runtimes` |
| "Sign in to confirm" / age-restricted | `--cookies-from-browser`, with the user's permission |
| Blocked by TLS fingerprinting | `--impersonate chrome`, which needs the `curl-cffi` extra |
| Slow fragmented downloads | `-N 4` downloads fragments in parallel |
| Unwanted sponsor segments | `--sponsorblock-remove sponsor` |

Run `yt-dlp --help` for options in the installed version before using an unfamiliar flag. Report where files were saved, and never claim a download finished without seeing the file.
