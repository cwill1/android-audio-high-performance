---
layout: page
title: OpenSL ES for Android
permalink: /guides/opensl_es.html
site_nav_category_order: 145
is_site_nav_category2: true
site_nav_category: guides
---
{::options toc_levels="2"/}

* TOC
{:toc}

<p>
This article describes the Android native audio APIs based on the
Khronos Group OpenSL ES&#8482; 1.0.1 standard.
</p>
<p>
Unless otherwise noted,
all features are available at Android API level 9 (Android platform
version 2.3) and higher.
Some features are only available at Android API level 14 (Android
platform version 4.0) and higher; these are noted.
</p>
<p>
OpenSL ES provides a C language interface that is also callable from C++. It
exposes features similar to the audio portions of these Android Java APIs:
</p>
<ul>
<li><a href="http://developer.android.com/reference/android/media/MediaPlayer.html">
android.media.MediaPlayer</a>
</li>
<li><a href="http://developer.android.com/reference/android/media/MediaRecorder.html">
android.media.MediaRecorder</a>
</li>
</ul>

<p>
As with all of the Android Native Development Kit (NDK), the primary
purpose of OpenSL ES for Android is to facilitate the implementation
of shared libraries to be called using the Java Native
Interface (JNI).  NDK is not intended for writing pure C/C++
applications.  That said, OpenSL ES is a full-featured API, and we
expect that you should be able to accomplish most of your audio
needs using only this API, without up-calls to code running in the Android runtime.
</p>

<p>
Note: though based on OpenSL ES, the Android native audio API
is <i>not</i> a conforming implementation of any OpenSL ES 1.0.1
profile (game, music, or phone). This is because Android does not
implement all of the features required by any one of the profiles.
Any known cases where Android behaves differently than the specification
are described in section "Android extensions" below.
</p>

## Getting started ##

### Example code

<h4 id="recommended">Recommended</h4>

<p>
Supported and tested example code, usable as a model
for your own code, is located in NDK folder
<code>platforms/android-9/samples/native-audio/</code> and the
<a href="https://github.com/googlesamples/android-ndk/tree/master/audio-echo">audio-echo</a>
and
<a href="https://github.com/googlesamples/android-ndk/tree/master/native-audio">native-audio</a>
folders of the repository
<a href="https://github.com/googlesamples/android-ndk">https://github.com/googlesamples/android-ndk</a>.
</p>

<h4 id="notRecommended">Not recommended</h4>

<p>
The OpenSL ES 1.0.1 specification contains example code in the
appendices (see section "References" below for the link to this
specification).  However, the examples in Appendix B: Sample Code
and Appendix C: Use Case Sample Code use features
not supported by Android. Some examples also contain
typographical errors, or use APIs that are likely to change.
Proceed with caution in referring to these;
though the code may be helpful in understanding the full OpenSL ES
standard, it should not be used as is with Android.
</p>

### Adding OpenSL ES to your application source code

<p>
OpenSL ES is a C API, but is callable from both C and C++ code.
</p>
<p>
At a minimum, add the following line to your code:
</p>
<pre>
#include &lt;SLES/OpenSLES.h&gt;
</pre>

<p>
If you use Android extensions, also include this header:
</p>
<pre>
#include &lt;SLES/OpenSLES_Android.h&gt;
</pre>
<p>
which automatically includes these headers as well (you don't need to
include these, they are shown as an aid in learning the API):
</p>
<pre>
#include &lt;SLES/OpenSLES_AndroidConfiguration.h&gt;
#include &lt;SLES/OpenSLES_AndroidMetadata.h&gt;
</pre>

### Makefile

<p>
Modify your Android.mk as follows:
</p>
<pre>
LOCAL_LDLIBS += -lOpenSLES
</pre>

### Audio content

<p>
There are many ways to package audio content for your
application, including:
</p>

<dl>

<dt>Resources</dt>
<dd>
By placing your audio files into the <code>res/raw/</code> folder,
they can be accessed easily by the associated APIs for
<a href="http://developer.android.com/reference/android/content/res/Resources.html">
Resources</a>.  However there is no direct native access to resources,
so you will need to write Java programming language code to copy them out before use.
</dd>

<dt>Assets</dt>
<dd>
By placing your audio files into the <code>assets/</code> folder,
they will be directly accessible by the Android native asset manager
APIs.  See the header files <code>android/asset_manager.h</code>
and <code>android/asset_manager_jni.h</code> for more information
on these APIs.  The example code
located in NDK folder
<code>platforms/android-9/samples/native-audio/</code> uses these
native asset manager APIs in conjunction with the Android file
descriptor data locator.
</dd>

<dt>Network</dt>
<dd>
You can use the URI data locator to play audio content directly from the
network. However, be sure to read section "Security and permissions" below.
</dd>

<dt>Local filesystem</dt>
<dd>
The URI data locator supports the <code>file:</code> scheme for local files,
provided the files are accessible by the application.
Note that the Android security framework restricts file access via
the Linux user ID and group ID mechanism.
</dd>

<dt>Recorded</dt>
<dd>Your application can record audio data from the microphone input,
store this content, and then play it back later.
The example code uses this method for the "Playback" clip.
</dd>

<dt>Compiled and linked inline</dt>
<dd>
You can link your audio content directly into the shared library,
and then play it using an audio player with buffer queue data locator.  This is most
suitable for short PCM format clips.  The example code uses this
technique for the "Hello" and "Android" clips. The PCM data was
converted to hex strings using a <code>bin2c</code> tool (not supplied).
</dd>

<dt>Real-time synthesis</dt>
<dd>
Your application can synthesize PCM data on the fly and then play it
using an audio player with buffer queue data locator.  This is a
relatively advanced technique, and the details of audio synthesis
are beyond the scope of this article.
</dd>

</dl>

<p>
Finding or creating useful audio content for your application is
beyond the scope of this article, but see the "References" section
below for some suggested web search terms.
</p>
<p>
Note that it is your responsibility to ensure that you are legally
permitted to play or record content, and that there may be privacy
considerations for recording content.
</p>

### Debugging

