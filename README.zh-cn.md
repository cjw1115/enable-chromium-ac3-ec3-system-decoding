# enable-chromium-ac3-ec3-system-decoding
在 Chromium 里面为杜比 AC3/EC3 启用系统级解码。 AC3 就是 Dolby Digital, EC3 是 Dolby Digital Plus。 

##### 简体中文 | [English](./README.md)

## 背景
AC3 和 EC3 是杜比的音频编码格式，因为版权问题等不能直接集成解码器到 Chromium 里面（Chromium 使用 FFMPEG 解码）。
在 Windows 上， 微软从 Windows 8 开始在系统镜像中自带杜比 AC3/EC3 的解码器，以 MFT 的形式存在。第三方可以手动调用这个解码器来解码杜比 AC3/EC3, Chromium 同样可以使用这种方式。 
## Chromium 如何使用 AC3/EC3 MFT 解码器
Chromium 默认使用 FFMPEG 作为它的音频解码器，与此同时，chromium 也使用基于 MOJO 的外部音频解码器，如果目标音频格式不被 FFMPEG 支持，Chromium 会尝试使用外部的解码器。

MediaFoundationAudioDecoder 用来在 Windows 调用外部解码器，杜比音频解码器 MFT 会在其内部被选择使用。

## 编译一个带有 AC3/EC3 支持的 Chromium 版本
目前为止，即使用来调用系统级 AC3/EC3 解码器的代码已经被合并入 Chromium，但是这些代码默认没有被启用。

这儿是提交列表： [Add Dolby AC3, EC3 codec support on Windows Platform](https://chromium-review.googlesource.com/c/chromium/src/+/4116077)

你必须手动开启编译选项 `enable_platform_ac3_eac3_audio ` 在你自己的编译环境里。

这是一个示例的 GN 编译参数：

    # //arg.gn
    is_clang=true
    enable_nacl=false
    ffmpeg_branding="Chrome"
    proprietary_codecs=true
    enable_platform_ac3_eac3_audio = true

`enable_platform_ac3_eac3_audio` 是一个新的编译选项，用来控制是否启用 AC3/EC3解码器。

## [Releases](https://github.com/cjw1115/enable-chromium-ac3-ec3-system-decoding/releases)
因为编译 chromium 需要消耗比较长的时间，我在这个仓库的 Release 里面上传了我自己编译的版本，已经默认打开了支持 AC3/EC3 的编译选项。

这是我用到的所有的 GN 编译参数，你可以用类似的参数来自己编译一个版本。

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

## 如何验证是否支持 AC3/EC3?

### 通过 API 验证
#### Media Capabilities API
```javascript
const audioConfiguration = {
    type: "media-source",
    audio: {
      // 对于 AC3, 使用 'ac-3'
      // 对于 EC3, 使用 'ec-3'
      contentType: 'audio/mp4; codecs="ac-3"',
      // spatialRendering 目前还不被 Chromium 支持.
      // 这个字段默认会被忽略.
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
// 对于 AC3, 使用 'ac-3'
// 对于 EC3, 使用 'ec-3'
if (MediaSource.isTypeSupported('audio/mp4; codecs="ac-3"')) {
  console.log('Dolby AC3 is supported!');
}
```

#### HTMLMediaElement.canPlayType()
```javascript
// 对于 AC3, 使用 'ac-3'
// 对于 EC3, 使用 'ec-3'
let obj = document.createElement("video");
if (obj.canPlayType('audio/mp4; codecs="ac-3"') === 'probably') {
  console.log('Dolby AC3 is supported!');
}
```
### 通过播放音频内容验证
你可以在当前这个 Readme 页面播放下面的音频文件，如果可以播放，意味着你现在的浏览器已经支持 AC3/EC3 解码。
#### bear-eac3-only-frag
https://user-images.githubusercontent.com/13924086/228438871-96a90b6e-7309-41e8-84ab-00f2d00a1f2c.mp4
#### bear-ac3-only-frag
https://user-images.githubusercontent.com/13924086/228438950-b67f8939-2d5b-4b13-babc-caeb31cd5c92.mp4

### 通过进程内加载的模块验证
如果系统级解码工作，那么杜比 MFT 会被 Chromium 加载，你可以通过一些工具来查看 Chromium 加载的所有模块。

用 listdll.exe 举个例子：[下载 listdll](https://learn.microsoft.com/en-us/sysinternals/downloads/listdlls)

```bash
listdll.exe chrome.exe
```
如果模块 `C:\Windows\System32\DolbyDecMFT.dll` 被加载了，那么意味着杜比 AC3/EC3 解码正在工作。
