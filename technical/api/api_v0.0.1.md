---
version: v0.0.1
date: 04/14/2024
---

## API for ConFlux 0.0.1

**basic datastruct**

- ROOM_INFO

  ```javascript
  {
    id: '123456789',
    title: 'Meeting Title',
    organizer: {
      muid: 'abcdefgh-1234-5678-901a-bcdefghijklm',
      name: 'organizer_name',
    },
    start_time: 1234567890123,
    participants: [{USER_INFO}],
  }
  ```

- USER_INFO

  ```javascript
  {
    muid: 'muid',
    name: 'user_name',
    role: 'HOST' | 'PARTICIPANT',
    state: {
      mic: true,
      camera: true,
      screen: false,
    },
    avatar: '',
    expandCamera: true,
    mirrorCamera: true
  }
  ```

**Config**

- [turn,stun] /
- [ws] /peer_signal
- [http/get] /config/:user_id
  - Description: get site config
  - Response:
    ```
    {
      token: 'token',
      COTURN_PREFIX: 'coturn',
      PEER_SERVER_PATH: '/peer_signal',
      COTURN_USERNAME: 'conflux',
      COTURN_PASSWORD: 'conflux',
      BUILD_TIME: '2024-04-21T23:41:48.940Z',
      BUILD_VERSION: '0.0.1',
    }
    ```

**Meeting Management**

- [http/post] /meeting
  - Description: create a meeting
  - Request:
    ```
    header: {
      Authorization: 'Bearer token'
    }
    body: {
      title: 'meeting_title',
      organizer_name: 'user_name',
    }
    ```
  - Response:
    ```
    {
      code: 0,
      data: {
        id: 'meeting_id',
      }
      msg: 'success'
    }
    ```
- [http/get] /meeting/:meeting_id/:name
  - Description: get room info
  - Response:
    ```
    {
      code: 0,
      data: {ROOM_INFO},
      msg: 'success'
    }
    ```

**Room Control**

- [ws] JOIN_MEETING
  - Request:
    ```
    {
      room_id: 'room_id',
      ...{USER_INFO}
    }
    ```
  - Response:
    ```
    {
      message: 'SUCCESS',
      data: {ROOM_INFO},
      type: 'RES_JOIN_MEETING',
    }
    ```
- [ws] LEAVE_MEETING
  - Request:
    ```
    {
      room_id: 'room_id',
      token: 'token',
    }
    ```
  - Response:
    ```
    {
      message: 'SUCCESS',
      type: 'RES_LEAVE_MEETING',
    }
    ```
- [ws] FINISH_MEETING
  - Request:
    ```
    {
      room_id: 'room_id',
      token: 'token',
    }
    ```
  - Response:
    ```
    {
      message: 'SUCCESS',
      type: 'RES_FINISH_MEETING',
    }
    ```

**Meeting Control**

- [ws] UPDATE_USER_STATE
  - Request:
    ```
    {
      room_id: 'room_id',
      token: 'token',
      participant_diff: Partial{USER_INFO}
    }
    ```
  - Response:
    ```
    {
      message: 'SUCCESS',
      type: 'RES_UPDATE_USER_STATE',
    }
    ```
- [ws] REMOVE_USER
  - Request:
    ```
    {
      room_id: 'room_id',
      token: 'token',
      muid: 'muid',
    }
    ```
  - Response:
    ```
    {
      message: 'SUCCESS',
      type: 'RES_REMOVE_USER',
    }
    ```

