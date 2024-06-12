### 推流
`ffmpeg -re -stream_loop -1 -i in.flv -c copy -f flv outurl`
### 推流追加时间戳
`ffmpeg -stream_loop -1 -re -i move.flv -vf "settb=AVTB,setpts='trunc(PTS/1K)*1K+st(1,trunc(RTCTIME/1K))-1K*trunc(ld(1)/1K)',drawtext=fontfile=arial.ttf:text='%{localtime}.%{eif\:1M*t-1K*trunc(t*1K)\:d\:3}':fontsize=40:fontcolor=red: x=50: y=50" -vcodec libx264 -r 25 -g 75 -acodec copy -f flv rtmp://xxxxxxx`

### 推流时去除视频或者音频
**去除音频**：`-c:v copy -an`
**去除视频**：`-c:a copy -vn`
例如：
**转h264** : `ffmpeg -i .\cut.mp4 -c:v copy -an cut.h264`
**转aac**：`ffmpeg -i .\cut.mp4 -vn -c:a copy test.aac`

### 常用的转码参数
`ffmpeg -i .\cut.mp4 -vf "drawtext=fontfile=simhei.ttf: text=‘720_4M_250’:x=10:y=10:fontsize=24:fontcolor=red:shadowy=2" -vcodec h264 -acodec copy -bf 0 -g 250 -keyint_min 250 -sc_threshold 0 -r 25 -b:v 4M -maxrate 4M -s 1280:720 720p_4M_250.flv`
**-bf 0** 代表不含B帧
**-g 250** 代表关键帧之间间隔是250帧
**-sc threshold 0** 代表关闭关键帧补偿
**-r 25** 代表帧率
**-b:v 4M** 代表码率是4兆
**-s 1280:720** 代表分辨率
**-vf "drawtext=fontfile=simhei.ttf: text=‘720_4M_250’:x=10:y=10:fontsize=24:fontcolor=red:shadowy=2"**  代表在屏幕左上角添加水印 720_4M_250 红色字体

### 查看是否含有B帧
**mac**
`ffprobe -v quiet -show_frames -select_streams v test.mp4 | grep "pict_type=B"`
**windows**
`ffprobe -v quiet -show_frames -select_streams v test.mp4 | findstr "pict_type=B"`

### 查看音频反向
`ffplay -showmode 1 "rtmp://xxxxxxxxx"`
### ffplay播放时取消缓存
`ffplay -fflags nobuffer`

### 查看当前设备的音视频设备列表
**mac**
`ffmpeg -f avfoundation -list_devices true -i ""`
**windows**
`ffmpeg -list_devices true -f dshow -i dummy`

### 通过ffprobe分析流
`ffprobe -show_packets -i "rtmp:/xxxxxxx" -of xml`
`ffprobe -show_frames -i "rtmp:/xxxxxxx" -of xml`

### 设置屏幕宽高比
`ffmpeg -i input.mp4 -aspect 16:9 output.mp4`
**常用的宽高比** 16:9，4:3，16:10，5:4，2:2，1:1，2:3，5:1，2:3，9:1
### ffmpeg录制屏幕
**mac**
`ffmpeg -f avfoundation -framerate 25 -i "1" output.mp4`
`ffmpeg -f avfoundation -framerate 30 -pixel_format uyvy422 -i "2" output.mp4`
**windows**
`ffmpeg -f gdigrab -framerate 25 -i desktop output.mp4`

### ffmpeg录制屏幕和声音
**mac**
`ffmpeg -f avfoundation -framerate 25 -i "1:0" -f avfoundation -i ":0" output.mp4`
-f avfoundation -i ":0": 指定音频输入为默认设备，可以改为其他音频设备的index

**windows**
`ffmpeg -f gdigrab -framerate 25 -i desktop -f dshow -i audio="Microphone (Realtek High Definition Audio)" output.mp4`
-f dshow -i audio="Microphone (Realtek High Definition Audio)": 指定音频输入设备，可以吧Microphone (Realtek High Definition Audio)换为其他音频设备的名称
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg4MjUwNzk0Niw3MzA5OTgxMTZdfQ==
-->