<p>
For robustness, we recommend that you examine the <code>SLresult</code>
value which is returned by most APIs. Use of <code>assert</code>
vs. more advanced error handling logic is a matter of coding style
and the particular API; see the Wikipedia article on
<a href="http://en.wikipedia.org/wiki/Assertion_(computing)">assert</a>
for more information. In the supplied example, we have used <code>assert</code>
for "impossible" conditions which would indicate a coding error, and
explicit error handling for others which are more likely to occur
in production.
</p>
<p>
Many API errors result in a log entry, in addition to the non-zero
result code. These log entries provide additional detail which can
be especially useful for the more complex APIs such as
<code>Engine::CreateAudioPlayer</code>.
</p>
<p>
Use <a href="http://developer.android.com/guide/developing/tools/adb.html">
adb logcat</a>, the
<a href="http://developer.android.com/tools/help/adt.html">
Eclipse ADT plugin</a> LogCat pane, or
<a href="http://developer.android.com/guide/developing/tools/ddms.html#logcat">
ddms logcat</a> to see the log.
</p>

## Supported features from OpenSL ES 1.0.1

<p>
This section summarizes available features. In some
cases, there are limitations which are described in the next
sub-section.
</p>

### Global entry points

<p>
Supported global entry points:
</p>
<ul>
<li><code>slCreateEngine</code>
</li>
<li><code>slQueryNumSupportedEngineInterfaces</code>
</li>
<li><code>slQuerySupportedEngineInterfaces</code>
</li>
</ul>

### Objects and interfaces

<p>
The following figure indicates objects and interfaces supported by
Android's OpenSL ES implementation.  A green cell means the feature
is supported.
</p>

<p>
<img src="{{ site.baseurl }}/guides/images/chart1.png" alt="Supported objects and interfaces" />
</p>

### Limitations

<p>
This section details limitations with respect to the supported
objects and interfaces from the previous section.
</p>

<h4 id="bufferQueueDataLocator">Buffer queue data locator</h4>

<p>
An audio player or recorder with buffer queue data locator supports
PCM data format only.
</p>

<h4 id="deviceDataLocator">Device data locator</h4>

<p>
The only supported use of an I/O device data locator is when it is
specified as the data source for <code>Engine::CreateAudioRecorder</code>.
It should be initialized using these values, as shown in the example:
</p>
<pre>
SLDataLocator_IODevice loc_dev =
  {SL_DATALOCATOR_IODEVICE, SL_IODEVICE_AUDIOINPUT,
  SL_DEFAULTDEVICEID_AUDIOINPUT, NULL};
</pre>

<h4 id="dynamicInterfaceManagementSupported">Dynamic interface management</h4>

<p>
<code>RemoveInterface</code> and <code>ResumeInterface</code> are not supported.
</p>

<h4 id="effectCombinations">Effect combinations</h4>

<p>
It is meaningless to have both environmental reverb and preset
reverb on the same output mix.
</p>
<p>
The platform may ignore effect requests if it estimates that the
CPU load would be too high.
</p>

<h4 id="effectSend">Effect send</h4>

<p>
<code>SetSendLevel</code> supports a single send level per audio player.
</p>

<h4 id="environmentalReverb">Environmental reverb</h4>

<p>
Environmental reverb does not support the <code>reflectionsDelay</code>,
<code>reflectionsLevel</code>, or <code>reverbDelay</code> fields of
<code>struct SLEnvironmentalReverbSettings</code>.
</p>

<h4 id="mimeDataFormat">MIME data format</h4>

<p>
The MIME data format can be used with URI data locator only, and only
for player (not recorder).
</p>
<p>
The Android implementation of OpenSL ES requires that <code>mimeType</code>
be initialized to either <code>NULL</code> or a valid UTF-8 string,
and that <code>containerType</code> be initialized to a valid value.
In the absence of other considerations, such as portability to other
implementations, or content format which cannot be identified by header,
we recommend that you
set the <code>mimeType</code> to <code>NULL</code> and <code>containerType</code>
to <code>SL_CONTAINERTYPE_UNSPECIFIED</code>.
</p>
<p>
Supported formats include WAV PCM, WAV alaw, WAV ulaw, MP3, Ogg
Vorbis, AAC LC, HE-AACv1 (aacPlus), HE-AACv2 (enhanced aacPlus),
AMR, and FLAC [provided these are supported by the overall platform,
and AAC formats must be located within an MP4 or ADTS container].
MIDI is not supported.
WMA is not part of the open source release, and compatibility
with Android OpenSL ES has not been verified.
</p>
<p>
The Android implementation of OpenSL ES does not support direct
playback of DRM or encrypted content; if you want to play this, you
will need to convert to cleartext in your application before playing,
and enforce any DRM restrictions in your application.
</p>

<h4 id="object">Object</h4>

<p>
<code>Resume</code>, <code>RegisterCallback</code>,
<code>AbortAsyncOperation</code>, <code>SetPriority</code>,
<code>GetPriority</code>, and <code>SetLossOfControlInterfaces</code>
are not supported.
</p>

<h4 id="pcmDataFormat">PCM data format</h4>

<p>
The PCM data format can be used with buffer queues only. Supported PCM
playback configurations are 8-bit unsigned or 16-bit signed, mono
or stereo, little endian byte ordering, and these sample rates:
8000, 11025, 12000, 16000, 22050, 24000, 32000, 44100, or 48000 Hz.
For recording, the supported configurations are device-dependent,
however generally 16000 Hz mono 16-bit signed is usually available.
</p>
<p>
Note that the field <code>samplesPerSec</code> is actually in
units of milliHz, despite the misleading name. To avoid accidentally
using the wrong value, you should initialize this field using one
of the symbolic constants defined for this purpose (such as
<code>SL_SAMPLINGRATE_44_1</code> etc.)
</p>
<p>
For API level 21 and above, see section "Floating-point data" below.
</p>

<h4 id="playbackRate">Playback rate</h4>

<p>
<b>Note:</b>
An OpenSL ES <em>playback rate</em> indicates the speed at which an
object presents data, expressed in
<a href="https://en.wikipedia.org/wiki/Per_mille">per mille</a>
units.  A <em>rate range</em> is a closed interval that expresses
possible rate ranges.
</p>
<p>
The supported playback rate range(s) and capabilities may vary depending
on the platform version and implementation, and so should be determined
at runtime by querying with <code>PlaybackRate::GetRateRange</code>
or <code>PlaybackRate::GetCapabilitiesOfRate</code>.
</p>
<p>
That said, some guidance on typical rate ranges may be useful:
In Android 2.3 a single playback rate range from 500 per mille to 2000 per mille
inclusive is typically supported, with property
<code>SL_RATEPROP_NOPITCHCORAUDIO</code>.
In Android 4.0 the same rate range is typically supported for a data source
in PCM format, and a unity rate range of 1000 per mille to 1000 per mille for other formats.
</p>

<h4 id="record">Record</h4>

<p>
The <code>SL_RECORDEVENT_HEADATLIMIT</code> and
<code>SL_RECORDEVENT_HEADMOVING</code> events are not supported.
</p>

