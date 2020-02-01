### Query: [how to parse more complex human-oriented text output to machine-friently style?](https://stackoverflow.com/questions/59913240/how-to-parse-more-complex-human-oriented-text-output-to-machine-friently-style)
([jump to the answer]())

This is the question about how to parse "unparseable" output into json, or to something easily consumable as json. This is "little" bit behind trivial stuff, so I'd like to know, how do you solve these things in principle, it's not about this specific example only. But example:

We have this command, which shows data about audio inputs: 

    pacmd list-sink-inputs

it prints something like this:

    2 sink input(s) available.
        index: 144
    	driver: <protocol-native.c>
    	flags: 
    	state: RUNNING
    	sink: 4 <alsa_output.pci-0000_05_00.0.analog-stereo>
    	volume: front-left: 15728 /  24% / -37.19 dB,   front-right: 15728 /  24% / -37.19 dB
    	        balance 0.00
    	muted: no
    	current latency: 70.48 ms
    	requested latency: 210.00 ms
    	sample spec: float32le 2ch 44100Hz
    	channel map: front-left,front-right
    	             Stereo
    	resample method: copy
    	module: 13
    	client: 245 <MPlayer>
    	properties:
    		media.name = "UNREAL! Tetris Theme on Violin and Guitar-TnDIRr9C83w.webm"
    		application.name = "MPlayer"
    		native-protocol.peer = "UNIX socket client"
    		native-protocol.version = "32"
    		application.process.id = "1543"
    		application.process.user = "mmucha"
    		application.process.host = "vbDesktop"
    		application.process.binary = "mplayer"
    		application.language = "C"
    		window.x11.display = ":0"
    		application.process.machine_id = "720184179caa46f0a3ce25156642f7a0"
    		application.process.session_id = "2"
    		module-stream-restore.id = "sink-input-by-application-name:MPlayer"
        index: 145
    	driver: <protocol-native.c>
    	flags: 
    	state: RUNNING
    	sink: 4 <alsa_output.pci-0000_05_00.0.analog-stereo>
    	volume: front-left: 24903 /  38% / -25.21 dB,   front-right: 24903 /  38% / -25.21 dB
    	        balance 0.00
    	muted: no
    	current latency: 70.50 ms
    	requested latency: 210.00 ms
    	sample spec: float32le 2ch 48000Hz
    	channel map: front-left,front-right
    	             Stereo
    	resample method: speex-float-1
    	module: 13
    	client: 251 <MPlayer>
    	properties:
    		media.name = "Trombone Shorty At Age 13 - 2nd Line-k9YUi3UhEPQ.webm"
    		application.name = "MPlayer"
    		native-protocol.peer = "UNIX socket client"
    		native-protocol.version = "32"
    		application.process.id = "2831"
    		application.process.user = "mmucha"
    		application.process.host = "vbDesktop"
    		application.process.binary = "mplayer"
    		application.language = "C"
    		window.x11.display = ":0"
    		application.process.machine_id = "720184179caa46f0a3ce25156642f7a0"
    		application.process.session_id = "2"
    		module-stream-restore.id = "sink-input-by-application-name:MPlayer"

very nice. But we don't want to show user all of this, we just want to show index (id of input), application.process.id, application.name and media.name, in some reasonable format. It would be great to parse it _somehow_ to json, but even if I preprocess it somehow, the `jq` is way beyond my capabilities and quite complex. I tried multiple approaches using jq, with regex or without, but I wasn't able to finish it. And I guess we cannot rely on order or presence of all fields.

I was able to get the work "done", but it's messy, inefficient, and namely expects no semicolons in media name or app name. Not acceptable solution, but this is the only thing I was able to bring to the "end".

incorrect solution:

    cat exampleOf2Inputs | 
    grep -e "index: \|application.process.id = \|application.name = \|media.name = " | 
    sed "s/^[ \t]*//;s/^\([^=]*\) = /\1: /" | 
    tr "\n" ";" | 
    sed "s/$/\n/;s/index:/\nindex:/g" | 
    tail -n +2 | 
    while read A; do 
    index=$(echo $A|sed "s/^index: \([0-9]*\).*/\1/");
    pid=$(echo $A|sed 's/^.*application\.process\.id: \"\([0-9]*\)\".*$/\1/'); 
    appname=$(echo $A|sed 's/^.*application\.name: \"\([^;]*\)\".*$/\1/'); 
    medianame=$(echo $A|sed 's/^.*media\.name: \"\([^;]*\)\".*$/\"\1\"/'); 
    
    echo "pid=$pid index=$index appname=$appname medianame=$medianame"; 
    done

~ I grepped the interessant part, replaced newlines with semicolon, split to multiple lines, and just extract the data multiple times using sed. Crazy.

Here the output is:

    pid=1543 index=144 appname=MPlayer medianame="UNREAL! Tetris Theme on Violin and Guitar-TnDIRr9C83w.webm"
    pid=2831 index=145 appname=MPlayer medianame="Trombone Shorty At Age 13 - 2nd Line-k9YUi3UhEPQ.webm"

