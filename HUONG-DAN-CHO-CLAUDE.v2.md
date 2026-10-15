# Dựng máy Mac mới giống hệt máy cũ — gửi Claude trên máy mới (bản đã rà soát)

> Viết ngày 15/10/2026 trên máy cũ **"MacBook Air của vu"** (user `vuducquang`, chip Apple,
> macOS 26.1). Quang nhờ Claude máy mới đọc file này và tự dựng máy cho giống máy cũ.
> Trả lời Quang bằng **tiếng Việt, ngắn**, báo từng mốc. Không chi tiền, không tự đăng nhập hộ.
> Đọc thêm `~/.claude/CLAUDE.md` + `MEMORY.md` nếu đã có trên máy (đi theo Trợ lý Di chuyển).
>
> 🆕 = chỗ bản rà soát thêm/sửa so với bản gốc. Mục có "(hỏi Quang)" là suy đoán, chưa chắc máy cũ có.

---

## Bước 0 — Máy này dựng theo cách nào?

```bash
whoami; ls ~/.claude/projects 2>/dev/null | head; find ~/.claude/projects -name '*.jsonl' | wc -l
```

- **Có `~/.claude/projects` và ≥116 file `.jsonl`** → Quang đã dùng Trợ lý Di chuyển. Phần lớn đã
  có sẵn → chỉ làm **phần KIỂM TRA + SỬA** (Bước 7), bỏ qua các bước cài.
- **Không có** → máy trắng, làm lần lượt Bước 0.5 → 7.
- `whoami` **khác `vuducquang`** → lịch sử session Claude sẽ không hiện (Claude nhớ theo đường dẫn
  `/Users/vuducquang`) và mọi file launchd/đường dẫn cứng phải sửa. Báo Quang trước khi làm tiếp.

## Bước 0.5 🆕 — Công cụ nền của Apple (bắt buộc trước Homebrew và `/usr/bin/python3`)

```bash
xcode-select -p || xcode-select --install     # Quang bấm "Cài đặt" trong hộp thoại, chờ xong
/usr/bin/python3 --version                     # phải ra 3.9.x
```

Máy mới chưa có Command Line Tools thì `/usr/bin/python3` chỉ là cái vỏ, Homebrew cũng không cài được.

---

## Bước 1 — App có giao diện (Quang tự tải, hoặc Claude cài bằng brew cask)

| App | Dùng để | Cách cài |
|---|---|---|
| Claude (desktop) | Claude Code + Cowork | `brew install --cask claude` |
| ChatGPT | Codex đăng nhập bằng gói ChatGPT | tải từ openai.com |
| Google Chrome | Claude in Chrome, YouTube, Vercel | `brew install --cask google-chrome` |
| Telegram, Zalo | nhận đơn, bot | App Store / web |
| zoom.us, TeamViewer | họp, điều khiển từ xa | web |
| **VOICEVOX** | giọng Nhật local 0đ | voicevox.hiroshiba.jp |
| CapCut | sửa video tay | web |
| LibreOffice | đọc/xuất pptx-docx-pdf | `brew install --cask libreoffice` |
| Cursor, Antigravity | trình sửa code (không bắt buộc) | web |
| Figma | thiết kế | `brew install --cask figma` |

**App tự làm, không tải được trên mạng** → phải chép từ máy cũ (AirDrop/USB):
`/Applications/AOC Tool.app`, `/Applications/QVvoice.app`, `/Applications/Sticky Notepad.app`,
`~/Applications/QuangVideoStudio.app`, `~/Applications/VoicePrompt.app`.
🆕 Nếu mở báo "không xác minh được nhà phát triển": `xattr -cr "<đường dẫn app>"` rồi mở lại
(chỉ làm với app của chính Quang).

Bỏ qua (thuộc IIJOBS — đã bàn giao cho anh Tường 18/08): Android Studio, android-commandlinetools,
temurin@17/21, cocoapods. Chỉ cài khi Quang yêu cầu.

---

## Bước 2 — Homebrew + công cụ dòng lệnh

⚠️ **Homebrew trên máy cũ đang HỎNG** (`brew.sh` thiếu `utils/os.sh`, mọi lệnh brew lỗi). Nếu
`/opt/homebrew` bị chép sang thì cũng hỏng theo → chạy `brew doctor`; lỗi thì cài lại Homebrew sạch
(lệnh trên brew.sh — Quang phải tự gõ mật khẩu).

```bash
brew tap supabase/tap; brew tap steipete/tap
brew install ffmpeg-full yt-dlp aria2 whisper-cpp tesseract poppler rubberband \
  p7zip ripgrep gh node@22 pnpm deno python@3.12 python@3.13 gnupg \
  supabase/tap/supabase steipete/tap/remindctl
brew install --cask gcloud-cli
```

