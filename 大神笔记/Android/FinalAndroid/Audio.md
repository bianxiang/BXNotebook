[toc]


## 管理音频播放

为确保用户体验，你的应用要正确管理音频焦点（audio focus），防止多个应用同时播放音频。

### 控制应用的音量和播放

- 让用户能控制音量。
- 让播放控制按键（硬件）能控制播放，如播放、暂停、下一首等。

#### 识别使用哪个音频流

Android为以下内容提供独立的音频流：播放音乐、闹钟（alarms）、通知、来电铃声、系统声音、in-call volume, DTMF tones。这样做的目的是允许用户独立的控制每个流的音量。

这些流多数限于系统事件使用，除非你的应用是一个闹钟的替代品，否则你几乎总是使用 [音乐流](http://developer.android.com/reference/android/media/AudioManager.html#STREAM_MUSIC) 播放音乐。

#### 利用硬件声音键控制你的App的音量

默认，按下音量控制修改当前活动的（active）音频流的音量。如果你的App目前未播放任何东西，按下音量键调整的是铃声音量。

在游戏或音乐App中，按下音量键最好控制的是游戏或音乐音量，即使当前未播放。为此你必须监听音量控制键。Android提供`setVolumeControlStream()`，将音量键定向到你指定的音频流。该方法要尽量早的调用，一般在活动的`onCreate()`方法中：

```java
	setVolumeControlStream(AudioManager.STREAM_MUSIC);
```

从此刻起，只要你的活动或Fragment是可见的，按下设备的音量键影响的只是你指定的音频流。

#### 使用播放控制键（硬件）

一些设备或耳机上有播放控制按钮，如播放、暂停、停止、跳过、上一个等。当用户按下这样的键后，系统广播一个`Intent`，`action`是`ACTION_MEDIA_BUTTON`。接收者要判断按下的是哪个键，用`Intent`的`EXTRA_KEY_EVENT`键可以获取该信息。`KeyEvent`类定义了一组`KEYCODE_MEDIA_*`常量，表示可能的媒体按钮，如`KEYCODE_MEDIA_PLAY_PAUSE`和`KEYCODE_MEDIA_NEXT`。

```xml
	<receiver android:name=".RemoteControlReceiver">
	    <intent-filter>
	        <action android:name="android.intent.action.MEDIA_BUTTON" />
	    </intent-filter>
	</receiver>
```

```java
	public class RemoteControlReceiver extends BroadcastReceiver {
	    @Override
	    public void onReceive(Context context, Intent intent) {
	        if (Intent.ACTION_MEDIA_BUTTON.equals(intent.getAction())) {
	            KeyEvent event = (KeyEvent)intent.getParcelableExtra(Intent.EXTRA_KEY_EVENT);
	            if (KeyEvent.KEYCODE_MEDIA_PLAY == event.getKeyCode()) {
	                // Handle key press.
	            }
	        }
	    }
	}
```

因可能存在多个应用需要监听音频按键，你必须通过编程的方式控制应用开始和停止接收音频按键。下面的代码，使用`AudioManager`，注册或解除注册音频按键事件的接收器。注册执行后，你的广播接收器将成为排他的音频键接收器。

```java
	AudioManager am = mContext.getSystemService(Context.AUDIO_SERVICE);
	...
	// Start listening for button presses
	am.registerMediaButtonEventReceiver(RemoteControlReceiver);
	...
	// Stop listening for button presses
	am.unregisterMediaButtonEventReceiver(RemoteControlReceiver);
```

应用应在其不可见或不活跃时（如在`onStop()`回调中）解除注册。但对于音乐播放器应用，反而是在其不可见时，也最好要能够通过音频键控制播放。

更好的注册/解除注册的时机是，当你的应用获取和失去音频焦点时。见下节。

### 管理音频焦点

为避免多个音乐App同时播放，Android使用音频焦点控制，只有获取音频焦点的App才能播放。应用在播放音频前，要先获取和接收音频焦点。**它还要能监听音频焦点的丢失**。

#### 请求音频焦点

请求音频焦点通过`requestAudioFocus()`方法。若成功，方法返回`AUDIOFOCUS_REQUEST_GRANTED`。请求时，要指明使用哪个流，请求的是短暂的（transient）还是持久的（permanent）音频焦点。短暂焦点适于播放短暂的音频，如导航。持久的焦点适合音乐等。请求音频焦点应在播放开始前。

```java
	AudioManager am = mContext.getSystemService(Context.AUDIO_SERVICE);
	...
	// 请求音频焦点：音乐流、持久的焦点
	int result = am.requestAudioFocus(afChangeListener,
    	AudioManager.STREAM_MUSIC, AudioManager.AUDIOFOCUS_GAIN);
	if (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
	    am.registerMediaButtonEventReceiver(RemoteControlReceiver);
	    // Start playback.
	}
```

播放完成后记得调用`abandonAudioFocus()`，告诉系统你不再需要焦点，并解除注册关联的`AudioManager.OnAudioFocusChangeListener`。例如，放弃短暂焦点后，被打断的App可以继续播放。

```java
	am.abandonAudioFocus(afChangeListener);
```

请求短暂焦点时，还有一个附加选项：是否要启用`ducking`。若启用，则其他音频引用可以继续播放，只要他们降低音量，直到焦点返回给它们。

```java
	// Request audio focus for playback
	int result = am.requestAudioFocus(afChangeListener,
    	AudioManager.STREAM_MUSIC,
        AudioManager.AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK);
	if (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
	    // Start playback.
	}
```

Ducking特别适合需要间歇性的使用音频的应用。

当另一个应用请求音频焦点时，你之前注册的监听器会获知它选择的是哪种焦点。

#### 处理焦点丢失

If your app can request audio focus, it follows that it will in turn lose that focus when another app requests it. How your app responds to a loss of audio focus depends on the manner of that loss.

请求音频焦点时注册的、音频焦点监听器的回调方法，`onAudioFocusChange()`，接受一个参数，描述焦点的变化。

Generally speaking, a transient (temporary) loss of audio focus should result in your app silencing it’s audio stream, but otherwise maintaining the same state. You should continue to monitor changes in audio focus and be prepared to resume playback where it was paused once you’ve regained the focus.

If the audio focus loss is permanent, it’s assumed that another application is now being used to listen to audio and your app should effectively end itself. In practical terms, that means stopping playback, removing media button listeners—allowing the new audio player to exclusively handle those events—and abandoning your audio focus. At that point, you would expect a user action (pressing play in your app) to be required before you resume playing audio.

In the following code snippet, we pause the playback or our media player object if the audio loss is transient and resume it when we have regained the focus. If the loss is permanent, it unregisters our media button event receiver and stops monitoring audio focus changes.

```java
	OnAudioFocusChangeListener afChangeListener = new OnAudioFocusChangeListener() {
	    public void onAudioFocusChange(int focusChange) {
	        if (focusChange == AUDIOFOCUS_LOSS_TRANSIENT
	            // Pause playback
	        } else if (focusChange == AudioManager.AUDIOFOCUS_GAIN) {
	            // Resume playback 
	        } else if (focusChange == AudioManager.AUDIOFOCUS_LOSS) {
	            am.unregisterMediaButtonEventReceiver(RemoteControlReceiver);
	            am.abandonAudioFocus(afChangeListener);
	            // Stop playback
	        }
	    }
	};
```

In the case of a transient loss of audio focus where ducking is permitted, rather than pausing playback, you can "duck" instead.

#### Duck!

Ducking指的是降低你的音频流音量，让另一个应用的短暂音频流让你能听到。

In the following code snippet lowers the volume on our media player object when we temporarily lose focus, then returns it to its previous level when we regain focus.

```java
	OnAudioFocusChangeListener afChangeListener = new OnAudioFocusChangeListener() {
	    public void onAudioFocusChange(int focusChange) {
	        if (focusChange == AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK) {
	            // Lower the volume
	        } else if (focusChange == AudioManager.AUDIOFOCUS_GAIN) {
	            // Raise it back to normal
	        }
	    }
	};
```

A loss of audio focus is the most important broadcast to react to, but not the only one. The system broadcasts a number of intents to alert you to changes in user’s audio experience. The next lesson demonstrates how to monitor them to improve the user’s overall experience.

### 处理音频输出硬件

设备上有内置的扬声器、耳机插孔，还可能有蓝牙连接的设备。

#### 检查正在使用哪种硬件

通过`AudioManager`了解你的音频当前输出到扬声器、耳机还是其他设备：

```java
    if (isBluetoothA2dpOn()) {
        // Adjust output for Bluetooth.
    } else if (isSpeakerphoneOn()) {
        // Adjust output for Speakerphone.
    } else if (isWiredHeadsetOn()) {
        // Adjust output for headsets
    } else {
        // If audio plays and no one can hear it, is it still playing?
    }
```

#### 音频硬件改变

当耳机或蓝牙设备断开后，音频流会被自动定向到内建的扬声器。此时若之前音量很高，会让人一惊。
幸运的是系统此时会广播一个`ACTION_AUDIO_BECOMING_NOISY`。对于音乐播放器，最好暂停播放；对于游戏等，则应该降低音量。

```java
    private class NoisyAudioStreamReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            if (AudioManager.ACTION_AUDIO_BECOMING_NOISY.equals(intent.getAction())) {
                // Pause the playback
            }
        }
    }

    private IntentFilter intentFilter = new IntentFilter(AudioManager.ACTION_AUDIO_BECOMING_NOISY);

    private void startPlayback() {
        registerReceiver(myNoisyAudioStreamReceiver(), intentFilter);
    }

    private void stopPlayback() {
        unregisterReceiver(myNoisyAudioStreamReceiver);
    }
```
