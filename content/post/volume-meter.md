---
title:      "A recording app with a volume level meter"
date: 2018-05-01T01:37:56+08:00
lastmod: 2017-05-01T01:37:56+08:00
author:     "Adrian"
tags: ["c#", "winform"]
categories: ["Coding"]
draft: false

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---

Prerequisites: Install NAudio, an open-source .NET library written by Mark Heath. After that, see [this tutorial](https://github.com/naudio/NAudio/blob/master/Docs/RecordWavFileWinFormsWaveIn.md), figure it out and implement the code. Our code will be based on that. 

Now you have a WinForm app where you can record audio to a WAV file. Now we want to add a feature to allow us to test the mic before we start recording. We'll add a real-time volume level meter to our app.

First let's define a `isRecording` flag right after the `closing` flag.

```c#
bool closing = false;
bool isRecording = false;
```

Now let's add a `Test mic` button, and a progress bar which serves as our volume meter, right after the existing buttons.

```c#
var buttonTest = new Button() { Text = "Test mic", Top = buttonStop.Bottom };
var volume = new ProgressBar(){ Top = buttonTest.Bottom, Size = new Size(800,20) };
```

Then we'll modify some event handlers. When we click `Test mic` button, we want the recording starts but we don't want the audio to be saved.  After we click `Record` button, we set `isRecording` to be true.

```c#
buttonTest.Click += (s, a) => { waveIn.StartRecording();};
buttonRecord.Click += (s, a) =>
{
	writer = new WaveFileWriter(outputFilePath, waveIn.WaveFormat);
	isRecording = true;
	buttonRecord.Enabled = false;
	buttonStop.Enabled = true;
};
```

 The recorded audio samples are represented as a byte array. Each sample is 16bit integer, so we need to convert every two array items into a sample. We calculate a max value in every buffer to represent the volume value in that period of time.

```c#
waveIn.DataAvailable += (s, a) =>
{
	if (isRecording)
	{
		writer.Write(a.Buffer, 0, a.BytesRecorded);
		// Limit the length to 30s.
		if (writer.Position > waveIn.WaveFormat.AverageBytesPerSecond * 30)
		{
         	waveIn.StopRecording();
         }
     }
	float max = 0;
	for (int index = 0; index < a.BytesRecorded; index += 2)
	{
		short sample = (short)((a.Buffer[index + 1] << 8) | a.Buffer[index + 0]);
		// to floating point
		var sample32 = sample / 32768f;
		// absolute value 
		if (sample32 < 0) sample32 = -sample32;
		// is this the max value?
		if (sample32 > max) max = sample32;
     }
	volume.Value = (int) (100*max);
};         
```
This is what our simple app looks like:

![](/img/in-post/post-recorder/Recorder.JPG)

The full program code:

```c#
using System;
using System.Drawing;
using System.IO;
using System.Windows.Forms;
using NAudio.Wave;

namespace Recorder
{
    static class Program
    {
        static void Main()
        {
            // Set output path. Save recording to desktop/recorded.wav
            var outputFolder = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.Desktop), "NAudio");
            Directory.CreateDirectory(outputFolder);
            var outputFilePath = Path.Combine(outputFolder, "recorded.mp3");
            var waveIn = new WaveInEvent();
            WaveFileWriter writer = null;
            bool closing = false;
            bool isRecording = false;

            var f = new Form();
            var buttonRecord = new Button(){Text = "Record"};
            var buttonStop = new Button(){Text = "Stop", Left = buttonRecord.Right, Enabled = false};
            
            var volume = new ProgressBar(){ Top = buttonStop.Bottom, Size = new Size(800,20) };
            var buttonTest = new Button() { Text = "Test mic", Top = volume.Bottom };
            f.Controls.AddRange(new Control[]{buttonRecord, buttonStop, volume, buttonTest});

            buttonRecord.Click += (s, a) =>
            {
                writer = new WaveFileWriter(outputFilePath, waveIn.WaveFormat);
                isRecording = true;
                buttonRecord.Enabled = false;
                buttonStop.Enabled = true;
            };
            buttonTest.Click += (s, a) =>
            {               
                waveIn.StartRecording();
            };
            buttonStop.Click += (s, a) => waveIn.StopRecording();

            waveIn.DataAvailable += (s, a) =>
            {
                if (isRecording)
                {
                    writer.Write(a.Buffer, 0, a.BytesRecorded);
                    // Limit the length to 30s.
                    if (writer.Position > waveIn.WaveFormat.AverageBytesPerSecond * 30)
                    {
                        waveIn.StopRecording();
                    }
                }
                float max = 0;
                for (int index = 0; index < a.BytesRecorded; index += 2)
                {
                    short sample = (short)((a.Buffer[index + 1] << 8) | a.Buffer[index + 0]);
                    // to floating point
                    var sample32 = sample / 32768f;
                    // absolute value 
                    if (sample32 < 0) sample32 = -sample32;
                    if (sample32 > max) max = sample32;
                }

                volume.Value = (int) (100*max);
            };
            // RecordingStopped event handler. Disposes writer
            waveIn.RecordingStopped += (s, a) =>
            {
                writer?.Dispose();
                writer = null;
                buttonRecord.Enabled = true;
                buttonStop.Enabled = false;
                if (closing)
                    waveIn.Dispose();
            };
            // Stop recording when form is closed
            f.FormClosing += (s, a) =>
            {
                closing = true;
                waveIn.StopRecording();
            };
            f.ShowDialog();
        }
    }
}

```