<h4 id="seek">Seek</h4>

<p>
<code>SetLoop</code> can only loop the whole file and not a portion of it;
the parameters should be set as follows: the <code>startPos</code>
parameter should be zero and the <code>endPos</code> parameter should
be <code>SL_TIME_UNKNOWN</code>.
</p>

<h4 id="uriDataLocator">URI data locator</h4>

<p>
The URI data locator can be used with MIME data format only, and
only for an audio player (not audio recorder). Supported schemes
are <code>http:</code> and <code>file:</code>.
A missing scheme defaults to the <code>file:</code> scheme. Other
schemes such as <code>https:</code>, <code>ftp:</code>, and
<code>content:</code> are not supported.
<code>rtsp:</code> is not verified.
</p>

### Data structures

<p>
Android supports these OpenSL ES 1.0.1 data structures:
</p>
<ul>
<li>SLDataFormat_MIME
</li>
<li>SLDataFormat_PCM
</li>
<li>SLDataLocator_BufferQueue
</li>
<li>SLDataLocator_IODevice
</li>
<li>SLDataLocator_OutputMix
</li>
<li>SLDataLocator_URI
</li>
<li>SLDataSink
</li>
<li>SLDataSource
</li>
<li>SLEngineOption
</li>
<li>SLEnvironmentalReverbSettings
</li>
<li>SLInterfaceID
</li>
</ul>

### Platform configuration

<p>
OpenSL ES for Android is designed for multi-threaded applications,
and is thread-safe.
</p>
<p>
OpenSL ES for Android supports a single engine per application, and
up to 32 objects per engine. Available device memory and CPU may further
restrict the usable number of objects.
</p>
<p>
<code>slCreateEngine</code> recognizes, but ignores, these engine options:
</p>
<ul>
<li><code>SL_ENGINEOPTION_THREADSAFE</code>
</li>
<li><code>SL_ENGINEOPTION_LOSSOFCONTROL</code>
</li>
</ul>

<p>
OpenMAX AL and OpenSL ES may be used together in the same application.
In this case, there is internally a single shared engine object,
and the 32 object limit is shared between OpenMAX AL and OpenSL ES.
The application should first create both engines, then use both engines,
and finally destroy both engines.  The implementation maintains a
reference count on the shared engine, so that it is correctly destroyed
at the second destroy.
</p>

## Planning for future versions of OpenSL ES

<p>
The Android native audio APIs are based on Khronos
Group OpenSL ES 1.0.1 (see section "References" below).
Khronos has released
a revised version 1.1 of the standard. The revised version
includes new features, clarifications, correction of
typographical errors, and some incompatibilities. Most of the expected
incompatibilities are relatively minor, or are in areas of OpenSL ES
not supported by Android. However, even a small change
can be significant for an application developer, so it is important
to prepare for this.
</p>
<p>
The Android team is committed to preserving future API binary
compatibility for developers to the extent feasible. It is our
intention to continue to support future binary compatibility of the
1.0.1-based API, even as we add support for later versions of the
standard. An application developed with this version should
work on future versions of the Android platform, provided that
you follow the guidelines listed in section "Planning for
binary compatibility" below.
</p>
<p>
Note that future source compatibility will <i>not</i> be a goal. That is,
if you upgrade to a newer version of the NDK, you may need to modify
your application source code to conform to the new API. We expect
that most such changes will be minor; see details below.
</p>

### Planning for binary compatibility

<p>
We recommend that your application follow these guidelines,
to improve future binary compatibility:
</p>
<ul>
<li>
Use only the documented subset of Android-supported features from
OpenSL ES 1.0.1.
</li>
<li>
Do not depend on a particular result code for an unsuccessful
operation; be prepared to deal with a different result code.
</li>
<li>
Application callback handlers generally run in a restricted context,
and should be written to perform their work quickly and then return
as soon as possible. Do not do complex operations within a callback
handler. For example, within a buffer queue completion callback,
you can enqueue another buffer, but do not create an audio player.
</li>
<li>
Callback handlers should be prepared to be called more or less
frequently, to receive additional event types, and should ignore
event types that they do not recognize. Callbacks that are configured
with an event mask of enabled event types should be prepared to be
called with multiple event type bits set simultaneously.
Use "&amp;" to test for each event bit rather than a switch case.
</li>
<li>
Use prefetch status and callbacks as a general indication of progress, but do
not depend on specific hard-coded fill levels or callback sequence.
The meaning of the prefetch status fill level, and the behavior for
errors that are detected during prefetch, may change.
</li>
<li>
See section "Buffer queue behavior" below.
</li>
</ul>

### Planning for source compatibility

<p>
As mentioned, source code incompatibilities are expected in the next
version of OpenSL ES from Khronos Group. Likely areas of change include:
</p>

<ul>
<li>The buffer queue interface is expected to have significant changes,
especially in the areas of <code>BufferQueue::Enqueue</code>, the parameter
list for <code>slBufferQueueCallback</code>,
and the name of field <code>SLBufferQueueState.playIndex</code>.
We recommend that your application code use Android simple buffer
queues instead, because we do not plan to change that API.
In the example code supplied with the NDK, we have used
Android simple buffer queues for playback for this reason.
(We also use Android simple buffer queue for recording and decode to PCM, but
that is because standard OpenSL ES 1.0.1 does not support record or decode to
a buffer queue data sink.)
</li>
<li>Addition of <code>const</code> to input parameters passed by reference,
and to <code>SLchar *</code> struct fields used as input values.
This should not require any changes to your code.
</li>
<li>Substitution of unsigned types for some parameters that are
currently signed.  You may need to change a parameter type from
<code>SLint32</code> to <code>SLuint32</code> or similar, or add a cast.
</li>
<li><code>Equalizer::GetPresetName</code> will copy the string to
application memory instead of returning a pointer to implementation
memory. This will be a significant change, so we recommend that you
either avoid calling this method, or isolate your use of it.
</li>
<li>Additional fields in struct types. For output parameters, these
new fields can be ignored, but for input parameters the new fields
will need to be initialized. Fortunately, these are expected to all
be in areas not supported by Android.
</li>
<li>Interface
<a href="http://en.wikipedia.org/wiki/Globally_unique_identifier">
GUIDs</a> will change. Refer to interfaces by symbolic name rather than GUID
to avoid a dependency.
</li>
<li><code>SLchar</code> will change from <code>unsigned char</code>
to <code>char</code>. This primarily affects the URI data locator
and MIME data format.
</li>
<li><code>SLDataFormat_MIME.mimeType</code> will be renamed to <code>pMimeType</code>,
and <code>SLDataLocator_URI.URI</code> will be renamed to <code>pURI</code>.
We recommend that you initialize the <code>SLDataFormat_MIME</code>
and <code>SLDataLocator_URI</code>
data structures using a brace-enclosed comma-separated list of values,
rather than by field name, to isolate your code from this change.
In the example code we have used this technique.
</li>
<li><code>SL_DATAFORMAT_PCM</code> does not permit the application
to specify the representation of the data as signed integer, unsigned
integer, or floating-point. The Android implementation assumes that
8-bit data is unsigned integer and 16-bit is signed integer.  In
addition, the field <code>samplesPerSec</code> is a misnomer, as
the actual units are milliHz. These issues are expected to be
addressed in the next OpenSL ES version, which will introduce a new
extended PCM data format that permits the application to explicitly
specify the representation, and corrects the field name.  As this
will be a new data format, and the current PCM data format will
still be available (though deprecated), it should not require any
immediate changes to your code.
</li>
</ul>

