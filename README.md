# F5-ddos-custom-snmp-trap
F5 DDOS custom snmp trap configuration 



1.	/config/user_alert.conf
This file by default is empty.
Add following lines:

alert BIGIP_TS_TS_DOS_ATTACK_DETECTED_ERR_CUSTOM_START "CUSTOM_ASM_DOSL7_ATTACK_START - " {
        snmptrap OID=".1.3.6.1.4.1.3375.2.4.0.500";
}

alert BIGIP_TS_TS_DOS_ATTACK_DETECTED_ERR_CUSTOM_END "CUSTOM_ASM_DOSL7_ATTACK_END - " {
        snmptrap OID=".1.3.6.1.4.1.3375.2.4.0.501";
}


2.	Restart alertd daemon
tmsh restart sys service alertd


3.	custom trap script
(files on folder /var/tmp/ may survive an upgrade, but don’t bet on that)

create the script in /var/tmp folder

vi /var/tmp/dosl7d-logparser.sh

And add below content, save and exit.

#!/bin/bash

# check last minute l7dos logs entries
MYDATE=`date --date='-1 minutes' +"%b %d %H:%M"`
MYLOG=/var/log/dosl7/dosl7d.log
#Set the field separator to new line
IFS=$'\n'


############ START OF A DOS ATTACK #####################
COUNTER=`grep -i "$MYDATE" $MYLOG | grep -i "ATTACK START" | wc -l`
echo "date is $MYDATE counter is $COUNTER"

if [ $COUNTER -gt 0 ]; then
        LOGLINE=`grep -i "$MYDATE" $MYLOG | grep -i "ATTACK START"`
        for j in $LOGLINE
        do
            echo "LOG content is: $j"
            logger -p local0.info "CUSTOM_ASM_DOSL7_ATTACK_START - $j"
        done
else
        echo "nothing to do"
fi


############ END OF A DOS ATTACK   #####################
COUNTER2=`grep -i "$MYDATE" $MYLOG | grep -i "END ATTACK" | wc -l`
echo "date is $MYDATE counter is $COUNTER2"

if [ $COUNTER2 -gt 0 ]; then
        LOGLINE2=`grep -i "$MYDATE" $MYLOG | grep -i "END ATTACK"`
        for j2 in $LOGLINE2
        do
            echo "LOG content is: $j2"
            logger -p local0.info "CUSTOM_ASM_DOSL7_ATTACK_END - $j2"
        done
else
        echo "nothing to do"
fi


Make the script executable :
chmod +x /var/tmp/dosl7d-logparser.sh


4.	cronjob
Configure crontab to run the script every minutes.

crontab –e

and then add below line , save and exit.

* * * * * /var/tmp/dosl7d-logparser.sh


