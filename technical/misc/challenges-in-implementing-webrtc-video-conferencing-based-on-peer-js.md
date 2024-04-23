# Challenges in Implementing WebRTC Video Conferencing Based on Peer.js

## Limitations Brought by the WebRTC Protocol

- **Randomization of Media Stream Attributes**

  - The RTC protocol requires the recipient to overwrite attributes such as id, label, and contentHint on the media track after receiving the media stream to ensure the properties of the stream do not leak information about the sender's media devices, making the stream unique in the P2P network.

    This makes it impossible for the receiver to distinguish between similar types of tracks within the stream. For example, if the sender combines video tracks from a camera and screen sharing into a single stream, the sender does not know what the ids of these tracks are when they reach the receiver, nor can the receiver tell which is screen sharing and which is the video stream.

  - **Solution**: Establish multiple P2P connections when transmitting several tracks that need to be separated (although this may seem impractical).

- **FireFox Prohibits Multiple Connections Between the Same Nodes**

  - FireFox prohibits establishing multiple media connections between the same nodes, meaning node A can only call node B once.

  - **Solution**: If there is a need to establish two streams (e.g., camera and screen sharing streams), let nodes A and B call each other once.

- **Inability to Add or Remove Tracks Once a Connection is Established**

  - Although a stream is passed when establishing an RTC connection, the protocol acquires all the tracks from the stream and transmits these tracks, which means the protocol does not reference the stream object, and adding or removing tracks from the stream does not affect the RTC.

    For example, if a user is currently using the front camera and switches to the rear camera, deleting the front media stream and adding the rear media stream in the RTC connection will not switch tracks but will result in an inability to retrieve track information.

  - **Solution**: Create placeholder streams with the maximum possible number of tracks that might be used when establishing the connection. Use the RTC API's [replaceTrack](https://developer.mozilla.org/en-US/docs/Web/API/RTCRtpSender/replaceTrack) method to replace the placeholder stream when adding or removing devices. For instance, when transmitting data from a camera and microphone, placeholder audio tracks and two video tracks might be necessary. When the camera is activated, switch the placeholder video track to the actual track and switch back when the camera is disabled.

- **RTC Connection Status Statistics**

  RTC does not support directly summarizing the status of all RTC connections, only providing statistics for individual connections. Developers can obtain reports on different aspects of a single connection through [`RTCPeerConnection.getStats`](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection/getStats).

  - **Statistics on Bitrate**:
    1. Obtain the total number of bytes sent or received by the current connection.
    2. Sum the total bytes sent and received by all connections.
    3. Bitrate = `8 * difference in bytes between two checks / interval between checks`

    Obtaining Byte Counts

    - Downlink bytes: Reports where `report.type === 'inbound-rtp'` and `report.bytesReceived`.
    - Uplink bytes: Reports where `report.type === 'outbound-rtp'` and `report.bytesReceived`.

    Differentiating Media Types

    - Video: Identify reports where `report.mediaType === 'video'`.
    - Audio: Identify reports where `report.mediaType === 'audio'`.

  - **Statistics on Downlink Packet Loss Rate**:
    - Cumulative packet loss for a connection: `report.type === 'inbound-rtp'` and `report.packetsLost`.
    - Packets received in a connection: `report.type === 'inbound-rtp'` and `report.packageReceived`.
    - Total downlink packet loss rate: `sum of all connections' lost packets / (sum of all connections' lost packets + sum of all connections' received packets)`

  - **Network Latency**:
    - Latency for a connection: `report.currentRoundTripTime` (seconds).
    - Total latency: `sum of all latencies / number of connections`

  - **Video Frame Rate**:
    - Uplink frame rate: `report.type === 'outbound-rtp'` and `report.framesPerSecond`.
    - Downlink frame rate: `report.type === 'inbound-rtp'` and `report.framesPerSecond`.
    - Total frame rate: `sum of connection frame rates / number of connections`

  - **Video Resolution**:
    - Uplink resolution: `report.type === 'outbound-rtp'` and `report.frameWidth`, `report.frameHeight`.
    - Downlink resolution: `report.type === 'inbound-rtp'` and `report.frameWidth`, `report.frameHeight`.

  For implementation reference: <https://github.com/KairuiLiu/conflux-client/blob/master/src/utils/report.ts>

- **RTC Connection Status Report for Debugging**: <chrome://webrtc-internals/>

## Limitations of the Peer.js Library

