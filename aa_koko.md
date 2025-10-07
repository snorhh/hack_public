## h1 ei mitään
## h2 


OWASP Broken Access Control
https://owasp.org/Top10/A01_2021-Broken_Access_Control/
Hyökkäystapoja:

jos ei ole varmistettu, niin hyökkääjä voi esim. muuttaa   
 https://example.com/app/accountInfo?acct=notmyacct

voi yrittää löytää sivun, jolle pääsee ilman lupaa

### Ffuf
https://terokarvinen.com/2023/fuzz-urls-find-hidden-directories/
#### Vaiheet
MUISTA, kaksi äffää alussa!

- tarvitaan sanakirja
- katso parametrit ./ffuf -> KUN OLET BIN-kansiossa (kokeile | less tai & less)
- __./ffuf -w common.txt -u http://1270.0.0.2:800/FUZZ__
- eli -w worldist ja -u target url
- nyt siis fuzzataan wordlista sinne urlinperään
- jos näkyy pelkkiä status-Ok niin filteröidään

  $ ./fuff |& less
FILTER OPTIONS:
 -fc Filter HTTP __status codes__ from response. Comma separated list of codes and ranges
 -fl Filter by __amount of lines__ in response. Comma separated list of line counts and ranges
 -fmode Filter set operator. Either of: and, or (default: or)
 -fr Filter regexp
 -fs Filter __HTTP response size__. Comma separated list of sizes and ranges
 -ft Filter by __number of milliseconds__ to the first response byte, either greater or less than. EG: >100 or <10
 -fw Filter by __amount of words__ in response. Comma separated list of word counts and ranges

esim. `./ffuf -w common.txt -u http://127.0.0.2:8000/FUZZ -fs 132`  
eli ottaa pois ne, jotka ovat 132 tavua

Ffufillla voi etsiä myös headereita, POST-parametrejä

# ffuf — Fuzzing Headers & POST Parameters (Cheat-sheet)

## Quick flags reminder

* `-u` URL (use `FUZZ` in path or in headers/body)
* `-w` wordlist (`wordlist[:placeholder]` for multiple `-w`)
* `-H` add header (can include `FUZZ`)
* `-d` request body (for POST; include `FUZZ` where needed)
* `-X` HTTP method (e.g. `POST`)
* `-mode` `clusterbomb` / `pitchfork` / `sniper`
* `-mc` match status codes (e.g. `-mc 200,301`)
* `-fc` filter status codes (e.g. `-fc 404`)
* `-fs` filter by response size
* `-fw` filter by word count
* `-fl` filter by line count
* `-o` output file, `-of` output format (`json`, `html`, `e` etc.)

---

## 1) Fuzzing header **values**

Replace header value with `FUZZ`.

```bash
# fuzz X-Forwarded-For header values
ffuf -u https://target.tld/ -H "X-Forwarded-For: FUZZ" \
     -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
     -mc 200 -fc 404 -o ffuf_headers.json -of json
```

Use multiple `-H` to add static headers (Auth/Cookie/Content-Type) while fuzzing one header.

---

## 2) Fuzzing header **names**

Discover hidden headers by fuzzing the header name and sending a small value.

```bash
ffuf -u https://target.tld/ -H "FUZZ: test" \
     -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt \
     -mc 200 -fc 404
```

Look for unexpected status codes, changes in body length, or headers in response.

---

## 3) Fuzzing POST parameter **values** (x-www-form-urlencoded)

Put `FUZZ` where the parameter value should be.

```bash
# fuzz password
ffuf -u https://target.tld/login -X POST \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "username=admin&password=FUZZ" \
     -w /usr/share/wordlists/rockyou.txt \
     -mc 200 -fc 401,403,404
```

If the site uses cookies or CSRF tokens, include them via `-H "Cookie: ..."` and `-H "X-CSRF-Token: ..."` respectively.

---

## 4) Fuzzing POST parameter **names**

Send `FUZZ` as the parameter name to find undocumented parameters.

```bash
ffuf -u https://target.tld/api -X POST \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "FUZZ=test" \
     -w small-params.txt \
     -mc 200 -fc 404
```

---

## 5) JSON bodies

Place `FUZZ` inside JSON and set header.

```bash
ffuf -u https://target.tld/api/login -X POST \
     -H "Content-Type: application/json" \
     -d '{"username":"admin","password":"FUZZ"}' \
     -w /usr/share/wordlists/rockyou.txt -mc 200 -fc 401
```

Tip: if your shell interprets quotes, use a file for the body and `--data-binary @file.json`.

---

## 6) Multiple fuzz positions (credential combos, param name + value, etc.)

Use multiple `-w` with placeholders and `-mode clusterbomb` to test all combinations.

```bash
# clusterbomb: try every username with every password
ffuf -u https://target.tld/login -X POST \
     -d "username=FUZZ&password=FUZ2Z" \
     -w users.txt:FUZZ -w passwords.txt:FUZ2Z \
     -mode clusterbomb -mc 200 -fc 401
```

`pitchfork` pairs wordlists item-by-item (useful when you want 1-to-1 pairs).

---

## 7) Filtering & matching — common useful combos

* Show only 200s: `-mc 200`
* Exclude 404s: `-fc 404`
* Exclude responses of size 0 or specific sizes: `-fs 0` / `-fs 1234`
* Filter by number of words: `-fw 0`

Use a combination to reduce noise. Save `-o results.json -of json` for later analysis.

## 9) Notes & tips (practical)

* **Auth & CSRF:** include cookies / tokens as static headers (`-H "Cookie: ..."`) or pre-fetch them.
* **Proxies:** use `-x http://127.0.0.1:8080` to proxy through Burp.
* **Follow redirects:** ffuf reports redirects; filter by status codes or review Location headers.
* **Response inspection:** save JSON output and grep for anomalies (`jq`) — large size differences or unusual status codes usually indicate interesting finds.
* **Wordlists:** prefer focused small lists for parameters and large curated lists (rockyou, seclists) for credentials.

---

### Tehtävä h2 c OK
Ratkaise dirfuzt-1 artikkelista Karvinen 2023: Find Hidden Web Directories - Fuzz URLs with ffuf.
https://terokarvinen.com/2023/fuzz-urls-find-hidden-directories/

Oma ratkaisu:  
Suoritin ffufin komennolla `ffuf -w common.txt -u http://127.0.0.2:8000/FUZZ`. Yritin ensin ajaa sitä samassa komentokehoteikkunassa, kunnes muistin, että __piti avata toinen ikkuna__.

Filtteröin tuloksia pituuden mukaan, ja koska 154 oli yleinen pituus, niin käytin sitä filtteriehtona. `ffuf -w common.txt -u http://127.0.0.2:8000/FUZZ -fs 154`

Vinkki: kokeile `curl -i http.....git/HEAD`, joka löytyy kaikista gittirepoista

