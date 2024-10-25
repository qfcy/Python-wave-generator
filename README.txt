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