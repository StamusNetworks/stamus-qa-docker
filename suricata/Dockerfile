FROM debian:jessie

env DEBIAN_FRONTEND noninteractive
env FILEBEAT_DEB_URL https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.5.2-amd64.deb
env METRICBEAT_DEB_URL https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-5.5.2-amd64.deb

run echo 'APT::Default-Release "jessie";' > /etc/apt/apt.conf.d/00apt_default_release
add sources.list /etc/apt/sources.list
run apt-get update
run apt-get install -y wget
run wget -O - -q http://packages.stamus-networks.com/packages.stamus-networks.com.gpg.key | apt-key add -
add stamus-packages.list /etc/apt/sources.list.d/
run apt-get update
run apt-get install -y psmisc sudo openssh-server supervisor rsync monit \
        libcap-ng0 libgeoip1 libjansson4 libluajit-5.1-2 \
        libnet1 libnspr4 libnss3 libpcap0.8 libyaml-0-2 python-simplejson \
        pciutils
add debs/* /root/
run dpkg -i /root/*.deb
run mkdir /data /var/run/sshd
# Set supervisord buildbot and sshd processes
run /bin/echo -e "[program:sshd]\ncommand=/usr/sbin/sshd -D\n" > \
    /etc/supervisor/conf.d/sshd.conf
run /bin/echo -e "[program:monit]\ncommand=/usr/bin/monit -I -c /etc/monit/monitrc\n" > \
    /etc/supervisor/conf.d/monit.conf
add monitrc /etc/monit/monitrc
run chmod 600 /etc/monit/monitrc
run /bin/echo -e 'check process ~suricata with pidfile /var/run/suricata.pid\n  start program = "/etc/init.d/suricata start"\n  stop program = "/etc/init.d/suricata stop"\n' > /etc/monit/conf.d/suricata
run useradd -m -d /home/snuser -p "wqm/2NFEczfE6" snuser
run adduser snuser sudo
run useradd -m -d /home/ams -p sa1aY64JOY94w ams
run adduser ams sudo
run sed -Ei 's/^(\%sudo.*)ALL/\1NOPASSWD:ALL/' /etc/sudoers
run mkdir /var/run/suricata/
run mkdir -p /etc/ansible/facts.d/
add version.fact /etc/ansible/facts.d/version.fact
add model.fact /etc/ansible/facts.d/model.fact
run cd /tmp && wget $FILEBEAT_DEB_URL $METRICBEAT_DEB_URL && dpkg -i filebeat*.deb metricbeat*.deb
add suricata.initd /etc/init.d/suricata
USER root
cmd ["/usr/bin/supervisord", "-n"]
