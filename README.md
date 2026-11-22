# luci-app-esim


Native LuCI frontend for OpenWrt's `lpac` eSIM manager, installed as
`luci-app-esim` under **Modem → eSIM Manager**. The application uses
the packaged `/usr/bin/lpac` entrypoint and `/etc/config/lpac`.


## Banana Pi BPI-R4 with Quectel RM551E-GL

This fork targets the latest OpenWrt stable release, 25.12.6 (apk packages),
on the Banana Pi BPI-R4 (`mediatek/filogic`, package architecture
`aarch64_cortex-a53`). Build the bundle with the **Bundled OpenWrt packages**
workflow. Its default architecture is `aarch64_cortex-a53`. Then install both packages:

```sh
apk update
apk add --allow-untrusted ./lpac-2.3.0-r5.apk ./luci-app-esim-*.apk
```

The LuCI package was previously named `luci-app-lpac`. If that package is
installed, remove it first with `apk del luci-app-lpac`, then install the new
packages. If the eSIM pages then ask you to choose a backend, save
**Settings** once more.

Recommended RM551E-GL setup (module in the BPI-R4 M.2 Key-B slot, USB):

1. Put the modem in QMI mode and reboot it: `AT+QCFG="usbnet",0`.
2. In **Modem → eSIM Manager → Settings**, choose the **uqmi** backend and
   `/dev/cdc-wdm0` (Detect also finds ports bound to Quectel's `qmi_wwan_q`).
3. If the eUICC card is in the second SIM slot, select that UIM slot:

   ```sh
   uci set lpac.uqmi.uim_slot='2'
   uci commit lpac
   ```

The **AT** backend on `/dev/ttyUSB2` can be used as a fallback. This lpac build
tolerates the RM551E-GL rejecting `AT+CCHO=?` and its `+CME ERROR` replies.
Do not use the **MBIM** backend with this modem: Quectel has said it does not
plan to support eSIM over MBIM on the RM551E-GL.

RM551E-GL firmware still has known eSIM issues. If APDUs stop responding after
a reboot or a profile switch, reset the modem (`AT+CFUN=1,1`) and retry. Keep
other tools (ModemManager, a second lpac, AT scripts) off the same control
port while the eSIM manager is running.

## License

The LuCI application is licensed under Apache-2.0; see [LICENSE](LICENSE).
Bundled lpac binaries retain the upstream AGPL/LGPL and component licenses
shipped by [estkme-group/lpac](https://github.com/estkme-group/lpac). 
