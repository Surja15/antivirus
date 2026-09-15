# Antivirus — Setup Guide

---

## 1. Install Dependencies

```bash
sudo apt update
sudo apt install gcc make python3 python3-pip libyara-dev yara git -y
pip install pillow
```

---

## 2. Project Folder Structure

Create everything in one place:

```bash
mkdir ~/blackswan
cd ~/blackswan
mkdir myrule myrule/compiled quarantine
```

Your final structure will look like:
```
~/blackswan/
├── engine.c
├── quarantine.c
├── restore2.c
├── rtm.c
├── gui.py
├── exceptions.txt        ← create empty for now
├── images/               ← your GUI images go here
├── myrule/
│   ├── yourule.yar       ← raw rules from github
│   └── compiled/         ← compiled .yarac files go here
└── quarantine/           ← auto-created by quarantine.c
```

---

## 3. Get YARA Rules

```bash
cd ~/blackswan/myrule
git clone <your-rules-repo-url> .
```

---

## 4. Compile YARA Rules

Each `.yar` file needs to be compiled to `.yarac`:

```bash
cd ~/blackswan/myrule
for f in *.yar; do
    yarac "$f" "compiled/${f%.yar}.yarac"
done
```

Verify:
```bash
ls compiled/
```
You should see `.yarac` files.

---

## 5. Fix File Paths in Each Code

### engine.c
Change rules_dir to your path:
```c
const char* rules_dir = "/home/YOUR_USERNAME/blackswan/myrule/compiled/";
```

### quarantine.c
Change quarantine dir:
```c
#define QUARANTINE_DIR "/home/YOUR_USERNAME/blackswan/quarantine"
```

### restore2.c
Change quarantine dir:
```c
#define QUARANTINE_DIR "/home/YOUR_USERNAME/blackswan/quarantine"
```

### rtm.c
Change engine path:
```c
execl("/home/YOUR_USERNAME/blackswan/engine",
      "./engine", data->filePath, (char *)NULL);
```

### gui.py
Change both paths:
```python
ENGINE_PATH     = "/home/YOUR_USERNAME/blackswan/engine"
ENGINE_RTM_PATH = "/home/YOUR_USERNAME/blackswan/rtm"
```

Replace YOUR_USERNAME with your actual Linux username. Check it with:
```bash
whoami
```

---

## 6. Create exceptions.txt

```bash
touch ~/blackswan/exceptions.txt
```

Add any folders to exclude from RTM, one per line:
```
/proc
/sys
/dev
```
Leave empty if you don't need exclusions.

---

## 7. Compile

```bash
cd ~/blackswan

# Engine (links quarantine in)
gcc engine.c quarantine.c -o engine -lyara

# RTM
gcc rtm.c -o rtm -lpthread

# Restore utility (standalone)
gcc restore.c -o restore
```

---

## 8. Run

```bash
cd ~/blackswan
python3 gui.py
```

---

## 9. Quick Test (without GUI)

Test engine on a file directly:
```bash
# Download EICAR test file (harmless AV test string)
echo 'X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*' > eicar.txt
./engine eicar.txt
```

Test RTM:
```bash
./rtm ~/Downloads &
# Now create/modify a file in Downloads and watch output
```

Test restore:
```bash
./restore2 eicar.txt
./restore2 --> GUI Enter Filename --> Enter Master Key --> File Restored
```

---

## Recompile After Any Code Change

```bash
gcc engine.c quarantine.c -o engine -lyara   # if engine.c or quarantine.c changed
gcc rtm.c -o rtm -lpthread                   # if rtm.c changed
gcc restore2.c -o restore2                   # if restore.c changed
```

---

## Common Errors

| Error | Fix |
|---|---|
| `libyara not found` | `sudo apt install libyara-dev` |
| `FileNotFoundError` in GUI | Wrong path in ENGINE_PATH or ENGINE_RTM_PATH |
| `multiple definition of main` | quarantine.c still has main() — delete it |
| `unknown type FILE` in rtm.c | Add `#include <stdio.h>` at top |
| Rules not matching | Check compiled/ has .yarac files |
| Permission denied on engine/rtm | `chmod +x engine rtm` |

## Screenshots
<img width="1440" height="871" alt="BlackSwanFileUpload" src="https://github.com/user-attachments/assets/e60bfd07-c960-4300-8d94-58bbc69f29c5" />
<img width="1427" height="862" alt="BlackSwanDirectoryScan" src="https://github.com/user-attachments/assets/a81b49d4-81eb-48a2-85fd-f6dfaae03d1c" />
<img width="1425" height="872" alt="BlackSwanRTM" src="https://github.com/user-attachments/assets/c623e6ef-c695-47db-b1cf-db0f330ea4e5" />
<img width="1212" height="878" alt="restore2" src="https://github.com/user-attachments/assets/07a45228-fa1e-4c2a-b326-66c9b9fcb4ff" />
<img width="428" height="57" alt="BlackSwanRestore2" src="https://github.com/user-attachments/assets/81b88692-01f5-4c6f-b651-cd309bd5f84b" />
