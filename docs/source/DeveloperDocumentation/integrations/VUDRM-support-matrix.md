# VUDRM SUPPORT MATRIX

**See bottom of page for caveats.**

| Browsers | PlayReady | Widevine | Fairplay |
|-|-|-|-|
| **Chrome** | <p>&#10008;</p> | <p>&#10004;</p> | <p>&#10008;</p> |
| **Edge** _Windows 10^_ | <p>&#10004;</p> | <p>&#10004;</p> | <p>&#10008;</p> |
| **Firefox** | <p>&#10008;</p> | <p>&#10004;</p> | <p>&#10008;</p> |
| **Internet Explorer 11** _Windows 8.1^_ | <p>&#10004;</p> | <p>&#10008;</p> | <p>&#10008;</p> |
| **Safari** | <p>&#10008;</p> | <p>&#10008;</p> | <p>&#10004;</p> |
| **Opera** | <p>&#10008;</p> | <p>&#10004;</p> | <p>&#10008;</p> |

| Mobile | PlayReady | Widevine | Fairplay |
|-|-|-|-|
| **Android 4.4^** _VUDRMWidevine SDK: offline playback available_ | <p>&#10008;</p> | <p>&#10004;</p> | <p>&#10008;</p> |
| **Chrome mobile (Android)** | <p>&#10008;</p> | <p>&#10004;</p> | <p>&#10008;</p> |
| **iOS** _VUDRMfairplay SDK: 10^ offline available, 9^ online only_ | <p>&#10008;</p> | <p>&#10008;</p> | <p>&#10004;</p> |
| **Safari mobile (iOS)** _iOS 11.2^_ | <p>&#10008;</p> | <p>&#10008;</p> | <p>&#10004;</p> |
| **Windows phone** | <p>&#10004;</p> | <p>&#10008;</p> | <p>&#10008;</p> |

| Consoles | PlayReady | Widevine | Fairplay |
|-|-|-|-|
| **PlayStation** | <p>&#10004;</p> | <p>&#10008;</p> | <p>&#10008;</p> |
| **Xbox** | <p>&#10004;</p> | <p>&#10008;</p> | <p>&#10008;</p> |

| Smart TVs | PlayReady | Widevine | Fairplay |
|-|-|-|-|
| **Amazon Fire** | <p>&#10004;</p> | <p>&#10004;</p> | <p>&#10008;</p> |
| **Android TV** _VUDRMWidevine SDK_ | <p>&#10008;</p> | <p>&#10004;</p> | <p>&#10008;</p> |
| **Apple TV** _VUDRMfairplay SDK_ | <p>&#10008;</p> | <p>&#10008;</p> | <p>&#10004;</p> |
| **Google Chromecast** | <p>&#10008;</p> | <p>&#10004;</p> | <p>&#10008;</p> |
| **Google TV** | <p>&#10008;</p> | <p>&#10004;</p> | <p>&#10008;</p> |
| **Roku** | <p>&#10004;</p> | <p>&#10004;</p> | <p>&#10008;</p> |
| **Samsung Tizen** _2017 models^_ | <p>&#10004;</p> | <p>&#10008;</p> | <p>&#10008;</p> |
| **tvOS** _VUDRMfairplay SDK_ | <p>&#10008;</p> | <p>&#10008;</p> | <p>&#10004;</p> |
| **Smart TV Alliance** _LG, PANASONIC, PHILIPS, TOSHIBA_ | <p>&#10004;</p> | <p>&#10004;</p> | <p>&#10008;</p> |

- Widevine mandates all browser CDM implementations to stay current with Chrome stable releases to ensure that the latest updates are applied. Older versions of Chrome, Firefox, and Opera may not be able to use DRM. See [Widevine's deprecation schedule](https://www.widevine.com/news).

- VUDRM uses Widevine Modular by default (Widevine Classic is not supported).

- Some Smart TV Alliance models may not support Widevine.

- In Linux based systems, Widevine may not be supported on some versions of Chrome and Firefox.