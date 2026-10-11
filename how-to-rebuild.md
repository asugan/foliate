# Rebuilding and installing the fork

The fork is installed as a user Flatpak (`com.github.johnfactotum.Foliate`, remote: `foliate-fork`).
After changing the code, rebuild and reinstall it with:

```sh
~/Projects/foliate-flatpak/rebuild.sh
```

To run it:

```sh
flatpak run com.github.johnfactotum.Foliate
```

## What the script does

`flatpak-builder` doesn't work on this system (`linkat: No such file or directory`),
so the script uses plain `flatpak build` commands:

```sh
SRC=$HOME/Projects/foliate
D=$HOME/Projects/foliate-flatpak
cd "$D"
rm -rf app _build
flatpak build-init app com.github.johnfactotum.Foliate org.gnome.Sdk org.gnome.Sdk 49
flatpak build --filesystem="$SRC":ro --filesystem="$D" app meson setup --prefix=/app "$D/_build" "$SRC"
flatpak build --filesystem="$SRC":ro --filesystem="$D" app ninja -C "$D/_build" install
flatpak build-finish app --command=foliate --share=network --share=ipc \
    --socket=fallback-x11 --device=dri --socket=wayland \
    --filesystem=xdg-run/speech-dispatcher:ro \
    "--add-policy=Tracker3.dbus:org.freedesktop.Tracker3.Miner.Files=tracker:Documents"
flatpak build-export repo app
flatpak update --user -y --noninteractive com.github.johnfactotum.Foliate
```

## Requirements

- GNOME SDK 49: `flatpak install --user flathub org.gnome.Sdk//49`
- The `src/foliate-js` submodule: `git submodule update --init --recursive`

## First-time install

The script only updates an existing install. If the app isn't installed yet, run `rebuild.sh` and then:

```sh
flatpak remote-add --user --no-gpg-verify --if-not-exists foliate-fork ~/Projects/foliate-flatpak/repo
flatpak install --user foliate-fork com.github.johnfactotum.Foliate
```