🆕 Chỉnh so với bản gốc: bỏ `node` thường khỏi lệnh (bản gốc có cả `node` lẫn `node@22`, dễ lẫn bản).

- **ffmpeg phải là `ffmpeg-full`** (máy cũ: `/opt/homebrew/bin/ffmpeg → Cellar/ffmpeg-full/8.1_1`).
  Bản `ffmpeg` thường thiếu bộ lọc dựng video (libass, rubberband, vidstab…). Nếu cả hai cùng có
  thì `brew unlink ffmpeg && brew link --overwrite ffmpeg-full`.
- 🆕 **`node@22` là "keg-only"** — cài xong chưa có lệnh `node` trong PATH. Chạy
  `brew link --overwrite --force node@22` (hoặc thêm `/opt/homebrew/opt/node@22/bin` vào PATH),
  rồi `node -v` phải ra v22.x **trước khi** chạy `npm i -g`.
- Công cụ Node cài toàn cục:
  ```bash
  npm i -g @openai/codex vercel firebase-tools docx ui-ux-pro-max-cli @google/design.md
  ```
  (máy cũ còn `openclaw` — hệ thống cũ đã tắt, KHÔNG cài lại trừ khi Quang muốn.)
- `uv` (`curl -LsSf https://astral.sh/uv/install.sh | sh`) rồi `uv tool install markitdown-mcp`.
- Trình duyệt để dựng video bằng HTML: `npx playwright install chromium`.
- 🆕 **File mô hình của whisper-cpp**: `brew` chỉ cài chương trình, KHÔNG kèm mô hình (`ggml-*.bin`).
  Hỏi Quang máy cũ để mô hình ở đâu (`find ~ -name 'ggml-*.bin' 2>/dev/null` trên máy cũ) rồi chép
  sang, nếu không `whisper-cli` sẽ chạy không ra chữ.
- 🆕 (hỏi Quang) Nếu có OCR tiếng Nhật/Việt: `brew install tesseract-lang`.

---

## Bước 3 — Python

⚠️ **Luật của máy cũ: script video/văn phòng chạy bằng `/usr/bin/python3` (3.9.6 của Apple)**,
KHÔNG dùng `python3` của Homebrew (3.14 lỗi `pyexpat`, đọc pptx/docx/xml đều chết).

```bash
/usr/bin/python3 -m pip install --user --upgrade pip      # 🆕 pip của Apple quá cũ, nâng trước
/usr/bin/python3 -m pip install --user -r ~/Desktop/CAI-MAY-MOI/python-packages.txt
```

File `python-packages.txt` (135 gói, đã kiểm: đủ 135 dòng, phiên bản khớp Python 3.9) chép từ máy cũ.
Gói nặng: torch, torchvision, onnxruntime, insightface, gfpgan, rembg, opencv. Gói hay dùng:
python-pptx, python-docx, openpyxl, edge-tts, pykakasi, jaconv, google-api-python-client.
Nếu một gói cài lỗi thì bỏ qua gói đó, cài tiếp, cuối cùng báo danh sách gói lỗi.

🆕 Hai chỗ dễ lỗi trong danh sách này:
- `tb-nightly==2.21.0a20251023` là bản nightly, hay bị gỡ khỏi PyPI → lỗi thì bỏ qua (chỉ để xem
  tensorboard, không ảnh hưởng dựng video).
- `basicsr` / `gfpgan` với `torchvision` mới hay báo
  `No module named 'torchvision.transforms.functional_tensor'`. Cách sửa: mở
  `.../site-packages/basicsr/data/degradations.py`, đổi
  `from torchvision.transforms.functional_tensor import rgb_to_grayscale` thành
  `from torchvision.transforms.functional import rgb_to_grayscale`. Hỏi Quang trước khi sửa file thư viện.
- `opencv-python` và `opencv-python-headless` cùng có là đúng như máy cũ, đừng gỡ một bản.

**Môi trường ảo riêng của dự án** (không chép được giữa 2 máy, phải dựng lại):
- `~/quangvideostudio/.venv` ← `requirements.txt` + `requirements-worker.txt`
- `~/claude-telegram-bot/video-pipeline/.depthflow-venv`

**Mô hình AI tải sẵn** (tự tải lại khi chạy lần đầu, hoặc chép từ máy cũ cho nhanh):
`~/.cache/huggingface` (5,2 GB), `~/.insightface` (600 MB), `~/.u2net` (170 MB).

---

## Bước 4 — Dự án + khoá bí mật + cấu hình shell

