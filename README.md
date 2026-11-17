# chromium-search

An agent friendly CLI for [Chromium source code](https://source.chromium.org).

A single-file Python script with zero dependencies. Talks directly to the Chromium Code Search API.

Forked from [jclay/chromium-search](https://github.com/jclay/chromium-search). This fork adds `ls`, `log`, `repos`, other repos and path scoping (`--repo`, `--project`, `--path`), `find -L`, `--json` on every command, and clearer error messages.

![screenshot of chromium-search](screenshot.svg)

## Install

The script is a single standalone script. Fetch it and customize it to fit your needs.

```
curl -o chromium-search https://raw.githubusercontent.com/92kns/chromium-search/main/chromium-search.py
chmod +x chromium-search
```

Put it somewhere on your `PATH`. Requires Python 3.11+ or [`uv`](https://docs.astral.sh/uv/).

## Usage

```
chromium-search <query>                   # search source code
chromium-search find <name>               # find files by name
chromium-search cat <path>                # print file contents
chromium-search ls [path]                 # list a directory
chromium-search log <path>                # commit history for a path
chromium-search repos                     # repositories in the project
chromium-search syntax                    # query syntax reference
```

### Search

```
chromium-search base::span
chromium-search 'lang:cpp class:WebContents'
chromium-search 'usage:TabStripModel -file:out/' -C 3
chromium-search 'file:*_test.cc base::span' -L 10
chromium-search 'function:CreateForTesting lang:cpp' --json
```

`-L <n>` limits results (default 30). `-C <n>` adds context lines. `--json` outputs JSON.

Supports filters: `lang:`, `file:`, `class:`, `function:`, `symbol:`, `usage:`, `comment:`, `case:yes`, `pcre:`. Run `chromium-search syntax` for the full reference.

### Find

```
chromium-search find web_contents.h
```

Returns file paths matching the name (fuzzy, 10 results by default; `-L` for more).

### Cat

```
chromium-search cat content/public/browser/web_contents.h
chromium-search cat base/containers/span.h -n
chromium-search cat chrome/browser/BUILD.gn --ref refs/tags/144.0.7559.98
```

`-n` adds line numbers. `--ref` fetches at a specific git ref or tag.

### ls, log

```
chromium-search ls third_party/crossbench
chromium-search log base/containers/span.h -L 5
chromium-search log third_party/blink/renderer/core/scroll --ref refs/tags/144.0.7559.98
```

`ls` lists one directory level (dirs first). `log` shows commits that touched a path, newest first (`-L`, default 10).

### Other repos and scoping

```
chromium-search repos
chromium-search gclient_eval --repo chromium/tools/depot_tools
chromium-search ScrollEnd --path third_party/blink/renderer/core
```

`--repo` / `--project` (env `CR_SEARCH_REPO`, `CR_SEARCH_PROJECT`) pick another repository; `repos` lists the ones in the project, and `(xrefs)` marks those with a semantic index. Repos without one may not be searchable. `--path` limits search and find to a directory.

Note that search and find always use the current main index. `--ref` only applies to `cat`, `ls` and `log`; the tool prints a note if you pass it to search.

### JSON

`--json` works on every command. Search takes an optional field list (`path,line,snippet,context,matchRanges,url`). The other commands return fixed shapes: find returns `[{path,url}]`, cat returns `{path,ref,content}`, ls returns `[{name,type}]`, log returns `[{commit,time,author,subject,position,review}]`, and repos returns `[{name,semantic,lastCommit}]`.

## How it works

Queries the Grimoire API that powers source.chromium.org. Search, find, log and repos use the batch REST transport; cat and ls use gRPC-Web to the FileService. No authentication required.

## License

MIT
