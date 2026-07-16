# Catatan_FileMd
Catatan 
# Tambah scope
nj scope add nextarjuna.com

# Restart
nj restart

# Scan app sendiri
nj pentest scan nextarjuna.com

# Update templates
nuclei -update-templates

# Tambah scope nextarjuna
nj scope add nextarjuna.com

# Restart
nj restart

# Scan
nj pentest scan nextarjuna.com


# Quick scan dulu
nj pentest quick nextarjuna.com

# Full scan
nj pentest scan nextarjuna.com

# Cek report
nj pentest report

# Generate dashboard
nj dashboard generate findings


# Cek scope sekarang
nj scope list

# Cek guardrail
curl -sk -X POST https://127.0.0.1:9545/ \
  -d '{"action":"cognitiveStatus"}' \
  | python3 -c "
import sys,json
d=json.load(sys.stdin)
g=d['modules']['guardrail']
print('=== SCOPE ===')
for domain in g['allowed_domains']:
    print(f'  ✅ {domain}')
print()
print('=== BLOCKED ===')
for domain in g['blocked_domains']:
    print(f'  ❌ {domain}')
"


# Restart nextarjuna
nj restart

# Tambah scope
nj scope add nextarjuna.com

# Scan app sendiri
nj pentest scan nextarjuna.com

# Generate report
nj pentest report

# Export
nj pentest export html
nj pentest export json


# Cari file scope config
find ~/nextarjuna/ -type f -name "*.json" \
  -exec grep -l "allowed_domains" {} \;

# Edit file tersebut dan tambah nextarjuna.com

# Scan ulang
nj pentest scan nextarjuna.com

# Cek hasil
nj pentest report

# Export
nj pentest export html

# Cek DNS resolve
nslookup nextarjuna.com

# Cek apakah server hidup
ping -c 3 nextarjuna.com

# Cek port 80 dan 443
nmap -p 80,443 nextarjuna.com

# Export dari scan terakhir ke markdown
nj pentest export md





# Lihat report yang tersedia
nj pentest reports

# Cari file JSON report
ls ~/nextarjuna/reports/*.json

# Convert JSON ke MD manual
cat ~/nextarjuna/reports/NAMA_FILE.json | python3 -c "
import sys, json
data = json.load(sys.stdin)

md = []
md.append(f'# Scan Report')
md.append(f'')
md.append(f'Target: {data.get(\"target\", \"N/A\")}')
md.append(f'Date: {data.get(\"timestamp\", \"N/A\")}')
md.append(f'Scan ID: {data.get(\"scan_id\", \"N/A\")}')
md.append(f'')
md.append(f'## Findings')
md.append(f'')

for f in data.get('findings', []):
    md.append(f'### {f.get(\"type\", \"Unknown\")}: {f.get(\"subtype\", \"\")}')
    md.append(f'- Severity: {f.get(\"severity\", \"N/A\")}')
    md.append(f'- URL: {f.get(\"url\", \"N/A\")}')
    md.append(f'- File: {f.get(\"file\", \"N/A\")}')
    if f.get('line'):
        md.append(f'- Line: {f.get(\"line\")}')
    if f.get('description'):
        md.append(f'- Description: {f.get(\"description\")}')
    if f.get('context'):
        md.append(f'')
        md.append(f'**Context:**')
        md.append(f'\`\`\`')
        md.append(f'{f.get(\"context\", \"\")}')
        md.append(f'\`\`\`')
    md.append(f'')

output = '\n'.join(md)
with open('report.md', 'w') as out:
    out.write(output)
print(output)
print('\n[*] Saved to report.md')
"




# Simpan script ini
cat > ~/nextarjuna/export_md.sh << 'SCRIPT'
#!/bin/bash
REPORT=$1
if [ -z "$REPORT" ]; then
    # Cari report terbaru
    REPORT=$(ls -t ~/nextarjuna/reports/*.json 2>/dev/null | head -1)
fi

if [ -z "$REPORT" ]; then
    echo "[!] No report found"
    exit 1
fi

OUTPUT="${REPORT%.json}.md"

python3 << PYEOF
import json

with open("$REPORT") as f:
    data = json.load(f)

md = []
md.append("# Nextarjuna Scan Report\n")
md.append(f"**Target:** {data.get('target', 'N/A')}")
md.append(f"**Date:** {data.get('timestamp', 'N/A')}")
md.append(f"**Scan ID:** {data.get('scan_id', 'N/A')}")
md.append(f"**Total Findings:** {data.get('total_findings', 0)}")
md.append("\n---\n")

for f in data.get("findings", []):
    severity = f.get("severity", "info").upper()
    md.append(f"## [{severity}] {f.get('type', 'Unknown')}: {f.get('subtype', '')}\n")
    md.append(f"| Field | Value |")
    md.append(f"|---|---|")
    md.append(f"| Severity | {severity} |")
    md.append(f"| URL | {f.get('url', 'N/A')} |")
    md.append(f"| File | {f.get('file', 'N/A')} |")
    md.append(f"| Line | {f.get('line', 'N/A')} |")
    md.append(f"| Verified | {f.get('verified', False)} |")
    md.append(f"| Confidence | {f.get('confidence', 0)} |")
    md.append("")

    if f.get("description"):
        md.append(f"### Description\n{f['description']}\n")

    if f.get("context"):
        md.append(f"### Source Context\n")
        md.append(f"```\n{f['context']}\n```\n")

    if f.get("value"):
        md.append(f"### Exposed Value\n")
        md.append(f"```\n{f['value']}\n```\n")

    if f.get("verification"):
        v = f["verification"]
        md.append(f"### Verification\n")
        md.append(f"- Live Confirmed: {v.get('live_confirmed', 'N/A')}")
        md.append(f"- Method: {v.get('method', 'N/A')}")
        if v.get("payloads_to_test"):
            md.append(f"- Payloads to test:")
            for p in v["payloads_to_test"]:
                md.append(f"  - `{p}`")
        md.append("")

    md.append("---\n")

output = "\n".join(md)
with open("$OUTPUT", "w") as out:
    out.write(output)

print(f"[*] Report exported to: $OUTPUT")
print(f"[*] Total findings: {len(data.get('findings', []))}")
PYEOF
SCRIPT

chmod +x ~/nextarjuna/export_md.sh




# Export report terbaru
~/nextarjuna/export_md.sh

# Export report spesifik
~/nextarjuna/export_md.sh ~/nextarjuna/reports/nama_file.json

# Lihat hasil
cat ~/nextarjuna/reports/nama_file.md


# Cek report yang ada
nj pentest reports

# Lalu scan dulu jika belum ada
nj pentest scan nextarjuna.com

# Generate dashboard markdown
nj dashboard generate full
nj dashboard generate findings
nj dashboard generate reports
nj dashboard generate disclosure

# Lihat file .md yang dihasilkan
nj dashboard list



nj pentest reports










# Cek HTTP langsung
curl -v https://nextarjuna.com 2>&1 | head -30

# Cek HTTP tanpa SSL
curl -v http://nextarjuna.com 2>&1 | head -30


