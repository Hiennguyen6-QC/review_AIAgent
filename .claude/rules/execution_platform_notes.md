# Execution Platform Notes — Windows / PowerShell 5.1

## When to read this file
Read this file before writing any Bash or PowerShell commands, especially for:
- Bulk find-and-replace operations inside markdown files
- Renumbering sequences (e.g., QnA STT, BA#X IDs)
- Any script that iterates over file content

Applies to all agents and skills running on this machine (Windows 11, PowerShell 5.1).

---

## PowerShell 5.1 Constraints

### Scriptblock in `-replace` — DOES NOT EXECUTE

```powershell
# ❌ WRONG — prints literal "{...}" text, does not execute the scriptblock
$text -replace 'BA#(\d+)', { "BA#$([int]$matches[1] - 1)" }

# ✅ CORRECT — use a foreach loop with fixed string substitution
foreach ($n in @(10..1)) {
    $text = $text -replace "BA#$n\b", "BA#$($n - 1)"
}
```

**Why**: PowerShell 5.1 `-replace` operator does not support scriptblock evaluation on the right-hand side. This is a .NET/C# pattern that does not work in PS 5.1. The scriptblock is printed as a literal string.

---

### Python availability

```
❌ python3   → not available by default on this machine
❌ python    → not available by default on this machine
✅ Use Read + Edit tools or PowerShell for all file manipulation tasks
```

---

### No Vietnamese string literals in `.ps1` scripts

```powershell
# ❌ WRONG — Write tool saves .ps1 with Windows-1252 encoding by default.
# Vietnamese + special quote chars inside string literals → garbled → PS parse error
$msg = "Lưu thành công"   # → broken on execution

# ✅ CORRECT — split content from logic
# Step 1: Write Vietnamese content to a separate .txt file (Write tool handles UTF-8 OK)
# Step 2: In the .ps1 script, read that file with explicit UTF-8:
$lines = [System.IO.File]::ReadAllLines($txtPath, [System.Text.Encoding]::UTF8)
```

**Rule**: Never embed Vietnamese string literals in `.ps1` scripts. Store content in a `.txt` file → read via `ReadAllLines` with UTF-8 encoding in the script. Also: always use `-File` flag to invoke scripts (not `-Command $variable`) — Bash escaping breaks `$` in `-Command` mode.

---

### PS5.1 — Build list from function calls: use `List[string]`, not array literal

```powershell
# ❌ WRONG — PS5.1 does not evaluate function calls as array literal elements
# Results in: "Missing expression after ','" parse error
$rows = @(BuildRow '1' 'val1', BuildRow '2' 'val2')

# ✅ CORRECT — explicit List[string] + .Add() per line
$rows = New-Object System.Collections.Generic.List[string]
$rows.Add((BuildRow '1' 'val1'))
$rows.Add((BuildRow '2' 'val2'))
```

**Why**: PS5.1 array literal `@(a, b, c)` does not evaluate function-call expressions separated by commas. This works in PS7 but fails silently or throws in PS5.1.

---

## Tool Priority Order (when choosing how to edit files)

| Situation | Preferred tool |
|---|---|
| 1–5 targeted changes in a file | Read + Edit (dedicated tools) — always first choice |
| Search before editing | Grep + Read → then Edit |
| Bulk changes > 5 entries, ASCII-only patterns | PowerShell `foreach` loop |
| Bash needed (POSIX logic) | Bash tool — only when PowerShell insufficient |

**Rule**: Always prefer dedicated tools (Read / Grep / Edit / Write) over terminal commands. Reserve PowerShell/Bash for operations that genuinely cannot be done with dedicated tools.

---

## String Replacement Tool Selection (MANDATORY decision gate)

Before choosing between Edit tool vs PowerShell `.Replace()`, classify the target string:

| String characteristics | Tool |
|---|---|
| ASCII-only, simple pattern, ≥5 identical occurrences | PowerShell batch |
| Contains Vietnamese diacritics OR backticks OR table cell syntax | **Edit tool** — always |
| Unique per-line (different surrounding context each time) | **Edit tool** — always |

**Rule**: Backtick + Vietnamese + markdown table cell = Edit tool. No exceptions. Using PowerShell `.Replace()` on these strings silently fails (returns unchanged content with no error).

---

## Post-Replacement Verification (MANDATORY)

PowerShell `.Replace()` silently returns the original string when no match is found — it never throws an error. Always verify immediately after each replacement:

```powershell
$content = $content.Replace($old, $new)
# Verify — never skip this
if ($content.Contains($old)) {
    Write-Host "WARNING: Replace failed — string still present: $old"
} else {
    Write-Host "OK"
}
```

After completing ALL replacements in a batch, run a Grep scan to confirm 0 residual old patterns before reporting Done to user. Reporting "Done" without verification = silent failure.

---

## Safe Renumber Pattern — Avoiding Collision

When removing entries and renumbering IDs (e.g., removing BA#3, BA#12, BA#13 → shift remaining IDs):

```powershell
# ❌ WRONG — direct sequential replace causes collision
# Replacing BA#4→BA#3 then BA#5→BA#4 etc. overwrites already-replaced values

# ✅ CORRECT — two-pass TEMP tag approach
# Pass 1: rename to TEMP tags (no collision possible)
foreach ($n in @(4..20)) {
    $content = $content -replace "BA#$n\b", "BA#TEMP$n"
}
# Pass 2: shift TEMP tags to final values
foreach ($n in @(4..20)) {
    $content = $content -replace "BA#TEMP$n\b", "BA#$($n - 1)"
}
```

**Why**: Direct in-place renumber collides — if BA#4 → BA#3 first, then the original BA#3 (already replaced) and the new BA#3 (from BA#4) become indistinguishable in subsequent passes. TEMP tags act as an intermediate namespace that eliminates this ambiguity.

---

## TC Table Structural Integrity Check (MANDATORY before reporting Done)

Sau bất kỳ batch edit nào trên TC file, **TRƯỚC KHI báo Done**, phải chạy đủ 3 checks:

**Check 1 — Blank line scan:**

```text
Grep pattern `^\s*$` trên file → ghi lại tất cả line numbers trả về
→ Với mỗi blank line: xác định nó nằm TRONG table body hay ngoài (section divider, sau last row, etc.)
→ Nếu có blank line nào nằm giữa `| --- |` separator row và dòng `---` kết thúc section → LỖI, phải fix trước khi báo Done
```

**Check 2 — Row count:**

```text
Count data rows trong ## #2 UI TCs và ## #3 FUNCTION TCs
→ So sánh với số TC đã khai báo trong ## #1 OVERALL SUMMARY
→ Mismatch = LỖI
```

**Check 3 — Section headers inside table body:**

```text
Grep pattern `^#{2,4}\s` trong range từ sau `| --- |` đến `\n---\n` tiếp theo
→ Nếu có kết quả → còn section header chưa xóa, table bị break → LỖI
```

"Đã review" chỉ hợp lệ khi kèm theo evidence của ít nhất Check 1. Không được dùng từ "đã review" hoặc "Done" dựa trên mental simulation.

---

## Remove-Header Atomicity Rule

Khi xóa bất kỳ `###` hoặc `##` header nào nằm bên trong TC table body:

**PHẢI xóa cả blank line TRƯỚC và SAU header đó trong cùng 1 Edit call.** Không được xóa header mà để lại blank lines xung quanh — chúng trở thành ghost artifacts phá cấu trúc Markdown table.

```text
❌ WRONG — chỉ xóa header, để lại blank lines:
  [blank line]           ← còn lại
  ~~### UC-190 Pagination~~  ← đã xóa
  [blank line]           ← còn lại → phá table
  | row... |

✅ CORRECT — xóa header kèm cả 2 blank lines xung quanh:
  old_string: "\n\n### UC-190 Pagination\n\n"
  new_string: "\n"   (giữ lại 1 newline để nối 2 rows)
```

---

## TC Skeleton Pre-flight (Skill Execution)

Trước khi generate bất kỳ TC file nào:

1. **Read Skill file** → xác nhận cấu trúc sections bắt buộc: `## #1` / `## #2` / `## #3`
2. **Read reference TC file** (VD: `DinhMucBHYT_TCs_v3.md`) lines 1–60 → verify skeleton structure bằng mắt
3. **Tạo file skeleton trước** (header + section shells + table headers, không có data rows) → Write → confirm structure đúng
4. **Sau đó điền content** vào skeleton đã verified

Không được generate toàn bộ file trong 1 lần Write mà không verify skeleton trước. Approach này loại bỏ risk "tạo sai skeleton" vì skeleton được confirm trước khi có content.

---

## Reference

- Lesson learned from: `Memory for Claude code/LessonLearned_02_M10_F1_DM05_DinhMucBHYT.md` (Errors #5, #6)
- Lesson learned from: `02_M10_F1_DM05_MucHuongBHYT` session (blank lines in table body, wrong skeleton with Phase sections)