1. Dự án đã có trên GitHub (tài khoản cá nhân **QuangAIjp**) → `git clone` về **đúng đường dẫn cũ**
   (`~/<tên>`): QVsocial-studio, aitoolfree-pdf, claude-telegram-bot, iphat-bot-legal,
   quangvideostudio, quangvideostudio-v2, qv-short-engine, translateaoc.
   Chưa có GitHub → phải chép từ máy cũ: `VIDEO`, `ai-reel-factory`, `employee-photo-register`,
   `kham-suc-khoe-aoc`, `kho-hieu-ung-qv`, `qv-vox-viet`, `voicepilot`, `mendan-tokutei`.
   ⚠️ `~/APPS/` toàn **lối tắt (symlink)** — dựng lại lối tắt, KHÔNG dời thư mục thật.
2. Dự án Node → `npm install` trong: QVsocial-studio, claude-telegram-bot, ai-reel-factory,
   quangvideostudio-v2, mendan-tokutei.
3. **Khoá API KHÔNG có trên GitHub** (đã cố ý chặn). Quang phải chép tay từ máy cũ qua
   **AirDrop hoặc USB** — không gửi qua chat, iCloud, Telegram. Danh sách file:
   ```
   ~/quangvideostudio/.env  .groq_key  .ausynclab_key
   ~/quangvideostudio-v2/.env.local
   ~/claude-telegram-bot/.env   youtube/client_secret.json
   ~/claude-telegram-bot/youtube/token_monitor.py   (bị chặn nhầm vì tên có chữ "token")
   ~/kham-suc-khoe-aoc/.env   ~/employee-photo-register/.env   ~/mendan-tokutei/.env.local
   ~/translateaoc/.env   ~/.config/supabase-personal/.env   ~/.telegram-codex-bridge/.env
   ~/VIDEO/aoc/ton-tu-kinh-doanh/may-dung/.azure_key_hd
   ```
   Claude **không được in nội dung khoá** ra màn hình, chỉ kiểm file có tồn tại.
4. 🆕 **Cấu hình môi trường — bản gốc bỏ sót, dễ làm máy mới "chạy khác máy cũ"** (hỏi Quang rồi chép
   bằng AirDrop/USB, các file này có thể chứa khoá nên cũng không in ra màn hình):
   ```
   ~/.zshrc  ~/.zprofile  ~/.gitconfig
   ~/.ssh/   (khoá SSH cho 2 tài khoản GitHub — cực kỳ nhạy cảm, chỉ chép trực tiếp)
   ~/Library/Fonts/   (font tiếng Nhật/phụ đề video, nếu Quang có cài thêm)
   ```
   Kiểm trên máy cũ có `crontab -l` không, và danh sách Login Items
   (Cài đặt hệ thống → Cài đặt chung → Mục đăng nhập). Có thì chép/dựng lại.

---

## Bước 5 — Claude Code + Codex

- Thư mục `~/.claude` (skill, lệnh, rules, agents, **memory**, CLAUDE.md, lịch sử session) phải đi
  theo nguyên thư mục từ máy cũ. Thư mục app: `~/Library/Application Support/Claude`.
  Lịch sử Codex: `~/.codex`.
- Nối Codex vào Claude (nếu `claude mcp list` chưa có `codex`):
  ```bash
  claude mcp add codex -s user -- codex mcp-server -c sandbox_mode=workspace-write -c approval_policy=never
  ```
  (`approval_policy=never` nghĩa là Codex tự chạy lệnh không hỏi — đúng như máy cũ, Quang tự cân nhắc.)
- Plugin đang dùng: `vercel-plugin@vercel-vercel-plugin`.
- 🆕 Chrome: đăng nhập Chrome bằng tài khoản Google rồi cài lại tiện ích **Claude in Chrome**
  (Chrome profile không đi theo `~/.claude`).

---

## Bước 6 — Việc QUANG phải tự làm (Claude chỉ nhắc, không làm hộ)

**Đăng nhập:**
- App Claude → `aocducquang@gmail.com`
- Codex → `codex login` (dùng tài khoản ChatGPT, không tốn phí API)
- GitHub → `gh auth login`. Có 2 tài khoản, **không được trộn**: cá nhân `QuangAIjp`, công việc
  `iijobs128-sys` (xem skill `git-dual-machine`).
- Vercel → **chỉ tài khoản cá nhân** `aocducquang@gmail.com`. Trước khi `vercel login` phải đăng
  xuất mọi tài khoản iijobs trong trình duyệt; sau đó `npx vercel whoami` phải ra `aocducquang-4347`.
