# enable-chromium-ac3-ec3-system-decoding
Enable system level decoding of Dolby AC3/EC3 in Chromium.

##### English | [简体中文](./README.zh_cn.md)

## Background
AC3 and EC3 are Dolby's audio coding formats, it's Dolby's IP and can not be integrated into Chromium(ffmpeg).

On Windows, since Windows 8, Microsoft preload AC3/EC3 decoder in windows image as MFT(Media Foundation Transform). Third party can use it to decode ac3/ec3 audio by load MFT manually. This gives us a chance to use it in Chromium. 

## How Chromium use AC3/EC3 MFT decoder
Natively Chromium use ffmpeg as it's decoder factory. And chromium also support Mojo Audio Decoder, if target format is not supported by ffmpeg, chromium will try to find platfrom decoder, it's a kind of Mojo Audio Decoder.

MediaFoundationAudioDecoder is an implementation of Mojo Audio Decoder on Windows, Dolby Audio Decoder MFT will be selected by input audio format internally. 

## Build a custom chromium with AC3/EC3 support
Even if code for support AC3/EC3 playback integrated in chromium, but all of them are disabled by default.

Here is change list: [Add Dolby AC3, EC3 codec support on Windows Platform](https://chromium-review.googlesource.com/c/chromium/src/+/4116077)

You have to turn build flag `enable_platform_ac3_eac3_audio ` enabled on your custom build.

Here is a sample GN args for build chromium:

    # //arg.gn
    is_clang=true
    enable_nacl=false
    ffmpeg_branding="Chrome"
    proprietary_codecs=true
    enable_platform_ac3_eac3_audio = true

`enable_platform_ac3_eac3_audio` is a new build flag for controlling AC3,EC3 support on or off.

## [Releases](https://github.com/cjw1115/enable-chromium-ac3-ec3-system-decoding/releases)
Since build chromium will spend much time, I attached a build from my machine to Github. You can download it and try this feature.
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

### Verify with WEB API
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

### Verify with Process modules
If on system decoding is working, the Dolby MFT will be loaded by chromium. You can use some tool to dump chromium loaded modules.

Take listdll.exe as example: [Download listdll](https://learn.microsoft.com/en-us/sysinternals/downloads/listdlls)

```bash
listdll.exe chrome.exe
```
if module `C:\Windows\System32\DolbyDecMFT.dll` is loaded, it means Dolby AC3/EC3 decoding worked.
