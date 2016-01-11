#!/bin/bash -e

if [ $# -lt 1 ]
then
  echo "No command?"
  exit 1
fi

if [ "$(pidof spotify)" = "" ] 
then
  echo "Spotify is not running"
  exit 2
fi

function track-id {
  track \
  | grep trackid \
  | cut -d'|' -f2 \
  | cut -d':' -f3 
}

function track {
  dbus-send --print-reply --session --dest=org.mpris.MediaPlayer2.spotify /org/mpris/MediaPlayer2 org.freedesktop.DBus.Properties.Get string:'org.mpris.MediaPlayer2.Player' string:'Metadata' \
  | grep -Ev "^method"                           `# Ignore the first line.`   \
  | grep -Eo '("(.*)")|(\b[0-9][a-zA-Z0-9.]*\b)' `# Filter interesting fiels.`\
  | sed -E '2~2 a|'                              `# Mark odd fields.`         \
  | tr -d '\n'                                   `# Remove all newlines.`     \
  | sed -E 's/\|/\n/g'                           `# Restore newlines.`        \
  | sed -E 's/(xesam:)|(mpris:)//'               `# Remove ns prefixes.`      \
  | sed -E 's/^"//'                              `# Strip leading...`         \
  | sed -E 's/"$//'                              `# ...and trailing quotes.`  \
  | sed -E 's/"+/|/'                             `# Regard "" as seperator.`  \
  | sed -E 's/ +/ /g'                            `# Merge consecutive spaces.`
}

function track-format {
  track \
  | grep -E "(title)|(album)|(artist)" 
}

function album-url {
  trackid=$(track-id)
  curl -s https://api.spotify.com/v1/tracks/$trackid | jq '.album.images[0].url' | sed 's/"//g'
}

case $1 in
    "play")
        dbus-send --print-reply --dest=org.mpris.MediaPlayer2.spotify /org/mpris/MediaPlayer2 org.mpris.MediaPlayer2.Player.PlayPause
        ;;
    "next")
        dbus-send --print-reply --dest=org.mpris.MediaPlayer2.spotify /org/mpris/MediaPlayer2 org.mpris.MediaPlayer2.Player.Next
        ;;
    "prev")
        dbus-send --print-reply --dest=org.mpris.MediaPlayer2.spotify /org/mpris/MediaPlayer2 org.mpris.MediaPlayer2.Player.Previous
        ;;
   "getStatus")
        dbus-send --print-reply --dest=org.mpris.MediaPlayer2.spotify /org/mpris/MediaPlayer2 org.freedesktop.DBus.Properties.Get string:'org.mpris.MediaPlayer2.Player' string:'PlaybackStatus'|grep 'string "[^"]*"'|sed 's/.*"\(.*\)"[^"]*$/\1/'
        ;;
   "track")
	track
	;;
   "track-formatted")
	track-format
	;;

   "track-id")
	track-id
	;;
   "album-url")
	album-url
	;;
    *)
        echo "Unknown command: " $1
        ;;
esac
