# DEPLOY — Website pexis.vn

## Thong tin co ban
- **URL**: https://www.pexis.vn
- **Hosting**: GitHub Pages (repo `noland81/pexis-website`)
- **Source path**: `C:\code\Printolab\pexis-website\`
- **Branch**: `master`
- **Cau truc**: static site — chi co `index.html`, `icon.png`, `CNAME`

---

## GitHub Pages hoat dong nhu nao

- Repo `noland81/pexis-website` bat GitHub Pages tu branch `master`
- Moi khi push len `master`, GitHub tu dong deploy trong 1-2 phut
- Noi dung `index.html` duoc serve truc tiep tai domain

---

## CNAME + Cloudflare DNS

- File `CNAME` trong repo chua: `www.pexis.vn`
- **Cloudflare** quan ly domain `pexis.vn`:
  - `www.pexis.vn` -> CNAME tro ve GitHub Pages
  - A records (DNS Only) tro ve GitHub Pages IPs cho root domain
  - `api.pexis.vn` -> Cloudflare Proxied tro ve origin server (cho PexisCore tunnel)

---

## Download links — Base64 encoding

Website dung base64 encoded URLs trong attribute `data-dl` de tranh bot/crawler tim thay link truc tiep.

### Cach hoat dong
```html
<a href="javascript:void(0)"
   data-dl="BASE64_ENCODED_URL"
   onclick="window.location.href=atob(this.dataset.dl)">
   Tai ve
</a>
```

Khi user click, `atob()` decode base64 -> redirect den URL thuc.

### 6 cho data-dl trong index.html

| # | App | Platform | Vi tri trong file |
|---|-----|----------|-------------------|
| 1 | PexisOne Desktop | Windows | Section san pham (~dong 1207) |
| 2 | PexisOne Desktop | Mac source | Section san pham (~dong 1211) |
| 3 | PexisConnect | Windows | Section san pham (~dong 1317) |
| 4 | PexisOne Desktop | Windows | Section tai ve (~dong 1556) |
| 5 | PexisConnect | Windows | Section tai ve (~dong 1617) |
| 6 | PexisConnect | Mac source | Section tai ve (~dong 1629) |

---

## Cap nhat download links khi co release moi

### Buoc 1: Encode URL moi thanh base64

```bash
# PexisOne Desktop Win
echo -n "https://github.com/noland81/pexis-website/releases/download/v${NEW_TAG}/PexisOne-Desktop-v${VERSION}.exe" | base64 -w0

# PexisOne Desktop Mac source
echo -n "https://github.com/noland81/pexis-website/releases/download/v${NEW_TAG}/PexisOne-Desktop-v${VERSION}-mac-source.zip" | base64 -w0

# PexisConnect Win
echo -n "https://github.com/noland81/pexis-website/releases/download/v${NEW_TAG}/PexisConnect-v${VERSION}.exe" | base64 -w0

# PexisConnect Mac source
echo -n "https://github.com/noland81/pexis-website/releases/download/v${NEW_TAG}/PexisConnect-v${VERSION}-mac-source.zip" | base64 -w0
```

### Buoc 2: Tim base64 cu trong index.html

```bash
rtk grep 'data-dl=' C:/code/Printolab/pexis-website/index.html
```

### Buoc 3: Thay the bang sed

```bash
# Thay tung base64 cu -> moi
sed -i 's|BASE64_CU_PEXISONE_WIN|BASE64_MOI_PEXISONE_WIN|g' C:/code/Printolab/pexis-website/index.html
sed -i 's|BASE64_CU_PEXISONE_MAC|BASE64_MOI_PEXISONE_MAC|g' C:/code/Printolab/pexis-website/index.html
sed -i 's|BASE64_CU_CONNECT_WIN|BASE64_MOI_CONNECT_WIN|g' C:/code/Printolab/pexis-website/index.html
sed -i 's|BASE64_CU_CONNECT_MAC|BASE64_MOI_CONNECT_MAC|g' C:/code/Printolab/pexis-website/index.html
```

**Meo**: Moi URL co base64 unique, nen `sed -i 's|...|...|g'` se thay dung tat ca cac cho co cung base64 (vi du PexisOne Desktop Win xuat hien 2 lan voi cung base64).

---

## Push changes

```bash
cd C:/code/Printolab/pexis-website
rtk git add -A
rtk git commit -m "Update downloads to v${NEW_TAG}"
rtk git push origin master
```

GitHub Pages tu deploy trong 1-2 phut.

### Verify

```bash
# Check release ton tai
/tmp/gh/bin/gh.exe release view v${NEW_TAG} --repo noland81/pexis-website

# Check website (mo browser)
# https://www.pexis.vn -> bam Tai ve -> confirm download dung file moi
```

---

## gh CLI

- **Path**: `/tmp/gh/bin/gh.exe`
- **Account**: `noland81` (da auth san bang keyring)
- **Repo**: `noland81/pexis-website` (public)

Neu `/tmp/gh/bin/gh.exe` khong ton tai, download lai:
```bash
curl -sL https://github.com/cli/cli/releases/download/v2.65.0/gh_2.65.0_windows_amd64.zip -o /tmp/gh.zip && unzip -o /tmp/gh.zip -d /tmp/gh
```

---

## Tao GitHub Release (day du)

```bash
cd C:/code/Printolab/pexis-website

# Xem tag hien tai
/tmp/gh/bin/gh.exe release list --repo noland81/pexis-website --limit 1

# Tao release voi tat ca files
/tmp/gh/bin/gh.exe release create v${NEW_TAG} \
  "path/to/PexisOne-Desktop-v${VER1}.exe" \
  "path/to/PexisOne-Desktop-v${VER1}-mac-source.zip" \
  "path/to/PexisConnect-v${VER2}.exe" \
  "path/to/PexisConnect-v${VER2}-mac-source.zip" \
  --title "Pexis Apps v${NEW_TAG} — Mo ta ngan" \
  --notes "## PexisOne Desktop v${VER1}
- Windows: PexisOne-Desktop-v${VER1}.exe
- macOS source: PexisOne-Desktop-v${VER1}-mac-source.zip

## PexisConnect v${VER2}
- Windows: PexisConnect-v${VER2}.exe
- macOS source: PexisConnect-v${VER2}-mac-source.zip

## Thay doi
- Mo ta thay doi..." \
  --repo noland81/pexis-website
```

---

## Canh bao quan trong

- **PHAI** push len branch `master` (khong phai `main`)
- Sau khi tao GitHub Release, **PHAI** update base64 links trong `index.html` + push
- Moi release tag tang MINOR: v1.0.0 -> v1.1.0 -> v1.2.0 (tang MAJOR khi breaking change)
- Ten file trong release **PHAI** dung prefix: `PexisOne-Desktop` va `PexisConnect` (auto-update cua app dua vao prefix nay)
- **KHONG** sua CNAME file tru khi doi domain
