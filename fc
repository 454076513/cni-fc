#!/bin/bash
#
# Depend on tc,iptables and jq
# 

log=/var/log/fc.log
time=$(date "+%m/%d/%y %H:%M:%S")

# log tag content
function log {
	echo "[${time}]: $*" >> $log
}


cni_env_log=$(printf "cni_command:%s ,cni_ifname:%s, cni_containerid:%s, cni_netns:%s,cni_args:%s ,cni_path:%s " \
					$CNI_COMMAND  $CNI_IFNAME 	 $CNI_CONTAINERID    $CNI_NETNS   $CNI_ARGS	   $CNI_PATH )
log "CNI_Env"  "$cni_env_log"

dev=$CNI_IFNAME

#iptables
function ips {
    echo " $1 enter : $2"
    jump=`echo $1 | tr [:lower:] [:upper:]`
    for row in `echo $2 | jq -r ".[] | @base64"` ; do
        IPTABLES="iptables -A INPUT -i $dev -p tcp -j $jump" 
        function _jq {
            echo ${row} | base64 --decode | jq -r ${1}
        }
        ip=`_jq ".ip"`
        port=`_jq ".port"`

        if [ $ip != "null" ]; then  
            IPTABLES="$IPTABLES -s $ip"
        fi 
        if [ $port != "null" ]; then  
            IPTABLES="$IPTABLES --dport $port"
        fi 
        log $IPTABLES
        $($IPTABLES)
    done
}

function htb {
    log " $1 enter : $2"
    root="1:0"
    tc qdisc add dev eth0 root handle $root htb default 1
    classid=100
    for row in `echo $2 | jq -r ".[] | @base64"` ; do
        function _jq {
            echo ${row} | base64 --decode | jq -r ${1}
        }

        u32="tc filter add dev $dev parent $root protocol ip prio 16 u32 flowid 1:$classid" 
        
        ip=`_jq ".ip"`
        port=`_jq ".port"`
        
        if [ $ip != "null" ]; then  
            u32="$u32 match ip dst $ip"
        fi 
        if [ $port != "null" ]; then  
            u32="$u32 match ip sport $port 0xffff"
        fi 

        class="tc class add dev $dev parent $root classid 1:$classid htb" 

        rate=`_jq ".rate"`
        ceil=`_jq ".ceil"`
        burst=`_jq ".burst"`
        prio=`_jq ".prio"`

        if [ $rate != "null" ]; then  
            class="$class rate $rate"
        fi 
        if [ $ceil != "null" ]; then  
            class="$class ceil $ceil"
        fi 
        if [ $burst != "null" ]; then  
            class="$class burst $burst"
        fi 
        if [ $prio != "null" ]; then  
            class="$class prio $prio"
        fi 

        log $u32
        log $class
        $($u32)
        $($class)

        classid=$(( $classid + 1 ))
    done
}

function drop {
    # echo " drop enter :$1"
    ips "drop" "$1"
}

function reject {
    ips "reject" "$1"
}

function bandwidth {
    #TODO ? Classless:tbf,fifo,fifo,
    #TODO ? Classful:CBQ
    htb "htb" "$1"
}

function accept {
    ips "accept" "$1"
}



# Read stdin
config=$(cat /dev/stdin)

case $CNI_COMMAND in
ADD)
    log "Add command conf" ${config}

    # fc=$(echo $config | jq -r '.plugins[] | select(.type == "fc")'  )
    rules=$(echo $config | jq -r '.rules'  )
    rule_key=$(echo $rules | jq -r 'keys | .[]'  )
    
    for k in $rule_key ; do

        value=$(echo $rules | jq -r ".$k")
            
        case $k in
            accept)
                accept "$value"
            ;;
            drop)
                drop "$value"
            ;;
            reject)
                reject "$value"
            ;;
            bandwidth)
                bandwidth "$value"
            ;;
            *)
                log "Unknown rule: $k" 
                # exit 1
            ;;
        esac
    done

  	
    
;;

DEL)
   log "Del command conf" ${config} 
;;

GET)
    log "Get command conf" ${config} 
;;

VERSION)
    log "Version command conf" ${config} 
    echo '{
    "cniVersion": "0.3.1", 
    "supportedVersions": [ "0.3.0", "0.3.1", "0.4.0" ] 
    }'

;;

*)
  log "Unknown cni command: $CNI_COMMAND" 
  exit 1
;;

esac





