FROM debian:jessie
env DEBIAN_FRONTEND noninteractive
env FILEBEAT_DEB_URL https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.5.2-amd64.deb
env FILEBEAT_RPM_URL https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.5.2-x86_64.rpm
env METRICBEAT_DEB_URL https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-5.5.2-amd64.deb
env METRICBEAT_RPM_URL https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-5.5.2-x86_64.rpm

run  echo 'APT::Default-Release "jessie";' > /etc/apt/apt.conf.d/00apt_default_release
add  sources.list /etc/apt/sources.list

run  apt-get update -qy
run  apt-get install -y wget
run  echo 'deb http://packages.stamus-networks.com/debian/ jessie main' >> /etc/apt/sources.list
add  packages.stamus-networks.com.gpg.key /root/packages.stamus-networks.com.gpg.key
run  apt-key add /root/packages.stamus-networks.com.gpg.key
run  echo 'deb http://httpredir.debian.org/debian unstable main' >> /etc/apt/sources.list
run  apt-get update -qy
run  apt-get install -y python-pip python-psycopg2 python-gevent python-dev supervisor git sudo ssh \
        rabbitmq-server easy-rsa sshpass rsync libhtp1 libffi-dev libssl-dev suricata \
        vnc4server libldap2-dev libsasl2-dev virtualenv jq curl postgresql-client libpq-dev nginx

run  mkdir /data /var/run/sshd
run  useradd -m -d /home/admin -p sa1aY64JOY94w admin
run  adduser admin sudo
run  adduser admin www-data
user admin
run cat /dev/zero | ssh-keygen -q -N ""
add  config /home/admin/.ssh/config

user root
# Set VNC password to admin123
run  mkdir -p /root/.vnc && echo 'ZrO3AaiJtEI=' | base64 -d > /root/.vnc/passwd && chmod 600 /root/.vnc/passwd

run  sed -Ei 's/^(\%sudo.*)ALL/\1NOPASSWD:ALL/' /etc/sudoers

run  mkdir -p /var/log/celery/ && chown admin:admin /var/log/celery/

user admin
run touch /var/log/celery/ansible.log /var/log/celery/beat.log /var/log/celery/django-auth.log /var/log/celery/django-error.log /var/log/celery/django-security.log /var/log/celery/django-sql.log /var/log/celery/django.log
run  touch /home/admin/django-debug.log
run  mkdir -p /home/admin/ansible /home/admin/git-sources /home/admin/backups /home/admin/django \
    /home/admin/packages/deb /home/admin/packages/rpm /home/admin/upgrade /home/admin/coverage /home/admin/generated

user root
run apt-get install -y iceweasel
run cd /root && wget https://github.com/mozilla/geckodriver/releases/download/v0.24.0/geckodriver-v0.24.0-linux64.tar.gz
run cd /root && tar xvzf geckodriver-v0.24.0-linux64.tar.gz
run mv /root/geckodriver /usr/bin

# Add packages for log shipping
user admin
run cd /home/admin/packages/deb && wget "$FILEBEAT_DEB_URL"
run cd /home/admin/packages/rpm && wget "$FILEBEAT_RPM_URL"
run cd /home/admin/packages/deb && wget "$METRICBEAT_DEB_URL"
run cd /home/admin/packages/rpm && wget "$METRICBEAT_RPM_URL"

# Install Scirius packages
run virtualenv /home/admin/venv
run /home/admin/venv/bin/pip install --upgrade pip
run /home/admin/venv/bin/pip install --upgrade six cffi
add requirements.txt /home/admin/requirements.txt
run /home/admin/venv/bin/pip install -r /home/admin/requirements.txt --upgrade

# pylint-django<2.0, to keep support for Django 1.11
run /home/admin/venv/bin/pip install coverage pylint 'pylint-django<2.0'
#run chown -R admin:admin /home/admin/venv /home/admin/upgrade /home/admin/coverage /home/admin/.coveragerc

user root
add requirements-rf.txt /root/requirements-rf.txt
run virtualenv /root/rf/
run /root/rf/bin/pip install pip --upgrade
run /root/rf/bin/pip install -r /root/requirements-rf.txt --upgrade

#
## Work-around to https://github.com/ansible/ansible/issues/13981
## based on https://github.com/ansible/ansible/commit/70ac47ae61eb3c0cb2e33a342128ef1a7653b17b
#add lineinfile.py /root
#run mv /root/lineinfile.py /usr/local/lib/python2.7/dist-packages/ansible/modules/core/files/
#run rm -f /usr/local/lib/python2.7/dist-packages/ansible/modules/core/files/lineinfile.pyc

run mkdir /root/.ssh
run mkdir -p /etc/ansible
add yaml.py /var/lib/scirius-pro/ansible_library/

# Postgresql installation
run apt-get install -y postgresql-9.4
run mkdir -p /var/run/postgresql/9.4-main.pg_stat_tmp/ && chown postgres:postgres /var/run/postgresql -R
user postgres
run /etc/init.d/postgresql start && \
    psql --command "CREATE USER docker WITH SUPERUSER PASSWORD 'scient';" && \
    createdb -O docker scient && \
    /etc/init.d/postgresql stop
user root
run echo "host all  all    127.0.0.0/8  md5" >> /etc/postgresql/9.4/main/pg_hba.conf
run sed -e "s/^log_line_prefix .*/log_line_prefix = '%t [%p]: [%l-1] '/" -i /etc/postgresql/9.4/main/postgresql.conf
run (echo 'log_min_duration_statement = 0' && \
    echo 'log_checkpoints = on' && \
    echo 'log_connections = on' && \
    echo 'log_disconnections = on' && \
    echo 'log_lock_waits = on' && \
    echo 'log_temp_files = 0' && \
    echo 'log_autovacuum_min_duration = 0' && \
    echo 'log_error_verbosity = default') >> /etc/postgresql/9.4/main/postgresql.conf

# Install pgbadger
env PGBADGER_VER 10.1
run cd /root && wget https://github.com/darold/pgbadger/archive/v${PGBADGER_VER}.tar.gz
run cd /root && tar xzf v${PGBADGER_VER}.tar.gz && mv pgbadger-${PGBADGER_VER} pgbadger && rm v${PGBADGER_VER}.tar.gz

# nginx configuration
add nginx.conf /etc/nginx/sites-enabled/default
run touch /var/log/gunicorn-error.log /var/log/gunicorn-access.log && \
    chown admin:admin /var/log/gunicorn-error.log /var/log/gunicorn-access.log

run make-cadir /etc/nginx/ca && cp /etc/nginx/ca/openssl-1.0.0.cnf /etc/nginx/ca/openssl.cnf && \
    sed -e "s/^subjectAltName=.*/subjectAltName=\"DNS:stamus\"/" -i /etc/nginx/ca/openssl.cnf && \
    cd /etc/nginx/ca && bash -c "source vars && ./clean-all && \
        yes $'\n' | ./build-ca && \
        (yes $'\n' | head -n 5 ; echo $CERT_CN ; \
        yes $'\n' | head -n 4 ; sleep 5s ; echo y; echo y; sleep 5s) | ./build-key-server stamus"
run mkdir -p /etc/nginx/ssl/ && cp /etc/nginx/ca/keys/stamus.crt /etc/nginx/ssl/manager.crt && \
    cp /etc/nginx/ca/keys/stamus.key /etc/nginx/ssl/manager.key
