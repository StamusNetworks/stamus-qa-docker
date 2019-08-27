FROM centos:7

run yum update -y
run yum install -y epel-release
run yum install -y openssh-server openssl sudo python-pip suricata libselinux-python rsync monit screen pciutils
run pip install supervisor

run echo 'OPTIONS="-i eth0 --user root "' > /etc/sysconfig/suricata
run chown -R root:root /var/log/suricata

run useradd -m -d /home/ams -p sa1aY64JOY94w ams

run sed -e 's/^Defaults    requiretty/#Defaults    requiretty/' -i /etc/sudoers
run echo 'ams     ALL=(ALL)       NOPASSWD: ALL' > /etc/sudoers.d/ams

run mkdir -p /etc/supervisor/conf.d/
run /usr/bin/echo_supervisord_conf > /etc/supervisor/supervisord.conf
run /bin/echo -e "\n[include]\nfiles=conf.d/*" >> /etc/supervisor/supervisord.conf
run /bin/echo -e "[program:sshd]\ncommand=/usr/sbin/sshd -D\n" > \
    /etc/supervisor/conf.d/sshd.conf
run /bin/echo -e "[program:monit]\ncommand=/usr/bin/monit -I -c /etc/monitrc\n" > \
    /etc/supervisor/conf.d/monit.conf
run /bin/echo -e 'check process ~filebeat matching "filebeat"\n  start program = "/usr/bin/screen -d -m /bin/filebeat.sh -e -c /etc/filebeat/filebeat.yml"\n' > /etc/monit.d/filebeat
run /bin/echo -e 'check process ~metricbeat matching "metricbeat"\n  start program = "/usr/bin/screen -d -m /bin/metricbeat.sh -e -c /etc/metricbeat/metricbeat.yml"\n' > /etc/monit.d/metricbeat
run /bin/ssh-keygen -A

# Work-Around to prevent ansible from calling systemd
run ln -s /usr/bin/true /usr/bin/service

cmd ["/bin/supervisord", "-n"]
