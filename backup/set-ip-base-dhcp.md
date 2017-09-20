NICs=$(cat /proc/net/dev | awk {'print $1'} | grep -Ev "^lo|^face|^Inter|^$" | cut -d ":" -f1)

for x in $NICs
do
	DATA=$(ip a show $x | awk ' /inet / {print $2}') 
	IP=$(echo $DATA | cut -d "/" -f1)
	PREFIX=$(echo $DATA | cut -d "/" -f2)
	GATEWAY=$(ip route list | grep $x |awk ' /^default/ {print $3}')
	if [ -z $GATEWAY ]
	then
		GATEWAY=$( ip route list | grep ens7 | grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" | tail -n1)
		if [ ! -z $IP ]	
	then
	echo -e "TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
NAME=$x
DEVICE=$x
ONBOOT=yes
IPADDR=$IP
PREFIX=$PREFIX
DNS1=8.8.8.8" > ifcfg-$x
	fi
	else
		if [ ! -z $IP ]	
	then
	echo -e "TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
NAME=$x
DEVICE=$x
ONBOOT=yes
IPADDR=$IP
PREFIX=$PREFIX
GATEWAY=$GATEWAY
DNS1=8.8.8.8" > /etc/sysconfig/network-scripts/ifcfg-$x
	fi
	fi	
done