-i is useful for:

Seeing status codes (200 OK, 404 Not Found, etc.)

Viewing cookies, redirects, CORS, content type, etc.

Debugging or scripting HTTP responses (especially in CTFs or fuzzing).

### Access control vulnerabilities
https://portswigger.net/web-security/access-control

Vertical - käyttäjä pääsee admin-sivuille  

__Tsekkaa aina /robots.txt__

__Tsekkaa näkyy javascriptissä jotain (esim. url)__

__Tsekkaa toimiiko /ADMIN/DELETEUSER vaikka /admin/deleteuser ei toimi__

/admin/deleteUser.anything saattaa mätsätä /admin/deleteUser


tai jopa viimeinen / tai ilman

### Horizontal privilege escalation
https://insecure-website.com/myaccount?id=123  
Muuttaa id:n toisen käyttäjä id:ksi

Labissa oli tehtävä jossa piti etsiä Carlosin blogi, sieltä löysi ID:n ja sen pystsyi muuttamaan sen jälkeen kun oli kirjautunut sisään.

### Tehtävä 2a) Murtaudu 010-staff-only.

Oma tehtävä meni pieleen, katson toisesta

Sivulla piti laittaa salasana ja "reveal my password" näytti oliko se oikein. Syöttökenttä hyväksyi vain numeroita, mutta se oli selaimen tekemä rajoitus. Inspectorilla näkyi <input type="number" name="pin" value="0">. Siihen pystyy esim. laittamaan type="string" tai "text"

OR 1 = 1 eli SELECT password FROM pins WHERE pin='OR 1=1' tuottaa yhden rivin mutta 'OR 1=1 LIMIT 2,1-- toi halutun

tai 

SELECT password FROM pins WHERE pin='1' OR 1=1 ORDER BY password LIMIT 1 OFFSET 0 --';


# SQLi Cheat-sheet — quotes, comments, and `LIMIT`




## 1) Breaking out of a string: why start with `'`

When an application builds SQL like:

```sql
SELECT password FROM pins WHERE pin = '<user_input>';
```

The application *opens* a string literal with the first `'`. To inject SQL logic you must **close** that string so the rest becomes SQL instead of literal data. That’s why payloads often begin with a single quote:

```
0' OR 1=1 --
```

Constructed SQL becomes:

```sql
SELECT password FROM pins WHERE pin = '0' OR 1=1 -- ';
```

Step-by-step:

1. The first `'` in your input closes the application's opening quote.
2. `OR 1=1` is now regular SQL (a tautology returning all rows).
3. `-- ` starts a comment so the leftover `';` appended by the app is ignored.

## 2) Why add a comment (`--`, `#`, `/* ... */`)?

* The app usually appends a trailing quote and semicolon (`' ;`) after your input. If you don’t comment out that remainder you often end up with a syntax error (extra quote).
* Adding `-- ` (line comment) or `#` or a block comment `/* ... */` ignores the rest of the constructed query so it stays syntactically valid.

**Examples:**

* Without comment (will likely error):

```
user_input = 0' OR 1=1'
-- results: ... OR 1=1''  (extra quote -> syntax error)
```

* With comment (valid):

```
user_input = 0' OR 1=1 --
-- results: ... OR 1=1 -- ' ;  (rest ignored)
```

**Dialect notes:**

* Many DBs require a space after `--` (use `-- `). MySQL accepts `#` as a line comment. Block comments `/* ... */` work across many DBs.
* Some defenses strip comment tokens, so different payloads may be needed in labs.

## 3) Numeric vs. quoted contexts

If the application constructs the query without quotes (e.g., `WHERE pin = 0`), you **must not** start with a quote. Instead use unquoted logic:

```
0 OR 1=1 --
```

This produces `WHERE pin = 0 OR 1=1 --;` which is valid for numeric contexts.

## 4) Using `LIMIT` to enumerate rows

**MySQL-style `LIMIT <offset>,<count>`** means: skip `<offset>` rows, then return up to `<count>` rows.

* `LIMIT 2,1` → skip 2 rows, return 1 → returns the *third* row (zero-based offset).
* Equivalent standard form: `LIMIT 1 OFFSET 2`.

**Why use `LIMIT` in an injection?**

* `OR 1=1` can make the `WHERE` match all rows. `LIMIT` lets you pick a specific row from that result set — useful for enumerating entries one-by-one.

**Example payload used by another student:**

```
' OR 1=1 LIMIT 2,1--
```

Constructed SQL:

```sql
SELECT password FROM pins WHERE pin = '' OR 1=1 LIMIT 2,1 -- ';
```

This returns the third matching row (but only if ordering is consistent).

**Important caveat:**

* Without `ORDER BY`, row order is *not guaranteed*. Combine with `ORDER BY id` (or another stable column) for deterministic enumeration:

```sql
... WHERE ... ORDER BY id LIMIT 2,1
```

## 5) Practical checklist for lab payloads

* Inspect whether the application uses quotes around the parameter. If yes, start with `'` to close the string; if no, do not use the initial quote.
* Add injected logic (e.g., `OR 1=1` or a boolean condition targeting a field).
* Add a comment token (`-- `, `#`, or `/*`) to swallow the trailing characters.
* To extract specific rows, use `LIMIT offset,count` or `LIMIT count OFFSET offset`.
* URL-encode spaces and punctuation when sending via HTTP (`%20` for space, etc.).

## 6) Defensive fixes to mention in your assignment

* **Parameterized queries / prepared statements** (don’t concatenate user input into SQL):

  * Python (sqlite3): `cur.execute("SELECT password FROM pins WHERE pin = ?", (pin,))`
  * PHP (PDO): prepared statements with bound parameters
  * Node.js: `connection.execute('SELECT ... WHERE pin = ?', [pin])`

* **Input validation** (e.g., `^[0-9]{4}$` for a 4-digit PIN) as defense-in-depth (not a substitute for parameterization).



---
# ORDER BY and LIMIT Explained (Deterministic Selection)

## 1. Why ORDER BY matters

SQL databases do **not guarantee row order** unless you explicitly tell them how to sort the results. Without `ORDER BY`, rows may appear in any sequence, depending on:

* Index structure
* Execution plan
* Table updates or internal optimizations

Therefore, when using `LIMIT` (or `OFFSET`) to fetch a specific subset of rows, you must **combine it with ORDER BY** to make sure the same rows are returned every time.

---

## 2. Basic usage example

```sql
-- Get the third row (zero-based offset 2)
SELECT password
FROM pins
WHERE 1=1
ORDER BY id ASC
LIMIT 1 OFFSET 2;
```

**Explanation:**

* `ORDER BY id ASC` → sorts rows by the `id` column in ascending order.
* `LIMIT 1 OFFSET 2` → skip the first 2 rows, return 1.

This guarantees you get the *third* row by id.

