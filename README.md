# 🛸 Antigravity Toolkit & Maintenance

> **Repository สำหรับ Antigravity Global Rules และ Toolkit**

## 📁 Files

| File | Purpose |
|:-----|:--------|
| `GEMINI.md` | Global Rules → Replace ไปที่ `~/.gemini/GEMINI.md` |
| `README.md` | คู่มือ + Agent Instructions + Embedded Toolkit |

---

## 🚀 Quick Start

### 1. Sync Global Rules ไปยัง ~/.gemini/

พิมพ์: **"sync GEMINI"** หรือ **"update global rules"**

Agent จะ:
1. Compare ก่อนทุกครั้ง
2. แจ้ง diff ให้ user เห็น
3. Replace เมื่อ user approve

### 2. สร้าง Toolkit Script

พิมพ์: **"สร้าง antigravity_toolkit.sh ให้หน่อย"**

Agent จะสร้าง script ที่ `~/Documents/fix-model-selection/antigravity_toolkit.sh`

---

## 🛸 Factory Reset Guide

> **Purpose:** Complete reset to fix 413 errors, corruption, or critical issues.  
> **Safety:** Your project files (Documents) are untouched.

### 📋 Quick Reference

| Action | Command |
|:-------|:--------|
| **Full Reset + Restore** | `./antigravity_toolkit.sh full` ✅ RECOMMENDED |
| Factory Reset Only | `./antigravity_toolkit.sh factory` |
| Restore Rules | `./antigravity_toolkit.sh restore-rules` |
| Restore Brain | `./antigravity_toolkit.sh restore-brain` |

### 🔧 Usage

```bash
cd ~/Documents/fix-model-selection
./antigravity_toolkit.sh full
```

This will:
1. Factory reset (backup `~/.gemini` → `~/.gemini_OLD_*`)
2. Automatically restore Global Rules
3. Ready to start Antigravity!

### 🚨 Troubleshooting

| Issue | Solution |
|:------|:---------|
| 413 error | `./antigravity_toolkit.sh factory` |
| 413 returns after brain restore | Factory reset again, don't restore brain |
| Agent doesn't recognize Global Rules | Restart IDE after restore |
| Backup folder not found | `ls -la ~/.gemini_OLD_*` |

### 🧹 Cleanup (After 24-48 Hours)

```bash
rm -rf ~/.gemini_OLD_*
```

---

## 🤖 Agent Instructions

### Create Toolkit Script

> เมื่อ user ขอสร้าง toolkit ให้ทำตามขั้นตอนนี้:

1. **สร้าง directory** (ถ้ายังไม่มี):
   ```bash
   mkdir -p ~/Documents/fix-model-selection
   ```

2. **สร้างไฟล์** `~/Documents/fix-model-selection/antigravity_toolkit.sh` จาก embedded script ด้านล่าง

3. **ให้สิทธิ์ execute**:
   ```bash
   chmod +x ~/Documents/fix-model-selection/antigravity_toolkit.sh
   ```

4. **แจ้ง user** ว่าพร้อมใช้งาน

### Sync Global Rules

> เมื่อ user ต้องการ sync GEMINI.md ไปยัง ~/.gemini/

**⚠️ ALWAYS Compare First!**

1. **Compare ก่อนทุกครั้ง**:
   ```bash
   diff ~/Documents/startup/GEMINI.md ~/.gemini/GEMINI.md
   ```
   หรือใช้ `view_file` ทั้ง 2 ไฟล์แล้วเทียบ

2. **แจ้ง user ถึงความแตกต่าง**:
   - อะไรจะถูกเพิ่ม/ลบ/เปลี่ยน
   - ขอ approval ก่อน replace

3. **Replace เมื่อ user approve**:
   ```bash
   cp ~/Documents/startup/GEMINI.md ~/.gemini/GEMINI.md
   ```

4. **Verify**:
   ```bash
   head -5 ~/.gemini/GEMINI.md
   ```

---

## 📦 Embedded Toolkit Script

> **สำหรับ Agent:** เมื่อ user ต้องการสร้าง toolkit ให้ copy script ด้านล่างไปสร้างเป็นไฟล์ `.sh`

