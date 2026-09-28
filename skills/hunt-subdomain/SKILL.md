---
name: hunt-subdomain
description: Passive subdomain enumeration using only Subfinder and SubDog. Use when the user asks to discover subdomains with those two tools and no other enumeration tool, active probing, DNS resolution, takeover checks, or vulnerability testing.
report_count: 3
---

# Passive Subdomain Enumeration

Collect publicly discoverable hostnames for one exact root domain using only `subfinder` and `subdog`.

Do not invoke any other reconnaissance or enumeration tool. Standard shell utilities may be used only to create files, normalize output, filter the exact domain, deduplicate, and count results.

Do not resolve, probe, scan, or test discovered hostnames. Report them as passive discoveries, not as live or reachable assets.

## Workflow

Set the target without a scheme, port, path, or wildcard:

```bash
TARGET="example.com"
ESCAPED_TARGET="example\\.com"
OUTDIR="./recon/$TARGET"
TMPDIR="$OUTDIR/.tmp"
mkdir -p "$TMPDIR"
```

Check that the only permitted enumeration tools are available:

```bash
command -v subfinder subdog
```

Run Subfinder:

```bash
echo "$TARGET" | subfinder -duc -silent -all | unew "$TMPDIR/subfinder.raw"
```

Run SubDog:

```bash
echo "$TARGET" | subdog --silent --source all | unew "$TMPDIR/subdog.raw"
```

Merge, normalize, scope-filter, and deduplicate their output:

```bash
cat "$TMPDIR/subfinder.raw" "$TMPDIR/subdog.raw" \
  | tr '[:upper:]' '[:lower:]' \
  | sed 's/\.$//' \
  | grep -aE "^([A-Za-z0-9_-]+\\.)*$ESCAPED_TARGET$" \
  | unew | "$OUTDIR/allsubs.txt"
```

Count the final results:

```bash
wc -l "$OUTDIR/allsubs.txt"
```

Report the line count and output path. Do not add active-discovery or vulnerability conclusions.
