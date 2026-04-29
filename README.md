# Apple Unity Plugins - Unofficial Prebuilt Packages

Automatically built and published packages for [apple/unityplugins](https://github.com/apple/unityplugins).

Apple's repo includes a Python build script but does not ship prebuilt binaries. This repo uses GitHub Actions to watch for new upstream releases, build the native libraries on macOS with Xcode, and publish the results in two forms:

- **GitHub Release assets** - `.tgz` tarballs ready for local install
- **UPM Git branches** - one branch per plugin for direct `git` URL install in Unity

---

## Available Plugins

| Plugin | UPM Branch | Description |
|---|---|---|
| `Apple.Core` | `upm/Apple.Core` | Required dependency for all other plugins |
| `Apple.Accessibility` | `upm/Apple.Accessibility` | VoiceOver and assistive technology support |
| `Apple.CoreHaptics` | `upm/Apple.CoreHaptics` | Haptic patterns and UIFeedbackGenerator |
| `Apple.GameController` | `upm/Apple.GameController` | GameController framework |
| `Apple.GameKit` | `upm/Apple.GameKit` | Game Center, leaderboards, matchmaking |
| `Apple.PHASE` | `upm/Apple.PHASE` | Spatial audio via the PHASE framework |

---

## Installing in Unity

> **Note:** `Apple.Core` must be installed first - all other plugins depend on it.

### Option A - Direct Git URL (UPM)

No download needed. Unity's Package Manager installs straight from this repo's branches.

1. Open **Window → Package Manager** in the Unity Editor.
2. Click the **＋** button → **Add package from git URL…**
3. Paste the URL for each plugin you need:

```
https://github.com/aymisml/apple-unityplugins-prebuilt.git#upm/Apple.Core
https://github.com/aymisml/apple-unityplugins-prebuilt.git#upm/Apple.Accessibility
https://github.com/aymisml/apple-unityplugins-prebuilt.git#upm/Apple.CoreHaptics
https://github.com/aymisml/apple-unityplugins-prebuilt.git#upm/Apple.GameController
https://github.com/aymisml/apple-unityplugins-prebuilt.git#upm/Apple.GameKit
https://github.com/aymisml/apple-unityplugins-prebuilt.git#upm/Apple.PHASE
```

To target a specific version, append the release tag after `#`:
```
https://github.com/aymisml/apple-unityplugins-prebuilt.git#upm/Apple.Core@v2.0.0
```
### Option B - Tarball Download

1. Go to the [Releases](../../releases) page and download the `.tgz` files for the plugins you need.
2. In Unity: **Window → Package Manager → ＋ → Add package from tarball…**
3. Select the downloaded `.tgz` file.
4. Repeat for each plugin, starting with `Apple.Core`.

---

## How the Automation Works

```
┌─────────────────────────────────┐
│  check-upstream.yml             │
│  Runs every 6 hours             │
│                                 │
│  1. Fetch latest tag from       │
│     apple/unityplugins          │
│  2. Compare with latest tag     │
│     in this repo's Releases     │
│  3. If newer → trigger          │
│     build-release.yml           │
└────────────────┬────────────────┘
                 │ workflow_dispatch (with upstream_tag input)
                 ▼
┌─────────────────────────────────┐
│  build-release.yml              │
│  Runs on macos-latest           │
│                                 │
│  1. Clone apple/unityplugins    │
│     at the given tag            │
│  2. Run python3 build.py        │
│     (Xcode + npm)               │
│  3. Create GitHub Release       │
│     → upload all .tgz files     │
│  4. For each plugin:            │
│     • Extract tarball           │
│     • Force-push to             │
│       upm/<PluginName> branch   │
└─────────────────────────────────┘
```

### Workflow files

| File | Purpose |
|---|---|
| `.github/workflows/check-upstream.yml` | Polls apple/unityplugins every 6 hours; triggers build if a new release is found |
| `.github/workflows/build-release.yml` | Clones, builds, releases tarballs, and pushes UPM branches |

---

## Setting Up Your Own Copy

### 1. Create the repository

Fork this repo **or** create a new public repo and copy these files into it.

### 2. Required permissions for GitHub Actions

The default `GITHUB_TOKEN` is used for all operations. You must grant it write access:

1. Go to **Settings → Actions → General** in your repo.
2. Under **Workflow permissions**, select **Read and write permissions**.
3. Check **Allow GitHub Actions to create and approve pull requests** (optional but useful).
4. Click **Save**.

No additional secrets are required - `secrets.GITHUB_TOKEN` is provided automatically.

### 3. Update the README URLs

Replace all occurrences of `aymisml/apple-unityplugins-prebuilt` in this file with your actual GitHub path.

### 4. Trigger your first build manually

The automated check only fires when a new upstream release appears. To build immediately:

1. Go to **Actions → Build and Release**.
2. Click **Run workflow**.
3. Enter the upstream tag you want to build (e.g. `v2.0.0`).
4. Click **Run workflow**.

After it completes, check the **Releases** page for tarballs and verify the `upm/*` branches exist.

### 5. (Optional) Change the check schedule

Edit `.github/workflows/check-upstream.yml` and modify the `cron` expression:

```yaml
schedule:
  - cron: "0 */6 * * *"   # every 6 hours - change as needed
```

---

## Troubleshooting

**Build fails at the `build.py` step**

The upstream build script requires Xcode, Python 3, npm, and a compatible macOS environment. GitHub's `macos-latest` runner includes these, but the Xcode version may not always match what Apple's latest plug-ins expect. If a build fails, check the workflow logs and try pinning the Xcode version in `build-release.yml`:

```yaml
- uses: maxim-lobanov/setup-xcode@v1
  with:
    xcode-version: "16.0"   # pin to a specific version
```

**`--skip-codesign` flag not recognised**

Apple occasionally renames flags. If the build fails with an unknown argument error, check the current flags:

```
python3 build.py --help
```

Then update the `build-release.yml` `run` step accordingly.

**UPM branch install fails in Unity**

Ensure the branch name is typed exactly as listed (case-sensitive). Also confirm the branch exists in the repo under **Code → Switch branches**.

**Workflow has no permission to push**

Re-check step 2 of the setup above. The token needs **write** permission to create releases and push branches.

---

## Relationship to apple/unityplugins

This repo contains no original Apple source code. The build workflows clone [apple/unityplugins](https://github.com/apple/unityplugins) at a specific tag, build the native libraries, and publish the resulting compiled artifacts. All intellectual property belongs to Apple. Review Apple's [LICENSE](https://github.com/apple/unityplugins/blob/main/LICENSE.txt) before use.

---

## Contributing

Pull requests for workflow improvements are welcome. Please open an issue first if you're making significant changes to the build logic.