## Android extensions

<p>
The API for Android extensions is defined in <code>SLES/OpenSLES_Android.h</code>
and the header files that it includes.
Consult that file for details on these extensions. Unless otherwise
noted, all interfaces are "explicit".
</p>
<p>
Note that use these extensions will limit your application's
portability to other OpenSL ES implementations. If this is a concern,
we advise that you avoid using them, or isolate your use of these
with <code>#ifdef</code> etc.
</p>
<p>
The following figure shows which Android-specific interfaces and
data locators are available for each object type.
</p>

<p>
<img src="{{ site.baseurl }}/guides/images/chart2.png" alt="Android extensions" />
</p>

### Android configuration interface

<p>
The Android configuration interface provides a means to set
platform-specific parameters for objects. Unlike other OpenSL ES
1.0.1 interfaces, the Android configuration interface is available
prior to object realization. This permits the object to be configured
and then realized. Header file <code>SLES/OpenSLES_AndroidConfiguration.h</code>
documents the available configuration keys and values:
</p>
<ul>
<li>stream type for audio players (default <code>SL_ANDROID_STREAM_MEDIA</code>)
</li>
<li>record profile for audio recorders (default <code>SL_ANDROID_RECORDING_PRESET_GENERIC</code>)
</li>
</ul>
<p>
Here is an example code fragment that sets the Android audio stream type on an audio player:
</p>
<pre>
// CreateAudioPlayer and specify SL_IID_ANDROIDCONFIGURATION
// in the required interface ID array. Do not realize player yet.
// ...
SLAndroidConfigurationItf playerConfig;
result = (*playerObject)-&gt;GetInterface(playerObject,
    SL_IID_ANDROIDCONFIGURATION, &amp;playerConfig);
assert(SL_RESULT_SUCCESS == result);
SLint32 streamType = SL_ANDROID_STREAM_ALARM;
result = (*playerConfig)-&gt;SetConfiguration(playerConfig,
    SL_ANDROID_KEY_STREAM_TYPE, &amp;streamType, sizeof(SLint32));
assert(SL_RESULT_SUCCESS == result);
// ...
// Now realize the player here.
</pre>
<p>
Similar code can be used to configure the preset for an audio recorder.
</p>

### Android effects interfaces

<p>
The Android effect, effect send, and effect capabilities interfaces provide
a generic mechanism for an application to query and use device-specific
audio effects. A device manufacturer should document any available
device-specific audio effects.
</p>
<p>
Portable applications should use the OpenSL ES 1.0.1 APIs
for audio effects instead of the Android effect extensions.
</p>

### Android file descriptor data locator

<p>
The Android file descriptor data locator permits the source for an
audio player to be specified as an open file descriptor with read
access. The data format must be MIME.
</p>
<p>
This is especially useful in conjunction with the native asset manager.
</p>

### Android simple buffer queue data locator and interface

<p>
The Android simple buffer queue data locator and interface are
identical to the OpenSL ES 1.0.1 buffer queue locator and interface,
except that Android simple buffer queues may be used with both audio
players and audio recorders, and are limited to PCM data format.
[OpenSL ES 1.0.1 buffer queues are for audio players only, and are not
restricted to PCM data format.]
</p>
<p>
For recording, the application should enqueue empty buffers. Upon
notification of completion via a registered callback, the filled
buffer is available for the application to read.
</p>
<p>
For playback there is no difference. But for future source code
compatibility, we suggest that applications use Android simple
buffer queues instead of OpenSL ES 1.0.1 buffer queues.
</p>

### Dynamic interfaces at object creation

<p>
For convenience, the Android implementation of OpenSL ES 1.0.1
permits dynamic interfaces to be specified at object creation time,
as an alternative to adding these interfaces after object creation
with <code>DynamicInterfaceManagement::AddInterface</code>.
</p>

### Buffer queue behavior

<p>
The OpenSL ES 1.0.1 specification requires that "On transition to
the <code>SL_PLAYSTATE_STOPPED</code> state the play cursor is
returned to the beginning of the currently playing buffer." The
Android implementation does not necessarily conform to this
requirement. For Android, it is unspecified whether a transition
to <code>SL_PLAYSTATE_STOPPED</code> operates as described, or
leaves the play cursor unchanged.
</p>
<p>
We recommend that you do not rely on either behavior; after a
transition to <code>SL_PLAYSTATE_STOPPED</code>, you should explicitly
call <code>BufferQueue::Clear</code>. This will place the buffer
queue into a known state.
</p>
<p>
A corollary is that it is unspecified whether buffer queue callbacks
are called upon transition to <code>SL_PLAYSTATE_STOPPED</code> or by
<code>BufferQueue::Clear</code>.
We recommend that you do not rely on either behavior; be prepared
to receive a callback in these cases, but also do not depend on
receiving one.
</p>
<p>
It is expected that a future version of OpenSL ES will clarify these
issues. However, upgrading to that version would result in source
code incompatibilities (see section "Planning for source compatibility"
above).
</p>

### Reporting of extensions

<p>
<code>Engine::QueryNumSupportedExtensions</code>,
<code>Engine::QuerySupportedExtension</code>,
<code>Engine::IsExtensionSupported</code> report these extensions:
</p>
<ul>
<li><code>ANDROID_SDK_LEVEL_#</code>
where # is the platform API level, 9 or higher
</li>
</ul>

### Decode audio to PCM

<p>
This section describes a deprecated Android-specific extension to OpenSL ES 1.0.1
for decoding an encoded stream to PCM without immediate playback.
The table below gives recommendations for use of this extension and alternatives.
</p>
<table>
<tr>
  <th>API level</th>
  <th>Extension<br /> is available</th>
  <th>Extension<br /> is recommended</th>
  <th>Alternatives</th>
