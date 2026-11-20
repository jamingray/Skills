---
name: yt-dlp
description: >-
  Use when the user asks to download a YouTube video from a URL, extract audio,
  subtitles, clips, or metadata from YouTube, or when an agent needs to download
  a YouTube video locally using the workspace yt-dlp executable.
---

# yt-dlp

Download YouTube videos, audio, subtitles, clips, and metadata using the local `yt-dlp` binary.
Reference documentation: https://github.com/yt-dlp/yt-dlp

## Executable Path

Always invoke the hardcoded local executable:

```powershell
$ytdlp = "C:\Users\jamin\AI projects\Video\tools\yt-dlp.exe"
& $ytdlp <OPTIONS> "<URL>"
```

Or inline in PowerShell (the `&` call operator and quotes are **required** because the path contains a space):

```powershell
& "C:\Users\jamin\AI projects\Video\tools\yt-dlp.exe" <OPTIONS> "<URL>"
```

## Quick Reference

| Goal | Flags |
| --- | --- |
| Best video + audio (default) | `--no-playlist "<URL>"` |
| Best MP4 (H.264/AAC compatible) | `-t mp4 --no-playlist "<URL>"` |
| Cap resolution (e.g., 1080p) | `-S "res:1080" --no-playlist "<URL>"` |
| Audio only (MP3) | `-t mp3 --no-playlist "<URL>"` |
| Audio only (custom format) | `-x --audio-format m4a --audio-quality 0 "<URL>"` |
| Download into specific folder | `-P "<DIR>" "<URL>"` |
| Custom filename template | `-o "%(title)s.%(ext)s" "<URL>"` |
| Download specific time range | `--download-sections "*00:01:00-00:02:30" "<URL>"` |
| Download subtitles (no video) | `--skip-download --write-subs --write-auto-subs --sub-langs "en.*" "<URL>"` |
| List available formats | `-F "<URL>"` |
| Print metadata (no download) | `--skip-download -O "%(title)s | %(duration_string)s | %(uploader)s" "<URL>"` |
| Print final saved file path | `--print after_move:filepath "<URL>"` |
| Download entire playlist | `--yes-playlist -o "%(playlist_index)02d - %(title)s [%(id)s].%(ext)s" "<URL>"` |
| Update yt-dlp binary | `-U` |

## Standard Workflow

1. **Quote both the binary path and the URL.** YouTube URLs frequently contain `?` and `&`, which PowerShell interprets as operators if unquoted.
2. **Default to `--no-playlist`** whenever the user asks for a single video, in case the URL includes `&list=...`. Only omit it (or pass `--yes-playlist`) when the user explicitly wants the playlist.
3. **Specify the output directory with `-P`** (for example `-P "C:\Users\jamin\AI projects\Video\downloads"`) so downloads do not clutter the workspace root or `tools\` directory.
4. **Capture the resulting file path** by adding `--print after_move:filepath` (or `--print filename` with `--simulate` for a dry run).
5. **Verify completion**: confirm exit code `0` and verify the downloaded file exists and has a non-zero size before reporting success.

## Examples

### Download a single video as MP4 to a target folder

```powershell
& "C:\Users\jamin\AI projects\Video\tools\yt-dlp.exe" `
  -t mp4 `
  --no-playlist `
  -P "C:\Users\jamin\AI projects\Video\downloads" `
  --print after_move:filepath `
  "https://www.youtube.com/watch?v=VIDEO_ID"
```

### Download a 1080p clip with English subtitles

```powershell
& "C:\Users\jamin\AI projects\Video\tools\yt-dlp.exe" `
  -S "res:1080" `
  --download-sections "*00:01:15-00:02:00" `
  --write-subs --write-auto-subs --sub-langs "en.*" `
  --no-playlist `
  -P "C:\Users\jamin\AI projects\Video\downloads" `
  "https://www.youtube.com/watch?v=VIDEO_ID"
```

### Inspect available formats and metadata without downloading

```powershell
& "C:\Users\jamin\AI projects\Video\tools\yt-dlp.exe" `
  -F --no-playlist "https://www.youtube.com/watch?v=VIDEO_ID"
```

## Key Options Reference

### Format Selection (`-f`, `-S`, `-t`)

- Default selector is `bv*+ba/b` (best video + best audio merged, falling back to best pre-merged format).
- Use `-f <format_id>` or `-f <video_id>+<audio_id>` after inspecting `-F`.
- Use `-S` (`--format-sort`) to prioritize codecs, resolution, or size, e.g. `-S "res:1080,ext:mp4:m4a"`.
- Built-in preset aliases (`-t` / `--preset-alias`): `mp3`, `aac`, `mp4`, `mkv`, `sleep`.

### Output Templates (`-o`) and Paths (`-P`)

- Default template: `%(title)s [%(id)s].%(ext)s`.
- Common fields: `%(title)s`, `%(id)s`, `%(uploader)s`, `%(upload_date)s`, `%(duration)s`, `%(ext)s`, `%(playlist_index)s`.
- Length cap / fallback: `%(title).80s`, `%(uploader|Unknown)s`.
- `-P` is ignored if `-o` is an absolute path; prefer `-P "<dir>" -o "<template>"`.
- Use `--restrict-filenames` for ASCII-only filenames without spaces or `&` (useful when passing files to downstream scripts). `--windows-filenames` is applied on Windows by default.

### Post-Processing & FFmpeg Dependency

The following features require `ffmpeg` (and `ffprobe`) on `PATH` or passed via `--ffmpeg-location "<path>"`:
- Merging separate video and audio streams (`bv*+ba`)
- Audio extraction (`-x`, `-t mp3`, `-t aac`)
- Container remuxing/recoding (`--merge-output-format`, `--remux-video`, `--recode-video`, `-t mp4`, `-t mkv`)
- Downloading time ranges (`--download-sections`)
- Embedding thumbnails, subtitles, or metadata (`--embed-thumbnail`, `--embed-subs`, `--embed-metadata`)

If `ffmpeg` is not installed and a merge/post-process step fails, either:
- Download a pre-merged single stream with `-f b` (usually 360p/720p MP4/WebM), or
- Inform the user that `ffmpeg` is required for the requested format/post-processing.

## Common Mistakes

- **Missing `&` in PowerShell**: Running `"C:\Users\jamin\AI projects\Video\tools\yt-dlp.exe" ...` without `&` fails in PowerShell because quoted strings are not executed. Always prefix with `&`.
- **Unquoted YouTube URLs**: URLs containing `&list=...` or `&t=30s` break in PowerShell unless wrapped in quotes.
- **Unintended playlist downloads**: A URL copied from a playlist (`watch?v=...&list=...`) downloads the entire playlist unless `--no-playlist` is passed.
- **Stale binary breaking YouTube downloads**: YouTube frequently changes player/API signatures. If downloads fail with extractor or HTTP 403 errors, update the binary with `& "C:\Users\jamin\AI projects\Video\tools\yt-dlp.exe" -U`.
- **Browser cookies without consent**: Age-restricted or private videos may require `--cookies-from-browser <browser>`. Ask the user before reading their browser cookies.