---

## 3. MySQL shorthand syntax

MySQL and SQLite also support:

```sql
SELECT password
FROM pins
WHERE 1=1
ORDER BY id ASC
LIMIT 2,1;
```

Here:

* `2` = offset
* `1` = number of rows to return

Equivalent to `LIMIT 1 OFFSET 2`.

---

## 4. Ordering best practices

* Always use a **stable column** in `ORDER BY` — usually the primary key (`id`).
* If the column you order by isn’t unique, add a tiebreaker:

  ```sql
  ORDER BY created_at DESC, id ASC
  ```
* To reverse order, use `DESC` (descending):

  ```sql
  ORDER BY id DESC
  ```

---

## 5. Performance note: large OFFSETs

When you use a large `OFFSET`, the database still scans and discards earlier rows. For large tables or deep pagination, use **keyset pagination** instead.

### Keyset pagination example:

```sql
-- Fetch the next row after the last known id
SELECT password
FROM pins
WHERE id > :last_id
ORDER BY id ASC
LIMIT 1;
```

This is faster and more consistent than using a huge OFFSET.

---

## 6. Summary

| Goal                               | Recommended syntax                        |
| ---------------------------------- | ----------------------------------------- |
| Deterministic single-row selection | `ORDER BY id ASC LIMIT 1 OFFSET n`        |
| MySQL shorthand                    | `ORDER BY id ASC LIMIT n,1`               |
| Paginate efficiently               | Keyset pattern using `WHERE id > last_id` |

---

**Key takeaway:** Always combine `LIMIT` with `ORDER BY` when you need repeatable results.

### Tehtävä b Korjaa 010 staff only haavoittuvuus

Itse en tehnyt sitä. 

Eli siinä olisi pitänyt avata lähdekoodi (.py) editorissa ja tehdä siellä jotain... en jaksa edes katsoa.

### Tehtävä d Murtaudu 020-your-eyes-only. Ks. Karvinen 2024: Hack'n Fix

https://terokarvinen.com/hack-n-fix/
Itse en tehnyt.

Tämä oli se, jossa piti vain rekisteröityä sivulle ja sitten mennä sinne admin-console-sivulle, joka löytyi fuzzaamalla.

### Tehtävä e) Korjaa 020-your-eyes-only haavoittuvuus. Osoita testillä, että ratkaisusi toimii.

En tehnyt tätä lainkaan.

Tässä olisi pitänyt taas katsella lähdekoodia ja laittaa sinne myös toiseen kohtaan tarkistus, että käyttäjä "self.request.user.is_staff".

### 🧠 GDB and Conversion Reference Guide

* * *

## 🧩 GDB Basics

### `x/s $rdi`

Examine memory as a string at the address stored in the **RDI register**.

```bash
(gdb) x/s $rdi
```

→ Displays the string pointed to by RDI.

### `set $buf = $rsi`

Creates a custom GDB variable `$buf` with the same value as `$rsi`.

```bash
(gdb) set $buf = $rsi
(gdb) x/s $buf
```

→ Handy for tracking pointers while stepping.

### Stepping Commands

| Command | Level | Steps Into Functions? | Description |
| --- | --- | --- | --- |
| `n` | Source | ❌ No | Run next C line, skip calls |
| `s` | Source | ✅ Yes | Run next C line, step into calls |
| `ni` | Instruction | ❌ No | Run next assembly instruction, skip calls |
| `si` | Instruction | ✅ Yes | Run next assembly instruction, step into calls |

**In short:** `n/s` for source, `ni/si` for assembly. `next` skips, `step` enters.

* * *

## 🧰 File Tools

### Opening `.docx` Files in Linux

| Purpose | Command |
| --- | --- |
| View/Edit | `libreoffice document.docx` |
| Default App | `xdg-open document.docx` |
| View Text | `catdoc document.docx` or `docx2txt document.docx` |
| Inspect Internals | `7z x document.docx -odocx_contents` |

> `.docx` = ZIP archive. Main content: `word/document.xml`.

### 7-Zip `-o` Option

No space between `-o` and folder name.

```bash
7z x file.zip -ooutput_folder
7z x file.zip -o"folder with spaces"
```

### Binwalk `-M`

`-M` = Recursive scan mode ("Matryoshka mode").

```bash
binwalk -eM firmware.bin
```

→ Extracts and scans nested files inside firmware.

* * *

## 🧩 Python Conversion Cheatsheet

### `bytes.fromhex()`

Converts a hex string → bytes.

```python
b = bytes.fromhex("48656c6c6f")  # b'Hello'
print(b.hex())  # bytes → hex
```

**Preferred modern way.** Replaces older `binascii` or `codecs` methods.

* * *

## 🧩 Ultimate Encoding & Conversion Guide

### 1\. Hex ↔ Bytes

```python
b = bytes.fromhex("48656c6c6f")  # hex → bytes
h = b.hex()                      # bytes → hex
```

### 2\. Base64 ↔ Bytes

```python
import base64
b64 = base64.b64encode(b"Hello")
raw = base64.b64decode(b64)
```

### 3\. Strings ↔ Bytes

```python
b = "Hello".encode()
t = b.decode()
```

### 4\. Mixing Conversions

```python
data = base64.b64encode("Hello".encode())
text = base64.b64decode(data).decode()
```

### 5\. Int ↔ Bytes

```python
n = 1024
b = n.to_bytes(2, 'big')
i = int.from_bytes(b, 'big')
```

### 6\. CLI Tools

| Conversion | Linux Command |
| --- | --- |
| Hex dump | `xxd file` |
| Base64 encode | `base64 file` |
| Base64 decode | `base64 -d file` |

### Quick Summary

| Type | To Bytes | From Bytes |
| --- | --- | --- |
| **Hex** | `bytes.fromhex()` | `.hex()` |
| **Base64** | `b64decode()` | `b64encode()` |
| **String** | `.encode()` | `.decode()` |
| **Int** | `.to_bytes()` | `int.from_bytes()` |

> Always convert to **bytes** first — most encodings work from there.

# Ghidra and Reverse Engineering Guide — Consolidated

This document merges all the guides and walkthroughs from this chat: **Ghidra basics**, **example walkthrough (passtr)**, and **offset explanations**.

---

## 1. Ghidra — Concise Reverse‑Engineering Instructions (for simple programs)

**Goal:** fast, practical steps to analyze small binaries (C-style programs, challenge binaries). No fluff — just techniques you will use in an assignment.

### Quick setup

* Start Ghidra and create a new Project (Non‑Shared).
* File → Import File → choose the binary. Keep default loader; click **OK**.
* Double‑click the imported file to open it in CodeBrowser.
* When asked, run **Auto‑analysis** with default options (tick `Optimize for C/C++` if available). You can turn off expensive analyzers for bigger files.

### First pass: reconnaissance