</tr>
<tr>
  <td>13<br /> and below</td>
  <td>no</td>
  <td>N/A</td>
  <td>open source codec with suitable license</td>
</tr>
<tr>
  <td>14 to 15</td>
  <td>yes</td>
  <td>no</td>
  <td>open source codec with suitable license</td>
</tr>
<tr>
  <td>16 to 20</td>
  <td>yes</td>
  <td>no</td>
  <td>
    <a href="http://developer.android.com/reference/android/media/MediaCodec.html">android.media.MediaCodec</a><br />
    or open source codec with suitable license
  </td>
</tr>
<tr>
  <td>21<br /> and above</td>
  <td>yes</td>
  <td>no</td>
  <td>
    NDK MediaCodec in &lt;media/NdkMedia*.h&gt;<br />
    or <a href="http://developer.android.com/reference/android/media/MediaCodec.html">android.media.MediaCodec</a><br />
    or open source codec with suitable license
  </td>
</tr>
</table>

<p>
A standard audio player plays back to an audio device, and the data sink
is specified as an output mix.
However, as an Android extension, an audio player instead
acts as a decoder if the data source is specified as a URI or Android
file descriptor data locator with MIME data format, and the data sink is
an Android simple buffer queue data locator with PCM data format.
</p>
<p>
This feature is primarily intended for games to pre-load their
audio assets when changing to a new game level, similar to
<code>android.media.SoundPool</code>.
</p>
<p>
The application should initially enqueue a set of empty buffers to the Android simple
buffer queue. These buffers will be filled with the decoded PCM data.  The Android simple
buffer queue callback is invoked after each buffer is filled. The
callback handler should process the PCM data, re-enqueue the
now-empty buffer, and then return.  The application is responsible for
keeping track of decoded buffers; the callback parameter list does not include
sufficient information to indicate which buffer was filled or which buffer to enqueue next.
</p>
<p>
The end of stream is determined implicitly by the data source.
At the end of stream a <code>SL_PLAYEVENT_HEADATEND</code> event is
delivered. The Android simple buffer queue callback will no longer
be called after all consumed data is decoded.
</p>
<p>
The sink's PCM data format typically matches that of the encoded data source
with respect to sample rate, channel count, and bit depth. However, the platform
implementation is permitted to decode to a different sample rate, channel count, or bit depth.
There is a provision to detect the actual PCM format; see section "Determining
the format of decoded PCM data via metadata" below.
</p>
<p>
Decode to PCM supports pause and initial seek.  Volume control, effects,
looping, and playback rate are not supported.
</p>
<p>
Depending on the platform implementation, decoding may require resources
that cannot be left idle.  Therefore it is not recommended to starve the
decoder by failing to provide a sufficient number of empty PCM buffers,
e.g. by returning from the Android simple buffer queue callback without
enqueueing another empty buffer.  The result of decoder starvation is
unspecified; the implementation may choose to either drop the decoded
PCM data, pause the decoding process, or in severe cases terminate
the decoder.
</p>

### Decode streaming ADTS AAC to PCM

<p>
Note: this feature is available at API level 14 and higher.
</p>
<p>
An audio player acts as a streaming decoder if the data source is an
Android buffer queue data locator with MIME data format, and the data
sink is an Android simple buffer queue data locator with PCM data format.
The MIME data format should be configured as:
</p>
<dl>
<dt>container</dt>
<dd><code>SL_CONTAINERTYPE_RAW</code>
</dd>
<dt>MIME type string
</dt>
<dd><code>"audio/vnd.android.aac-adts"</code> (macro <code>SL_ANDROID_MIME_AACADTS</code>)
</dd>
</dl>
<p>
This feature is primarily intended for streaming media applications that
deal with AAC audio, but need to apply custom processing of the audio
prior to playback.  Most applications that need to decode audio to PCM
should use the method of the previous section "Decode audio to PCM",
as it is simpler and handles more audio formats.  The technique described
here is a more specialized approach, to be used only if both of these
conditions apply:
</p>
<ul>
<li>the compressed audio source is a stream of AAC frames contained by ADTS headers
</li>
<li>the application manages this stream, that is the data is <i>not</i> located within
a network resource identified by URI or within a local file identified by file descriptor.
</li>
</ul>
<p>
The application should initially enqueue a set of filled buffers to the Android buffer queue.
Each buffer contains one or more complete ADTS AAC frames.
The Android buffer queue callback is invoked after each buffer is emptied.
The callback handler should re-fill and re-enqueue the buffer, and then return.
The application need not keep track of encoded buffers; the callback parameter
list does include sufficient information to indicate which buffer to enqueue next.
The end of stream is explicitly marked by enqueuing an EOS item.
After EOS, no more enqueues are permitted.
</p>
<p>
It is not recommended to starve the decoder by failing to provide full
ADTS AAC buffers, e.g. by returning from the Android buffer queue callback
without enqueueing another full buffer.  The result of decoder starvation
is unspecified.
</p>
<p>
In all respects except for the data source, the streaming decode method is similar
to that of the previous section:
</p>
<ul>
<li>initially enqueue a set of empty buffers to the Android simple buffer queue
</li>
<li>the Android simple buffer queue callback is invoked after each buffer is filled with PCM data;
the callback handler should process the PCM data and then re-enqueue another empty buffer
</li>
<li>the <code>SL_PLAYEVENT_HEADATEND</code> event is delivered at end of stream
</li>
<li>the actual PCM format should be detected using metadata rather than by making an assumption
</li>
<li>the same limitations apply with respect to volume control, effects, etc.
</li>
<li>starvation for lack of empty PCM buffers is not recommended
</li>
</ul>
<p>
Despite the similarity in names, an Android buffer queue is <i>not</i>
the same as an Android simple buffer queue.  The streaming decoder
uses both kinds of buffer queues: an Android buffer queue for the ADTS
AAC data source, and an Android simple buffer queue for the PCM data
sink.  The Android simple buffer queue API is described in this document
in section "Android simple buffer queue data locator and interface".
The Android buffer queue API is described in the Android native media
API documentation, located in $NDK_ROOT/docs/Additional_library_docs/openmaxal/index.html
</p>

### Determining the format of decoded PCM data via metadata

