# Building Mnemosyne.app on an Apple Silicon Mac

These instructions build a native Apple Silicon version of Mnemosyne on your
own Mac. You end up with a normal `Mnemosyne.app` in your Applications folder
that no longer needs Rosetta, so the "Support Ending for Intel-based Apps"
warning goes away.

You do not need to know how to program. You will copy commands into the
Terminal app and press Return. Each step says what you should see.

**What you need**

- A Mac with an Apple Silicon chip (M1 or newer) and macOS 14.4 or newer.
  Check under Apple menu → About This Mac: "Chip" should say Apple M…
- About 4 GB of free disk space.
- An internet connection.
- About 30–60 minutes. Most of it is waiting for downloads.

**How to use Terminal**

Open Terminal: press Cmd+Space, type `Terminal`, press Return.

For each step, copy the whole grey box, paste it into Terminal (Cmd+V), and
press Return. Wait until Terminal shows its prompt again (a line ending in
`%`) before you go to the next step.

When Terminal asks for a password, it means your Mac login password. Nothing
appears on the screen while you type it. That is normal. Type it and press
Return.

---

## Step 1. Back up your cards

Your cards, learning history and media are in the folder
`~/Library/Mnemosyne`. The new version may update that database the first
time it opens it, so make a copy first.

1. Quit Mnemosyne (Mnemosyne menu → Quit).
2. Paste this into Terminal:

```
ditto ~/Library/Mnemosyne ~/Desktop/Mnemosyne-backup-$(date +%Y-%m-%d)
```

You should now have a folder named something like `Mnemosyne-backup-2026-11-04`
on your Desktop. Keep it until you are sure the new version works well.
Copying it to a USB stick or an external drive as well is a good idea.

## Step 2. Install Homebrew

Homebrew is a free tool that installs developer software on a Mac. Its
installer also installs Apple's "Command Line Tools", which the build needs.

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

- It asks for your Mac password, then says "Press RETURN to continue".
  Press Return.
- If a window appears asking to install the Command Line Tools, click
  Install and accept the license.
- This can take 10–30 minutes.
- At the end it prints "Next steps" with some commands. **You can ignore
  them.** The commands below use Homebrew's full path, so they work anyway.

If Homebrew is already installed, it just says so. That is fine.

## Step 3. Install Python

```
/opt/homebrew/bin/brew install python@3.15
```

Check that it worked:

```
/opt/homebrew/bin/python3.15 --version
```

This should print `Python 3.15.` followed by a number.

## Step 4. Download the Mnemosyne source code

```
git clone https://github.com/MichaelNedzelsky/mnemosyne.git ~/mnemosyne-src
```

This creates a folder `mnemosyne-src` in your home folder. It ends with a
line like `Resolving deltas: 100%`.

## Step 5. Install the libraries Mnemosyne needs

Paste all four lines at once:

```
cd ~/mnemosyne-src
/opt/homebrew/bin/python3.15 -m venv venv
venv/bin/pip install --upgrade pip
venv/bin/pip install PyQt6 PyQt6-WebEngine matplotlib cheroot webob pillow gTTS argon2-cffi legacy-cgi pyinstaller
```

This downloads about 400 MB and takes a few minutes. It should end with a
line starting `Successfully installed`.

The libraries go into `~/mnemosyne-src/venv`. They don't change anything
else on your Mac.

## Step 6. Build the app

```
cd ~/mnemosyne-src
source venv/bin/activate
make macos && echo "*** BUILD OK ***"
deactivate
```

This takes about a minute and prints many lines. **Lines like these are
normal and can be ignored:**

```
ERROR: Hidden import 'argon2-cffi' not found
ERROR: Hidden import 'google' not found
dist/Mnemosyne.app: replacing existing signature
```

The build worked if one of the last lines is:

```
*** BUILD OK ***
```

If you don't see `*** BUILD OK ***`, the build failed. See
[If something goes wrong](#if-something-goes-wrong).

## Step 7. Put the app in Applications

1. Keep the old version for now. In Finder, open Applications, find
   `Mnemosyne`, and rename it to `Mnemosyne Intel` (click it, press Return,
   type the new name, press Return).
2. Show the new app in Finder:

   ```
   open -R ~/mnemosyne-src/dist/Mnemosyne.app
   ```

3. A Finder window opens with the new app already selected. Drag it onto
   Applications in the Finder sidebar. (Next to it there is also a plain
   folder named `Mnemosyne`. That is a leftover from the build: leave it.)

## Step 8. Start it and check

Start Mnemosyne from Applications or Launchpad as usual. It opens without a
security warning, because you built it on your own Mac.

Check:

- Your cards are there. Look at the card count at the bottom of the window,
  and open Browse cards.
- It runs natively. Open Activity Monitor (Cmd+Space, type
  `Activity Monitor`). In the CPU tab, find Mnemosyne. The "Kind" column
  should say **Apple**, not Intel.

Do a normal review session. When you are happy with it, you can delete
`Mnemosyne Intel` from Applications. Keep the backup from Step 1 for a while
longer.

Don't run the old and the new version at the same time.

---

## Updating later

When there is a new version of the code, run:

```
cd ~/mnemosyne-src
git pull
rm -rf build dist
source venv/bin/activate
make macos && echo "*** BUILD OK ***"
deactivate
```

Then quit Mnemosyne and repeat Step 7, replacing the app in Applications.
Before updating, make a fresh backup as in Step 1.

## If something goes wrong

**`zsh: command not found: brew`**
Type the commands exactly as written above, starting with `/opt/homebrew/bin/`.

**`xcrun: error: invalid active developer path`**
The Command Line Tools are missing. Run `xcode-select --install`, click
Install, wait until it finishes, then repeat the step that failed.

**`error: externally-managed-environment`**
You ran `pip` without the `venv/bin/` in front of it. Repeat Step 5 exactly
as written.

**The build doesn't end with `*** BUILD OK ***`**
Run the build again and save everything it prints to a file:

```
cd ~/mnemosyne-src
source venv/bin/activate
make macos > ~/Desktop/mnemosyne-build.log 2>&1; deactivate
```

Then send the file `mnemosyne-build.log` from your Desktop to the
[Mnemosyne users group](https://groups.google.com/g/mnemosyne-proj-users)
or to whoever is helping you.

**Starting over**
Nothing here touches your cards. To start again from scratch, delete the
build folder and repeat from Step 4:

```
rm -rf ~/mnemosyne-src
```

**Getting your old data back**
Quit Mnemosyne. In Finder, use Go → Go to Folder, type `~/Library`, and
rename the `Mnemosyne` folder to `Mnemosyne-new`. Then copy your backup
folder from the Desktop into `~/Library` and rename it to `Mnemosyne`.

## Good to know

- The app you build only works on Macs with the same or a newer version of
  macOS. It also works on your Mac after future macOS updates.
- The Google Translate button is not available in this build, because its
  add-on is not installed. Everything else, including Google text-to-speech,
  is included.
- These steps were tested on 4 November 2026 on macOS 27.2 with Python
  3.15.2, PyQt6 6.12.0 and PyInstaller 6.22.3. Python 3.14
  (`brew install python@3.14`, then `python3.14` instead of `python3.15`)
  works as well.