* **Program Tree / Symbols:** open **Symbol Tree → Functions**. Look for `main` and other obvious names.
* **Strings:** Window → Defined Strings. Scan for human text, format strings, filenames, flags.
* **Imports:** check **Imports** (External Symbols) to infer library use (printf, strcmp, fopen, system, etc.).

### Decompiler and code reading

* Double‑click `main` (or top function). Press **F3** for decompiler view.
* Use the C-like output as your primary view. The Listing is for exact instructions.
* Look for input handling (`scanf`, `fgets`, etc.) and comparisons (`strcmp`, loops, conditional jumps).

### Rename and comment

* Rename functions/variables: select → **L** or right‑click → **Rename**.
* Add comments with **;** or right‑click → **EOL Comment**.
* Use short, meaningful names (`check_pass`, `buf`, `len`).

### Cross‑references and control flow

* Right‑click → **References → Show References to** (who calls or uses it).
* Use **Function Graph** to visualize calls.

### Adjusting data types

* Right‑click → **Data Type** to fix wrong variable types (char*, int, etc.).

### Identify comparisons and crypto checks

* Watch for `strcmp`, `memcmp`, or byte loops comparing to constants.
* If it’s all bit shifts or XORs, it may be an obfuscated check or simple cipher.

### Patch & re‑test

* **Patch Program** → modify instructions or constants (e.g., change `jne` → `je` to flip a check).
* File → Export Program → save patched binary.

### Shortcuts

* **F3** toggle decompiler
* **L** rename label
* **C** convert operand to code
* **Space** switch Listing/Decompiler
* **T** change data type

### Assignment checklist

* List analyzed functions and their roles.
* Note important strings and constants.
* Describe how input is checked.
* Provide patch evidence (diff or bytes).
* Screenshot renamed variables and logic path.

---

## 2. Example Walkthrough — Binary: `passtr`

**Summary:**

* File: ELF 64-bit, x86‑64, PIE, **not stripped**.
* Flag found directly in strings: `FLAG{Tero-d75ee66af0a68663f15539ec0f46e3b1}`.

### Reconnaissance

* `file` → confirms 64‑bit PIE.
* `strings` → shows key text:

  ```
  What's the password?
  Yes! That's the password. FLAG{Tero-d75ee66af0a68663f15539ec0f46e3b1}
  ```

### Ghidra steps

1. Import the binary and run Auto‑analysis.
2. Open **Defined Strings**; locate both text lines and the flag.
3. Right‑click → **References → Show References to** → likely `main`.
4. In the **Decompiler**, typical structure:

   ```c
   printf("What's the password?");
   scanf("%s", input_buf);
   if (strcmp(input_buf, "sala-hakkeri-321") == 0)
       puts("Yes! That's the password. FLAG{...}");
   else
       puts("Sorry, no bonus.");
   ```
5. Rename variables for clarity: `input_buf`, `pw_check`, etc.
6. Confirm control flow: correct password prints the flag.

### Optional patch

To make it always print success:

* In Listing, locate conditional jump after `strcmp`.
* **Patch Program** → replace `jne` with `je` or NOP the branch.
* Export patched binary.

### Deliverables

* Architecture summary.
* Key strings and functions (`strcmp`, `scanf`).
* Screenshot of decompiler and string view.
* The discovered flag.

---

## 3. Understanding Offsets vs Addresses

**Offset** = byte position of data *in the file* (starting from 0).
**Address (VMA)** = where that data is mapped *in memory* when loaded.

### Viewing offsets

```bash
strings -o passtr       # decimal offsets
strings -t x passtr     # hex offsets
```

### Inspecting file bytes

```bash
hexdump -C -s <offset> -n <len> passtr
xxd -s <offset> -l <len> passtr
```

### Mapping to memory addresses

```
VMA = p_vaddr + (file_offset - p_offset)
```

Where `p_offset` and `p_vaddr` are from the program headers (`readelf -l passtr`).

For **PIE binaries**, runtime adds a random base (ASLR):

```
runtime address = base + VMA
```

### In Ghidra

* The addresses shown are **virtual memory addresses**, not file offsets.
* To locate a string or instruction by file offset, use the program header mapping or search for the bytes directly.

---

## 4. Combined Practical Flow (Summary)

1. Inspect with `file`, `readelf -h`, `strings`.
2. Import into Ghidra, run auto‑analysis.
3. Examine `main`, rename, and comment.
4. Trace strings and library calls.
5. Patch conditional logic or constants as needed.
6. Verify by running patched binary.
7. Document findings: logic flow, key functions, and discovered flag.

---

End of consolidated Ghidra reverse‑engineering guide.



### 🧭 Seeing Imports in Ghidra

1. **Window → Imports** (if available) – shows imported libraries and functions.
2. **Symbol Tree → Imports** – expand to see DLLs/so files and imported functions.
3. **Search → For External Locations** – find all references to imported functions.

If you don’t see any imports:

* The binary might be **statically linked** (no external libs).
* It could be **packed** (e.g., UPX hides imports until runtime).
* It could be a **minimal binary** using syscalls directly.

You can confirm by checking for `.idata`, `.dynsym`, `.got` sections or using `readelf -d`.

---

### 🧱 When There Are No Imports

If **Imports** isn’t even listed under the *Window* tab, it means Ghidra found no import table at all. This usually means:

* The program is **statically linked** (all code included in binary).
* The binary is **packed** (imports hidden until unpacked).
* The binary is **handcrafted/minimal**, using raw syscalls.

---



---

### 🧩 External Programs = Imports

In Ghidra, **External Programs** lists external libraries (like `libc.so`, `KERNEL32.dll`) and their imported functions.

Example:

```
External Programs
└── libc.so.6
    ├── printf
    ├── malloc
    ├── free
```

These are your **imports**. If empty, the binary is probably statically linked or packed.

| Ghidra Term           | Meaning                                      |
| --------------------- | -------------------------------------------- |
| External Programs     | The external libraries the binary depends on |
| Functions inside them | Imported functions                           |
| Empty                 | Statically linked or packed binary           |

---

### 💬 Adding Comments in Ghidra

1. Click a line in the **Listing** window.
2. Press `;` to add a **plate comment** (appears above the line).
3. Press `Shift+;` for an **end-of-line comment**.
4. Or right-click → **Set Comment...** → choose type.

| Type       | Shortcut    | Appears            | Use                            |
| ---------- | ----------- | ------------------ | ------------------------------ |
| Plate      | `;`         | Above line         | Descriptive note               |
| EOL        | `Shift+;`   | End of line        | Quick inline comment           |
| Pre/Post   | Right-click | Before/after line  | Section headers/notes          |
| Repeatable | Right-click | Shown at all xrefs | Good for function explanations |

Example:

```asm
CALL FUN_0010234a
; likely calls printf("Access granted")
```

---