<p>
The metadata extraction interface <code>SLMetadataExtractionItf</code>
is a standard OpenSL ES 1.0.1 interface, not an Android extension.
However, the particular metadata keys that
indicate the actual format of decoded PCM data are specific to Android,
and are defined in header <code>SLES/OpenSLES_AndroidMetadata.h</code>.
</p>
<p>
The metadata key indices are available immediately after
<code>Object::Realize</code>. Yet the associated values are not
available until after the first encoded data has been decoded.  A good
practice is to query for the key indices in the main thread after Realize,
and to read the PCM format metadata values in the Android simple
buffer queue callback handler the first time it is called.
</p>
<p>
The OpenSL ES 1.0.1 metadata extraction interface
<code>SLMetadataExtractionItf</code> is admittedly cumbersome, as it
requires a multi-step process to first determine key indices and then
to get the key values.  Consult the example code for snippets showing
how to work with this interface.
</p>
<p>
Metadata key names are stable.  But the key indices are not documented
and are subject to change.  An application should not assume that indices
are persistent across different execution runs, and should not assume that
indices are shared for different object instances within the same run.
</p>

### Floating-point data

<p>
As of API level 21 and above, data can be supplied to an AudioPlayer in
single-precision floating-point format.
</p>
<p>
Example code fragment, to be used during the Engine::CreateAudioPlayer process:
</p>
<pre>
#include &lt;SLES/OpenSLES_Android.h&gt;
...
SLAndroidDataFormat_PCM_EX pcm;
pcm.formatType = SL_ANDROID_DATAFORMAT_PCM_EX;
pcm.numChannels = 2;
pcm.sampleRate = SL_SAMPLINGRATE_44_1;
pcm.bitsPerSample = 32;
pcm.containerSize = 32;
pcm.channelMask = SL_SPEAKER_FRONT_LEFT | SL_SPEAKER_FRONT_RIGHT;
pcm.endianness = SL_BYTEORDER_LITTLEENDIAN;
pcm.representation = SL_ANDROID_PCM_REPRESENTATION_FLOAT;
...
SLDataSource audiosrc;
audiosrc.pLocator = ...
audiosrc.pFormat = &amp;pcm;
</pre>

## Programming notes

<p>
These notes supplement the OpenSL ES 1.0.1 specification,
available in the "References" section below.
</p>

### Objects and interface initialization

<p>
Two aspects of the OpenSL ES programming model that may be unfamiliar
to new developers are the distinction between objects and interfaces,
and the initialization sequence.
</p>
<p>
Briefly, an OpenSL ES object is similar to the object concept
in programming languages such as Java and C++, except an OpenSL ES
object is <i>only</i> visible via its associated interfaces. This
includes the initial interface for all objects, called
<code>SLObjectItf</code>.  There is no handle for an object itself,
only a handle to the <code>SLObjectItf</code> interface of the object.
</p>
<p>
An OpenSL ES object is first "created", which returns an
<code>SLObjectItf</code>, then "realized". This is similar to the
common programming pattern of first constructing an object (which
should never fail other than for lack of memory or invalid parameters),
and then completing initialization (which may fail due to lack of
resources).  The realize step gives the implementation a
logical place to allocate additional resources if needed.
</p>
<p>
As part of the API to create an object, an application specifies
an array of desired interfaces that it plans to acquire later. Note
that this array does <i>not</i> automatically acquire the interfaces;
it merely indicates a future intention to acquire them.  Interfaces
are distinguished as "implicit" or "explicit".  An explicit interface
<i>must</i> be listed in the array if it will be acquired later.
An implicit interface need not be listed in the object create array,
but there is no harm in listing it there.  OpenSL ES has one more
kind of interface called "dynamic", which does not need to be
specified in the object create array, and can be added later after
the object is created.  The Android implementation provides a
convenience feature to avoid this complexity; see section "Dynamic
interfaces at object creation" above.
</p>
<p>
After the object is created and realized, the application should
acquire interfaces for each feature it needs, using
<code>GetInterface</code> on the initial <code>SLObjectItf</code>.
</p>
<p>
Finally, the object is available for use via its interfaces, though
note that some objects require further setup. In particular, an
audio player with URI data source needs a bit more preparation in
order to detect connection errors. See the next section
"Audio player prefetch" for details.
</p>
<p>
After your application is done with the object, you should explicitly
destroy it; see section "Destroy" below.
</p>

### Audio player prefetch

<p>
For an audio player with URI data source, <code>Object::Realize</code> allocates resources
but does not connect to the data source (i.e. "prepare") or begin
pre-fetching data. These occur once the player state is set to
either <code>SL_PLAYSTATE_PAUSED</code> or <code>SL_PLAYSTATE_PLAYING</code>.
</p>
<p>
Note that some information may still be unknown until relatively
late in this sequence. In particular, initially
<code>Player::GetDuration</code> will return <code>SL_TIME_UNKNOWN</code>
and <code>MuteSolo::GetChannelCount</code> will either return successfully
with channel count zero
or the error result <code>SL_RESULT_PRECONDITIONS_VIOLATED</code>.
These APIs will return the proper values once they are known.
</p>
<p>
Other properties that are initially unknown include the sample rate
and actual media content type based on examining the content's header
(as opposed to the application-specified MIME type and container type).
These too, are determined later during prepare / prefetch, but there are
no APIs to retrieve them.
</p>
<p>
The prefetch status interface is useful for detecting when all
information is available. Or, your application can poll periodically.
Note that some information may <i>never</i> be known, for example,
the duration of a streaming MP3.
</p>
<p>
The prefetch status interface is also useful for detecting errors.
Register a callback and enable at least the
<code>SL_PREFETCHEVENT_FILLLEVELCHANGE</code> and
<code>SL_PREFETCHEVENT_STATUSCHANGE</code> events. If both of these
events are delivered simultaneously, and
<code>PrefetchStatus::GetFillLevel</code> reports a zero level, and
<code>PrefetchStatus::GetPrefetchStatus</code> reports
<code>SL_PREFETCHSTATUS_UNDERFLOW</code>, then this indicates a
non-recoverable error in the data source.
This includes the inability to connect to the data source because
the local filename does not exist or the network URI is invalid.
</p>
<p>
The next version of OpenSL ES is expected to add more explicit
support for handling errors in the data source. However, for future
binary compatibility, we intend to continue to support the current
method for reporting a non-recoverable error.
</p>
<p>
In summary, a recommended code sequence is:
</p>
<ol>
<li>Engine::CreateAudioPlayer
</li>
<li>Object:Realize
</li>
<li>Object::GetInterface for SL_IID_PREFETCHSTATUS
</li>
<li>PrefetchStatus::SetCallbackEventsMask
</li>
<li>PrefetchStatus::SetFillUpdatePeriod
</li>
<li>PrefetchStatus::RegisterCallback
</li>
<li>Object::GetInterface for SL_IID_PLAY
</li>
<li>Play::SetPlayState to SL_PLAYSTATE_PAUSED or SL_PLAYSTATE_PLAYING
</li>
<li>preparation and prefetching occur here; during this time your
callback will be called with periodic status updates
</li>
</ol>

