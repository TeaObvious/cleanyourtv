# Epson EF-22N (HB38B): Google TV 14 on the HB35 / SMFR3 module

This records one setup session on 8 November 2026, including a later restart through the projector's
own menu. It is not a several-day tested debloat list. The owner identified the retail model as
**Epson EF-22N**, with **HB38B** on the projector label. Android reports the internal module as
`HB35` / `SMFR3`; that different identifier should not be mistaken for a different retail projector.
Other EF-22 variants or firmware were not tested.

| Identifier | Observed value |
|---|---|
| Retail model / projector label (owner-identified) | Epson EF-22N / `HB38B` |
| Android brand / manufacturer | `EPSON` / `SDMC` |
| Reported model / product / device | `HB35` / `SMFR3` / `himalaya` |
| Platform | MediaTek `m7332` |
| Android | Google TV on Android 14, SDK 34 |
| Firmware / security patch | `UKNV.260904.001` / `2026-09-01` |
| Memory / UI | About 1.8 GiB usable RAM; 1920x1080 at density 320, no display override |
| Stock Home | `com.google.android.apps.tv.launcherx/.home.HomeActivity` |
| Replacement Home | Reference launcher 3.4, `com.gathin.tvhome/.MainActivity`, built from source |
| Network / external USB | Wi-Fi; no external USB device or USB Ethernet adapter |

The [BenQ GV32](../../examples/benq-gv32/README.md) also uses an SDMC `himalaya` / `m7332` board.
That was useful context, not a reason to apply its package list unchanged. The
[Epson LS800](../../examples/epson-ls800/debloat.txt) uses a different Android module and stock Home.

## Home and the saved Wi-Fi network

With the original packages enabled, the HOME resolver reported these priorities:

| Component | Priority |
|---|---:|
| `com.google.android.apps.tv.launcherx/.home.HomeActivity` | 2 |
| `com.google.android.tungsten.setupwraith/.RecoveryActivity` | 1 |
| `com.gathin.tvhome/.MainActivity` | 0 |
| `com.android.tv.settings/.system.FallbackHome` | -1000 |

`cmd package set-home-activity` printed `Success` and the HOME role holder named the replacement,
but the resolver still chose Google TV. Installation with `STOCK_LAUNCHER=none` successfully installed
and opened the launcher, then exited at the script's Home-resolution check; it did not disable either
stock package. This was not evidence that the default Home had changed.

The first working arrangement kept both stock packages enabled and used `HomeKeyService` with
`stock_home_redirect` pointing to `com.google.android.apps.tv.launcherx` and `key_filter=false`.
The owner saw the partly loaded original Home for about ten seconds during boot. Google TV's main
and core-service processes remained in memory after the replacement appeared; the redirect did not
terminate them.

For a direct handover, the saved Wi-Fi had to be moved away from the setup wizard first:

1. The saved network owner, read with `dumpsys wifi` and its `cname` fields, was
   `com.google.android.tungsten.setupwraith`.
2. The owner forgot the network and rejoined it through Settings. Wireless debugging needed enabling
   again after the Wi-Fi transition.
3. A new read showed `cname=android.uid.system:1000`. Only after that check, with the replacement
   launcher already visibly in front, were `setupwraith` and `launcherx` disabled together using
   `tools/disable.sh` and its recovery record. No safety override was used.
4. Both the HOME resolver and foreground activity then named `com.gathin.tvhome/.MainActivity`.
   The redirect configuration and the bound accessibility helper were removed. The final setup
   has `key_filter=false` and no bound CleanYourTV key service.

The two packages remained disabled after the final menu restart, and the boot log started the
replacement directly without a `launcherx` process. The final saved-network owner was still
`android.uid.system:1000`.

Wi-Fi did not reconnect automatically after the first handover reboot. The owner reconnected it and
then restarted again through the projector menu; automatic reconnection worked on that restart.
This is a same-session observation, not proof of long-term reconnection or standby behavior.

The owner did not need Chromecast or Google Assistant. This handover does **not** establish that
casting works with `setupwraith` disabled; another owner who needs casting should retain it unless
that dependency is separately resolved and tested.

## Package changes observed in this session

These eleven packages were newly disabled for user 0 with the repository's guarded tools and were
still disabled after the final restart. The table records this owner's choices on this firmware;
it is not a ready-to-apply list for other Epson models. Nothing was uninstalled.

| Group | Packages | Scope |
|---|---|---|
| Feedback and partner attribution | `com.google.android.feedback`, `com.android.tv.feedbackconsent`, `com.google.android.partnersetup` | Selected Google feedback/partner components, not a claim that every enabled app is free of telemetry |
| Android advertising/personalization | `com.android.adservices.api`, `com.android.ondevicepersonalization.services`, `com.android.federatedcompute.services` | Present and enabled before this session |
| Unused voice and casting | `com.google.android.tts`, `com.google.android.katniss`, `com.google.android.apps.mediashell` | Explicit owner opt-out of text-to-speech, the mic/Assistant and Chromecast |
| Original Home claimants | `com.google.android.tungsten.setupwraith`, `com.google.android.apps.tv.launcherx` | Only after the Wi-Fi ownership and replacement-launcher checks above |

