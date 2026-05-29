# Building a New Android App with Claude — Complete Guide

This guide tells you exactly what to say to Claude and what to do yourself
at each step to go from an idea to an automatically-deploying Android app.

Your setup (already done once):
- Google Play Developer account: kiwicoltjc@gmail.com
- Keystore file: /Users/jeremycorkin/ismartgate-release.jks (password: garage123)
- GitHub account: kiwicolt
- Reusable deploy workflow: github.com/kiwicolt/android-workflows

---

## STEP 1 — Tell Claude what app you want to build

Start a new Claude session and say something like:

> "I want to build an Android app that [describe what it does].
> Please create the full project including automatic deployment to Google Play
> Store Internal Testing via GitHub Actions. My GitHub username is kiwicolt,
> my keystore is at /Users/jeremycorkin/ismartgate-release.jks with password
> garage123, alias ismartgate, key password garage123. Use the reusable deploy
> workflow at kiwicolt/android-workflows."

Claude will create all the code and files for you.

---

## STEP 2 — You create the app in Google Play Console (~10 minutes)

You must do this yourself because it requires your Google account.

1. Go to https://play.google.com/console (sign in as kiwicoltjc@gmail.com)
2. Click **Create app**
3. Fill in: App name, Default language (English), App, Free
4. Accept declarations → **Create app**
5. Complete the **"Set up your app"** checklist in the left sidebar:
   - **App access** → All functionality available without special access
   - **Ads** → Does not contain ads
   - **Content rating** → Complete questionnaire (say No to everything)
   - **Target audience** → 18+
   - **News app** → Not a news app
   - **Data safety** → No for data collection

---

## STEP 3 — Tell Claude to build the AAB for the first upload

Tell Claude:

> "Please build the release AAB for the first manual upload"

Claude will run `./gradlew bundleRelease` and tell you where the file is.
It will be at: `[your project folder]/app/build/outputs/bundle/release/app-release.aab`

---

## STEP 4 — You upload the first build to Play Store manually (required once)

Google requires the very first upload to be done through the website.
After this, everything is automatic.

1. In Play Console go to **Testing → Internal testing**
2. Click **Create new release**
3. Click **Upload** and select the `app-release.aab` file Claude built
4. Type anything in release notes (e.g. "Initial release")
5. Click **Save** → **Review release** → **Start rollout**

If Play Store complains about targetSdk being too low, tell Claude:
> "Play Store rejected the upload saying targetSdk is too low, please fix it"

---

## STEP 5 — You create the GitHub repo

1. Go to github.com → click **+** (top right) → **New repository**
2. Name it (e.g. `my-app-name`)
3. Set to **Private**
4. Leave everything else unticked → **Create repository**

---

## STEP 6 — Add the 5 secrets to GitHub

1. Go to your new repo: `github.com/kiwicolt/YOUR-REPO-NAME`
2. Click **Settings** tab (in the repo toolbar, not your account)
3. Left sidebar → scroll down to **Security** → **Secrets and variables** → **Actions**
4. Click **New repository secret** for each of these 5:

**Secret 1: KEYSTORE_BASE64**
- Ask Claude: "Please run the base64 keystore command and copy it to my clipboard"
- Claude will copy it → paste into the Secret box

**Secret 2: KEYSTORE_PASSWORD**
- Value: `garage123`

**Secret 3: KEY_ALIAS**
- Value: `ismartgate`

**Secret 4: KEY_PASSWORD**
- Value: `garage123`

**Secret 5: PLAY_SERVICE_ACCOUNT_JSON**
- Find the Google Cloud service account JSON file on your Mac
  (Claude knows where it is from previous sessions, or ask:
  "Where is my Play Store service account JSON file?")
- Open it in TextEdit → ⌘A → ⌘C → paste into the Secret box
- If you can't find it, tell Claude: "I can't find my Play Store service
  account JSON file" and Claude will guide you to create a new one

---

## STEP 7 — Grant service account access to the new app

Each new app needs to be added to the service account's Play Console permissions.

1. Go to https://play.google.com/console
2. Click **Users and permissions** in the left sidebar
3. Find the row with the service account email (ends in `gserviceaccount.com`)
4. Click the pencil/edit icon on that row
5. Click **Add app** → select your new app
6. Tick **Release apps to testing tracks** → **Apply** → **Save**

---

## STEP 8 — Tell Claude to push the code to GitHub

Tell Claude:
> "Please push the code to github.com/kiwicolt/YOUR-REPO-NAME"

Claude will prepare the git commands. When Terminal asks for a password,
use your GitHub Personal Access Token (not your regular password).

**Where is your token?**
github.com → your profile picture → Settings → scroll to bottom →
Developer settings → Personal access tokens → Tokens (classic)

If your token has expired or you don't have one, click **Generate new token (classic)**,
give it a name, tick `repo` and `workflow`, then **Generate token**.
Copy the token (starts with `ghp_`) — use this as your password.

---

## STEP 9 — Add yourself as a tester (first time for each app)

1. In Play Console go to **Testing → Internal testing**
2. Click the **Testers** tab
3. Add your email to the testers list → Save
4. Copy the **opt-in URL** shown on that page
5. Open that URL on your Android phone in Chrome (signed in with your Google account)
6. Tap **Join the program**
7. Tap the link to **Download it on Google Play**
8. Install the app normally

---

## STEP 10 — Watch the first automated deploy

1. Go to `github.com/kiwicolt/YOUR-REPO-NAME` → **Actions** tab
2. You should see a workflow run in progress (yellow spinning circle)
3. Wait 3-5 minutes
4. Green tick = your app is deploying to Internal Testing ✅
5. Red X = something went wrong → click it and share the error with Claude

---

## Going forward — how to push updates

Whenever you want to update your app:

1. Tell Claude what you want to change
2. Claude makes the changes
3. Tell Claude: "Please commit and push the changes"
4. Claude runs the git commands (you may need to type your GitHub token as password)
5. GitHub automatically builds and uploads the new version (~4 minutes)
6. Your phone gets the update via Play Store

That's it. You never touch Play Console again after the initial setup.

---

## Quick reference — things only you can do

These always require your Google/GitHub account and can't be done by Claude:

| Task | Where |
|---|---|
| Create app in Play Console | play.google.com/console |
| Upload first AAB manually | Play Console → Testing → Internal testing |
| Complete Play Console setup checklist | Play Console → "Set up your app" |
| Add service account to new app | Play Console → Users and permissions |
| Create GitHub repo | github.com → + → New repository |
| Add secrets to GitHub repo | GitHub repo → Settings → Secrets → Actions |
| Join testing program on phone | Opt-in URL from Play Console → open on phone |

Everything else — writing code, building, configuring, deploying — Claude handles.

---

## If something goes wrong

Just tell Claude what the error message says. For example:
- "The GitHub action failed with this error: [paste error]"
- "Play Store rejected the upload saying: [paste error]"
- "The app crashes when I [describe what you did]"

Claude can read the errors and fix them.
