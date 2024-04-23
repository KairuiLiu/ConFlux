---
version: v0.0.1
date: 03/23/2024
---

## PRD for ConFlux 0.0.1

### Requirements Description

- Implement the website homepage, including options to join a meeting and create a meeting instantly.

  Display three buttons centered on the page: Create Meeting, Join Meeting, User Settings.

- Implement the meeting configuration page (meeting name, meeting number, username, microphone settings, camera settings, speaker settings, whether to fill the camera, whether to mirror the camera).

  Display the meeting title at the top.

  Show preview information on the left; if the camera is active, display its feed, otherwise, show a camera-off button. Below, arrange speakers and a dropdown menu; click speakers to toggle mute, and the dropdown to switch playback devices. Similarly for microphones and cameras. On the right, display a settings button to toggle local user settings.

  Arrange vertically on the right, from top to bottom, the meeting number, username, and Join Meeting button.

  For small screens, arrange items vertically.

- Implement video conference network statistics:

  - Network: bandwidth, packet loss, latency
  - Audio: bitrate
  - Video: resolution, frame rate, bitrate
  - Screen sharing: resolution, frame rate, bitrate

- Implement the meeting interface (meeting title, meeting time, meeting invitation link, microphone settings, camera settings, speaker settings, screen sharing, user management, leave meeting, end meeting).

  Display the meeting title, share button, and meeting duration at the top, show participants in the meeting in the middle using a 3x4 layout to paginate user information. If a user is sharing their screen or has their camera on, display their video, otherwise, show a user's raised hand or a random avatar. Below, from left to right: microphone settings, camera settings, speaker settings, screen sharing button, user management button, local user settings, leave meeting button.

- Meeting panel layout
  - Number of elements on the page: 1-12, element sizes should adjust adaptively.
  - Aspect ratio of elements: 0.75 - 1.
  - Up to 4 rows (if only one row, row height is container height, but element height can vary).
  - Up to 4 columns (if only one column, column width is container width, but element width can vary).
  - Element sizes should be as large as possible.
  - Prioritize displaying ongoing screen shares, then oneself, then the host, and finally everyone.

- Implement the user management interface (display users, set mute, kick out users).

  Floating panel showing the current user list, with user avatars on the left, followed by user nicknames, then network status (red signal icon when the connection is poor) and finally user microphone (icon visible when on, red microphone when off), camera (icon visible when on, otherwise hidden), and screen sharing status (icon visible when on, otherwise hidden). Admins can toggle these states by clicking the icons and also have a dropdown button for renaming users, switching roles, muting, turning off cameras, stopping screen sharing, and kicking out users.

  List oneself first, then the host, and finally everyone else.

- Implement local user settings panel.

  - Users can set their username, avatar (if no avatar is set, update it when updating the username).
  - Users can configure default behaviors:
    - Whether to automatically turn on the camera when joining a meeting.
    - Whether to automatically turn on the microphone when joining a meeting.
    - Whether to automatically mute when joining a meeting.
    - Display network quality information.
    - Default camera to use.
    - Default microphone to use, with a volume bar (display using fft, softmax).
    - Default speakers to use, with a volume bar, and a test button.


### Business Processes

**Creating a Meeting**

- The host clicks to create a meeting immediately.
- Enter the meeting name, host name, and click start meeting.

**Joining a Meeting**

- Users click to join a meeting, enter their username, meeting number, and configure media capture devices to enter the meeting.
- Users can choose whether to turn on the camera, which camera to use, whether to use a microphone, which microphone to use, and the meeting volume. Provide a settings button to access detailed settings.

**Screen Sharing**

- Only one user can activate screen sharing at a time.
- The sharer can click the screen sharing button again to share a new screen.
- While sharing the screen, a new panel will be displayed for setting shared computer audio and stopping sharing.

**Host Management**

- Initially, there is only one host in a meeting. The host can designate or remove other users as hosts.
- If all hosts leave the meeting without ending it, the system will randomly select a user to become the host.

**User Management**

- Both users and hosts can view current participants.
- Hosts can

 modify usernames, set user roles, mute, kick out users.

**Exiting a Meeting**

- Ordinary users and hosts can leave and rejoin the meeting at any time, with audible cues during exiting and joining.
- Hosts can choose to end the meeting voluntarily, which will cause all participants to exit the meeting.

**Handling Unexpected Exits**

- Screen sharing stops automatically if it exceeds 15 seconds.
- If there is no voice activity for 1 minute, the user will automatically exit the meeting.

### Plans for Next Version

**Creating a Meeting**

- Hosts can choose to create a meeting immediately or later.
- Invite only specific users (based on invitation codes).

**Meeting Room**

- Display mode for meetings.
- Raise hand / emoji reactions.
- Shortcut to toggle mute.
- Render only the video streams of users currently visible on the page.

**Meeting Chat Room**

- Users can interact one-on-one or in groups in the chat room.

**Meeting Collaboration Board**

- Users can annotate on others' shared screens, with annotations visible globally.
- Users can open a blank shared panel for annotations, visible globally.

**User Virtual Backgrounds**

- Users can choose to use virtual backgrounds when joining a meeting, in user settings, or during the meeting.

**User Settings**

- Chat content display options.
- Default virtual background.

**Network Status Statistics**

- Automatically clear network status statistics.
