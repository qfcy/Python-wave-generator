**The English documentation is shown below the Chinese version.**

本模块`wave_generator`实现了一套底层的声波生成、音频合成的算法，提供了生成正弦波、三角波、方波，生成白噪声、红噪声，音频拼接和反转，音量调节，混音，离散傅里叶变换等多种功能。  
并提供了与`wave`模块的接口，用于支持`wav`格式。程序还实现了调用`matplotlib`库显示波形以及频谱的功能。  
由于生成器能够生成音频流，程序内部使用了生成器实现。  
## 0.模块使用的生成器接口
生成器用于生成音频数据流，输出的每个值都是整数。
**生成器输出值的范围**
在位宽为8位时，输出值的取值范围为[0,256)，也就是`unsigned char`类型，和wav文件内部使用的类型相同。  
位宽为16位时，取值范围为[0,65536)，也就是`unsigned short`类型。  
不过位宽为16位时，wav文件的内部是有符号的`short`类型。由于`to_raw_data`函数已经实现了类型转换，为了简化生成器的实现，生成器依然使用`unsigned short`，这一点在`pyaudio`中写入生成器输出的值时需要注意。  

## 各个函数的说明文档

### 一. 音频生成

#### 1. `generate_sine(freq, seconds, volume, samprate, sampwidth, phase=0)`

生成一段正弦波，返回生成正弦波的生成器。

- **参数：**
  - `freq`：正弦波的频率（赫兹）。
  - `seconds`：生成的音频持续时间（秒）。
  - `volume`：音量，范围为0到1。
  - `samprate`：采样频率（Hz），通常为44100。
  - `sampwidth`：采样宽度（比特），通常为1（8位）或2（16位）。
  - `phase`：正弦波的相位，范围为0到2π。

#### 2. `generate_triangular(freq, seconds, volume, samprate, sampwidth)`

生成一段三角波，返回三角波的生成器。参数的用法和`generate_sine`相同。

#### 3. `generate_square(freq, seconds, volume, samprate, sampwidth)`

生成方波，参数的用法和`generate_sine`相同。

#### 4. `generate_white_noise(volume, seconds, samprate, sampwidth)`

生成一段白噪声。

### 二. 音频处理

#### 5. `integral(generator, volume, sampwidth)`

对`generator`音频信号进行积分，并会自动处理值到达上下限的情况。  
如果`generator`是一个白噪声，则可以生成粉红噪声。

- **参数：**
  - `generator`：一个音频生成器，作为输入源。

#### 6. `convert(generator, from_samprate, from_sampwidth, to_samprate, to_sampwidth)`

一个生成器，转换输入生成器`generator`到新的采样率和量化位数。

- **参数：**
  - `generator`：一个音频生成器，作为输入源。
  - `from_samprate`：源采样频率。
  - `from_sampwidth`：源量化位宽(1或2)。
  - `to_samprate`：目标采样频率。
  - `to_sampwidth`：目标量化位宽(1或2)。

#### 7. `adjust_volume(generator, sampwidth, volume)`

一个生成器，调整输入音频生成器`generator`的音量。

- **参数：**
  - `generator`：一个音频生成器，作为输入源。
  - `volume`：音量，范围0到1。

#### 8. `concat(*generators)`

按顺序拼接多个音频生成器的输出，返回拼接后的生成器。

- **参数：**
  - `*generators`：一个列表，包含多个音频生成器，作为输入源。

#### 9. `reverse(generator)`

反转音频信号，返回反转后的生成器。

- **参数：**
  - `generator`：一个音频生成器，作为输入源。

#### 10. `mixer(generators)`

一个生成器，混合多个音频生成器的输出，返回混合后的生成器。

- **参数：**
  - `generators`：一个列表，每个元素是一个元组，分别是输入生成器和音量振幅(0~1)。

#### 11. `wave_mixer(sounds, seconds, volume, samprate, sampwidth)`

