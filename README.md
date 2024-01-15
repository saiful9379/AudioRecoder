# AudioRecoder
Recording Audio file using pyaudio module

# Requirements

```
pip install pyaudio==0.2.13
```

# CodeBase

Run the code and it will enable for multiple recoding file save into ```logs``` folder as ```.wav``` format audio.

For Interactive Testing Check the jupyter ![notebook](audio_recoder.ipynb)

```py

import os
import pyaudio
import wave
import ipywidgets as widgets
from IPython.display import display, Audio

class AudioRecorder:
    def __init__(self, file_path='recorded_audio.wav', channels=2, sample_rate=44100, chunk=1024):
        self.file_path = file_path
        self.file_dir = "logs"
        self.channels = channels
        self.increment = 0
        self.sample_rate = sample_rate
        self.chunk = chunk
        self.frames = []
        self.recording = False
        self.p = pyaudio.PyAudio()

        self.record_button = widgets.Button(description="Start Recording")
        self.stop_button = widgets.Button(description="Stop Recording")
        
        self.record_button.on_click(self.start_recording)
        self.stop_button.on_click(self.stop_recording)

    def callback(self, in_data, frame_count, time_info, status):
        if self.recording:
            self.frames.append(in_data)
        return None, pyaudio.paContinue

    def start_recording(self, _):
        self.frames = []
        self.recording = True
        self.record_button.disabled = True
        self.stop_button.disabled = False
        os.makedirs(self.file_dir, exist_ok=True)
#         print(self.p.get_default_input_device_info())

        self.stream = self.p.open(format=pyaudio.paInt16,
                channels=self.channels,
                rate=self.sample_rate,
                input=True,
                frames_per_buffer=self.chunk,
                stream_callback=self.callback)

        print("Recording...")
        self.stream.start_stream()

    def stop_recording(self, _):
        self.recording = False
        self.record_button.disabled = False
        self.stop_button.disabled = True
        self.stream.stop_stream()
        self.increment+=1
        self.stream.close()
        self.p.terminate()
        self.p = pyaudio.PyAudio()
        
        path_dir = os.path.join(self.file_dir,  self.file_path[:-4]+"_"+str(self.increment)+".wav")
        print("save file path : ", path_dir)

        with wave.open(path_dir, 'wb') as wf:
            wf.setnchannels(self.channels)
            wf.setsampwidth(self.p.get_sample_size(pyaudio.paInt16))
            wf.setframerate(self.sample_rate)
            wf.writeframes(b''.join(self.frames))

        print(f"Audio recorded and saved to {self.file_path}")
        display(Audio(filename=self.file_path))

recorder = AudioRecorder()
widgets.VBox([recorder.record_button, recorder.stop_button])

```

# Reference
1. https://pypi.org/project/PyAudio/