### 💡 Follow Constants and Reverse Arithmetic

**Follow constants** = track fixed numeric values through the code to see what they represent (like ASCII, offsets, or keys).
**Reverse arithmetic** = simplify or undo math to recover original logic.

Example:

```c
eax = param_1 + 0x41;   // 0x41 = 65 = 'A'
```

Likely part of character encoding.

If code says:

```c
eax = input * 3 + 6;
if (eax == 15)
```

Then: `input = 3` → by reversing arithmetic.

| Phrase             | Meaning                                            |
| ------------------ | -------------------------------------------------- |
| Follow constants   | Track and interpret fixed values to reveal meaning |
| Reverse arithmetic | Undo math to recover original logic                |

These techniques make it easier to understand obfuscated or compiled code in Ghidra.



### H3 No strings attached (Tero)

**Using the `strings` Tool in Linux**

`strings` is a simple but powerful Linux tool for extracting readable text from binary files (like executables, images, or dumps). It's often used in reverse engineering, malware analysis, and CTFs.

---

### **Basic Syntax**

```
strings [options] <file>
```

Example:

```
strings a.out
```

→ Prints all readable ASCII strings from the binary file `a.out`.

---

### **Common Use Cases**

#### 1. **Find Hidden Text in Executables**

```
strings /bin/ls | head
```

→ Shows the first few readable strings inside the `ls` binary.

This can reveal version info, messages, or function names.

#### 2. **Search for Keywords**

```
strings secret.bin | grep password
```

→ Extracts all strings from `secret.bin` and filters only lines containing `password`.

#### 3. **Inspect Unknown Files**

```
strings unknown_file
```

→ Useful to check if a file contains readable text or metadata (like URLs, emails, or file paths).

#### 4. **Extract Unicode Strings**

```
strings -e l binaryfile
```

→ `-e l` tells `strings` to look for 16-bit little-endian Unicode text.

Other encodings:

* `-e b` → 16-bit big-endian
* `-e L` → 32-bit little-endian
* `-e B` → 32-bit big-endian

#### 5. **Set Minimum String Length**

```
strings -n 6 malware.bin
```

→ Only shows strings with 6 or more printable characters (default is 4).

#### 6. **Extract Strings from a Process in Memory**

```
sudo strings /proc/<PID>/mem | grep flag
```

→ Useful in forensics or CTFs to find flags or secrets in running processes.

*(Replace `<PID>` with the process ID.)*

#### 7. **Show Offsets**

```
strings -t x binaryfile
```

→ Prints each string with its offset in hexadecimal.

Useful for knowing where in memory or the file a string appears.

Example output:

```
  12ab Hello world
  13f2 /etc/passwd
```

---

### **Combine with Other Tools**

#### With `grep` to Filter Interesting Output

```
strings program | grep -E "flag|key|pass"
```

→ Finds words like *flag*, *key*, or *pass*.

#### With `less` for Interactive Browsing

```
strings binary | less
```

→ Lets you scroll through output conveniently.

---

### **Quick Recap Table**

| Option | Meaning                           | Example |
| ------ | --------------------------------- | ------- |
| `-n`   | Min string length                 | `-n 6`  |
| `-e`   | Encoding (Unicode etc.)           | `-e l`  |
| `-t`   | Show offsets                      | `-t x`  |
| `-f`   | Print filename before each string | `-f`    |

---

### **Practice Tips**

* Try `strings /bin/bash | grep version`
* Use it on captured network dumps (`.pcap`) or memory dumps.
* Combine with `xxd` or `file` to get deeper insight into binaries.

---

**In short:** `strings` helps you read between the bytes — use it to uncover clues, hardcoded secrets, or debugging hints in compiled or unknown files.


**Using the `xxd` Tool in Linux**

`xxd` is a command-line tool that creates a **hex dump** of a file or standard input. It can also do the reverse — **convert a hex dump back to binary**. This is useful for inspecting, editing, or injecting binary data, often in reverse engineering, exploit development, or CTF challenges.

---

### **Basic Syntax**

```
xxd [options] [infile [outfile]]
```

Example:

```
xxd file.bin
```

→ Displays the file content in hexadecimal and ASCII side-by-side.

---

### **Example Output**

```
00000000: 4865 6c6c 6f20 576f 726c 6421       Hello World!
```

* `00000000`: Offset (hex position in file)
* `4865 6c6c 6f20 576f 726c 6421`: Bytes in hex
* `Hello World!`: ASCII interpretation of those bytes

---

### **Common Use Cases**

#### 1. **View File Contents as Hex + ASCII**

```
xxd binaryfile | head
```

→ Shows the start of the file in both hex and readable text.

#### 2. **Limit the Number of Bytes Displayed**

```
xxd -l 64 file.bin
```

→ Dumps only the first 64 bytes.

#### 3. **Show Only Hex (Without ASCII)**

```
xxd -p file.bin
```

→ Produces a continuous plain hex dump.

Example output:

```
48656c6c6f20576f726c6421
```

This is useful for scripting or when you need clean hex values.

#### 4. **Reverse a Hex Dump Back to Binary**

```
xxd -r -p hex.txt output.bin
```

→ Converts a plain hex dump (like one created with `xxd -p`) back to a binary file.

#### 5. **Edit a File in Hex and Rebuild It**

1. Dump file into editable hex text:

   ```
   xxd -p file.bin > dump.txt
   ```
2. Edit the hex manually (e.g., change `41` to `42`).
3. Recreate the binary:

   ```
   xxd -r -p dump.txt newfile.bin
   ```

#### 6. **Specify Starting Offset**

```
xxd -s 128 file.bin
```

→ Skip the first 128 bytes and start dumping from there.

You can also use negative offsets to count from the file end:

```
xxd -s -64 file.bin
```

→ Shows the last 64 bytes.

#### 7. **Control Columns per Line**

```
xxd -c 8 file.bin
```

→ Prints 8 bytes per line instead of the default 16.

---

### **Combine with Other Tools**

#### Filter for Specific ASCII Text

```
xxd file | grep Hello
```

→ Finds hex lines containing readable text.

#### Compare Two Binaries

```
diff <(xxd file1.bin) <(xxd file2.bin)
```

→ Quickly see byte-level differences.

---

### **Quick Recap Table**

| Option | Meaning               | Example         |
| ------ | --------------------- | --------------- |
| `-l`   | Limit number of bytes | `-l 32`         |
| `-p`   | Plain hex (no ASCII)  | `-p`            |
| `-r`   | Reverse (hex → bin)   | `-r -p hex.txt` |
| `-s`   | Start at offset       | `-s 128`        |
| `-c`   | Bytes per line        | `-c 8`          |

---

### **Practice Tips**

* Run `xxd /bin/ls | less` to see what executables look like in hex.
* Extract hidden messages from image files using offsets.
* Combine `xxd` with `grep`, `cut`, or `awk` for quick analysis.