混合多个音频波形，返回混合后的生成器。和`mixer`不同，`wave_mixer`可以指定详细的声波参数，如正弦波的频率和振幅。如：
```python
[(SINE_WAVE,450,0),
 (SINE_WAVE,850,-5),
 (SINE_WAVE,2500,-10)]
```
但`wave_mixer`只能传入`SINE_WAVE`,`TRIANGULAR_WAVE`,`SQUARE_WAVE`三种基本的声波类型，而不能传入生成器。

- **参数：**
  - `sounds`：一个列表，每个元素是一个元组，包含声波类型、频率和音量（分贝）。声波类型可以是`SINE_WAVE`,`TRIANGULAR_WAVE`和`SQUARE_WAVE`。
  - `seconds`：每个声音的持续时间（秒）。
  - `volume`：输出的音量(振幅)，范围为0~1。
  - `samprate`：输出的采样频率。
  - `sampwidth`：输出的采样宽度。

#### 12. `fourier_transform(generator, freq, samprate, sampwidth)`

在频率`freq`上执行离散傅里叶变换。

- **参数：**
  - `generator`：输入音频生成器的迭代器。
  - `freq`：目标频率。
  - `samprate`：采样频率。
  - `sampwidth`：采样宽度。

- **返回值：**
  - 该频率上的振幅和相位。

### 三. wav格式接口

#### 13. `get_wav_info(filename)`

获取wav文件的信息。

- **参数：**
  - `filename`：wav文件的路径。
  
- **返回值：**
  - 返回一个包含采样频率、采样宽度和持续时间(秒)的元组。

#### 14. `read_wav(filename)`

读取wav文件的音频数据（目前仅支持单声道）。

- **参数：**
  - `filename`：wav文件的路径。
  
- **返回值：**
  - 一个音频生成器。如果要获取wav文件的采样率等信息，可结合`get_wav_info()`函数。

#### 15. `to_raw_data(generator, sampwidth, size)`

根据量化位宽，将生成器的输出转换为原始二进制音频数据。

- **参数：**
  - `generator`：输入音频生成器的迭代器。
  - `sampwidth`：采样宽度（比特），支持1（8位）或2（16位）。
  - `size`：生成的音频数据大小（字节）。
  
- **返回值：**
  - `bytes`类型的原始音频数据，不包含wav文件头。

#### 16. `to_wav_data(raw_data, samprate, sampwidth, channels=1)`

根据二进制的原始音频数据`raw_data`，构造wav文件的二进制数据，包括wav文件头。

- **参数：**
  - `raw_data`：原始音频数据的字节表示。
  - `samprate`：采样频率（Hz）。
  - `sampwidth`：采样宽度（比特）。
  - `channels`：声道数，默认为1（单声道）。
  
- **返回值：**
  - `bytes`类型的wav文件数据，包括文件头。

### 四. 杂项

#### 17. `view_wave(generator, seconds)`

调用`matplotlib`，显示生成的声波的波形图。

- **参数：**
  - `generator`：输入音频生成器的迭代器。
  - `seconds`：音频的持续时间（秒）。

#### 18. `view_freq_spectrum(generator, samprate, sampwidth, min=10, max=10000, count=100)`

调用`matplotlib`，显示生成声波的傅里叶变换频谱，使用DFT。

- **参数：**
  - `generator`：输入音频生成器的迭代器。
  - `samprate`：采样频率（Hz）。
  - `sampwidth`：采样宽度（比特）。
  - `min`：频率范围的最小值，默认为10Hz。
  - `max`：频率范围的最大值，默认为10000Hz。
  - `count`：用于频谱分析的采样数，默认为100。

#### 19. `Beep(frequency, duration, type=SINE_WAVE, use_cache=True)`

发出一段蜂鸣声音，用于替代`winsound`模块的`Beep`函数，以及Windows的`Beep`这个API。

- **参数：**
  - `frequency`：声音的频率（赫兹）。
  - `duration`：声音持续时间（毫秒）。
  - `type`：声音波形类型，支持正弦波（`SINE_WAVE`）、三角波（`TRIANGULAR_WAVE`）和方波（`SQUARE_WAVE`），默认为 `SINE_WAVE`。
  - `use_cache`：是否使用之前调用`Beep`生成音频数据的缓存，默认为 `True`，以提高性能。

