## Control Snapcast and MPD using MIDI controller "SubZero MiniControl"

Snapcast can be controlled using a JSON-RPC API over HTTP.
For example: set client's volume and mute clients.
curl is used to send requests and receive responses.
jq is used to filter JSON data and make it more readable

MPD can be controlled by MPC.

aseqdump (ALSA sequencer dump) is used to read the MIDI controller data.
Like: number of the controller (fader, knob, button) and controller value

## SubZero MiniControl Layout
 
  Channels 1-4

###   Buttons, faders, knobs and their controller numbers
|ids[0]| ids[1]| ids[2]| ids[3]| ---- |
|-|-|-|-|-|
| K 14 | K 15 | K 16 | K 17 |  ids index = controller no - 14
| F 3  | F 4  | F 5  | F 6  |  ids index = controller no - 3
| B 23 | B 24 | B 25 | B 26 |  ids index = controller no - 23
 |    1 |    2 |    3 |    4 |

     ids[]: Snapclient IDs assigned to channels
     K = knob / F = fader / B = button
     1-4: channel number printed on the SubZero MiniControl

### Media buttons and their controller numbers
| Repeat | Previous | Next | Stop    | Play | Record  |
|---------|---------|---------|---------|---------|---------|   
|  49 |  47 |  48 |  46 |  45 |  44 |
