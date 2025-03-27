FROM debian:bullseye

env DEBIAN_FRONTEND noninteractive
env FILEBEAT_DEB_URL https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-oss-7.16.2-amd64.deb
env METRICBEAT_DEB_URL https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-oss-7.16.2-amd64.deb

run apt-get update
run apt-get install -y psmisc sudo openssh-server supervisor rsync monit \
    libcap-ng0 libgeoip1 libjansson4 libluajit-5.1-2 \
    libnet1 libnspr4 libnss3 libpcap0.8 libyaml-0-2 python3 \
    pciutils iproute2 suricata wget redis easy-rsa
run cd /tmp && wget $FILEBEAT_DEB_URL $METRICBEAT_DEB_URL && dpkg -i filebeat*.deb metricbeat*.deb
run mkdir -p /data /var/run/sshd /var/run/suricata /etc/ansible/facts.d/ /etc/monit/monitrc.d /etc/stamus \
    /var/log/suricata/core
# Set supervisord buildbot and sshd processes
run /bin/echo -e "[program:sshd]\ncommand=/usr/sbin/sshd -D\n" > \
    /etc/supervisor/conf.d/sshd.conf
run /bin/echo -e "[program:monit]\ncommand=/usr/bin/monit -I -c /etc/monit/monitrc\n" > \
    /etc/supervisor/conf.d/monit.conf
add monitrc /etc/monit/monitrc
run chmod 600 /etc/monit/monitrc
run /bin/echo -e 'check process ~suricata with pidfile /var/run/suricata.pid\n  start program = "/etc/init.d/suricata start"\n  stop program = "/etc/init.d/suricata stop"\n' > /etc/monit/conf.d/suricata
run /bin/echo -e 'check process ~redis-server with pidfile /var/run/redis/redis-server.pid\n  start program = "/etc/init.d/redis-server start"\n  stop program = "/etc/init.d/redis-server stop"' > /etc/monit/conf.d/redis
run useradd -m -d /home/snuser -p snpasswd snuser
run adduser snuser sudo
run echo snuser:snpasswd | chpasswd
run useradd -m -d /home/ams -p admin ams
run adduser ams sudo
run echo ams:admin | chpasswd
run sed -Ei 's/^(\%sudo.*)ALL/\1NOPASSWD:ALL/' /etc/sudoers
add version.fact /etc/ansible/facts.d/version.fact
add model.fact /etc/ansible/facts.d/model.fact
run sed -e 's/RUN=.*/RUN=yes/' -i /etc/default/suricata
run sed -e 's/nfqueue/af-packet/' -i /etc/default/suricata
run sed -e 's/# unixsocket\(.*\)$/unixsocket\1/' -i /etc/redis/redis.conf
run sed -e 's/redis-server\.sock/redis.sock/' -i /etc/redis/redis.conf
run rm -f /usr/bin/systemctl && ln -s /bin/true /usr/bin/systemctl
run ln -s /var/suricata-command.socket /run/
USER root
cmd ["/usr/bin/supervisord", "-n"]