## 使用示例
这是播放基本的声波如正弦波、三角波、方波的示例：
```python
Beep(220.5,1800,type=SINE_WAVE)
Beep(220.5,1800,type=TRIANGULAR_WAVE)
Beep(220.5,1800,type=SQUARE_WAVE)
```
关于如何调用`generate_sine`这些具体生成器，以及如何控制音量、采样频率和量化位数，具体可看源代码中的`Beep`函数。

这是生成并播放白噪声的示例：
```python
def test_white_noise(): # 生成白噪声
    # 设置采样率等参数
    samprate = 44100 # 采样频率
    sampwidth = 2 # 量化位宽(字节)，现仅支持1或2
    seconds = 1.8 # 音频长度(秒)
    volume_db = -40 # 音量(db)。分贝数每降低20，振幅减小10倍，0为最大音量
    volume = 1 * 10**(volume_db/20) # 计算实际音量 (振幅)
    size = int(samprate * seconds * sampwidth)
    gen=generate_white_noise(volume,seconds,samprate,sampwidth) # 调用生成器
    data=to_raw_data(gen,sampwidth,size) # 将生成器的输出，转换为音频的原始二进制数据，为bytes类型
    wav_data=to_wav_data(data,samprate,sampwidth) # 转换为wav文件的数据，为bytes类型
    with open("white_noise.wav","wb") as f:
        f.write(wav_data) # 保存为wav文件
    PlaySound(wav_data,SND_MEMORY) # 播放

def test_red_noise(): # 生成红噪声，类似于风的声音
    samprate = 44100;sampwidth = 2
    seconds = 1.8
    volume_db = -20
    volume = 1 * 10**(volume_db/20)
    size = int(samprate * seconds * sampwidth)
    white=generate_white_noise(volume,seconds,samprate,sampwidth)
    gen=list(integral(white,volume,sampwidth))
    view_freq_spectrum(gen,samprate,sampwidth) # 调用matplotlib，显示音频的频谱
    data=to_raw_data(gen,sampwidth,size)
    wav_data=to_wav_data(data,samprate,sampwidth)
    PlaySound(wav_data,SND_MEMORY)

test_white_noise()
test_red_noise()
```
以及读取wav文件、转换wav文件格式并播放的示例：
```python
def test_convert(filename="test.wav"):
    samprate = 11025;sampwidth = 1
    rate,width,seconds=get_wav_info(filename) # 获取wav文件的采样率等信息
    size = int(samprate * seconds * sampwidth)
    wav=read_wav(filename) # 输出wav文件音频的生成器
    converted=convert(wav,rate,width,samprate,sampwidth) # 转换到新的采样率和量化位数
    sound=adjust_volume(converted,sampwidth,0.3) # 调节音量(振幅)为原先的0.3倍
    data=to_raw_data(sound,sampwidth,size)
    wav_data=to_wav_data(data,samprate,sampwidth)
    PlaySound(wav_data,SND_MEMORY)

test_convert()
```
更复杂的示例，通过多个正弦波的干涉，实现了简单的乐器音色：
```python
def _instrument_sound(freq,decline=5): # 返回指定频率的乐器音色，decline为衰减的分贝数
    return [(SINE_WAVE,freq,0),
            (SINE_WAVE,freq*2,-decline),
            (SINE_WAVE,freq*3,-decline*2),
            (SINE_WAVE,freq*4,-decline*3)]
_sound2 = [(SINE_WAVE,450,0), # 混合多个正弦波
           (SINE_WAVE,850,-5),
           (SINE_WAVE,2500,-10)]

def test_mixed_sound(sounds):
    samprate = 44100;sampwidth = 2
    seconds = 1.8;volume=0.2
    size = int(samprate * seconds * sampwidth)
    gen=wave_mixer(sounds,seconds,volume,samprate,sampwidth) # 调用混音器
    data=to_raw_data(gen,sampwidth,size)
    wav_data=to_wav_data(data,samprate,sampwidth)
    PlaySound(wav_data,SND_MEMORY)

test_mixed_sound(_instrument_sound(220))
test_mixed_sound(_sound2)
```


