# API Specifications

## TL;DR:

API Endpoint | Implemented? | Arguments | Purpose
------------ | ------------ | --------- | -------
/brightness  | No | None  | Get ambient light intensity
/speed       | Yes | None  | Get current speed (-8 to 8)
/speed/set-relative | Yes | value:int, [force:int] | Set speed with relative offset
/speed/set-absolute | Yes | value:int | Set absolute speed
/heading/set-absolute | Yes | value:int | Command a turn correct to 15 degrees.
/status | Partial | None | Request for Status Report *(Obstacle flag is not complete.)
/status/enable-parasitic | No | None | Enable Parasitic Reports
/status/disable-parasitic | No |  None | Disable Parasitic Reports


## Sensor Readout

### Reading of Ambient Light Intensity

You can read out the photocell on the device, to figure out how bright the ambience is.

`GET /brightness`

##### Parameters:

- None.

##### Return value:

`{"status": "ok", "brightness": 384}`

The brightness value is a value between 0 to 1024. More details to follow.

## Commanding Vehicle

### Setting and Getting Speed

The speed of the vehicle is defined as a value from -8 to 8. Positive or negative denotes going forward or backward. To read out the speed, simply call:

`GET /speed`

##### Parameters:

- None.

##### Return value:

`{"status": "ok", "speed": 7}`

#### Setting speed with absolute value

If you wish to set the speed with an absolute speed value, use this API:

`GET /speed/set-absolute`

##### Parameters:

- `value`: Integer. Target speed value. If you set a value out of bound (such as 9), the result will be the maximum vehicle speed in that direction.

##### Return value:

`{"status": "ok", "speed": 7}`

#### Setting speed with relative value

If you wish to perform an acceleration or deceleration, use this API:

`GET /speed/set-relative`

##### Parameters:

- `value`: Integer. Delta value added to the speed. If the vehicle is moving forward, positive delta means acceleration, else if the vehicle is moving backwards, negative delta means acceleration.
- `force`: Integer, 0 or 1. Defaults to 0, meaning if the delta causes the vehicle to change direction (e.g. current speed 2, you want to set a delta of -3), it will simply stop. Setting this flag to 1 allows you to change direction directly.

##### Return value:

`{"status": "ok", "speed": 7}`

### Setting Heading

When commanding vehicle to perform a turn, it will not move forward, and lateral speed is **reset** to 0.

`GET /heading/set-relative`

##### Parameters:

- `value`: Integer. Clockwise angle of turn. A positive value turns to the right, vice versa.

##### Return value:

`{"status": "ok"}`


## Status Update

A vehicle status update always contain the following parameters:
- `steps-traversed`: Integer. Motor steps traversed since last report.
- `obstacle`: Integer. 0 if no obstacle was encountered since last report, 1 if the vehicle has stopped due to obstacle.
- `sys-status`: Integer. 0 if system is running fine. (I think there won't be any error codes but just leave it there)

### Passive Status Request

You may request the vehicle for a status update at any time:

`GET /status`

##### Parameters:

- None.

##### Return value:

`{"status": "ok", "steps-traversed": 10, "obstacle": 0, "sys-status":0}`

### Parasitic Status Request

You may also ask the vehicle to append the status update during any vehicle drive functions. This includes all `/speed` and `/heading` APIs. You basically have these three fields in the JSON text of the API returns.

Parasitic Status is disabled by default. **Enable with caution** because all status reports are **relative to the last status report**, even parasitic ones. I.e. If you didn't note down the values in your app, they are lost.

To enable or disable them, just:

`GET /status/enable-parasitic`

`GET /status/disable-parasitic`
