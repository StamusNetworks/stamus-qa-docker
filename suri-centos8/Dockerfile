FROM centos:8

run yum update -y
run yum install -y epel-release
run yum install -y openssh-server openssl sudo suricata rsync monit screen pciutils python3

run echo 'OPTIONS="-i eth0 --user root "' > /etc/sysconfig/suricata
run chown -R root:root /var/log/suricata

run useradd -m -d /home/ams -p admin ams
run echo ams:admin | chpasswd
run echo root:snpasswd | chpasswd

run sed -e 's/^Defaults    requiretty/#Defaults    requiretty/' -i /etc/sudoers
run echo 'ams     ALL=(ALL)       NOPASSWD: ALL' > /etc/sudoers.d/ams

run /bin/echo -e 'check process ~filebeat matching "filebeat"\n  start program = "/usr/bin/screen -d -m /bin/filebeat -e -c /etc/filebeat/filebeat.yml"\n' > /etc/monit.d/filebeat
run /bin/echo -e 'check process ~metricbeat matching "metricbeat"\n  start program = "/usr/bin/screen -d -m /bin/metricbeat -e -c /etc/metricbeat/metricbeat.yml"\n' > /etc/monit.d/metricbeat
run /bin/echo -e 'check process ~ssh matching "sshd"\n  start program = "/usr/bin/screen -d -m /usr/sbin/sshd -D -e"\n' > /etc/monit.d/sshd
run /bin/ssh-keygen -A

# Work-Around to filebeat/metricbeat restart failing
run echo 'exit 0' > /etc/sysconfig/filebeat
run echo 'exit 0' > /etc/sysconfig/metricbeat

cmd ["bash", "-c", "rm /run/nologin ; /usr/bin/monit -I"]
