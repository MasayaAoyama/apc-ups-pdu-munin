# APC UPS/PDU plugins for Munin

See also https://github.com/ndonegan/munin-contrib/tree/master/plugins/ups

# How to Use

If you have a question, please contact to me.

## Install Munin and Munin Node

First of all, install software.
(Example of server IP: 192.168.1.1)

    yum --enable-repo=epel install munin munin-cgi mod_fcgid munin-node
    yum install snmpwalk

And then, set attribute for access control.

    cat << 'EOF' >> /etc/munin/munin-node.conf
    allow ^192\.168\.1\.1$
    EOF

## APC Setting

To use this plugin, there are 3 steps.

1. Download MIB file from APC website.
2. Download and set up this plugin.
3. Set up Munin setting.

### 1. Download MIB file from APC website.

Set up MIB file directory.

    mkdir -p /root/.snmp/mibs
    cat << 'EOF' >> /root/.snmp/snmp.conf
    mibdirs /usr/share/snmp/mibs:/root/.snmp/mibs
    mibs all
    EOF

Download MIB file.

    cd /root/.snmp/mibs/
    wget ftp://ftp.apc.com/apc/public/software/pnetmib/mib/414/powernet414.mib
    chmod 644 ./powernet414.mib

### 2. Download this plugin.

Download this plugin.

    cd
    git clone https://github.com/MasayaAoyama/apc-ups-pdu-munin.git
    cp -ai apc-ups-pdu-munin/apc__upspdu /usr/share/munin/plugins/apc__upspdu

Set community name to variable $COMMUNITYNAME in line 39 of this script.
Set path of powernet mib file to variable $POWERNETMIB in line 40 of this script.

    vi /usr/share/munin/plugins/apc__upspdu

    ...
    COMMUNITYNAME="public"
    POWERNETMIB="/root/.snmp/mibs/powernet414.mib"
    ...

Chenge permission.

    chmod 755 /usr/share/munin/plugins/apc__upspdu

### 3. Set up Munin setting.

Add execute permission.

    cat << 'EOF' > /etc/munin/plugin-conf.d/apc
    [apc_*]
    user root
    EOF

Register each UPS or PDU equipments.

    ln -s /usr/share/munin/plugins/apc__upspdu /etc/munin/plugins/apc_ups01.a-msy.jp_ups
    ln -s /usr/share/munin/plugins/apc__upspdu /etc/munin/plugins/apc_pdu01.a-msy.jp_pdu

    cat << 'EOF' > /etc/munin/conf.d/apc.conf
    # UPS
    [ups01.a-msy.jp]
        address 192.168.1.1
        use_node_name no
    # PDU
    [pdu01.a-msy.jp]
        address 192.168.1.1
        use_node_name no
    EOF


