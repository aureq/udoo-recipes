#!/bin/bash

# xbmc binary
XBMC_BIN='/imx6/xbmc/lib/xbmc/xbmc.bin'

# timestamp file
TIMESTAMP_FILE='/var/run/xbmc-timestamp'


while true
do
	date -u +%s > $TIMESTAMP_FILE

	$XBMC_BIN 2>&1 | logger -t xbmc

	RET="$?"

	LAST_EXEC_TIME=$(cat $TIMESTAMP_FILE)
	CURRENT_TIME=$(date -u +%s)
	TIME_DIFF=$(($CURRENT_TIME - $LAST_EXEC_TIME))

	echo "$XBMC_BIN has exited with code $RET after ${TIME_DIFF}s" | logger -t xbmc

	case "$RET" in
		0) # user initiated quit
			echo "XBMC has exited" | wall -n
			break;
		;;
		64) # shutdown system.
			/sbin/poweroff
			break
		;;
		65) # warm Restart xbmc
			echo "Restarting XBMC..." | wall -n
			sleep 2
		;;
		66) # Reboot System
			/sbin/reboot
			break
		;;
		*) # this should not happen
			if [ "$TIME_DIFF" -le 30 ]
			then
				# xbmc has exited too quick, so we don't restart it
				break
			fi
			sleep 2
		;;
	esac
done