The [peer.js](https://github.com/peers/peerjs) project is currently not very active and has accumulated many issues. It might be worth considering switching to [simple-peer](https://github.com/feross/simple-peer).

- **Establishment must use an active stream for `call()` and `answer()`** [#323](https://github.com/peers/peerjs/issues/323)

  - When establishing a connection with `call()`, an active stream must be provided, otherwise it defaults to failure.

    When responding to a connection with `answer()`, an active stream must be provided, otherwise the initiator cannot enter the `on(stream)` event.

  - Rationale: The WebRTC protocol does not allow adding or removing tracks to a stream, so if you pass an empty stream, that connection will always remain empty.

  - Unreasonable: If the caller has no media information, they cannot initiate a connection, even if they just want to receive the media stream from the other party.

  - Solution: Create an empty active stream to serve as a placeholder.

    ```ts
    export const createEmptyAudioTrack = (): MediaStreamTrack => {
      const ctx = new AudioContext();
      const oscillator = ctx.createOscillator();
      const destination = ctx.createMediaStreamDestination();
      oscillator.connect(destination);
      oscillator.start();
      const track = destination.stream.getAudioTracks()[0];
      return Object.assign(track, { enabled: false }) as MediaStreamTrack;
    };

    export const createEmptyVideoTrack = (): MediaStreamTrack => {
      const canvas = Object.assign(document.createElement('canvas'), {
        width: 1,
        height: 1,
      });
      canvas.getContext('2d')!.fillRect(0, 0, 1, 1);
      const stream = canvas.captureStream();
      const track = stream.getVideoTracks()[0];
      return Object.assign(track, { enabled: false }) as MediaStreamTrack;
    };

    fakeVidioTrack = createEmptyVideoTrack();
    fakeAudioTrack = createEmptyAudioTrack();
    mediaStream = new MediaStream([audioTrack, videoTrack]);
    ```

- **Only the `call()` method can transmit metadata, the `answer()` method cannot**

  - During `call()`, you can add `{metadata: any}` in the second parameter to send some context to the receiver, but the receiver cannot return context through `answer()`.
  - Solution: Establish a dedicated data stream for transmitting data. Also, note that the methods supported by data stream connections differ from those supported by media stream connections. Before sending data in a data stream connection, not only must the stream be open, but also checked if it is connected (because the data stream is established first and then exchanges data multiple times, whereas the media stream is connected at the time of the data call).


## Media Stream Handling and Compatibility Issues

- **Acquiring Media Permissions and Browser Compatibility Issues**

  The browser prompts the user for permission upon the first call to `navigator.mediaDevices.getUserMedia()`. However, this causes the device list to refresh only when the user's devices are accessed, which is inconvenient for UI display. The following methods are recommended:

  - **Check for Permission**:

    - Modern method: Use `navigator.permissions`

      ```ts
      const permissionName = name === 'microphone' // or 'camera';
      navigator.permissions
            .query({ name: permissionName as unknown as PermissionName })
            .then((permissionStatus) => permissionStatus.state === 'granted')
      ```

      However, this API has compatibility issues; only Chrome and Safari support the `microphone` and `camera` permission names. Firefox will report an error.

    - Fallback method: Check in `navigator.mediaDevices.enumerateDevices()` whether a certain type is missing or the label is empty. If so, it indicates either the user doesn't have such a device or hasn't granted permission. This function does not prompt the browser for authorization.

    - Summary:
      ```ts
      function checkPermission(name: 'audio' | 'video') {
        const permissionName = name === 'audio' ? 'microphone' : 'camera';
        try {
          return navigator.permissions
            .query({ name: permissionName as unknown as PermissionName })
            .then((permissionStatus) => permissionStatus.state === 'granted')
            .catch(() => {
              return navigator.mediaDevices
                .enumerateDevices()
                .then((devices) =>
                  devices.some(
                    (device) => device.kind === `${name}input` && device.label !== ''
                  )
                );
            });
        } catch (_) {
          return navigator.mediaDevices
            .enumerateDevices()
            .then((devices) =>
              devices.some(
                (device) => device.kind === `${name}input` && device.label !== ''
              )
            );
        }
      }
      ```

  - **Request Permission**:

      ```ts
      function requestPermission(name: 'audio' | 'video') {
        return navigator.mediaDevices
          .getUserMedia({ video: name === 'video', audio: name === 'audio' })
          .then((stream) => {
            stream.getTracks().forEach((track) => track.stop());
          })
          .catch(() => {});
      ```

  - **Monitor Permission Status Changes**

    ```ts
    try {
      navigator.permissions
        .query({ name: 'camera' as unknown as PermissionName })
        .then((permissionStatus) => {
          permissionStatus.onchange = () => {
            refreshMediaDevice('video', setState);
          };
        })
        .catch(() => {});
      navigator.permissions
        .query({ name: 'microphone' as unknown as PermissionName })
        .then((permissionStatus) => {
          permissionStatus.onchange = () => {
            refreshMediaDevice('audio', setState);
          };
        })
        .catch(() => {});
    } catch (error) {
      console.info('[ERROR] on getting permission ', error);
    }
    ```

  - **Monitor Device Changes**

    ```ts
    navigator.mediaDevices.addEventListener('devicechange', cb)
    ```

- **Selecting Audio Output Device**

  Note: Only desktop Chrome and Firefox browsers support this feature, and TypeScript does not have this method.

  ```ts
  // @ts-ignore
  audioElement?.setSinkId?.(speaker.deviceId);
  ```

- **Selecting Audio and Video Capture Devices**

  ```ts
  const constraints: MediaStreamConstraints = {
    [type]: { deviceId: { exact: deviceId } },
  };
  const stream = await navigator.mediaDevices.getUserMedia(constraints);
  ```

- **Audio Track Volume Statistics**

  Note: The volume range is $[0,1]$. When the microphone is far away, the collected volume is consistently low, keeping the progress bar at a low point. It's advisable to use softmax to alter the results, allowing users to clearly see volume changes.

  Note: `AudioContext` cannot specify audio playback devices, so it cannot be used to play audio concurrently.

  Principle:

  - Create an `AnalyserNode` to obtain real-time audio signal data in time and frequency domains (default using FFT sampling).
  - Use `getByteTimeDomainData` to acquire the most recent time domain data, representing amplitude values (range 0-256, with 128 being silence).
  - Subtract 128 from each sample value to center the waveform data at zero (silence as zero).
  - Square each centered value to calculate the energy of each sample, compute the average energy of all samples.
  - Calculate the square root of the average energy to get the RMS (root mean square) value (when the volume is at its maximum, RMS is 255).

  Implementation

:

  ```tsx
  import React, { useEffect, useRef, useState } from 'react';
  import ProgressBar from '../progress';

  const SpeakerVolume: React.FC<{
    element?: HTMLAudioElement;
    relitive?: boolean;
  }> = ({ element, relitive = false }) => {
    const [volume, setVolume] = useState(0);
    // fucking react re-render on dev mode
    const animationFrameRef = useRef<number>();
    const audioContextRef = useRef<AudioContext>();
    const mediaStreamAudioSourceNodeRef = useRef<MediaElementAudioSourceNode>();
    const analyserNodeRef = useRef<AnalyserNode>();
    const linked = useRef(false);
    const maxVolumeRef = useRef(0);

    useEffect(() => {
      if (!element || linked.current) return;
      maxVolumeRef.current = 0;
      linked.current = true;
      audioContextRef.current = new AudioContext();
      mediaStreamAudioSourceNodeRef.current =
        audioContextRef.current.createMediaElementSource(element);
      analyserNodeRef.current = audioContextRef.current.createAnalyser();
      mediaStreamAudioSourceNodeRef.current.connect(analyserNodeRef.current);
      const dataArray = new Uint8Array(analyserNodeRef.current.frequencyBinCount);

      const onFrame = () => {
        if (analyserNodeRef.current) {
          analyserNodeRef.current.getByteTimeDomainData(dataArray);
          let sum = 0;
          for (let i = 0; i < dataArray.length; i++) {
            sum += (dataArray[i] - 128) * (dataArray[i] - 128);
          }
          const realVolume = Math.sqrt(sum / dataArray.length) / 255;
          maxVolumeRef.current = Math.max(maxVolumeRef.current, realVolume);
          if (relitive) setVolume(realVolume / maxVolumeRef.current);
          else setVolume(realVolume);
        }
        animationFrameRef.current = window.requestAnimationFrame(onFrame);
      };
      window.requestAnimationFrame(onFrame);

      return () => {
        mediaStreamAudioSourceNodeRef.current?.disconnect?.();
        analyserNodeRef.current?.disconnect?.();
        cancelAnimationFrame(animationFrameRef.current!);
        linked.current = false;
        setVolume(0);
      };
    }, [element]);

    return <ProgressBar progress={volume} />;
  };

  export default SpeakerVolume;
  ```


