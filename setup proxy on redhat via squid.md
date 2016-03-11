## Set the proxy computer
Check avaliable ports：
`netstat -tanp | grep LISTEN`

Download squid：
`yum install squid`

Backup config file：
`cp /etc/squid/squid.conf /etc/squid/squid.conf.original`

Modify config file：
`vim /etc/squid/squid.conf`

config file on going：
```
#
# Recommended minimum configuration:
#

# Example rule allowing access from your local networks.
# Adapt to list your (internal) IP networks from where browsing
# should be allowed
acl localnet src 10.0.0.0/8     # RFC1918 possible internal network
acl localnet src 169.254.0.0/16 # RFC1918 possible internal network
acl localnet src 192.168.0.0/16 # RFC1918 possible internal network
acl localnet src fc00::/7       # RFC 4193 local private network range
acl localnet src fe80::/10      # RFC 4291 link-local (directly plugged) machines

acl SSL_ports port 443
acl Safe_ports port 80          # http
acl Safe_ports port 21          # ftp
acl Safe_ports port 443         # https
acl Safe_ports port 70          # gopher
acl Safe_ports port 210         # wais
acl Safe_ports port 1025-65535  # unregistered ports
acl Safe_ports port 280         # http-mgmt
acl Safe_ports port 488         # gss-http
acl Safe_ports port 591         # filemaker
acl Safe_ports port 777         # multiling http
acl CONNECT method CONNECT

#
# Recommended minimum Access Permission configuration:
#
# Deny requests to certain unsafe ports
http_access deny !Safe_ports

# Deny CONNECT to other than secure SSL ports
http_access deny CONNECT !SSL_ports

# Only allow cachemgr access from localhost
http_access allow localhost manager
http_access deny manager

# We strongly recommend the following be uncommented to protect innocent
# web applications running on the proxy server who think the only
# one who can access services on "localhost" is a local user
#http_access deny to_localhost

#
# INSERT YOUR OWN RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS
#

# Example rule allowing access from your local networks.
# Adapt localnet in the ACL section to list your (internal) IP networks
# from where browsing should be allowed
http_access allow localnet
http_access allow localhost

# And finally deny all other access to this proxy
http_access deny all

# Squid normally listens to port 3128
http_port 3128
icp_port 0

# Uncomment and adjust the following to add a disk cache directory.
cache_dir ufs /var/lib/ceph/squidcache 100 16 256

# Leave coredumps in the first cache dir
coredump_dir /var/lib/ceph/squidcache

#
# Add any of your own refresh_pattern entries above these.
#
refresh_pattern ^ftp:           1440    20%     10080
refresh_pattern ^gopher:        1440    0%      1440
refresh_pattern -i (/cgi-bin/|\?) 0     0%      0
refresh_pattern .               0       20%     4320
refresh_pattern (Release|Packages(.gz)*)$      0       20%     2880
#
# redirect the flow to the main proxy.
#
cache_peer 135.245.48.12 parent 8000 0 no-query default
never_direct allow all
```

modify cache directory：
`mkdir /var/lib/ceph/squidcache`

modify cache directory owner：
`chown squid:squid /var/lib/ceph/squidcache`

set up squid：
```
chkconfig --level 345 squid on
chkconfig --list squid    
squid -z    
```

restart squid in case of modifing the config file：
```
restorecon -R /var/spool/squid
service squid restart
```

##Set environment on LAN computers：
```
export http_proxy=192.168.10.50:3128
export ftp_proxy=192.168.10.50:3128
export rsync_proxy=192.168.10.50:3128
export https_proxy=192.168.10.50:3128
```
