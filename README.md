
:local hsactiveuptime; 
:local hslimit;
:local hsuser;
:local hsusedtime;
:local hsuseduptime;
:do {
    
    :foreach i in=[/system script find where comment="temp"] do={
        :set hsuser [/system script get $i name];
        :set hsusedtime [/system script get $i source]
        :set hsuseduptime [/ip hotspot user get $hsuser uptime];
        :set hslimit [/ip hotspot user get $hsuser limit-uptime];
        :if ( [/ip hotspot active find where user=$hsuser] = "") do
            :if ( $hsuseduptime > 0) do={
                /ip hotspot user set $hsuser limit-uptime=($hslimit - ($hsuseduptime - $hsusedtime));
                /ip hotspot user reset-counters $hsuser;
            }
            /system script remove $i;
        }
    }
    
    # Begin Algorithm
    # Check IP hotspot active is not empty
    :if ( [/ip hotspot active print count-only] > 0 ) do={
        # 1. Get IP Hotspot Active Uptime
        # Go through every active users in IP hotspot
        :foreach i in=[/ip hotspot active find] do={
            # Store uptime, user and limit-uptime
            :set hsactiveuptime [/ip hotspot active get $i uptime];
            :set hsuser [/ip hotspot active get $i user];
            :set hslimit [/ip hotspot user get $hsuser limit-uptime];
            # 2.    
            # Check if there is already a script name for that active user
            # !="" means there is a match
            :if ( [/system script find where name=$hsuser] != "") do={
                # hsusedtime gets its value from what is inside the source
                :set hsusedtime [/system script get $hsuser source];
                # Scenario 1: No blackout
                # hsactiveuptime is greater than hsusedtime 
                :if ( $hsactiveuptime > $hsusedtime ) do={
                    :if ( $hslimit > 1 ) do={
                        /ip hotspot user set $hsuser limit-uptime=($hslimit - ($hsactiveuptime - $hsusedtime)); 
                    }
                    :if ( $hslimit <= 1 ) do={
                        /ip hotspot user set $hsuser limit-uptime="00:00:01" disabled=yes comment="No More Uptime";
                        /ip hotspot cookies remove $hsuser;
                        /ip hotspot active remove $hsuser;
                        /system script remove $hsuser;
                    }
                    
                } else={
                    # For future scenario
                    # hsusedtime is less than hsactiveuptime
                    #/ip hotspot user set $hsuser limit-uptime=($hslimit - $hsactiveuptime);
                }
                 /system script set $hsuser source=$hsactiveuptime comment="temp";
            
            # No temp script for that user, will add a temp script 
            } else={
                    /system script add name=$hsuser source=$hsactiveuptime comment="temp";
                    :set hsusedtime [/system script get $hsuser source]; 
                    /ip hotspot user set $hsuser limit-uptime=($hslimit - $hsusedtime);
                    }
        }
    }
}
