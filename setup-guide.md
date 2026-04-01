# Squid Proxy Setup Guide.

This guide explains how to deploy and configure an enterprise Squid proxy server.

---

## 1. Environment Details

Proxy Server: 192.168.198.149
Client Server: 192.168.198.150
Operating System: RHEL 9.7
Proxy Port: 3128
Authenticated Proxy Port: 3129

---

## 2. Install Squid Proxy

Update the system and install the Squid package.

```
dnf install squid -y
```

Verify installation:

```
rpm -qa | grep squid
```

Start and enable the service:

```
systemctl enable --now squid
systemctl status squid
```

---

## 3. Basic Squid Configuration

Edit the main configuration file:

```
vi /etc/squid/squid.conf
```

Add or configure the following rules.

```
acl localnet src 192.168.198.0/24

http_access allow localhost
http_access allow localnet
http_access deny all

http_port 3128
```

Restart the service:

```
systemctl restart squid
```

Test the proxy:

```
curl -x http://192.168.198.149:3128 http://example.com
```

---

## 4. Configure Blocked Websites

Create a blocked websites list.

```
vi /etc/squid/blocked_sites.txt
```

Example:

```
.facebook.com
.youtube.com
.instagram.com
```

Add ACL in squid.conf:

```
acl blocked_sites dstdomain "/etc/squid/blocked_sites.txt"
http_access deny blocked_sites
```

Restart Squid:

```
systemctl restart squid
```

Test:

```
curl -x http://192.168.198.149:3128 http://facebook.com
```

---

## 5. Configure Allowed Websites

Create whitelist file.

```
vi /etc/squid/allowed_sites.txt
```

Example:

```
.google.com
.github.com
.redhat.com
```

Add ACL in squid.conf:

```
acl allowed_sites dstdomain "/etc/squid/allowed_sites.txt"
http_access allow allowed_sites
```

Restart Squid:

```
systemctl restart squid
```

---

## 6. Configure Proxy Authentication

Install required package:

```
dnf install httpd-tools -y
```

Create authentication file:

```
htpasswd -c /etc/squid/passwd user1
```

Add authentication configuration in squid.conf:

```
auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic realm Squid Proxy Authentication
acl authenticated proxy_auth REQUIRED
```

Allow authenticated users:

```
http_access allow authenticated
```

Restart Squid:

```
systemctl restart squid
```

Test authentication:

```
curl -U user1:password -x http://192.168.198.149:3129 http://google.com
```

---

## 7. Time-Based Access Control

Add time-based ACL in squid.conf:

```
acl office_hours time MTWHF 09:00-18:00
```

Block social media during office hours:

```
http_access deny blocked_sites office_hours
```

Restart Squid:

```
systemctl restart squid
```

---

## 8. Transparent Proxy Configuration

Enable intercept mode in squid.conf:

```
http_port 3128 intercept
```

Enable IP forwarding:

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Permanent configuration:

```
vi /etc/sysctl.conf
```

Add:

```
net.ipv4.ip_forward = 1
```

Apply settings:

```
sysctl -p
```

Add firewall redirect rule:

```
firewall-cmd --permanent --direct --add-rule ipv4 nat PREROUTING 0 -s 192.168.198.150 -p tcp --dport 80 -j REDIRECT --to-ports 3128
```

Reload firewall:

```
firewall-cmd --reload
```

Restart Squid:

```
systemctl restart squid
```

Test transparent proxy:

```
curl http://example.com
```

---

## 9. Verify Proxy Logs

Monitor Squid logs:

```
tail -f /var/log/squid/access.log
```

Example log entry:

```
192.168.198.150 TCP_MISS/200 GET http://example.com
```

---

## 10. Project Outcome

This lab demonstrates how enterprise networks deploy proxy servers to:

* control internet access
* enforce security policies
* monitor web traffic
* authenticate users
* block unwanted websites
* implement transparent proxying
