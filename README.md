# Student Extension — Deploy Kit

Self-host the **Student Extension** server for your school: GCSE Computer Science homework where teachers author tasks, students solve them in VS Code, and tests mark automatically. This kit is the public deployment half of the project (ADR-0012): a pinned-image Compose stack and a plain-language README. The extension comes from the VS Code Marketplace — this repo contains **no source code**.

**You are the data controller.** The stack runs on your machine, holds your students' data, and talks to nobody but your Google Workspace for sign-in. Nothing reaches the project's authors (details in [Privacy](#privacy)).

## What you need

- A server (a small VPS is plenty) reachable from school and student homes, with **Docker Engine + the compose plugin** installed (`docker compose version` must answer).
- **Google Workspace admin rights** (to create an OAuth client) — or a friendly IT colleague who has them.
- The VS Code extension on every teacher's and student's machine (Marketplace listing linked from the project README).

## 1 · Create the Google OAuth client

1. In the [Google Cloud Console](https://console.cloud.google.com/) create a project (e.g. “Student Extension”), then **APIs & Services → Credentials → Create credentials → OAuth client ID**.
2. Application type: **Web application**.
3. **Authorized redirect URI:** add `http://127.0.0.1/callback`. This is the sign-in loopback (RFC 8252) — the port in the browser bar varies per sign-in but Google matches loopback URIs on the path. If sign-in ever reports a `redirect_uri_mismatch`, add the exact URI from that error as a second entry.
4. Note the **Client ID** and **Client secret**.

## 2 · Configure and start

```sh
git clone https://github.com/veritymedia/student-extension-deploy-kit.git
cd student-extension-deploy-kit
cp .env.example .env
# fill in the three values (your favourite editor)
nano .env
docker compose up -d
docker compose ps
```

The server listens on port **8090**; its admin console is `http://<your-server>:8090/_/`. Data lands in the named volume `pb_data` and survives restarts and updates.

**Exposure:** port 8090 is published on all interfaces so the stack works over Tailscale or a school VPN. If this box has a public interface instead, put a firewall (or an HTTPS reverse proxy) in front — student data should not travel plain-HTTP across the open internet.

## 3 · First login = Super-admin

On a fresh server, **the very first sign-in becomes the Super-admin** — the one account that can designate Teachers. Do it yourself:

1. Install the Student Extension in VS Code on your machine.
2. Set the **Server URL** setting (`se.serverUrl`) to your server, e.g. `https://se.your-school.example`.
3. **Student Extension: Sign in** → the browser opens → pick your school Google account.

Only accounts whose email ends in your `SE_WORKSPACE_DOMAIN` can sign in at all. Everyone after you starts as a plain Student (see [Privacy](#privacy)).

## 4 · Teachers, Classes, Join Codes

- Each colleague signs in once the same way; you promote them in the extension's **Accounts** view (right-click → **Toggle Teacher**).
- Teachers create **Classes** in the **Classes** view — each class gets a six-character **Join Code**, regenerated any time one leaks.
- Students install the extension, set the same Server URL, sign in, and enter the Join Code. The in-extension walkthrough talks them through it.

## 5 · Update

The stack runs **pinned images** — updates never arrive by surprise:

1. Check the project's GitHub releases for a new `vX.Y.Z` tag (images are published to `ghcr.io` per release).2. Edit `docker-compose.yml` — bump the two `image:` tags (`student-extension-server` and `student-extension-runner`) to the same version.
3. `docker compose pull && docker compose up -d`. Data survives (`pb_data` volume).

## Privacy

Plain answers, because this holds children's work (the project's ADR-0014; the DPIA, if you need one, belongs to the school).

**What is stored** — a student's name and email (from your Google Workspace at sign-in), the code they submit for assignments, and their marks. Nothing else: no photos, no age, no location, no tracking, no analytics.

**Where** — only on **your** server above. The school is the data controller; the software's authors cannot see any of it. The only third parties ever involved are Google (sign-in) and, on student machines, GitHub's release CDN for the pinned Python download.

**Who can see it** — the student's teachers and the Super-admin account. No one else; there are no recordings or screenshots, and no telemetry anywhere in the software.

**Retention** — the recommended posture is the **current academic year plus one**, in line with school records practice. There is no automatic purge: deletion is deliberate (next paragraph).

**Deleting data** — the Super-admin's **Erase** action permanently deletes one student's account and all their data (work, submissions, marks). Removing a student from a class only hides that class from them. Deleting the server itself (the `pb_data` volume) removes everything irrecoverably.

**Backups** — yours to run; the software builds none. Either use PocketBase's built-in backups (admin console → Settings → Backups — a zip of everything), or simply stop the stack and copy the `pb_data` volume. Erased data ages out of your backups on your own schedule.

**Logs** — container logs exist for security and error diagnosis only. They never contain request bodies or student code. They rotate by staying bounded — three files of 10 MB per container (roughly the last month at typical school traffic); tune the `logging` section in the compose file to taste.

## License

No open-source license is declared yet — the kit and the containers it pins are © Verity Media, all rights reserved. Schools may deploy and run this kit for their own teaching. The license decision lives with the main project and will land here when made.

## Support

The in-extension **Get Started** walkthrough covers day-to-day use for students and teachers. For deployment help with this kit, contact the department that shared it with you — the project itself offers no public support channel.