- firebase, supabase, gcloud. ⚠️ gcloud dự án `claude-496014` là **tài khoản TRẢ TIỀN** — đăng nhập
  được, nhưng không chạy gì tốn tiền nếu Quang chưa đồng ý.
- 🆕 Telegram, Zalo, ChatGPT, Figma: đăng nhập lại trong app.

**Cấp quyền** (Cài đặt hệ thống → Quyền riêng tư & Bảo mật): Truy cập toàn bộ ổ đĩa, Trợ năng,
Ghi màn hình → cấp cho Claude, Terminal, Chrome.

**Chạy 24/7 khi gập nắp:** `sudo pmset -c disablesleep 1` (cần mật khẩu).

---

## Bước 7 — Tác vụ chạy nền (launchd) + KIỂM TRA

Máy cũ có các tác vụ này đang bật trong `~/Library/LaunchAgents/`:

| File | Việc |
|---|---|
| com.aoc.claude-telegram-bot | bot Telegram chính |
| com.quang.telegram-codex-bridge | cầu nối Telegram ↔ Codex |
| com.qv.qvsocialstudio | QVsocial Studio cổng 4040 |
| com.qv.autobackup | QVStudio tự backup GitHub 5 phút/lần |
| com.qv.autofinish, com.qv.devscan, com.qv.tokenmonitor | phụ trợ QVStudio |
| com.aoc.youtube-token-refresh, com.aoc.youtube-synthetic-backfill | giữ token YouTube |
| com.qv.aoc-keepalive, local.keepawake | giữ máy thức |

Thư mục `_tat-theo-yeu-cau-Quang/` và `_tat-tam-thoi/` là các tác vụ **Quang đã tắt** → GIỮ TẮT.

🚨 **QUAN TRỌNG: KHÔNG để 2 máy cùng chạy bot.** Trước khi bật tác vụ nền trên máy mới, Quang phải
tắt máy cũ (hoặc `launchctl unload` các tác vụ trên máy cũ). Nếu không, bot Telegram sẽ bị lỗi
409 xung đột, tin nhắn lặp, và 2 máy cùng đẩy backup.

🆕 **Chỉ bật sau khi** đã xong Bước 3–4: dự án đã clone, `npm install` xong, file khoá (`.env`...) đã
chép, venv đã dựng. Nếu chưa, tác vụ sẽ chạy lỗi và tự khởi động lại liên tục. Trước khi bật, kiểm
đường dẫn trong từng plist: `grep -A3 ProgramArguments ~/Library/LaunchAgents/com.*.plist` và chắc
các file đó tồn tại.

Bật lại (sau khi Quang xác nhận máy cũ đã tắt):
```bash
# 🆕 bản gốc dùng com.{aoc,qv,quang}.*.plist nên BỎ SÓT local.keepawake.plist
for p in ~/Library/LaunchAgents/com.{aoc,qv,quang}.*.plist ~/Library/LaunchAgents/local.keepawake.plist; do
  [ -f "$p" ] && launchctl load -w "$p"
done
launchctl list | grep -E 'qv|aoc|quang|keepawake'
```

**Danh sách kiểm tra — phải chạy THẬT rồi mới báo xong:**
```bash
# 🆕 bản gốc grep -cE 'ass|rubberband|vidstab' dễ đếm nhầm (chữ "ass" nằm trong nhiều tên khác)
ffmpeg -hide_banner -filters | grep -E ' (ass|subtitles|rubberband|vidstabdetect|vidstabtransform) '  # phải ra đủ 5 dòng
ls /opt/homebrew/opt/ | grep ffmpeg                                  # phải là ffmpeg-full, không có ffmpeg thường
whisper-cli --help >/dev/null && echo whisper OK
find ~ -maxdepth 4 -name 'ggml-*.bin' 2>/dev/null | head -3         # 🆕 có file mô hình whisper
/usr/bin/python3 -c "import pptx, docx, cv2, numpy, edge_tts, pykakasi; print('python OK')"
node -v                                                              # 🆕 v22.x
codex --version; claude mcp list; gh auth status; npx vercel whoami
find ~/.claude/projects -name '*.jsonl' | wc -l                     # >= 116
curl -s -o /dev/null -w '%{http_code}\n' http://localhost:4040     # QVsocial
launchctl list | grep -E 'qv|aoc|quang|keepawake'
```
Thêm 1 phép thử thật: dựng 1 video ngắn 5 giây bằng ffmpeg + playwright, mở được và có tiếng.

Báo cáo cho Quang dạng bảng: ✅ chạy được / ❌ hỏng (kèm dòng lỗi) / ⏳ chờ Quang làm (đăng nhập,
mật khẩu, chép khoá).
