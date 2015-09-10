# shairport-sync-metadata-reader
Sample Shairport Sync Metadata Player

This sample program reads the pipe of metadata optionally generated by Shairport Sync 2.4 and later. Shairport Sync is an AirTunes emulator with audio synchronization and multi-room capability -- https://github.com/mikebrady/shairport-sync. To use shairport-sync-metadata-reader, read the pipe via standard input, e.g.

```
shairport-sync-metadata-reader < /tmp/shairport-sync-metadata
```

You'll get an output like this:

```
"ssnc" "pfls": "".
"ssnc" "mdst": "".
Album Name: "Pergolesi: Stabat mater".
Artist: "Andreas Scholl, Barbara Bonney, Christophe Rousset & Les Talens Lyriques".
Comment: "".
Composer: "Giovanni Battista Pergolesi".
Genre: "Classical".
File kind: "Purchased AAC audio file".
Title: "Stabat Mater: I. Stabat mater".
Sort as: "Stabat Mater: I. Stabat mater".
"ssnc" "mden": "".
"ssnc" "sndr": "iTunes/12.1 (Macintosh; OS X 10.10.2)".
Picture received, length 39461 bytes.
"ssnc" "prgr": "2373925818/2373941178/2385081354".
"ssnc" "prsm": "".
```
Metadata is not used directly by Shairport Sync. Instead, it is routed to a pipe for other apps to use. All metadata received from the player is sent into the pipe in the order it is received. In addition, some metadata is generated by Shairport Sync itself and sent through the pipe. Metadata is sent in a uniform XML-style format, where each item comprises a `type`, a `code`, the `length` of the data and finally the base64-encoded data, if any. The `type` and `code` are 4-character codes each encoded as 8 hexadecimal digits -- they can be read into C as 32-bit integers. Here are more details:

* We use two 4-character codes to identify each piece of data, the `type` and the `code`.
* The first 4-character code, called the `type`, is either:
 * `core` for all the regular metadadata coming from iTunes, etc., or
 * `ssnc` (for 'shairport-sync') for all metadata coming from Shairport Sync itself, such as start/end delimiters, etc.

* For `core` metadata, the second 4-character code is the 4-character metadata code that comes from iTunes etc. See, for example, https://code.google.com/p/ytrack/wiki/DMAP for information about the significance of the codes. The original data supplied by the source, if any, follows, and is encoded in base64 format. The length of the data is also provided.
* For `ssnc` metadata, the second 4-character code is used to distinguish the messages. Cover art, coming from the source, is not tagged in the same way as other metadata, it seems, so is sent as an `ssnc` type metadata message with the code `PICT`. Progress information, similarly, is not tagged like other source-originated metadata, so it is sent as an `ssnc` type with the code `prgr`.

Here are the 'ssnc' codes defined so far:
 * `PICT` -- the payload is a picture, either a JPEG or a PNG. Check the first few bytes to see which.
 * `pbeg` -- play stream begin. No arguments
 * `pend` -- play stream end. No arguments
 * `pfls` -- play stream flush. No arguments
 * `prsm` -- play stream resume. No arguments
 * `pvol` -- play volume. The volume is sent as a string -- "airplay_volume,volume,lowest_volume,highest_volume", where "volume", "lowest_volume" and "highest_volume" are given in dB. The "airplay_volume" is what's sent by the source (e.g. iTunes) to the player, and is from 0.00 down to -30.00, with -144.00 meaning "mute". This is linear on the volume control slider of iTunes or iOS AirPlay
 * `prgr` -- progress -- this is metadata from AirPlay consisting of RTP timestamps for the start of the current play sequence, the current play point and the end of the play sequence. The timestamps probabably wrap at 2^32.
 * `mdst` -- a sequence of metadata is about to start
 * `mden` -- a sequence of metadata has ended
 * `snam` -- a device e.g. "Joe's iPhone" has started a play session. Specifically, it's the "X-Apple-Client-Name" string.
 * `snua` -- a "user agent" e.g. "iTunes/12..." has started a play session. Specifically, it's the "User-Agent" string.
 * `stal` -- this is an error message meaning that reception of a large piece of metadata, usually a large picture, has stalled; bad things may happen. The stalling phenomenon seems to be a bug in iTunes.

The next two two tokens are to facilitiate remote control of the source. The is some information at http://nto.github.io/AirPlay.html about remote control of the source.
 * `daid` -- this is the source's DACP-ID (if it has one -- it's not guaranteed), useful if you want to remotely control the source. Use this string to identify the source's remote control on the network.
 * `acre` -- this is the source's Active-Remote token, necessary if you want to send commands to the source's remote control (if it has one).

Building Instructions
=====
```
$autoreconf -i -f
$./configure
$make
$sudo make install
```