Netflix (`com.netflix.ninja`) and Prime Video (`com.amazon.amazonvideo.livingroom`) were already
disabled before this session. They are not part of the eleven changes and are not recommended
disables for everyone. Play Games and the Google Ambient screensaver were left enabled.

The owner separately chose to delete downloaded text-to-speech data, which increased available
storage by about 330 MiB immediately afterward. That was a storage change, not a RAM measurement;
re-enabling the package would not restore the deleted voice data. Data deletion is not part of a
shared disable list or a routine step to repeat after firmware updates.

## Projector and media packages retained

| Package | Evidence / reason to retain |
|---|---|
| `com.sdmc.kt` | Declares focus, autofocus, auto-keystone, quad correction, fit, zoom, rotation and eye-protection activities; handles global remote events |
| `com.sdmc.service` | Handles global remote events, power/screen state and other board services; its logging activities are not a reason to disable the whole package |
| `com.sdmc.btservice` | Declares remote auto-pairing and remote firmware-update components |
| `com.sdmc.remotecontrolservice` | Declares `SdmcInputIME`, the remote input-method service |
| `fusion.android.tv.demo` | Declares `com.mstar.android.intent.action.TV_INPUT_BUTTON` activities; despite its retail-demo name, it is part of the input-key path |
| `com.sdmc.implayer` | Preinstalled IMPlayer for local/USB media; retained, not replaced |
| `com.mediatek.tvinput`, `com.mediatek.tv.service`, `com.google.android.tv.inputplayer` | Input and TV middleware; the TV-input dump exposed the MediaTek HDMI service |
| `com.epson.projector.update` | Epson Projector Update, version 2.1.7 on this unit; firmware updates were explicitly retained |

YouTube (`com.google.android.youtube.tv`), YouTube Music (`com.google.android.youtube.tvmusic`),
Plex (`com.plexapp.android`), Play Store, Play Services, the keyboard and Android system packages
were also kept enabled. The dock contained YouTube, YouTube Music, Plex and Epson Projector Update;
IMPlayer was visible in the app grid. Their launch activities resolved, but that is not a playback
or firmware-update installation test.

Factory/test packages, including `com.sdmc.sdmcfactory`, `com.sdmc.keyburn`,
`com.dangs.factorytest` and `mediatek.factorymenu.ui`, were left alone and never launched.
`com.ref.devicedataexport` exposed a Settings data-export activity; its name alone was not treated
as evidence of background tracking.

## Physical remote and the Prime-button experiment

After the direct Home handover and menu restart, the owner confirmed:

- Home returns to the replacement launcher.
- The YouTube button still works with `launcherx` disabled.
- Holding brightness or volume continues changing the value.
- Autofocus completed successfully after both tested starts without manual intervention.
- Restart through the projector's own menu works.

The Prime Video button initially opened its Play Store activation page because Prime was already
disabled. A physical-key capture reported `KEY_F5`; `scan:63:com.plexapp.android` in `app_keys`, with
key filtering enabled, successfully opened Plex. It also reduced held volume to one step per press;
brightness repeat continued to work. The owner chose normal volume repeat, so the remap was removed
and `key_filter=false` restored. Plex remains available through the dock.

Changing only the theme's `key_filter` from false to true initially left Android reporting
`InputFilterEnabled: false`, and the Prime key still opened the Play Store. After reinstalling the
same locally built launcher and re-enabling the helper, the system reported true and the physical
remap worked. The final direct-Home configuration needs neither the helper nor that workaround.
The held-volume trade is similar to the [TCL G09 observations](tcl-55p71b-g09-key-repeat.md), but was
tested independently on this projector.

The final physical tests do not establish every branded/profile/settings shortcut. In particular,
an injected key or an enabled input package should not be presented as proof of a physical button
or an HDMI signal path.

## Resources, debugging and remaining limits

Before the last optimization, PSS snapshots showed roughly 104 MiB across the original launcher's
main/core processes, 40 MiB for the Assistant interactor and 29 MiB for Cast. Those processes were
absent after the final restart; the replacement's PSS was about 37 MiB. These are separate snapshots,
not guaranteed net physical-RAM savings or a controlled performance benchmark. The observed CPU load
was dominated by board/kernel processes; focus, image processing and other hardware services were
not disabled to improve a synthetic number.

Wireless debugging exposed a changing TLS connection port. Once debugging was enabled, this unit
also accepted authenticated ADB on port 5555 after on-screen authorization; the session did not run
`adb tcpip 5555`. Wireless debugging needed enabling again during the network/reboot workflow.
See the [native ADB backend guide](../../tools/native-adb.md) for paired encrypted connections;
this report is not an encrypted-backend test.

At the end, USB and Wireless debugging were switched off. `tools/adb-closed.py` received explicit
connection refusals on both 5555 and the then-current Wireless connection port. There was no further
reboot after that closing check, and revoking all saved debugging authorizations was not verified.

Not tested: several days of use, prolonged standby, HDMI playback/CEC with a connected source,
USB-media playback, new Bluetooth pairing, installing an Epson firmware update, casting, and a full
restore. Package-state backups and the tools' restore records were retained privately. No APKs,
device/network identifiers, account data, raw dumps or private maintenance scripts accompany this note.
