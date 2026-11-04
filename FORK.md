# About this fork

This is a fork of [mnemosyne-proj/mnemosyne](https://github.com/mnemosyne-proj/mnemosyne)
with fixes for running and building Mnemosyne on Apple Silicon Macs.

## What is different from upstream

| Branch | What it fixes |
|---|---|
| `fix-macos-tree-checkboxes` | With Qt 6.10/6.12, macOS does not draw the check boxes in tree views. In *Browse cards* and *Activate cards* the tag and card-type trees looked like plain lists, and choosing tags seemed to do nothing. The fix draws only those check boxes with the Fusion style (`mnemosyne/pyqt_ui/tag_tree_wdgt.py`). |
| `macos-arm64-build` | `make macos` failed on Apple Silicon (missing Intel-only mplayer/XQuartz libraries), required `CODESIGN_IDENTITY`, and produced an app with a broken signature. Fixed in `mnemosyne.spec`, `makefile` and the README's Mac section. |

Both branches are merged into `master`. They are also kept as separate
branches so that each can be offered upstream as its own pull request.

## Remotes

```
origin    git@github.com:MichaelNedzelsky/mnemosyne.git   (this fork, push here)
upstream  git@github.com:mnemosyne-proj/mnemosyne.git      (original project)
```

To set this up in a fresh clone:

```
git clone git@github.com:MichaelNedzelsky/mnemosyne.git
cd mnemosyne
git remote add upstream git@github.com:mnemosyne-proj/mnemosyne.git
```

## Updating from upstream

```
cd ~/gits/mnemosyne
git switch master
git fetch upstream
git merge upstream/master        # resolve conflicts if any, then commit
git push origin master
```

Then check whether upstream has fixed the issues themselves. If it has,
the merge may touch the same lines, and the fork's change can be dropped:

- Check boxes: open *Browse cards* and look for check boxes next to the tags.
  To check whether Qt itself has been fixed, temporarily disable the
  `if sys.platform == "darwin":` block in `TagsTreeWdgt.__init__`.
- Build: run `make macos` as described below.

After updating, regenerate the Qt UI files (this also happens automatically
in the launchers below):

```
make -C mnemosyne/pyqt_ui PYTHON=~/.local/pipx/venvs/mnemosyne-proj/bin/python
```

## Running from this checkout

The launchers `~/mnemo.sh` and `~/mnemo_imported.sh` run the code in
`~/gits/mnemosyne` (whatever branch is checked out, normally `master`). They
use the Python of the pipx-installed `mnemosyne-proj` package, which has all
dependencies:

```
REPO="$HOME/gits/mnemosyne"
PY="$HOME/.local/pipx/venvs/mnemosyne-proj/bin/python"
make -s -C "$REPO" build PYTHON="$PY" >/dev/null || exit 1
cd "$REPO" && PYTHONPATH="$REPO" exec "$PY" -W ignore::SyntaxWarning \
    mnemosyne/pyqt_ui/mnemosyne -d <data directory> "$@"
```

Remember that checking out a branch other than `master` changes what the
launchers run.

### Qt version of the pipx environment

It currently uses PyQt6 6.12.0 and PyQt6-WebEngine 6.140.0. WebEngine
changed its version numbering after 6.11, so 6.140 is the release that goes
with Qt 6.12.

```
pipx runpip mnemosyne-proj list | grep -i pyqt          # show versions
pipx runpip mnemosyne-proj install "PyQt6==<ver>" "PyQt6-WebEngine==<ver>"
make -B -C ~/gits/mnemosyne/mnemosyne/pyqt_ui PYTHON=~/.local/pipx/venvs/mnemosyne-proj/bin/python
```

To go back to the previous versions, use `PyQt6==6.10.0 PyQt6-WebEngine==6.10.0`.
Always run `make -B` afterwards so the UI files are regenerated with the
matching `uic`.

## Building Mnemosyne.app

Step-by-step instructions for non-developers (Homebrew Python, fresh
clone) are in [BUILD-MAC-APP.md](BUILD-MAC-APP.md).

One-time setup of a build environment (`venv/` is ignored by git):

```
cd ~/gits/mnemosyne
/opt/homebrew/opt/python@3.14/bin/python3.14 -m venv venv
venv/bin/pip install --upgrade pip
venv/bin/pip install PyQt6 PyQt6-WebEngine matplotlib cheroot webob pillow \
    gTTS argon2-cffi legacy-cgi pyinstaller
```

Build:

```
cd ~/gits/mnemosyne
source venv/bin/activate
rm -rf build dist          # or `make clean`, see the note below
make macos
deactivate
```

The result is `dist/Mnemosyne.app`, a native arm64 app of about 530 MB.
`make macos` ends by re-signing the app and running
`codesign --verify --deep --strict`. If that last command fails, the app is
broken.

Notes:

- `make clean` also deletes a tracked file. Restore it afterwards:
  `git restore mnemosyne/android/app/src/main/assets/python/mnemosyne.zip`.
- Without `CODESIGN_IDENTITY` the app is signed ad hoc. That is fine on the
  Mac that built it. On other Macs Gatekeeper rejects it unless it is signed
  with an Apple Developer ID and notarized (`xcrun notarytool`).
- `TARGET_ARCH=x86_64` or `universal2` is possible, but needs a Python and
  wheels for that architecture. matplotlib/numpy wheels are not universal2.
- Before opening a real collection with a new build or a new Qt version,
  back up the data directory.

## Offering the fixes upstream

Each fix branch is based on upstream `master`. To refresh one before
opening a pull request:

```
git switch fix-macos-tree-checkboxes    # or macos-arm64-build
git rebase upstream/master
git push --force-with-lease origin HEAD
gh pr create --repo mnemosyne-proj/mnemosyne --base master \
    --head MichaelNedzelsky:fix-macos-tree-checkboxes
```
