# sky-releases

Public release artifacts and Sparkle auto-update feed for [Sky](https://github.com/SACNY/sky).

---

## Repository Structure

```
sky-releases/
├── appcast.xml          # Sparkle auto-update feed (consumed by the app)
├── README.md
└── releases/
    └── v0.1.0/
        ├── Sky-0.1.0.dmg
        └── release-notes.md
```

---

## How It Works

Sky uses [Sparkle](https://sparkle-project.org/) for automatic updates. On launch (and periodically thereafter), the app fetches `appcast.xml` from this repository's `main` branch via GitHub's raw content URL:

```
https://raw.githubusercontent.com/SACNY/sky-releases/main/appcast.xml
```

Sparkle parses the feed, compares the bundled `CFBundleVersion` against `<sparkle:version>` in the feed, and prompts the user to update if a newer version is available.

---

## Releasing a New Version

### 1. Build & sign the DMG

Build an archive in Xcode, export it as a notarized `.dmg`, and name it:

```
Sky-<version>.dmg
```

### 2. Generate the EdDSA signature

Use the `sign_update` tool that ships with Sparkle:

```
./bin/sign_update Sky-<version>.dmg
```

Copy the printed `sparkle:edSignature` value — you'll need it in step 4.

> Your private EdDSA key must **never** be committed to this repository.
> Store it in your keychain or a secrets manager.

### 3. Add release artifacts

Create a folder for the new version and drop in the DMG and release notes:

```
releases/
└── v<version>/
    ├── Sky-<version>.dmg
    └── release-notes.md
```

### 4. Update `appcast.xml`

Add a new `<item>` block **above** all existing items (newest first):

```xml
<item>
  <title>Sky <version></title>
  <link>https://github.com/SACNY/sky-releases/releases/tag/v<version></link>
  <sparkle:version><CFBundleVersion></sparkle:version>
  <sparkle:shortVersionString><version></sparkle:shortVersionString>
  <description>
    <![CDATA[
      <!-- paste release-notes.md content as HTML here -->
    ]]>
  </description>
  <pubDate><!-- RFC 2822 date, e.g. Mon, 16 Jun 2025 00:00:00 +0000 --></pubDate>
  <enclosure
      url="https://github.com/SACNY/sky-releases/releases/download/v<version>/Sky-<version>.dmg"
      sparkle:edSignature="<signature from step 2>"
      length="<byte length of the DMG>"
      type="application/octet-stream"
  />
  <sparkle:minimumSystemVersion>13.0</sparkle:minimumSystemVersion>
</item>
```

### 5. Create a GitHub Release

- Tag: `v<version>`
- Upload `Sky-<version>.dmg` as a release asset (this is what the `enclosure` URL points to)
- Paste the contents of `release-notes.md` as the GitHub Release description

### 6. Commit & push

```
git add appcast.xml releases/v<version>/
git commit -m "Release v<version>"
git push origin main
```

Once pushed, live users running an older version will be notified of the update automatically.

---

## Releases

| Version | Date | Notes |
|---------|------|-------|
| [0.1.0](releases/v0.1.0/release-notes.md) | 2025-06-16 | Initial release |

---

## Security

- Update signatures use **EdDSA (Ed25519)** via Sparkle 2.
- The **private signing key** is never stored in this repository.
- All DMGs are **notarized** with Apple before release.