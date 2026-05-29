# How to Deploy a New Android App to Play Store (Internal Testing)

This guide assumes you have already completed the one-time setup:
- Google Play Developer account (kiwicoltjc@gmail.com)
- `kiwicolt/android-workflows` repo exists on GitHub
- Keystore file exists at `/Users/jeremycorkin/ismartgate-release.jks`
- Google Cloud service account JSON already downloaded

After following this guide once, every future `git push` will automatically
build and deploy your app to Play Store Internal Testing.

---

## PART 1 — Prepare your Android project

### 1.1 — Add the deployment workflow file

In your project folder, create this file at exactly this path:
`.github/workflows/deploy.yml`

Paste this content exactly (change nothing):

```yaml
name: Deploy to Play Store

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  deploy:
    uses: kiwicolt/android-workflows/.github/workflows/android-deploy.yml@main
    secrets:
      KEYSTORE_BASE64: ${{ secrets.KEYSTORE_BASE64 }}
      KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
      KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
      KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
      PLAY_SERVICE_ACCOUNT_JSON: ${{ secrets.PLAY_SERVICE_ACCOUNT_JSON }}
```

### 1.2 — Add a .gitignore file

In your project's root folder, create a file called `.gitignore` with this content:

```
build/
app/build/
**/build/
.idea/
*.iml
.gradle/
local.properties
play-service-account.json
*.jks
.DS_Store
**/.DS_Store
```

This stops build files and credentials from being accidentally uploaded to GitHub.

### 1.3 — Configure app/build.gradle

Your `app/build.gradle` needs three things. Check it has all of these:

**① The Triple-T plugin at the top:**
```groovy
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'com.github.triplet.play'        // ← this line
}
```

**② A signing config that reads from environment variables:**
```groovy
signingConfigs {
    release {
        storeFile file(System.getenv("KEYSTORE_PATH") ?: "../ismartgate-release.jks")
        storePassword System.getenv("KEYSTORE_PASSWORD") ?: "garage123"
        keyAlias System.getenv("KEY_ALIAS") ?: "ismartgate"
        keyPassword System.getenv("KEY_PASSWORD") ?: "garage123"
    }
}

buildTypes {
    release {
        minifyEnabled false
        proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        signingConfig signingConfigs.release
    }
}
```

**③ A play block at the bottom (after the dependencies block):**
```groovy
play {
    serviceAccountCredentials = file("../play-service-account.json")
    track = "internal"
    defaultToAppBundles = true
}
```

### 1.4 — Configure the root build.gradle

Your root `build.gradle` (not the one inside `app/`) needs the Triple-T plugin declared:

```groovy
plugins {
    id 'com.android.application' version '8.2.2' apply false
    id 'org.jetbrains.kotlin.android' version '1.9.22' apply false
    id 'com.github.triplet.play' version '3.9.1' apply false   // ← this line
}
```

### 1.5 — Make sure gradle-wrapper.jar is present

Check that this file exists in your project:
`gradle/wrapper/gradle-wrapper.jar`

If it's missing, copy it from the garage app:
```bash
cp /Users/jeremycorkin/ismartgate-garage-app/gradle/wrapper/gradle-wrapper.jar \
   YOUR_PROJECT/gradle/wrapper/gradle-wrapper.jar
```

---

## PART 2 — Set up the app in Google Play Console

### 2.1 — Create the app

1. Go to https://play.google.com/console (sign in as kiwicoltjc@gmail.com)
2. Click **Create app**
3. Fill in: App name, Default language (English), App (not game), Free
4. Accept declarations → **Create app**

### 2.2 — Complete the minimum required setup

Play Console won't accept an upload until these sections are filled in.
Look for the **"Set up your app"** task list in the left sidebar and complete:

- **App access** → All functionality available without special access
- **Ads** → Does not contain ads
- **Content rating** → Complete questionnaire (answer No to everything)
- **Target audience** → 18+
- **News app** → Not a news app
- **Data safety** → Select No for data collection

---

## PART 3 — Do the first upload manually (required once per app)

Google Play requires the very first upload to be done manually through the website.
After that, GitHub does it automatically.

### 3.1 — Build the AAB on your Mac

Open Terminal and run:
```bash
cd /path/to/your/project
./gradlew bundleRelease
```

The file will be created at:
`app/build/outputs/bundle/release/app-release.aab`

