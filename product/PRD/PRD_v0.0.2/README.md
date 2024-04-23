---
version: v0.0.2
date: 04/23/2024
---

## PRD for ConFlux 0.0.2

### Requirement Description

**Fix Feature Bugs in v0.0.1**

- Implement personal avatar upload and display during meetings
- Fix the issue where meeting statistics are not cleared after the meeting ends

**Meeting Creation and Joining**

- Support meeting scheduling feature
- Support invitation code feature

**Video Background Replacement**

- Allow users to modify the background in settings, options include background blur or replacing with a specific image

**During Meetings**

- Network connection performance optimization
- Media stream display optimization
- Chatroom functionality during meetings
- In-meeting status settings (including raising hand)
- Shortcut to toggle mute
- Optimize meeting display modes, support list display and side-by-side display

### Business Process

- Scheduling a Meeting

  After clicking to create a meeting on the homepage, users are redirected to the join meeting settings. Clicking the create meeting button will directly create a meeting. Clicking the dropdown menu on the right allows choosing to schedule a future meeting. Upon clicking, a dialog box appears where users can select the meeting start time (year, month, day, hour, minute, timezone). After confirming, the meeting ID, organizer, meeting time, and meeting invitation link are displayed, with buttons to copy the meeting information and join the meeting immediately.

  Before the meeting starts, users can join the meeting, but the display will show: Meeting not started

- Implement Invitation Code Feature:

  - Joining a Meeting: After clicking to join a meeting, if an invitation code is required, an invitation code input box will be displayed under the meeting ID input field. After entering the code, click to join the meeting.
  - Creating a Meeting: When creating a meeting, users can select to use an invitation code in the checkbox, and the system will automatically generate a six-digit code.

- Video Background Replacement

  - Allow users to modify the background in settings, which takes effect immediately in the meeting

- Meeting Optimization

  - Fix the issue of redundant Peer connections
  - Only render the video streams of users currently visible on the page
  - Support chat room functionality, allowing users to see messages sent by all participants and send messages
  - In-meeting status settings, providing common emojis, raising hand, etc., on the left side, visible in the user management and user panel's top right corner
  - User panel displays user names, microphone status, video status, hand-raising status, and user avatar
  - Users can press space to toggle mute
  - Users in the meeting header can choose to switch display modes, supporting list display and side-by-side display

