---
version: v0.0.1-1
date: 03/23/2024
---

## PRD for ConFlux 0.0.1

### Requirement Description

- Implement the website homepage, including options to join a meeting and create a meeting instantly

  Display three buttons in the center of the page: Create Meeting, Join Meeting, User Settings

- Implement the meeting configuration page (meeting name, meeting number, username, microphone configuration, camera configuration, speaker configuration, more settings)

  Display the meeting title at the top

  Display preview information on the left; if the user turns on the camera, display the camera content, otherwise display a button to turn off the camera. Arrange speaker and dropdown menus below, click on the speaker to switch to mute, click on the dropdown menu to switch playback devices, microphone, camera, etc. Display the settings button on the right, click to switch local user settings

  Arrange from top to bottom on the right, meeting number, username, join meeting button

  If it's a small screen, arrange them vertically

- Implement the meeting interface (meeting title, meeting time, meeting invitation link, microphone configuration, camera configuration, speaker configuration, screen sharing, user management, leave meeting, end meeting)

  Display the meeting title, share button, meeting duration at the top, display users in the meeting in the middle using a 3*4 layout to paginate user information. If a user is sharing their screen or has turned on their camera, display user video, otherwise display a surrendering user or a random avatar. From left to right below: microphone configuration, camera configuration, speaker configuration, screen sharing button, user management button, local user settings, leave meeting button

- Implement the user management interface (display users, set mute, kick out users)

  A floating panel displays the current user list, with user avatars on the left, followed by user nicknames, then network status (a red signal symbol when the signal is bad), and finally user microphone (icon displayed when on, red microphone when off), camera (icon displayed when on, nothing when off), and screen sharing status (icon displayed when on, nothing when off). Admins can click on icons to toggle states, and a dropdown button is provided for admins to switch mute, turn off the camera, stop screen sharing, and kick out users

- Implement the local user settings panel

  - Users can set their own username, avatar (if the user hasn't set an avatar, update the avatar when updating the username)
  - Users can configure default behaviors
    - Whether to automatically turn on the camera when joining a meeting
    - Whether to automatically turn on the microphone when joining a meeting
    - Whether to automatically mute when joining a meeting
    - Current and default microphone volume
    - Current and default speaker volume
    - Display network quality information

### Business Process

**Creating a Meeting**

- The host clicks to create a meeting immediately
- Enter the meeting name, host name, click to start the meeting

**Joining a Meeting**

- The user clicks to join a meeting, enters the username, meeting number, configures media capture devices, and enters the meeting
- Users can choose whether to turn on the camera, which camera to use, whether to use a microphone, which microphone to use, and the meeting volume. Provide a settings button, click to jump to detailed settings

**Screen Sharing**

- Globally, only one user can share their screen at a time
- The sharer can click the screen share button again to share a new screen
- The sharer will display a new panel when sharing the screen to set sharing computer audio, stop sharing

**Host Management**

- There is only one host when a meeting is created, the host can choose to set a participating user as a host or remove a host
- If all hosts leave the meeting without ending it, the system will randomly select a user as the host

**User Management**

- Both users and hosts can view current users
- Hosts can mute users, unmute, change usernames, kick out users

**Leaving the Meeting**

- Both regular users and hosts can leave the meeting at any time and then rejoin. The system should make a sound alert when leaving and joining
- The host can choose to end the meeting actively, at which point everyone will leave the meeting

### Bugs

-

### TODO

**Creating a Meeting**

- The host can choose to create a meeting immediately or later
- Invite only certain users (based on online user settings / based on invitation code)

**Waiting Room**

- For scheduled meetings, users can log in in advance, enter the waiting room, until the host enters the meeting and clicks to start the meeting

**Meeting Room**

- Meeting display mode
- Raise hand / emoji
- Shortcut to toggle mute

**Meeting Chat Room**

- Users can communicate one-on-one with other users or group chat in the chat room

**Meeting Collaboration Board**

- Users can annotate on the screen shared by others, and the annotation content is visible globally
- Users can start a blank sharing panel for annotation, and the content is visible globally

**User Virtual Background**

- Users can choose to use a virtual background when joining a meeting, in user settings, or during the meeting

**User Settings**

- Configure online user settings
- Users can in their personal page
- Chat content display mode
- Default virtual background
- Adaptive transmission resolution