---

**In short:** `xxd` lets you peek directly into the raw bytes of a file and manipulate them at the lowest level. It’s your go-to hex viewer and editor on Linux.

**Understanding the ASCII Column in `xxd` Output**

When you run `xxd` on a binary file, each line of output shows three parts: the offset, the hexadecimal bytes, and the ASCII representation. That final part — the **ASCII column** — is what shows you the readable text mixed with dots.

---

### **Structure of a Typical `xxd` Line**

```
00000000: 7f45 4c46 0201 0100 0000 0000 0000 0000  .ELF............
```

Each line has three columns:

| Part             | Meaning                                                          | Example            |
| ---------------- | ---------------------------------------------------------------- | ------------------ |
| **Offset**       | The file position (in hexadecimal) of the first byte on the line | `00000000`         |
| **Hex bytes**    | The raw binary data, shown as pairs of hex digits                | `7f45 4c46 ...`    |
| **ASCII column** | Human-readable characters (dots for non-printable bytes)         | `.ELF............` |

---

### **What the ASCII Column Shows**

* If a byte represents a **printable ASCII character**, it displays that character (letters, numbers, symbols, space, etc.).
* If a byte is **non-printable** (like `00`, `0A`, or control codes), it shows a **dot (`.`)** instead.

Example:

```
4865 6c6c 6f0a                           Hello.
```

Here:

* `48 65 6c 6c 6f` → `Hello`
* `0a` → newline, displayed as a dot (`.`)

---

### **Why You See Program Text There**

Executable files often contain:

* String literals (e.g. `"Hello, world!"`)
* Function or variable names (in symbol tables)
* Paths, messages, or version info

These are all stored as ASCII inside the binary, so `xxd` reveals them. Reverse engineers and CTF players use this to find clues, hidden flags, or text constants.

---

**In short:** The ASCII column helps you visually connect the raw hex bytes with their readable text equivalents. Dots (`.`) mark non-printable bytes, while real characters show you the actual text embedded in the binary.

# C — Basics for Beginners (super concise)

---

## Compile & run

```bash
# compile
gcc -std=c17 -O2 -Wall hello.c -o hello
# debug
gcc -g -O0 -Wall hello.c -o hello_debug
# run
./hello
```

---

## Hello world (example)

```c
#include <stdio.h>
int main(void) {
    printf("Hello, world
");
    return 0;
}
```

---

## Core primitives (one-liners)

```c
int i = 42;            // integer
char c = 'A';          // char
float f = 3.14f;       // float
char s[] = "hi";     // string (NUL-terminated array)
int *p = &i;           // pointer
int a[3] = {1,2,3};    // array
```

`sizeof(type)` -> bytes. `NULL` is null pointer.

---

## I/O (safe)

```c
char buf[100];
if (fgets(buf, sizeof buf, stdin)) {
    printf("you wrote: %s", buf);
}
```

Use `fgets` (not `gets`). Use `snprintf` for bounded formatting.

---

## Dynamic memory

```c
#include <stdlib.h>
int *arr = malloc(10 * sizeof *arr);
if (!arr) return 1; // check
arr[0] = 5;
free(arr);
```

Always `free` what you `malloc`.

---

## Quick mistakes to avoid

* forgetting `;`
* using uninitialized variables
* buffer overflow (writing past array end)
* dereferencing `NULL`

---

## Annotated assembly — Hello world (x86_64, System V, Intel syntax)

This shows how the C `printf("Hello, world
")` maps to simple assembly calls.

```asm
; data
.LC0:
    .string "Hello, world
"

; text
main:
    push rbp               ; save old base pointer
    mov rbp, rsp           ; set up new base pointer (stack frame)
    lea rdi, [rip + .LC0]  ; argument 1 (const char *) -> RDI
    call puts@PLT          ; call puts (prints string)
    mov eax, 0             ; return value 0 in RAX
    pop rbp                ; restore base pointer
    ret                    ; return to caller
```

Notes:

* `RDI` is where the first function argument goes (pointer to string).
* `call puts@PLT` jumps to the procedure linkage table entry for `puts` (dynamic linking).
* `ret` uses the return address pushed by the caller.

Reading compiled assembly helps you map C variables -> registers/memory and find where buffers live.

---

## Exercises — skeletons & tests

### 1) Sum two integers (stdin)

```c
#include <stdio.h>
int main(void){
    int a,b;
    if (scanf("%d %d", &a, &b) != 2) return 1;
    printf("%d
", a + b);
    return 0;
}
```

Test: `echo "2 3" | ./sum`

---

### 2) is_prime (simple)

```c
#include <stdio.h>
#include <math.h>
int is_prime(int n){
    if (n <= 1) return 0;
    for (int i = 2; i <= (int)sqrt((double)n); ++i)
        if (n % i == 0) return 0;
    return 1;
}
int main(void){
    printf("%d
", is_prime(97)); // expected 1
}
```

---

### 3) allocate, fill, free

```c
#include <stdio.h>
#include <stdlib.h>
int main(void){
    size_t n = 5;
    int *arr = malloc(n * sizeof *arr);
    if (!arr) return 1;
    for (size_t i=0;i<n;++i) arr[i] = (int)i*i;
    for (size_t i=0;i<n;++i) printf("%d ", arr[i]);
    puts("
");
    free(arr);
}
```

---

## One-page printable cheat-sheet (compact)

**Compile**: `gcc -std=c17 -O2 -Wall file.c -o file`
**Debug**: `gcc -g -O0 -Wall file.c -o file`
**Run**: `./file`

**Types**: `char`(1), `short`(2), `int`(4), `long`(8 on x86_64), pointers(8)

**Pointers**: `int *p = &x; *p = 5;`
**Arrays**: `int a[3]; sizeof(a) == 12`
**Strings**: NUL-terminated `char[]`; use `fgets`/`printf("%s")`

**Memory**: `malloc`, `free` — always check `malloc` result

**IO**: `fgets` for input, `snprintf` for safe formatting

**Common tools**:

* `gdb ./prog` — debug
* `objdump -d -M intel prog` — disassemble
* `readelf -s prog` — symbols
* `strings prog` — printable strings
* `file prog`, `ldd prog` — type & dynamic libs

**Avoid**: `gets`, `strcpy` without bounds, dereferencing NULL

---



### H3tehtävät

#### a) ok
Strings. Lataa ezbin-challenges.zip Aja 'passtr'. Selvitä oikea salasana 'strings' avulla. Selvitä myös lippu. (Ensisijaisesti katsomatta sorsia, jos osaat.)

Tässä piti katsoa pienestä ohjelmasta stringsillä ja siellä näkyi suoraan salasana.

#### b) Tee passtr.c -ohjelmasta uusi versio, jossa salasana ei näy suoraan sellaisenaan binääristä. Osoita testillä, että salasana ei näy. (Obfuskointi riittää.)