This module `wave_generator` implements a set of low-level algorithms for sound wave generation and audio synthesis, providing functions to generate sine waves, triangular waves, square waves, white noise, red noise, audio concatenation and reversal, volume adjustment, mixing, and discrete Fourier transforms.

It also provides an interface with the `wave` module to support the `wav` format. The program implements features to display waveforms and spectra using the `matplotlib` library.

As the generator can produce audio streams, the program internally uses generators for implementation.
## 0. Generator Interface Used by the Module
Generators are used to produce audio data streams, with each output value being an integer.
**Range of Generator Output Values:**
When the bit width is 8 bits, the range of output values is [0,256), which is the same as the `unsigned char` type used internally in wav files.  
When the bit width is 16 bits, the range is [0,65536), which is the `unsigned short` type.  
However, when the bit width is 16 bits, the internal representation in wav files is a signed `short` type. Since the `to_raw_data` function has already implemented type conversion, in order to simplify the implementation of the generator, the generator still uses `unsigned short`. This needs to be noted when writing the values output by the generator to `pyaudio`.

## Documentation of Each Function

### I. Audio Generation

#### 1. `generate_sine(freq, seconds, volume, samprate, sampwidth, phase=0)`
Generates a segment of a sine wave, returning a generator for the sine wave.
- **Parameters:**
  - `freq`: Frequency of the sine wave (Hertz).
  - `seconds`: Duration of the generated audio (seconds).
  - `volume`: Volume, ranging from 0 to 1.
  - `samprate`: Sampling frequency (Hz), usually 44100.
  - `sampwidth`: Sampling width (bits), usually 1 (8 bits) or 2 (16 bits).
  - `phase`: Phase of the sine wave, ranging from 0 to 2π.

#### 2. `generate_triangular(freq, seconds, volume, samprate, sampwidth)`
Generates a segment of a triangular wave, returning a generator for the triangular wave. The parameters are used the same way as in `generate_sine`.

#### 3. `generate_square(freq, seconds, volume, samprate, sampwidth)`
Generates a square wave, with parameters used the same way as in `generate_sine`.

#### 4. `generate_white_noise(volume, seconds, samprate, sampwidth)`
Generates a segment of white noise.

### II. Audio Processing

#### 5. `integral(generator, volume, sampwidth)`
Integrates the audio signal from `generator`, automatically handling situations where the values reach limits. If `generator` is white noise, it can generate pink noise.
- **Parameters:**
  - `generator`: An audio generator as input source.

#### 6. `convert(generator, from_samprate, from_sampwidth, to_samprate, to_sampwidth)`
Converts the input generator `generator` to new sampling rate and bit width.
- **Parameters:**
  - `generator`: An audio generator as input source.
  - `from_samprate`: Source sampling rate.
  - `from_sampwidth`: Source bit width (1 or 2).
  - `to_samprate`: Target sampling rate.
  - `to_sampwidth`: Target bit width (1 or 2).

#### 7. `adjust_volume(generator, sampwidth, volume)`
Adjusts the volume of the input audio generator `generator`.
- **Parameters:**
  - `generator`: An audio generator as input source.
  - `volume`: Volume, ranging from 0 to 1.

#### 8. `concat(*generators)`
Concatenates the outputs of multiple audio generators in order, returning the concatenated generator.
- **Parameters:**
  - `*generators`: A list of multiple audio generators as input source.

#### 9. `reverse(generator)`
Reverses the audio signal, returning the reversed generator.
- **Parameters:**
  - `generator`: An audio generator as input source.

#### 10. `mixer(generators)`
Mixes outputs from multiple audio generators, returning the mixed generator.
- **Parameters:**
  - `generators`: A list, each element being a tuple containing the input generator and volume amplitude (0~1).

#### 11. `wave_mixer(sounds, seconds, volume, samprate, sampwidth)`
Mixes multiple audio waveforms, returning the mixed generator. Unlike `mixer`, `wave_mixer` allows specifying detailed wave parameters, such as frequency and amplitude of sine waves. For example:
```python
[(SINE_WAVE,450,0),
 (SINE_WAVE,850,-5),
 (SINE_WAVE,2500,-10)]
```
However, `wave_mixer` can only accept basic wave types: `SINE_WAVE`, `TRIANGULAR_WAVE`, `SQUARE_WAVE`, and cannot accept generators.

