# enable-chromium-ac3-ec3-system-decoding
Enable system level decoding of Dolby AC3/EC3 in Chromium (Windows and macOS only for now).

##### English | [简体中文](./README.zh-cn.md)

## Background
AC3 and EC3 are Dolby's audio coding formats, it's Dolby's IP and can not be integrated into Chromium (Chromium use FFMPEG decode codec AAC/FLAC/WAV etc..).

#### Windows
Since Windows 8, Microsoft preload AC3/EC3 decoder in windows image as MFT(Media Foundation Transform). Third party can use it to decode ac3/ec3 audio by load MFT manually. This gives us a chance to use it in Chromium. 

#### MacOS
AC3 codec has been supported since macOS 10.2, and EC3 codec has been supported since macOS 10.11. Using AudioToolbox and CoreAudio, third party now have the ability to integrate the support without loyalty issue, and Chromium can also use this way to do the decoding job.

## How Chromium use AC3/EC3 platform decoder
Natively Chromium use FFMPEG as it's decoder factory. And Chromium also support Mojo Audio Decoder, if target format is not supported by FFMPEG, Chromium will try to find platfrom decoder, it's a kind of Mojo Audio Decoder.

#### Windows
MediaFoundationAudioDecoder is an implementation of Mojo Audio Decoder on Windows, Dolby Audio Decoder MFT will be selected by input audio format internally. 

#### MacOS
AudioToolboxAudioDecoder is also an implementation of Mojo Audio Decoder on macOS, it will use system decoder to do the decoding job.

## Build a custom Chromium with AC3/EC3 support
Even if code for support AC3/EC3 playback integrated in Chromium, but all of them are disabled by default.

Here are change lists:

* [Add Dolby AC3, EC3 codec support on Windows Platform](https://crbug.com/1402182)
* [Implement AC3 / E-AC3 audio codec support for macOS](https://crbug.com/1430808)

You have to turn build flag `enable_platform_ac3_eac3_audio ` enabled on your custom build.

Here is a sample GN args for build Chromium:

    # //arg.gn
    is_clang=true
    enable_nacl=false
    ffmpeg_branding="Chrome"
    proprietary_codecs=true
    enable_platform_ac3_eac3_audio = true

`enable_platform_ac3_eac3_audio` is a new build flag for controlling AC3,EC3 support on or off.

## [Releases](https://github.com/cjw1115/enable-chromium-ac3-ec3-system-decoding/releases)
Since build chromium will spend much time, We already created builds and attached them on Github. You can download them and try this feature.

Here is used full GN args, you can also use below args to build on your machine.

    # Set build arguments here. See `gn help buildargs`.
    is_debug=false
    is_clang=true
    is_component_build=false
    is_official_build=true
    enable_nacl=false
    symbol_level=0
    ffmpeg_branding="Chrome"
    proprietary_codecs=true
    enable_platform_ac3_eac3_audio  = true
    target_cpu = "x64"

## How to verify if AC3,EC3 supported?

### Verify with Web API
#### Media Capabilities API
```javascript
const audioConfiguration = {
    type: "media-source",
    audio: {
      // for AC3, use 'ac-3'
      // for EC3, use 'ec-3'
      contentType: 'audio/mp4; codecs="ac-3"',
      // spatialRendering is not supported by Chromium.
      // This field will be ignored internally.
      spatialRendering: true
    },
  };

  navigator.mediaCapabilities
    .decodingInfo(audioConfiguration)
    .then((result) => {
      console.log(
        `Dolby AC3 is ${result.supported ? "" : "not "}supported,`
      );
    })
    .catch(() => {
      console.log(`decodingInfo error: ${contentType}`);
    });
```

#### MediaSource.isTypeSupported()
```javascript
// for AC3, use 'ac-3'
// for EC3, use 'ec-3'
if (MediaSource.isTypeSupported('audio/mp4; codecs="ac-3"')) {
  console.log('Dolby AC3 is supported!');
}
```

#### HTMLMediaElement.canPlayType()
```javascript
// for AC3, use 'ac-3'
// for EC3, use 'ec-3'
let obj = document.createElement("video");
if (obj.canPlayType('audio/mp4; codecs="ac-3"') === 'probably') {
  console.log('Dolby AC3 is supported!');
}
```
### Verify with test Content
You can play below content on this readme page. If it can be play, means your browser support AC3/EC3 decoding.
#### bear-eac3-only-frag
https://user-images.githubusercontent.com/13924086/228438871-96a90b6e-7309-41e8-84ab-00f2d00a1f2c.mp4
#### bear-ac3-only-frag
https://user-images.githubusercontent.com/13924086/228438950-b67f8939-2d5b-4b13-babc-caeb31cd5c92.mp4

#### Dolby Atmos content on bilibili
bilibili supoorts stream EC3 content on Safari on macOS, if you change Chromium's user-agent to Safari's, it will stream EC3 content too.

If you have bilibili account, you can try it.  

Change your user-agent: [User-Agent Switcher and Manager](https://chrome.google.com/webstore/detail/user-agent-switcher-and-m/bhchdcejhohfmigjafbampogmaanbfkg)

### Verify with Process modules(Windows only)
If on system decoding is working, the Dolby MFT will be loaded by Chromium. You can use some tool to dump Chromium loaded modules.

Take listdll.exe as example: [Download listdll](https://learn.microsoft.com/en-us/sysinternals/downloads/listdlls)

```bash
listdll.exe chrome.exe
```
if module `C:\Windows\System32\DolbyDecMFT.dll` is loaded, it means Dolby AC3/EC3 decoding worked.

## Welcome issues
If you have any problems when using this feature, please open an issue without hesitation! Thanks! 
