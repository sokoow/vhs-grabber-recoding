## So you want to record VHS tapes into MP4 (on Linux)?

### Equipment needed

VHS recoding is pretty easy, you just need 3 things:

- A VCR (highly recommend LG LV4787 or any other 6 head VCR).
- A USB Scart-compatible video grabber (I use MS210x compatible one from here: https://www.aliexpress.com/item/1005005039040169.html?spm=a2g0o.order_list.order_list_main.47.6a941802mLRCSr).
- (Optionally) A converter casette for VHS-C conversion into big S-VHS casettes - in case you wanted to recode camera footage.

### Putting it all together

It's as simple as connecting scart to VCR, USB to your laptop/PC and off you go.

### Determining addresses of your devices

#### Determining address of your audio recoding device

First, you need to issue ```arecord -l``` command, and check it's output. It will look like this:

```
$ arecord -l
**** List of CAPTURE Hardware Devices ****
card 1: Generic_1 [HD-Audio Generic], device 0: ALC257 Analog [ALC257 Analog]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 2: Device [USB Audio Device], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 3: MS210x [MS210x], device 0: USB Audio [USB Audio]
  Subdevices: 0/1
  Subdevice #0: subdevice #0
card 4: U0x46d0x81b [USB Device 0x46d:0x81b], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 5: acp [acp], device 0: DMIC capture dmic-hifi-0 []
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 6: USB [ThinkPad Thunderbolt 3 Dock USB], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

Here you can see that MS210x is card number 3, so your audio device will look like this: ```plughw:3,0```.

#### Determining address of your video recoding device

To determine your video recoding device, issue ```v4l2-ctl --list-devices```, you should see something like this in your terminal output:

```
$ v4l2-ctl --list-devices
Integrated Camera: Integrated C (usb-0000:06:00.0-2):
	/dev/video0
	/dev/video1
	/dev/media0

USB Video: USB Video (usb-0000:07:00.3-1.3):
	/dev/video2
	/dev/video3
	/dev/media1

UVC Camera (046d:081b) (usb-0000:07:00.3-1.4):
	/dev/video4
	/dev/video5
	/dev/media2
```

You're looking for first device under ```USB Video```, so this would be ```/dev/video2``` in this case.


## Running live preview of your VCR

To run live preview for your VCR output, issue command below, remembering to swap your video and audio devices to proper one from tutorial above:

```
mpv av://v4l2:/dev/video2 --audio-file=av://alsa:plughw:3,0
```

This will show you mpv player with usb grabber output (normally blue screen when vcr is on). Then just hit PLAY on VCR and watch the video.

## Recording the footage from VCR to a mkv RAW file

We'll be recommending you recording RAW video from grabber and then recoding it to x264/265 later on. Why this way? It's way easier on CPU, and thus avoids irritating glitches and jitter in final video.

**WARNING** Dumping RAW video from VHS grabber requires quite a lot of disk space - it will take about 240GB of disk space per every 4 hours of footage. 

To record RAW video using ffmpeg, use following command line:

```
sudo nice --20 ffmpeg -f v4l2 -i /dev/videoX -thread_queue_size 1024 -threads 0 -err_detect +crccheck+bitstream+buffer+compliant+aggressive -fflags +igndts -vcodec rawvideo -rtbufsize 702000k -vtag YV12 -f alsa -i plughw:Y,0 -acodec pcm_s16le -pix_fmt yuvj422p -c copy result-raw-video.mkv
```

Replacing X with your video device number, and Y with your audio device number. After ffmpeg is running, you need to press PLAY on your VCR and.. wait. I normally wait till the video casette is finished and it starts it's final rewind to the beginning (then I stop ffmpeg by pressing q key).

## Recoding RAW VCR footage to a MP4 video

Use the following ffmpeg command:

```
ffmpeg -i result-raw-video.mkv -c:v libx264 -profile:v baseline -acodec aac -ab 96k -filter:a "dynaudnorm=p=0.9:s=5" -enc_time_base 1:120 -vsync vfr -level 3.0 -pix_fmt yuv420p -threads 0 -movflags +faststart result-raw-video.mp4
```

This will use all cores to recode RAW grabber video to x264/mp4 format, this will be also Whatsapp/Telegram compatible pixel format if you wanted it, and will normalize the audio.