<details>
<summary>🔽 Click to expand: antigravity_toolkit.sh</summary>

```bash
#!/bin/bash
# ═══════════════════════════════════════════════════════════════════════════
# 🛸 ANTIGRAVITY TOOLKIT
# ═══════════════════════════════════════════════════════════════════════════
# Factory Reset, Backup & Restore operations.
# Usage: ./antigravity_toolkit.sh [command]
#
# Commands:
#   full          - Full reset: factory + auto restore rules (RECOMMENDED)
#   factory       - Factory reset only
#   restore-rules - Restore Global Rules from backup
#   restore-brain - Restore brain/conversations from backup
#   help          - Show this help
# ═══════════════════════════════════════════════════════════════════════════

set -e

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Paths
GEMINI_DIR="$HOME/.gemini"
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
STARTUP_GEMINI="$HOME/Documents/startup/GEMINI.md"

# ───────────────────────────────────────────────────────────────────────────
# Functions
# ───────────────────────────────────────────────────────────────────────────

show_help() {
    echo -e "${BLUE}🛸 Antigravity Toolkit${NC}"
    echo ""
    echo "Usage: $0 [command]"
    echo ""
    echo "Commands:"
    echo -e "  ${YELLOW}full${NC}          Full reset + restore rules (RECOMMENDED)"
    echo -e "  ${RED}factory${NC}       Factory reset only"
    echo -e "  ${GREEN}restore-rules${NC} Restore Global Rules"
    echo -e "  ${GREEN}restore-brain${NC} Restore brain/conversations"
    echo -e "  help          Show this help"
    echo ""
    echo "Quick Usage:"
    echo -e "  ${YELLOW}$0 full${NC}   # One command to reset and restore"
}

confirm() {
    echo -e "${YELLOW}⚠️  $1${NC}"
    echo "Press CTRL+C within 5 seconds to cancel..."
    sleep 5
}

find_latest_backup() {
    LATEST=$(ls -td "$HOME"/.gemini_OLD_* 2>/dev/null | head -1)
    echo "$LATEST"
}

# ───────────────────────────────────────────────────────────────────────────
# FACTORY: Complete factory reset
# ───────────────────────────────────────────────────────────────────────────
cmd_factory() {
    echo -e "${RED}☢️  FACTORY RESET${NC}"
    confirm "This will COMPLETELY reset the Agent. All history will be moved to backup."
    echo ""
    
    echo "🔪 Closing Antigravity..."
    pkill -f "Antigravity" 2>/dev/null || true
    sleep 2
    
    BACKUP_NAME="$HOME/.gemini_OLD_$(date +%Y%m%d_%H%M%S)"
    
    echo "📦 Moving .gemini to $BACKUP_NAME..."
    mv "$GEMINI_DIR" "$BACKUP_NAME"
    
    echo ""
    echo -e "${GREEN}✅ Factory reset complete!${NC}"
    echo ""
    echo "👉 Next steps:"
    echo "   1. Restart Antigravity"
    echo "   2. Run: $0 restore-rules"
    echo "   3. (Optional) Run: $0 restore-brain"
}

# ───────────────────────────────────────────────────────────────────────────
# RESTORE-RULES: Restore Global Rules from startup/GEMINI.md
# ───────────────────────────────────────────────────────────────────────────
cmd_restore_rules() {
    echo -e "${GREEN}📋 RESTORE GLOBAL RULES${NC}"
    echo ""
    
    if [ ! -d "$GEMINI_DIR" ]; then
        echo "Creating $GEMINI_DIR..."
        mkdir -p "$GEMINI_DIR"
    fi
    
    if [ -f "$STARTUP_GEMINI" ]; then
        echo "📄 Copying from $STARTUP_GEMINI..."
        cp "$STARTUP_GEMINI" "$GEMINI_DIR/GEMINI.md"
        
        if [ -s "$GEMINI_DIR/GEMINI.md" ]; then
            echo -e "${GREEN}✅ Global Rules restored to ~/.gemini/GEMINI.md${NC}"
            echo ""
            echo "Preview:"
            head -5 "$GEMINI_DIR/GEMINI.md"
        else
            echo -e "${RED}❌ Failed to copy Global Rules${NC}"
        fi
    else
        echo -e "${RED}❌ Source file not found: $STARTUP_GEMINI${NC}"
        echo "Please run: git clone https://github.com/mr-phariyawit/startup.git ~/Documents/startup"
    fi
}

# ───────────────────────────────────────────────────────────────────────────
# RESTORE-BRAIN: Restore brain from backup
# ───────────────────────────────────────────────────────────────────────────
cmd_restore_brain() {
    echo -e "${GREEN}🧠 RESTORE BRAIN${NC}"
    echo ""
    
    BACKUP=$(find_latest_backup)
    
    if [ -z "$BACKUP" ]; then
        echo -e "${RED}❌ No backup found!${NC}"
        echo "Looking for: ~/.gemini_OLD_*"
        exit 1
    fi
    
    echo "Found backup: $BACKUP"
    
    if [ -d "$BACKUP/antigravity" ]; then
        confirm "This will restore brain from: $BACKUP"
        echo ""
        
        echo "📦 Copying antigravity folder..."
        cp -r "$BACKUP/antigravity" "$GEMINI_DIR/antigravity"
        
        echo ""
        echo -e "${GREEN}✅ Brain restored!${NC}"
        du -sh "$GEMINI_DIR/antigravity" 2>/dev/null || true
    else
        echo -e "${RED}❌ No antigravity folder in backup${NC}"
    fi
}

# ───────────────────────────────────────────────────────────────────────────
# FULL: Complete reset + restore in one command
# ───────────────────────────────────────────────────────────────────────────
cmd_full() {
    echo -e "${YELLOW}🛸 FULL RESET + RESTORE${NC}"
    confirm "This will factory reset and restore Global Rules."
    echo ""
    
    echo "═══════════════════════════════════════════════════════════════════"
    echo -e "${RED}STEP 1: FACTORY RESET${NC}"
    echo "═══════════════════════════════════════════════════════════════════"
    
    pkill -f "Antigravity" 2>/dev/null || true
    sleep 2
    
    BACKUP_NAME="$HOME/.gemini_OLD_$(date +%Y%m%d_%H%M%S)"
    
    if [ -d "$GEMINI_DIR" ]; then
        echo "📦 Moving .gemini to $BACKUP_NAME..."
        mv "$GEMINI_DIR" "$BACKUP_NAME"
        echo -e "${GREEN}✅ Backup created${NC}"
    else
        echo "⚠️  No existing .gemini folder found"
    fi
    
    echo ""
    
    echo "═══════════════════════════════════════════════════════════════════"
    echo -e "${GREEN}STEP 2: RESTORE GLOBAL RULES${NC}"
    echo "═══════════════════════════════════════════════════════════════════"
    
    mkdir -p "$GEMINI_DIR"
    
    if [ -f "$STARTUP_GEMINI" ]; then
        echo "📄 Copying from $STARTUP_GEMINI..."
        cp "$STARTUP_GEMINI" "$GEMINI_DIR/GEMINI.md"
        
        if [ -s "$GEMINI_DIR/GEMINI.md" ]; then
            echo -e "${GREEN}✅ Global Rules restored${NC}"
        else
            echo -e "${RED}❌ Failed to copy Global Rules${NC}"
        fi
    else
        echo -e "${RED}❌ Source file not found: $STARTUP_GEMINI${NC}"
        echo "Please clone: git clone https://github.com/mr-phariyawit/startup.git ~/Documents/startup"
    fi
    
    echo ""
    echo "═══════════════════════════════════════════════════════════════════"
    echo -e "${GREEN}✅ COMPLETE!${NC}"
    echo "═══════════════════════════════════════════════════════════════════"
    echo ""
    echo "👉 Now:"
    echo "   1. Start Antigravity"
    echo "   2. (Optional) Run: $0 restore-brain"
}

# ───────────────────────────────────────────────────────────────────────────
# Main
# ───────────────────────────────────────────────────────────────────────────

case "${1:-help}" in
    full)          cmd_full ;;
    factory)       cmd_factory ;;
    restore-rules) cmd_restore_rules ;;
    restore-brain) cmd_restore_brain ;;
    *)             show_help ;;
esac
```

</details>

---

*🛸 Antigravity Toolkit v3.1*