### Destroy

<p>
Be sure to destroy all objects on exit from your application.  Objects
should be destroyed in reverse order of their creation, as it is
not safe to destroy an object that has any dependent objects.
For example, destroy in this order: audio players and recorders,
output mix, then finally the engine.
</p>
<p>
OpenSL ES does not support automatic garbage collection or
<a href="http://en.wikipedia.org/wiki/Reference_counting">reference counting</a>
of interfaces. After you call <code>Object::Destroy</code>, all extant
interfaces derived from the associated object become <i>undefined</i>.
</p>
<p>
The Android OpenSL ES implementation does not detect the incorrect
use of such interfaces.
Continuing to use such interfaces after the object is destroyed will
cause your application to crash or behave in unpredictable ways.
</p>
<p>
We recommend that you explicitly set both the primary object interface
and all associated interfaces to NULL as part of your object
destruction sequence, to prevent the accidental misuse of a stale
interface handle.
</p>

### Stereo panning

<p>
When <code>Volume::EnableStereoPosition</code> is used to enable
stereo panning of a mono source, there is a 3 dB reduction in total
<a href="http://en.wikipedia.org/wiki/Sound_power_level">
sound power level</a>.  This is needed to permit the total sound
power level to remain constant as the source is panned from one
channel to the other. Therefore, don't enable stereo positioning
if you don't need it.  See the Wikipedia article on
<a href="http://en.wikipedia.org/wiki/Panning_(audio)">audio panning</a>
for more information.
</p>

### Callbacks and threads

<p>
Callback handlers are generally called <i>synchronously</i> with
respect to the event, that is, at the moment and location where the
event is detected by the implementation. But this point is
<i>asynchronous</i> with respect to the application. Thus you should
use a non-blocking synchronization mechanism to control access
to any variables shared between the application and the callback
handler. In the example code, such as for buffer queues, we have
either omitted this synchronization or used blocking synchronization in
the interest of simplicity. However, proper non-blocking synchronization
would be critical for any production code.
</p>
<p>
Callback handlers are called from internal
non-application thread(s) which are not attached to the Android runtime and thus
are ineligible to use JNI. Because these internal threads are
critical to the integrity of the OpenSL ES implementation, a callback
handler should also not block or perform excessive work.
</p>
<p>
If your callback handler needs to use JNI, or execute work that is
not proportional to the callback, the handler should instead post an
event for another thread to process.  Examples of acceptable callback
workload include rendering and enqueuing the next output buffer (for an
AudioPlayer), processing the just-filled input buffer and enqueueing the
next empty buffer (for an AudioRecorder), or simple APIs such as most
of the "Get" family.  See section "Performance" below regarding the workload.
</p>
<p>
Note that the converse is safe: an Android application thread which has
entered JNI is allowed to directly call OpenSL ES APIs, including
those which block. However, blocking calls are not recommended from
the main thread, as they may result in "Application Not
Responding" (ANR).
</p>
<p>
The choice of which thread calls a callback handler is largely left up
to the implementation.  The reason for this flexibility is to permit
future optimizations, especially on multi-core devices.
</p>
<p>
The thread on which the callback handler runs is not guaranteed to have
the same identity across different calls.  Therefore do not rely on the
<code>pthread_t</code> returned by <code>pthread_self()</code>, or the
<code>pid_t</code> returned by <code>gettid()</code>, to be consistent
across calls.  Don't use the thread local storage (TLS) APIs such as
<code>pthread_setspecific()</code> and <code>pthread_getspecific()</code>
from a callback, for the same reason.
</p>
<p>
The implementation guarantees that concurrent callbacks of the same kind,
for the same object, will not occur.  However, concurrent callbacks of
<i>different</i> kinds for the same object are possible, on different threads.
</p>

### Performance

