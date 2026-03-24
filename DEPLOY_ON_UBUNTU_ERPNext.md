# FBR-Pakistan Deployment Guide (Ubuntu + ERPNext)

This guide is based on the current repository contents and is intended for ERPNext/Frappe **v15** environments.

## 1) Confirm supported versions

From the repository docs:
- **Frappe:** v15.x
- **ERPNext:** v15.x
- **Python:** >= 3.10
- **wkhtmltopdf:** patched build `0.12.6.1-2` (Jammy package shown)

## 2) Prerequisites on Ubuntu

> Recommended baseline: Ubuntu 22.04 LTS (Jammy), because the documented wkhtmltopdf package is the Jammy build.

- A working bench with ERPNext v15 site already running.
- A site such as `site1.local` (replace with your real site in all commands).

Install wkhtmltopdf runtime libraries:

```bash
sudo apt update
sudo apt install -y fontconfig xfonts-75dpi xfonts-base \
  libxrender1 libxext6 libfontconfig1 libfreetype6 libjpeg-turbo8
```

Install patched wkhtmltopdf:

```bash
wget -O wkhtmltox.deb \
https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_amd64.deb
sudo apt install -y ./wkhtmltox.deb
wkhtmltopdf --version
```

Configure bench to use it:

```bash
bench set-config -g wkhtmltopdf_path "$(which wkhtmltopdf)"
bench restart
```

## 3) Choose your deployment mode

This repo contains two practical approaches:

### Mode A (recommended): Install as Frappe app
Use this if you have/prepare a proper `fbr_integration` app repository.

```bash
cd ~/frappe-bench
bench get-app https://github.com/ERPNEXT-PAKISTAN/FBR_Integration.git --branch main
bench --site site1.local install-app fbr_integration
bench migrate
bench restart
```

### Mode B: Manual setup using this repo artifacts
Use this if you are applying scripts/doctype data manually from this repository.

1. Create/import master doctypes from `Doctype/`.
2. Import seed master data from `FBR Tax Type Excel Data/` via Data Import.
3. Add custom fields for Sales Invoice / Sales Invoice Item / Item / Customer as documented.
4. Install server/client scripts from `FBR Integration Apps/` and/or `FBR Integration WebHook/`.
5. Configure chart of accounts and item tax templates.

## 4) Core configuration after install

Open **FBR Invoice Settings** and fill:
- Enabled = checked
- Integration Type = Sandbox or Production
- API URL and Security Token for the selected mode

## 5) Verification flow

1. Create Sales Invoice with required FBR master values on each item.
2. Submit invoice (`docstatus = 1`).
3. Click **Send to FBR** (or enable on-submit hook if you choose auto-submit mode).
4. Confirm fields are filled:
   - `custom_fbr_invoice_no`
   - `custom_fbr_invoice_status`
   - `custom_fbr_digital_invoice_response`

## 6) Optional automation mode (server-side)

The hook file shows two modes:
- **Button mode (default):** `doc_events = {}`
- **Auto on submit:** uncomment Sales Invoice `on_submit` hook to call `after_submit_invoice`.

## 7) Common pitfalls from repo behavior

- If integration type is not Sandbox/Production, submission throws validation error.
- API errors are persisted back to Sales Invoice response fields.
- Repo docs mention UTF-8 cleanup if you see `UnicodeDecodeError` from copied script files.

## 8) Quick production readiness checklist

- [ ] ERPNext/Frappe both on v15.x
- [ ] Python >= 3.10
- [ ] wkhtmltopdf patched build installed and configured in bench
- [ ] FBR Invoice Settings saved with production token/url
- [ ] Mandatory custom fields + masters loaded
- [ ] End-to-end test invoice accepted by FBR