### 3.2 — Upload through Play Console

1. In Play Console go to **Testing → Internal testing**
2. Click **Create new release**
3. Click **Upload** and select the `.aab` file
4. Add release notes (e.g. "Initial release")
5. Click **Save** → **Review release** → **Start rollout**

---

## PART 4 — Create the GitHub repo and add secrets

### 4.1 — Create the repo on GitHub

1. Go to github.com → click **+** (top right) → **New repository**
2. Name it whatever you like
3. Set to **Private**
4. Leave everything else unticked → **Create repository**

### 4.2 — Push your code

Run in Terminal (replace `YOUR-REPO-NAME` with your repo name):
```bash
cd /path/to/your/project
git init
git add .
git commit -m "Initial release"
git remote add origin https://github.com/kiwicolt/YOUR-REPO-NAME.git
git branch -M main
git push -u origin main
```

When it asks for a password, use your GitHub Personal Access Token (not your password).
Your token is at: github.com → Profile picture → Settings → Developer settings →
Personal access tokens → Tokens (classic). Make sure it has `repo` and `workflow` scopes.

### 4.3 — Add the 5 secrets

1. Go to your repo page on GitHub: `github.com/kiwicolt/YOUR-REPO-NAME`
2. Click the **Settings** tab (in the repo, not your account)
3. Left sidebar → scroll to **Security** → **Secrets and variables** → **Actions**
4. Click **New repository secret** for each of these:

| Secret name | How to get the value |
|---|---|
| `KEYSTORE_BASE64` | Run this in Terminal, then paste: `base64 -i /Users/jeremycorkin/ismartgate-release.jks \| pbcopy` |
| `KEYSTORE_PASSWORD` | Type: `garage123` |
| `KEY_ALIAS` | Type: `ismartgate` |
| `KEY_PASSWORD` | Type: `garage123` |
| `PLAY_SERVICE_ACCOUNT_JSON` | Open the Google Cloud service account `.json` file in TextEdit, ⌘A to select all, ⌘C to copy, then paste |

> **Where is the service account JSON file?**
> It was downloaded from Google Cloud Console when the garage app was set up.
> If you can't find it, create a new one:
> 1. Go to https://console.cloud.google.com
> 2. IAM & Admin → Service Accounts → click your existing `deploy-bot` account
> 3. Keys tab → Add Key → JSON → download

### 4.4 — Grant the service account access to the new app

Each new app needs to be added to the service account's permissions:

1. Go to https://play.google.com/console
2. **Users and permissions** → find the service account email → click the pencil icon
3. Under **App permissions** → click **Add app** → select your new app
4. Tick **Release apps to testing tracks** → Apply → Save

---

## PART 5 — Add yourself as a tester

1. In Play Console go to **Testing → Internal testing**
2. Click the **Testers** tab
3. Add your email to the testers list → Save
4. Copy the **opt-in URL** shown on the page
5. Open that URL on your Android phone in Chrome
6. Tap **Join the program** → tap the link to download from Play Store
7. Install the app

---

## PART 6 — You're done! How it works going forward

From now on, whenever you make changes to the app:

1. Make your code changes
2. Run in Terminal:
   ```bash
   git add .
   git commit -m "Description of what you changed"
   git push
   ```
3. GitHub automatically builds and uploads to Internal Testing (takes ~3-4 minutes)
4. Your phone gets the update automatically via Play Store

You can watch it happen at:
`github.com/kiwicolt/YOUR-REPO-NAME` → **Actions** tab

Green tick = success, Red X = something went wrong (click it to see the error).

---

## Troubleshooting

**"Version code is too low or has already been used"**
The version code in your `app/build.gradle` is too high. The CI automatically sets it
to `1000 + run_number` (e.g. 1005, 1006...). If you've manually uploaded builds with
higher version codes, update the offset in:
`kiwicolt/android-workflows/.github/workflows/android-deploy.yml`
Change `1000` to a number higher than your last uploaded version code.

**"Repository not found" when pushing**
Check that you created the repo on github.com first, and that the URL matches
your GitHub username (kiwicolt).

**Red X on the Actions tab**
Click the failed run → click the failed step → read the error message.
The most common causes are: wrong secrets, version code conflict, or Play Console
setup not completed.

**App doesn't appear on your phone**
Make sure you've joined the Internal Testing program via the opt-in URL (Part 5).
You only need to do this once per app.