<p>
As OpenSL ES is a native C API, non-runtime application threads which
call OpenSL ES have no runtime-related overhead such as garbage
collection pauses. With one exception described below, there is no additional performance
benefit to the use of OpenSL ES other than this. In particular, use
of OpenSL ES does not guarantee a lower audio latency, higher scheduling
priority, etc. than what the platform generally provides.
On the other hand, as the Android platform and specific device
implementations continue to evolve, an OpenSL ES application can
expect to benefit from any future system performance improvements.
</p>
<p>
One such evolution is support for reduced audio output latency.
The underpinnings for reduced output latency were first included in
the Android 4.1 platform release ("Jellybean"), and then continued
progress occurred in the Android 4.2 platform.  These improvements
are available via OpenSL ES for device implementations that claim feature
"android.hardware.audio.low_latency". If the device doesn't claim this
feature but supports API level 9 (Android platform version 2.3) or later,
then you can still use the OpenSL ES APIs but the output latency may be higher.
The lower output latency path is used only if the application requests a
buffer size and sample rate that are
compatible with the device's native output configuration.
These parameters are device-specific and should be obtained as follows.
</p>
<p>
Beginning with API level 17 (Android platform version 4.2), an application
can query for the platform native or optimal output sample rate and buffer size
for the device's primary output stream.  When combined with the feature
test just mentioned, an app can now configure itself appropriately for
lower latency output on devices that claim support.
</p>
<p>
For API level 17 (Android platform version 4.2.2) and earlier,
a buffer count of 2 or more is required for lower latency.
Beginning with API level 18 (Android platform version 4.3),
a buffer count of 1 is sufficient for lower latency.
</p>
<p>
All
OpenSL ES interfaces for output effects preclude the lower latency path.
</p>
<p>
The recommended sequence is:
</p>
<ol>
<li>Check for API level 9 or higher, to confirm use of OpenSL ES.
</li>
<li>Check for feature "android.hardware.audio.low_latency" using code such as this:
<pre>
import android.content.pm.PackageManager;
...
PackageManager pm = getContext().getPackageManager();
boolean claimsFeature = pm.hasSystemFeature(PackageManager.FEATURE_AUDIO_LOW_LATENCY);
</pre>
</li>
<li>Check for API level 17 or higher, to confirm use of
<code>android.media.AudioManager.getProperty()</code>.
</li>
<li>Get the native or optimal output sample rate and buffer size for this device's primary output
stream, using code such as this:
<pre>
import android.media.AudioManager;
...
AudioManager am = (AudioManager) getSystemService(Context.AUDIO_SERVICE);
String sampleRate = am.getProperty(AudioManager.PROPERTY_OUTPUT_SAMPLE_RATE));
String framesPerBuffer = am.getProperty(AudioManager.PROPERTY_OUTPUT_FRAMES_PER_BUFFER));
</pre>
Note that <code>sampleRate</code> and <code>framesPerBuffer</code>
are <code>String</code>s.  First check for <code>null</code>
and then convert to <code>int</code> using <code>Integer.parseInt()</code>.
</li>
<li>Now use OpenSL ES to create an AudioPlayer with PCM buffer queue data locator.
</li>
</ol>
<p>
The number of lower latency audio players is limited. If your application
requires more than a few audio sources, consider mixing your audio at
application level.  Be sure to destroy your audio players when your
activity is paused, as they are a global resource shared with other apps.
</p>
<p>
To avoid audible glitches, the buffer queue callback handler must execute
within a small and predictable time window. This typically implies no
unbounded blocking on mutexes, conditions, or I/O operations. Instead consider
"try locks", locks and waits with timeouts, and non-blocking algorithms.
</p>
<p>
The computation required to render the next buffer (for AudioPlayer) or
consume the previous buffer (for AudioRecord) should take approximately
the same amount of time for each callback.  Avoid algorithms that execute in
a non-deterministic amount of time, or are "bursty" in their computations.
A callback computation is bursty if the CPU time spent in any
given callback is significantly larger than the average.
In summary, the ideal is for the CPU execution time of the handler to have
variance near zero, and for the handler to not block for unbounded times.
</p>
<p>
Lower latency audio is possible for these outputs only: on-device speaker, wired
headphones, wired headset, line out, and USB digital audio.
On some devices, speaker latency is higher than other paths due to
digital signal processing for speaker correction and protection.
</p>
<p>
As of API level 21, lower latency audio input is supported on select
devices. To take advantage of this feature, first confirm that lower
latency output is available as described above. The capability for lower
latency output is a prerequisite for the lower latency input feature.
Then create an AudioRecorder with the same sample rate and buffer size
as would be used for output.
OpenSL ES interfaces for input effects preclude the lower latency path.
The record preset SL_ANDROID_RECORDING_PRESET_VOICE_RECOGNITION
must be used for lower latency; this preset disables device-specific
digital signal processing which may add latency to the input path.
For more information on record presets,
see section "Android configuration interface" above.
</p>
<p>
For simultaneous input and output, separate buffer queue
completion handlers are used for each side.  There is no guarantee of
the relative order of these callbacks, or the synchronization of the
audio clocks, even when both sides use the same sample rate.
Your application should buffer the data with proper buffer synchronization.
</p>
<p>
One consequence of potentially independent audio clocks is the need
for asynchronous sample rate conversion.  A simple (though not ideal
for audio quality) technique for asynchronous sample rate conversion
is to duplicate or drop samples as needed near a zero-crossing point.
More sophisticated conversions are possible.
</p>

### Security and permissions

<p>
As far as who can do what, security in Android is done at the
process level. Java programming language code can't do anything more than native code, nor
can native code do anything more than Java programming language code. The only differences
between them are what APIs are available.
</p>
<p>
Applications using OpenSL ES must request whatever permissions they
would need for similar non-native APIs. For example, if your application
records audio, then it needs the <code>android.permission.RECORD_AUDIO</code>
permission. Applications that use audio effects need
<code>android.permission.MODIFY_AUDIO_SETTINGS</code>. Applications that play
network URI resources need <code>android.permission.NETWORK</code>.
See <a href="https://developer.android.com/training/permissions/index.html">Working with System Permissions</a>
for more information.
</p>
<p>
Depending on the platform version and implementation,
media content parsers and software codecs may run within the context
of the Android application that calls OpenSL ES (hardware codecs
are abstracted, but are device-dependent). Malformed content
designed to exploit parser and codec vulnerabilities is a known attack
vector. We recommend that you play media only from trustworthy
sources, or that you partition your application such that code that
handles media from untrustworthy sources runs in a relatively
sandboxed environment.  For example you could process media from
untrustworthy sources in a separate process. Though both processes
would still run under the same UID, this separation does make an
attack more difficult.
</p>

## Platform issues

<p>
This section describes known issues in the initial platform
release which supports these APIs.
</p>

### Dynamic interface management

<p>
<code>DynamicInterfaceManagement::AddInterface</code> does not work.
Instead, specify the interface in the array passed to Create, as
shown in the example code for environmental reverb.
</p>

## References and resources

<p>
Android:
</p>
<ul>
<li><a href="http://developer.android.com/resources/index.html">
Android developer resources</a>
</li>
<li><a href="http://groups.google.com/group/android-developers">
Android developers discussion group</a>
</li>
<li><a href="http://developer.android.com/sdk/ndk/index.html">Android NDK</a>
</li>
<li><a href="http://groups.google.com/group/android-ndk">
Android NDK discussion group</a> (for developers of native code, including OpenSL ES)
</li>
<li><a href="http://code.google.com/p/android/issues/">
Android open source bug database</a>
</li>
<li><a href="https://github.com/googlesamples/android-ndk">Android Studio NDK Samples</a>
</li>
<li><a href="http://developer.android.com/samples/index.html">Android Studio Samples</a>
</li>
</ul>

<p>
Khronos Group:
</p>
<ul>
<li><a href="http://www.khronos.org/opensles/">
Khronos Group OpenSL ES Overview</a>
</li>
<li><a href="http://www.khronos.org/registry/sles/">
Khronos Group OpenSL ES 1.0.1 specification</a>
</li>
<li><a href="https://forums.khronos.org/forumdisplay.php/91-OpenSL-ES-embedded-audio-acceleration">
Khronos Group public message board for OpenSL ES</a>
(please limit to non-Android questions)
</li>
</ul>
<p>
For convenience, we have included a copy of the OpenSL ES 1.0.1
specification with the NDK in
<code>docs/opensles/OpenSL_ES_Specification_1.0.1.pdf</code>.
</p>

<p>
Miscellaneous:
</p>
<ul>
<li><a href="http://en.wikipedia.org/wiki/Java_Native_Interface">JNI</a>
</li>
<li><a href="http://stackoverflow.com/search?q=android+audio">
Stack Overflow</a>
</li>
<li>web search for "interactive audio", "game audio", "sound design",
"audio programming", "audio content", "audio formats", etc.
</li>
<li><a href="http://en.wikipedia.org/wiki/Advanced_Audio_Coding">AAC</a>
</li>
</ul>