which is easily convertable to any format, but the question was about json, so to:

    [
      {
        "pid": 1543,
        "index": 144,
        "appname": "MPlayer",
        "medianame": "UNREAL! Tetris Theme on Violin and Guitar-TnDIRr9C83w.webm"
      },
      {
        "pid": 2831,
        "index": 145,
        "appname": "MPlayer",
        "medianame": "Trombone Shorty At Age 13 - 2nd Line-k9YUi3UhEPQ.webm"
      }
    ]

Now I'd like to see, please, how are these things done correctly.

### A:
\- [`jtc`](https://github.com/ldn-softdev/jtc) accepts as input only JSON, thus, to achive the ask here, first, the input needs to be
converted (line by line) into some JOSN, then process it with `jtc`:

1. converting input into a stream of JSONs:
```bash
bash $ <inputs.txt sed -E 's/^ *([^=: ]+)[: =]+(.+)/{"\1": "\2"}/; s/""/"/g' | grep '^{'
{"2": "sink input(s) available."}
{"index": "144"}
{"driver": "<protocol-native.c>"}
{"flags": " "}
{"state": "RUNNING"}
{"sink": "4 <alsa_output.pci-0000_05_00.0.analog-stereo>"}
{"volume": "front-left: 15728 /  24% / -37.19 dB,   front-right: 15728 /  24% / -37.19 dB"}
{"balance": "0.00"}
{"muted": "no"}
{"current": "latency: 70.48 ms"}
{"requested": "latency: 210.00 ms"}
{"sample": "spec: float32le 2ch 44100Hz"}
{"channel": "map: front-left,front-right"}
{"resample": "method: copy"}
{"module": "13"}
{"client": "245 <MPlayer>"}
{"media.name": "UNREAL! Tetris Theme on Violin and Guitar-TnDIRr9C83w.webm"}
{"application.name": "MPlayer"}
{"native-protocol.peer": "UNIX socket client"}
{"native-protocol.version": "32"}
{"application.process.id": "1543"}
{"application.process.user": "mmucha"}
{"application.process.host": "vbDesktop"}
{"application.process.binary": "mplayer"}
{"application.language": "C"}
{"window.x11.display": ":0"}
{"application.process.machine_id": "720184179caa46f0a3ce25156642f7a0"}
{"application.process.session_id": "2"}
{"module-stream-restore.id": "sink-input-by-application-name:MPlayer"}
{"index": "145"}
{"driver": "<protocol-native.c>"}
{"flags": " "}
{"state": "RUNNING"}
{"sink": "4 <alsa_output.pci-0000_05_00.0.analog-stereo>"}
{"volume": "front-left: 24903 /  38% / -25.21 dB,   front-right: 24903 /  38% / -25.21 dB"}
{"balance": "0.00"}
{"muted": "no"}
{"current": "latency: 70.50 ms"}
{"requested": "latency: 210.00 ms"}
{"sample": "spec: float32le 2ch 48000Hz"}
{"channel": "map: front-left,front-right"}
{"resample": "method: speex-float-1"}
{"module": "13"}
{"client": "251 <MPlayer>"}
{"media.name": "Trombone Shorty At Age 13 - 2nd Line-k9YUi3UhEPQ.webm"}
{"application.name": "MPlayer"}
{"native-protocol.peer": "UNIX socket client"}
{"native-protocol.version": "32"}
{"application.process.id": "2831"}
{"application.process.user": "mmucha"}
{"application.process.host": "vbDesktop"}
{"application.process.binary": "mplayer"}
{"application.language": "C"}
{"window.x11.display": ":0"}
{"application.process.machine_id": "720184179caa46f0a3ce25156642f7a0"}
{"application.process.session_id": "2"}
{"module-stream-restore.id": "sink-input-by-application-name:MPlayer"}
bash $ 
```
2. convert the stream of JSONs into the requested output:
bash $ <inputs.txt sed -E 's/^ *([^=: ]+)[: =]+(.+)/{"\1": "\2"}/; s/""/"/g' | grep '^{' \
                          | jtc -J /\
                                -w'<application.process.id>l:' -w'<index>l:' -w'<application.name>l:' -w'<media.name>l:' -jl /\
                                -jw[:] -T'{"appname":"{$a}", "pid":{$b}, "{$C}":{$c}, "medianame":"{$d}"}'
[
   {
      "appname": "MPlayer",
      "index": 144,
      "medianame": "UNREAL! Tetris Theme on Violin and Guitar-TnDIRr9C83w.webm",
      "pid": 1543
   },
   {
      "appname": "MPlayer",
      "index": 145,
      "medianame": "Trombone Shorty At Age 13 - 2nd Line-k9YUi3UhEPQ.webm",
      "pid": 2831
   }
]
bash $ 
```bash
```