Ehhehe oma obfuskointi aika nolo.

ROT13 byt ChatGPT
```
#include <stdio.h>

void rot13(char *s) {
    for (int i = 0; s[i] != '\0'; i++) {
        char c = s[i];
        if (c >= 'a' && c <= 'z')
            s[i] = 'a' + (c - 'a' + 13) % 26;
        else if (c >= 'A' && c <= 'Z')
            s[i] = 'A' + (c - 'A' + 13) % 26;
    }
}

int main() {
    char text[] = "Hello, World!";
    rot13(text);
    printf("%s\n", text);  // Uryyb, Jbeyq!
    return 0;
}

```

Tässä jonkun käyttämä
https://yurisk.info/2017/06/25/binary-obfuscation-string-obfuscating-in-C/index.html


### c) Packd. Aja 'packd' paketista ezbin-challenges.zip. Mikä on salasana? Mikä on lippu? (Tämä tehtävä on hieman haastavampi. Kirjaa ylös kokeilemasi lähestymistavat ja keksimäsi hypoteesit. Toivottavasti pääset itse maaliin, mutta jos et, läpikävely paljastuu tunnilla...) ok


Tässä binääri oli pakatta upx-ohjelmalla, joten se piti vain `upx -d ohjelma`, jotta se tuli pakkaamattomaan muotoon. Ja sitten stringsillä näkyi salasana. 

Vinkki: komennolla `upx -l ./ohjelma`saadaan tietoa sen pakkaamisesta.

### h4 Kääntöpaikka (Tero)
# Ghidra — Concise Reverse‑Engineering Instructions (simple programs)

**Goal:** fast, practical steps to analyze small binaries (C-style programs, challenge binaries). No fluff — just techniques you will use in an assignment.

---

## 1. Quick setup

* Start Ghidra and create a new Project (Non‑Shared).
* File → Import File → choose the binary. Keep default loader; click **OK**.
* Double‑click the imported file to open it in CodeBrowser.
* When asked, run **Auto‑analysis** with default options (tick `Optimize for C/C++` if available). You can turn off expensive analyzers for bigger files.

## 2. First pass: reconnaissance (2–5 minutes)

* **Program Tree / Symbols:** open **Symbol Tree → Functions**. Look for `main` and other obvious names.
* **Strings:** Window → Defined Strings. Scan for human text, format strings, filenames, flags.
* **Imports:** check **Imports** (External Symbols) to infer library use (printf, strcmp, fopen, system, etc.).

## 3. Navigate `main` and the decompiler

* Double‑click `main` (or the top function) to open Listing and Decompiler side by side (press **F3** for decompiler view).
* **Decompiler** shows a C‑like view — use it as your primary reading surface. The Listing is for low‑level details.
* Read the decompiled flow: identify argument parsing, loops, strcmp/strstr checks, file IO.

## 4. Rename symbols & create comments (critical)

* Rename functions/variables: select an identifier → press **L** (Label) or right‑click → **Rename**. Use short meaningful names (`check_password`, `read_line`, `compare`).
* Add comments: **EOL comment** (right side) or **Pre‑Comment**. Use `// TODO:` style notes for hypotheses.
* Rename local variables from `var_XX` to `buf`, `len`, `i` to make the decompiler output readable.

## 5. Cross‑references (xrefs) and call graph

* Right‑click a function or string → **References** → **Show References to**. This finds who calls/uses it.
* Use **Call Graph** (Function Graph) for simple functions to understand call order.

## 6. Data & types

* If the decompiler shows wrong types (e.g., pointer vs int), right‑click a variable → **Data Type** → pick correct type (char*, int, struct). Reanalyze if needed.
* For buffers: change array sizes to match observed usage to avoid misleading decompilation.

## 7. Identify comparisons and crypto checks

* Look for `strcmp`, `strncmp`, `memcmp`, or custom loops. Follow the operands: one side may be user input, other a constant or computed digest.
* If a function performs many bit ops/rotates, it may implement a simple hash — follow constants and reverse arithmetic where possible.

## 8. Strings & format vulnerabilities

* If `printf` uses user data as format, note potential exploit. For analysis: find `printf` call, check argument passed (xrefs from string table).

## 9. Patching & experiments

* Use **Patch Program** (right‑click Listing → **Patch Program**) to modify instructions or change constants (e.g., replace conditional jump with NOP to bypass a check). Save a copy before patching.
* Recompile/Export patched binary: File → Export Program → choose format.

## 10. Scripting & automation (optional)

* Ghidra supports Python (Jython) and Java scripts. Useful for repetitive renaming, extracting strings, or pattern matching.
* Tools → Script Manager. Run small scripts to list all functions calling `strcmp` or to dump constants.

## 11. Useful shortcuts

* **F3** — Toggle decompiler.
* **L** — Create/rename label.
* **C** — Convert operand to code (useful when data treated as code).
* **Space** — Switch between Listing/Decompiler/Other windows.
* **T** — Change data type at cursor.

## 12. Deliverables 

* List the functions you analyzed and what they do (1–2 sentences each).
* Show the key strings/constants used for checks.
* Explain the vulnerability or logic and how you reached the flag (one short path: e.g., `input -> strcmp -> success`).
* Include any small patches you made (diff or hex bytes) and a screenshot of the decompiler with renamed variables.

---

### Short worked example (read quickly in the decompiler)

1. Open `main` → see `read_line` then `check_password`.
2. Follow xrefs to `check_password` → decompiler shows loop comparing bytes to constants.
3. Rename `var_30` → `input_buf`, change type to `char *`.
4. Patch conditional jump `jne` → NOP to bypass check; export patched binary.

---

Keep the process iterative: *read decompiled C → confirm in Listing → rename & comment → follow xrefs → patch or script if needed.*


# Ghidra walkthrough — binary: passtr

**Quick summary (results):**

* File: ELF 64-bit, x86-64, PIE, **not stripped**.
* Found flag in the binary strings: `FLAG{Tero-d75ee66af0a68663f15539ec0f46e3b1}`.

---

## 1) Initial reconnaissance (what I did before opening Ghidra)

* Ran `file` and `readelf -h` to identify architecture (ELF64 x86-64, PIE, not stripped).
* Extracted printable strings and saw: `"What's the password?"`, `strcmp`, `__isoc99_scanf`, and the flag `FLAG{Tero-...}`. This already gives the answer for the assignment, but we’ll still show how to confirm and trace it inside Ghidra.

## 2) Load the binary into Ghidra

* Create a new Project → Import `/mnt/data/passtr` → open it in **CodeBrowser**.
* Run Auto‑analysis with defaults (leave `Optimize for C/C++` checked).

## 3) Check Strings and Symbols in Ghidra

