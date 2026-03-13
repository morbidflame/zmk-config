# ZMK Firmware Build Guide
> Ambiente: Bazzite KDE (Fedora immutabile) + Distrobox + Podman

---

## Setup iniziale

### 1. Crea la distrobox
```bash
distrobox create --name zmk --image docker.io/zmkfirmware/zmk-dev-arm:stable
distrobox enter zmk
```

### 2. Inizializza west
```bash
west init -l ~/Documenti/zmk-config/config
west update
```
> ⚠️ `west update` scarica ~4GB (Zephyr, ZMK, toolchain). Solo la prima volta.

### 3. Configura le variabili d'ambiente
```bash
source ~/Documenti/zmk-config/zephyr/zephyr-env.sh
export ZEPHYR_TOOLCHAIN_VARIANT=zephyr
export CMAKE_PREFIX_PATH=$ZEPHYR_BASE/share/zephyr-package/cmake
```
> Questi comandi vanno ripetuti ogni volta che si entra nella distrobox.

---

## Compilazione

### Lato sinistro
```bash
west build -s zmk/app -b nice_nano//zmk -p -- \
  -DSHIELD="corne_left nice_view_adapter nice_view" \
  -DZMK_CONFIG="$HOME/Documenti/zmk-config/config"
```

### Lato destro
```bash
west build -s zmk/app -b nice_nano//zmk -p -- \
  -DSHIELD="corne_right nice_view_adapter nice_view" \
  -DZMK_CONFIG="$HOME/Documenti/zmk-config/config"
```

> Il firmware compilato si trova sempre in:
> `~/Documenti/zmk-config/build/zephyr/zmk.uf2`

---

## Flash

1. Doppio click sul tasto reset del nice_nano
2. Attendi che appaia il volume `NICENANO`
3. Copia il firmware:

```bash
cp ~/Documenti/zmk-config/build/zephyr/zmk.uf2 /run/media/p/NICENANO/
```

> Il volume si smonta automaticamente dopo il flash.
> Ripeti per il lato destro.

---

## Modifica keymap

I file da modificare sono in `~/Documenti/zmk-config/config/`:

| File | Descrizione |
|------|-------------|
| `corne.keymap` | Layout tasti |
| `corne.conf` | Configurazione ZMK |
| `west.yml` | Versione ZMK/Zephyr |

Dopo ogni modifica ricompila e reflasha entrambi i lati.

---

## Cleanup

Quando non hai più bisogno dell'ambiente di build:

```bash
# Rimuovi i file scaricati da west (recupera ~4GB)
cd ~/Documenti/zmk-config
rm -rf zephyr zmk modules tools bootloader build .west

# Rimuovi la distrobox
distrobox stop zmk
distrobox rm zmk

# Rimuovi l'immagine container
podman rmi docker.io/zmkfirmware/zmk-dev-arm:stable
```

> I tuoi file di config in `~/Documenti/zmk-config/config/` non vengono toccati.

---

## Note

- Il flag `-p` nel comando di build forza una ricompilazione pulita
- Solo il lato **sinistro** (central) va flashato per modifiche al keymap
- Il lato **destro** va flashato solo se cambia la configurazione hardware
- La distrobox condivide la home con il sistema host — i file sono accessibili da entrambi
