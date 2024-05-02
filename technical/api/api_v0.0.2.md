---
version: v0.0.2
date: 05/03/2024
---

## API for ConFlux 0.0.2


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

**Avatar**

- [http/get] /avatar?fileName=123.png
  - Description: get avatar by file name
  - Response: image

- [http/post] /avatar
  - Description: get avatar by file name
  - Request: image
  - Response:
    ```json
    {
      "code": 0,
      "data": "123.png",
      "msg": "success"
    }
    ```


**Meeting Management**

- [http/post] /meeting
  - Description: create a meeting
  - Request:
    ```json
    "header": {
      "Authorization": "Bearer token"
    }
    "body": {
      "title": "meeting_title",
      "organizer_name": "user_name",
      "start_time": 123456789,
      "passcode": "" | "123456"
    }
    ```
  - Response:
    ```json
    {
      "code": 0,
      "data": {
        "id": 123456789,
      }
      "msg": "success"
    }
    ```
- [http/get] /meeting?id=123456789&name=myname&passcode=123456
  - Description: get room info
  - Response:
    ```json
    {
      "code": 0,
      "data": {ROOM_INFO},
      "msg": "success"
    }
    ```

**Room Control**

- [ws] BOARDCAST_CHAT
  - Request:
    ```json
    {
      "room_id": 123456789,
      "token": "token",
      "muid": "muid",
      "message": "hihi",
      "time": 123456789
    }
    ```