* Window → **Defined Strings**. Search for: `What's the password?` and `FLAG{`.
* You’ll find both strings. Right‑click a string → **References** → **Show References to** to see where it’s used.
* In this binary the string is referenced from `main` (or the top-level function). Since the binary is not stripped, function names and library imports (e.g., `strcmp`, `scanf`) are visible in the Symbol/Imports windows.

## 4) Open `main` and decompiler view

* Double‑click the function that references the string (likely `main`). Press **F3** to show the decompiler beside the Listing.
* In the decompiler you will typically see a call sequence similar to:

  ```c
  printf("What's the password?");
  scanf("%s", input_buf);
  if (strcmp(input_buf, "<constant or computed>") == 0) {
      puts("Yes! That's the password. FLAG{...}");
  }
  ```
* Confirm the constant being compared: it may be a hardcoded string or computed in a helper function. In this case, the flag string is present in the binary (so the comparison likely uses that constant or prints it directly).

## 5) Follow cross‑references and rename

* In Listing, locate the `strcmp` call. Right‑click → **References → Show References From** to see callers.
* Rename local variables (`var_...`) to `input_buf`, `ret`, etc. Add short comments where checks happen.

## 6) Confirm control flow and output

* Confirm that when the strcmp succeeds the program prints `Yes! That's the password. FLAG{...}` — that is the successful path for the assignment's input.

## 7) (Optional) Patch to bypass check

* If you need to produce a patched binary that always prints the success message, open **Patch Program** and change the conditional branch after `strcmp` (e.g., change `jne` to `je`, or replace the branch with NOPs and set the zero flag path). Save a copy before exporting.

## 8) Understanding Offsets vs Addresses

**Offset** = byte position where data starts *in the file* (counting from the beginning). It’s not the same as the runtime address (VMA).

* To list strings with offsets:

  ```bash
  strings -o passtr       # decimal offsets
  strings -t x passtr     # hex offsets
  ```
* To inspect bytes at a specific offset:

  ```bash
  hexdump -C -s <offset> -n <len> passtr
  xxd -s <offset> -l <len> passtr
  ```

**Mapping to memory (ELF):**

```
VMA = p_vaddr + (file_offset - p_offset)
```

Where `p_offset` and `p_vaddr` are from the program header (`readelf -l passtr`).

For PIE binaries, add the randomized base address at runtime:
`runtime address = base + VMA`.

In Ghidra, you see **virtual addresses**, not file offsets.

## 9) Deliverables you might include in your assignment

* Short description: architecture, tools used, strings of interest.
* Evidence: screenshot of decompiler with renamed variables and the `strcmp` call; screenshot of Defined Strings showing the flag.
* Exact flag: `FLAG{Tero-d75ee66af0a68663f15539ec0f46e3b1}`.
* (Optional) small hex patch or bytes changed to bypass the check.

---
### h4 a Asenna ghidra ok

### b) rever-C. Käänteismallinna packd-binääri C-kielelle Ghidralla. Etsi pääohjelma. Anna muuttujielle kuvaavat nimet. Selitä ohjelman toiminta. Ratkaise tehtävä binääristä, ilman alkuperäistä lähdekoodia. ezbin-challenges.zip

Eli tässä oli se kohta JNZ = jump if not zero, joka muutettiin päinvastaiseksi eli JZ jump if zero. Sitten se exportattiin uudelle nimelle ja piti vielä muistaa antaa oikeudet chmod+x.

Kun tuplaklikkaa decomplilerissa niin pääsee assembly-kohtaan.

__Ja muista, että ohjelma ajetaan komennolla ./ohjelma.__

Vinkki: Ghidra - Window - Function call graph


### Understanding "Follow Constants and Reverse Arithmetic"

#### 💡 What "Follow Constants" Means

"Follow constants" means tracking how fixed numerical values (constants) are used and transformed throughout the code. These constants often reveal what the program is doing or what kind of data it manipulates.

Constants can be things like:

* Magic numbers (`0xDEADBEEF`, `1337`, etc.)
* ASCII values (`0x41` = 'A')
* Offsets, array indexes, or masks (`0xFF`, `0x7F`, etc.)
* Hardcoded keys or configuration values

When you see constants in disassembly or decompiled pseudocode, follow how they move through the program. Ask: *Where did this value come from? What operations are done to it? What does it become?*

#### 🔍 Why It Matters

By following constants, you can:

* Recognize patterns like ASCII manipulation or encoding
* Identify key comparison values (`if (x == 5)`)
* Recover hardcoded keys or offsets
* Understand what arithmetic operations are meant to achieve

#### 🧩 Example

```c
eax = param_1 + 0x41;
```

If you follow `0x41`, you find:

* `0x41` = 65 decimal
* ASCII 65 = 'A'

So the program is likely converting or encoding characters — maybe part of a ROT or Caesar cipher.

#### ⚙️ "Reverse Arithmetic Where Possible"

This means simplifying or undoing math transformations to recover original logic.

Example:

```c
eax = input * 3 + 6;
```

Later:

```c
if (eax == 15)
```

Reverse the arithmetic:

```
input * 3 + 6 = 15 → input = 3
```

By reversing arithmetic, you uncover what inputs or values cause specific behavior.

#### ✅ Summary

| Phrase                 | Meaning                                                     |
| ---------------------- | ----------------------------------------------------------- |
| **Follow constants**   | Track and interpret fixed numeric values to reveal meaning  |
| **Reverse arithmetic** | Undo or simplify math operations to recover original intent |

Both techniques help translate low-level operations back into human-understandable logic during reverse engineering.




references decompile-ikkunassa

function call graph window-ikkunasta

retype variable

muuta listojen kokoa tarvittaessa

patch instruction - tee muutos. muuttaa binääriä välittömätsi. 

nuolilla voi undo ja redo

### 4d d) Nora CrackMe: Käännä binääreiksi Tindall 2023: NoraCodes / crackmes. Lue README.md: älä katso lähdekoodeja, ellet tarvitse niitä apupyöriksi. Näissä tehtävissä binäärejä käänteismallinnetaan. Binäärejä ei muokata, koska muutenhan jokaisen tehtävän ratkaisu olisi vaihtaa palautusarvoksi "return 0". ok

tässä piti vaan katso Ghidralla (tai stringillä) se salasana

Vinkki: sen sijaan että laittaa \merkki niin voi laittaa koko stringin 'sisään'.

### e) Nora crackme01. Ratkaise binääri.
piti vaan etsiä salasana

###) e) Nora crackme01e. Ratkaise binääri.
piti laittaa ettei se salasana hajoa

### f) Nora crackme02. Nimeä pääohjelman muuttujat käänteismallinnetusta binääristä ja selitä ohjelman toiminta. Ratkaise binääri.

__HUOM__ ascii lista
https://www.eso.org/~ndelmott/ascii.html

https://www.geeksforgeeks.org/dsa/ascii-table/





