- **Parameters:**
  - `sounds`: A list, each element being a tuple containing wave type, frequency, and volume (in decibels). Wave types can be `SINE_WAVE`, `TRIANGULAR_WAVE`, and `SQUARE_WAVE`.
  - `seconds`: Duration (seconds) for each sound.
  - `volume`: Output volume (amplitude), ranging from 0 to 1.
  - `samprate`: Output sampling frequency.
  - `sampwidth`: Output sampling width.

#### 12. `fourier_transform(generator, freq, samprate, sampwidth)`
Performs discrete Fourier transform at frequency `freq`.
- **Parameters:**
  - `generator`: Iterator of the input audio generator.
  - `freq`: Target frequency.
  - `samprate`: Sampling frequency.
  - `sampwidth`: Sampling width.
- **Return Value:**
  - Amplitude and phase at that frequency.

### III. wav Format Interface

#### 13. `get_wav_info(filename)`
Gets information about a wav file.
- **Parameters:**
  - `filename`: Path to the wav file.
- **Return Value:**
  - A tuple containing sampling frequency, sampling width, and duration (seconds).

#### 14. `read_wav(filename)`
Reads audio data from a wav file (currently only supports mono).
- **Parameters:**
  - `filename`: Path to the wav file.
- **Return Value:**
  - An audio generator. To get information such as sampling rate from the wav file, it can be combined with the `get_wav_info()` function.

#### 15. `to_raw_data(generator, sampwidth, size)`
Converts the output of the generator to raw binary audio data based on quantization bit width.
- **Parameters:**
  - `generator`: Iterator of the input audio generator.
  - `sampwidth`: Sampling width (bits), supports 1 (8 bits) or 2 (16 bits).
  - `size`: Size of the generated audio data (bytes).
- **Return Value:**
  - Raw audio data of type `bytes`, without wav file header.

#### 16. `to_wav_data(raw_data, samprate, sampwidth, channels=1)`
Constructs binary data for a wav file based on the raw binary audio data `raw_data`, including the wav file header.
- **Parameters:**
  - `raw_data`: Byte representation of the raw audio data.
  - `samprate`: Sampling frequency (Hz).
  - `sampwidth`: Sampling width (bits).
  - `channels`: Number of channels, default is 1 (mono).
- **Return Value:**
  - Bytes type wav file data, including file header.

### IV. Miscellaneous

#### 17. `view_wave(generator, seconds)`
Calls `matplotlib` to display the waveform of the generated sound wave.
- **Parameters:**
  - `generator`: Iterator of the input audio generator.
  - `seconds`: Duration of the audio (seconds).

#### 18. `view_freq_spectrum(generator, samprate, sampwidth, min=10, max=10000, count=100)`
Calls `matplotlib` to display the Fourier transform spectrum of the generated sound wave, using DFT.
- **Parameters:**
  - `generator`: Iterator of the input audio generator.
  - `samprate`: Sampling frequency (Hz).
  - `sampwidth`: Sampling width (bits).
  - `min`: Minimum value of frequency range, default is 10 Hz.
  - `max`: Maximum value of frequency range, default is 10000 Hz.
  - `count`: Number of samples used for spectrum analysis, default is 100.

#### 19. `Beep(frequency, duration, type=SINE_WAVE, use_cache=True)`
Issues a beep sound, replacing the `Beep` function of the `winsound` module and the `Beep` API in Windows.
- **Parameters:**
  - `frequency`: Frequency of the sound (Hertz).
  - `duration`: Duration of the sound (milliseconds).
  - `type`: Type of the sound waveform, supports sine wave (`SINE_WAVE`), triangular wave (`TRIANGULAR_WAVE`), and square wave (`SQUARE_WAVE`), default is `SINE_WAVE`.
  - `use_cache`: Whether to use cached audio data generated by previous calls to `Beep`, default is `True`, for performance improvement.

