# ytx-archive-filter
yt-dlp's archive-file based seen/watched videos filter from picker/results for yt-x

A POSIX-shell extension that checks each result’s video ID against yt-x’s archive files and removes matches before they reach the fzf selector. yt-x supports sourced extensions and autoloading them from its config. This extension uses only the archive file you specify—it does not read or update yt-x’s recent/watch history. It intercepts yt-dlp’s flat-playlist JSON, removes entries whose extractor and ID appear in your file, then passes the filtered JSON back to yt-x. yt-x supports POSIX-shell extensions and automatic loading; yt-dlp archive entries use extractor/ID pairs such as youtube VIDEO_ID. 

**IMPORTANT NOTE**: Archived entries are removed from each fetched result batch before the picker displays them. Since filtering happens after yt-x fetches a page, a page may show fewer items than its configured page size. This version does not fetch extra items to fill the gaps.


**Install and configure**

1. Save the code as archive-filter.ext in yt-x’s extensions directory:

`mkdir -p "${XDG_CONFIG_HOME:-$HOME/.config}/yt-x/extensions"
`

2. Edit _~/.config/yt-x/config_ and set the path to your archive file:

```
CONFIG_ARCHIVE_FILTER_FILE="$HOME/path/to/your/archive.txt"
CONFIG_AUTOLOADED_EXTENSIONS="archive-filter.ext"
```

If CONFIG_AUTOLOADED_EXTENSIONS already has entries, keep them 
and append ',archive-filter.ext'. yt-x documents that setting as a comma-separated list of extensions to load at startup. 
Make sure the archive is in yt-dlp’s standard format, with one extractor ID pair per line, for example:
_youtube dQw4w9WgXcQ_

The extension only reads this file; it doesn’t add IDs to it. 

Test it with:
`yt-x -x archive-filter.ext --search "a search term"`

After that, the automatic-loading config makes it apply on normal yt-x launches too.

To identify which assumption failed, add these temporary log lines:

Run yt-x, open the results page, then check:
`cat "${TMPDIR:-/tmp}/yt-x-archive-filter-debug.log"`

Also confirm the test video’s ID is actually in the archive file in yt-dlp format—for example, a line like youtube VIDEO_ID. The archive file only represents IDs written into it; merely watching a video won’t add it. 

If the log says the extension loaded but has no “shim invoked” line, my PATH-wrapper approach missed yt-x’s call path. If the shim is invoked but either flag is false, its filter condition is wrong for the command yt-x uses. The relevant log lines, plus the test video’s archive entry (you can redact the ID), will show which part needs correcting.
