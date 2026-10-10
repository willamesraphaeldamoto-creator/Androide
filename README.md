# Android Emulator Web

Run an Android emulator in a VPS and access it through a browser with HTTP Basic Authentication and automatic HTTPS via Caddy.

## Important limits

- GitHub Actions is for validation only. Hosted runners are temporary and are **not** a 24/7 server.
- A VPS with Docker, KVM support, and enough RAM/CPU is required. Android emulators commonly need hardware virtualization; confirm your VPS provider supports nested virtualization/KVM.
- This setup is a starting point. Test the chosen emulator image and browser UI on your VPS before relying on it.
- The Android container is not published directly to the internet. Only Caddy publishes ports 80/443.

## 1. Prepare a VPS and domain

1. Create an Ubuntu VPS with Docker Engine and the Docker Compose plugin installed.
2. Point a DNS `A` record for your domain (for example `android.example.com`) to the VPS public IP.
3. Allow inbound TCP ports 80 and 443 in the VPS firewall. Do not open ports 5555 or 6080 publicly.
4. Clone this repository on the VPS.

## 2. Set a private login

Copy the example environment file:

```bash
cp .env.example .env
```

Create a strong, unique password and generate a bcrypt hash:

```bash
docker run --rm caddy:2 caddy hash-password --plaintext 'CHOOSE_A_NEW_STRONG_PASSWORD'
```

Edit `.env` and set:
- `DOMAIN` to your real DNS name
- `LOGIN_USER` to your chosen username
- `LOGIN_PASSWORD_HASH` to the full bcrypt hash returned above

Keep `.env` private. Never commit it or paste passwords into issues or chat. The Wi-Fi password is not needed for this server login; do not reuse your Wi-Fi password.

## 3. Start it

From the project directory on the VPS:

```bash
docker compose up -d
docker compose logs -f
```

Open `https://YOUR_DOMAIN` and enter the login username and password. Caddy obtains and renews a TLS certificate automatically when DNS and ports are configured correctly.

## 4. Updates and maintenance

```bash
docker compose pull
docker compose up -d
docker compose ps
```

Back up anything important inside the emulator separately. Restart policy helps containers restart after a reboot, but it cannot prevent VPS outages or provider maintenance.

## Security notes

- Use a strong, unique password and HTTPS.
- Keep Ubuntu, Docker, and container images updated.
- Do not expose Android ADB (`5555`) or the emulator UI (`6080`) directly to the public internet.
- HTTP Basic Authentication protects the web interface, but it is not a substitute for patching and firewall rules.
- The workflow under `.github/workflows/` only validates configuration. It does not host or run the emulator.

## Free temporary browser session (no VPS)

This repository also has a manual workflow at **Actions → Temporary Android emulator in browser → Run workflow**. It attempts to run the emulator on a GitHub-hosted runner and publish it through a temporary Cloudflare Quick Tunnel. It is not a permanent host: the link expires when the workflow ends, and the job is limited to about 5 hours 50 minutes. The runner may not expose `/dev/kvm`; if so, the workflow will stop with an error instead of pretending the emulator started.

### One-time setup

1. Open **Settings → Secrets and variables → Actions** in this repository.
2. Choose **New repository secret**.
3. Name it exactly `EMULATOR_WEB_PASSWORD`.
4. Choose a new, strong password of at least 14 characters. Do not reuse your Wi-Fi, email, or GitHub password.
5. Open **Actions → Temporary Android emulator in browser → Run workflow** and start it.
6. Open the running workflow and its logs. If startup succeeds, the log prints the temporary `https://…trycloudflare.com` URL. Sign in with username `emulator` and the password you saved as the secret.

The tunnel URL is public, but the browser UI is protected by HTTP Basic Authentication. Keep the password secret. Do not put it in workflow YAML, commit it, or paste it into issues. Anyone who can write to this repository may be able to start workflows, so only grant write access to people you trust.

### What free mode does and does not do

- It can provide a temporary browser session if GitHub's runner supports the emulator's required virtualization and the image starts successfully.
- It is **not 24/7**, does not retain a stable URL, and the emulator's data disappears when the runner is destroyed.
- GitHub Actions is designed for jobs, not as a permanent remote desktop. Hosted jobs have execution limits; see [GitHub Actions limits](https://docs.github.com/en/actions/reference/limits).
- Public repositories can use standard GitHub-hosted runners free of charge under GitHub's current policy, but policies and limits can change. See [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions).
- For a stable 24/7 link and persistent data, you need a VPS or another always-on host.