## Usage Examples
Here are examples for playing basic sound waves like sine wave, triangular wave, square wave:
```python
Beep(220.5,1800,type=SINE_WAVE)
Beep(220.5,1800,type=TRIANGULAR_WAVE)
Beep(220.5,1800,type=SQUARE_WAVE)
```
For how to invoke specific generators like `generate_sine` and to control volume, sampling frequency, and quantization bit width, refer to the `Beep` function in the source code.

Here is an example for generating and playing white noise:
```python
def test_white_noise(): # Generate white noise
    # Set the sampling rate and other parameters
    samprate = 44100 # Sampling frequency
    sampwidth = 2 # Quantization bit width (bytes), currently only supports 1 or 2
    seconds = 1.8 # Audio length (seconds)
    volume_db = -40 # Volume (dB). Each decrease of 20 dB shrinks the amplitude by a factor of 10, 0 is maximum volume
    volume = 1 * 10**(volume_db/20) # Calculate actual volume (amplitude)
    size = int(samprate * seconds * sampwidth)
    gen=generate_white_noise(volume,seconds,samprate,sampwidth) # Call generator
    data=to_raw_data(gen,sampwidth,size) # Convert generator output to raw binary audio data of bytes type
    wav_data=to_wav_data(data,samprate,sampwidth) # Convert to wav file data of bytes type
    with open("white_noise.wav","wb") as f:
        f.write(wav_data) # Save as wav file
    PlaySound(wav_data,SND_MEMORY) # Play
    
def test_red_noise(): # Generate red noise, similar to wind sound
    samprate = 44100;sampwidth = 2
    seconds = 1.8
    volume_db = -20
    volume = 1 * 10**(volume_db/20)
    size = int(samprate * seconds * sampwidth)
    white=generate_white_noise(volume,seconds,samprate,sampwidth)
    gen=list(integral(white,volume,sampwidth))
    view_freq_spectrum(gen,samprate,sampwidth) # Call matplotlib to display audio spectrum
    data=to_raw_data(gen,sampwidth,size)
    wav_data=to_wav_data(data,samprate,sampwidth)
    PlaySound(wav_data,SND_MEMORY)

test_white_noise()
test_red_noise()
```
Additionally, here’s an example of reading a wav file, converting its format, and playing it:
```python
def test_convert(filename="test.wav"):
    samprate = 11025;sampwidth = 1
    rate,width,seconds=get_wav_info(filename) # Get wav file sampling rate and other info
    size = int(samprate * seconds * sampwidth)
    wav=read_wav(filename) # Output wav file audio generator
    converted=convert(wav,rate,width,samprate,sampwidth) # Convert to new sampling rate and quantization width
    sound=adjust_volume(converted,sampwidth,0.3) # Adjust volume (amplitude) to 0.3 times the original
    data=to_raw_data(sound,sampwidth,size)
    wav_data=to_wav_data(data,samprate,sampwidth)
    PlaySound(wav_data,SND_MEMORY)

test_convert()
```
More complex examples implement a simple instrument tone by the interference of multiple sine waves:
```python
def _instrument_sound(freq,decline=5): # Return instrument tone at specified frequency, decline is the amount of decay in decibels
    return [(SINE_WAVE,freq,0),
            (SINE_WAVE,freq*2,-decline),
            (SINE_WAVE,freq*3,-decline*2),
            (SINE_WAVE,freq*4,-decline*3)]
_sound2 = [(SINE_WAVE,450,0), # Mix multiple sine waves
           (SINE_WAVE,850,-5),
           (SINE_WAVE,2500,-10)]

def test_mixed_sound(sounds):
    samprate = 44100;sampwidth = 2
    seconds = 1.8;volume=0.2
    size = int(samprate * seconds * sampwidth)
    gen=wave_mixer(sounds,seconds,volume,samprate,sampwidth) # Call mixer
    data=to_raw_data(gen,sampwidth,size)
    wav_data=to_wav_data(data,samprate,sampwidth)
    PlaySound(wav_data,SND_MEMORY)

test_mixed_sound(_instrument_sound(220))
test_mixed_sound(_sound2)
```
