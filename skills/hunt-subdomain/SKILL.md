---
name: hunt-subdomain
description: Subdomain enumeration using Subfinder and SubDog for discovery, followed by a mandatory naabu port scan of every discovered host. Use when the user asks to discover subdomains with these tools, or asks to enumerate a domain's subdomains and their open ports. Covers passive hostname discovery plus active port scanning — not takeover checks or vulnerability testing.
---

# Subdomain Enumeration

Collect publicly discoverable hostnames for one exact root domain using only `subfinder` and `subdog`, then port-scan every discovered host with `naabu`.

Subdomain discovery is passive; the naabu port scan that follows it is active and is a required final step, not an optional extra. Do not invoke any other reconnaissance or enumeration tool. Standard shell utilities may be used only to create files, normalize output, filter the exact domain, deduplicate, and count results.

Report hostnames as passive discoveries unless naabu has actually confirmed them live. Do not add takeover or vulnerability conclusions.

## Workflow

Set the target without a scheme, port, path, or wildcard:

```bash
TARGET="example.com"
ESCAPED_TARGET="example\\.com"
OUTDIR="./recon/$TARGET"
TMPDIR="$OUTDIR/.tmp"
mkdir -p "$TMPDIR"
```

Check that all three required tools are available:

```bash
command -v subfinder subdog naabu
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
  | unew "$OUTDIR/allsubs.txt"
```

Count the final results:

```bash
wc -l "$OUTDIR/allsubs.txt"
```

Port scan every discovered host with naabu. This step is required — do not skip it or present results before it completes:

```bash
cat "$OUTDIR/allsubs.txt" | naabu -duc -silent | unew "$OUTDIR/naabu.txt"
```

Count the live hosts naabu confirmed:

```bash
wc -l "$OUTDIR/naabu.txt"
```

Report the subdomain line count, the naabu-confirmed host count, and both output paths. Do not add takeover or vulnerability conclusions.
