## Signal-Analyzis-Project

# Description

This project is designed to anyles independent component analysis (ICA) of audio signals. The main purpose of this code is to identify dominant frequencies in an audio signal using ICA analysis. Only problem is we can only analyze short audio files. Longer audio may have inaccurate output due to its length. In addition, complex musical tracks can be inaccurate compared to single instrument samples.

# Wav informations

Please note that we only use files with the extension .wav. 

Thanks to my friend, nicknamed MAX.IM (https://www.twitch.tv/maxim_land), I am in possession of two of his, self-recorded samples played on guitar - "melody.wav" and "melody2.wav".

This can also be tested on the file "tdh.wav", which is the song "I am machine" by Three Days Grace (https://www.youtube.com/watch?v=8Zx6RXGNISk)

You can also use your own samples or music, as long as they have a .wav extension

# Libraries

    numpy
    sklearn
    librosa (https://librosa.org/doc/latest/index.html)
    matplotlib
    pandas

# How it works?

Make note that im ignoring LIBRARIES cell

# 1st cell part 1 (Caution : this cell can take a long time to execute)

1) Load the audio file you want to analyze using the librosa.load function. It returns the audio signal as a one dimensional numpy arrayand sample rate

2) Calculate the spectrogram of the audio signal using librosa.stft function and convert it to a decibel scale. Add a small offset to avoid divide-by-zero errors.

A spectrogram is a frequency representation of a signal over time. It uses the short-time Fourier transform (STFT) to obtain detailed information about the time frequency distribution of the signal's energy.

The STFT of a signal x(t) is calculated using the following formula:

$$
X(t, f) = \int_{-\infty}^{\infty} x(\tau) w(t - \tau) e^{-j2\pi f \tau} d\tau
$$

Where:

    X(t, f) is the STFT result,
    x(tau) is the audio signal,
    w(t - tau) is the window function,
    t is the time,
    f is the frequency.

The decibel scale is a logarithmic scale used to describe quantities. In the context of sound, decibels are used to quantify the loudness of a sound.

In this code, the spectrogram is converted to the decibel scale using the following formula:

$$
L = 20 \log_{10}\left(\frac{A}{A_{0}}\right)
$$

Where:

    L is the value in decibels,
    A is the current amplitude value of the signal,
    A_{0} is the reference value of the amplitude.

Of course, we use built-in methods, and as a result we do not have to worry about the exact mapping of the formulas, but it is good to know what we use.

3) Applying the ICA algorithm to the spectrogram using the FastICA class from the sklearn library. Virtualize original spectrogram using librosa.display.specshow , also display 9 independent components.

A signal spectrum shows which frequency components are included in a given signal. A decibel spectrum shows the same information, but the scale of amplitudes is logarithmic, which is often clearer to the human ear, which perceives loudness in a logarithmic manner.

```
plt.figure(figsize=(10, 7))
plt.subplot(2, 1, 1)
librosa.display.specshow(D_db, sr=sr, x_axis='time', y_axis='linear')
plt.colorbar(format='%+2.0f dB')
plt.title('Spektrum oryginalne')
plt.show()
```

This produces a decibel spectrum graph of the original audio signal. On the X-axis we have the time, on the Y-axis the frequency, and the colour corresponds to the amplitude of a given frequency component at a given moment in time.

Then, after conducting an ICA and determining the components that are independent.

```
for i, s in enumerate(S_):
    if i >= 9: 
        break
    plt.subplot(3, 3, i + 1)
    plt.plot(s)
    plt.title('ICA component {}'.format(i+1))
```

This displays graphs of these components in the time domain. For each component, its behaviour over time is shown, allowing you to understand how the individual components affect the signal.

# 1nd cell part 2
This code snippet analyses the sound file by segmenting it and then performing ICA for each segment. In addition, for each ICA component, it calculates the dominant frequency.

Steps:
1) Segment extraction.
2) Calculation of the spectrogram.
3) Transformation of the spectrogram to decibel scale.
4) Execution of the ICA and transposition of the results.
5) For each ICA component (up to 9 components), calculation of FFT and identification of the dominant frequency.
6) Adding the results to the data frame.


Each line in the resulting data frame represents an ICA component for that segment, indicating the dominant frequency of that component. This allows analysis of how the different signal components (which are independent) affect the overall signal in a given time segment.

!! CAUTION !! 

Dominant frequency results may contain values of 0.0 or negative due to the characteristics of the Fourier transform.

A value of 0.0 is usually associated with the DC component of the signal. This means that it is a frequency that does not oscillate. This can depend on the nature of the signal being analysed. In the context of sound, the 0 Hz component is related to the mean value of the signal.

Negative frequencies appear as a result of Hermitian symmetry resulting from the Fourier transform for real signals. In short, for real input signals, the Fourier transform produces a symmetrical result, with half of the signal energy concentrated on positive frequencies and the other half on negative frequencies. In practice, for real signals, we often only analyse half of the results (usually for positive frequencies), because the other half is a mirror image.

In the context of sound, we tend to ignore negative frequencies because they have no direct physical meaning - we cannot hear 'negative' sounds. If you see negative frequencies in your results, they can probably be ignored or the results can be processed to include only positive frequencies.

# 2nd cell

In this code snippet, I perform a procedure to separate harmonics from noise, and then perform ICA on the resulting signal.

I use the librosa.effects.harmonic function, which separates harmonics from noise. The harmonics in an audio signal are those which are integer multiples of the fundamental frequency of the sound.
Specifically, if we have a sound with a fundamental frequency f, the harmonics are an integral part of the perception of the sound, giving it a characteristic 'color' or 'hue'.

The librosa.effects.harmonic function works by filtering the input signal using the local median. This is a type of non-linear filtering that is effective in eliminating short-lived noise, leaving longer-lived harmonics.

Please note that I am creating a new FastICA , in order to apply de-noising. In this case we are using it to identify and isolate harmonics. I then reconstruct the signal by performing a matrix multiplication between the resulting ICA components and the mixing matrix, and then summing the result along one of the axes. The result is the reconstructed signal.

The following is similar to cell 1 part 1. The difference is i am saving reconstructed signal to wav file.

It is worth noting that we will not fully remove the reduction due to the quality of the recording I tested on. It is not perfect, but the vast majority of noise is removed.


# 3rd cell

This code snippet is intended to separate the various instruments present in a sound file.

Process procedure:

1) Load a sound file and calculate its spectrum using the STFT.
2) Perform ICA on this spectrum. We assume that there are 4 instruments in our sound file, so we set n_components=4 (for tdh.wav).
3) Perform ICA on the transpose of the spectrum, adjusting for the time dimension.
4) For each instrument,  transform the resulting components back to the original spectral space, display the spectrum, and then reconstruct the audio signal using an inverse STFT transformation.
5) Scale the signal to the 16-bit int range and save the result to a wav file.

Why it will not work?

This is related to how music editing and digital music processing itself is done these days. This is a very complicated process, one of whose tasks is to equalise the frequencies and volumes of the individual instruments.

In order to perform the instrument retrieval process correctly, you would probably need to rely on a neural network and have a lot more knowledge about music editing itself in music studios. Presumably, you would also need some data about the specific song.

If you really want to check the output, I RECOMMEND turning down the volume! You do so at your own risk! You have been warned.