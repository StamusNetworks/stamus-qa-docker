FROM debian:buster
env DEBIAN_FRONTEND noninteractive
env FILEBEAT_DEB_URL https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.5.2-amd64.deb
env FILEBEAT_RPM_URL https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.5.2-x86_64.rpm
env METRICBEAT_DEB_URL https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-5.5.2-amd64.deb
env METRICBEAT_RPM_URL https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-5.5.2-x86_64.rpm

run  echo 'APT::Default-Release "buster";' > /etc/apt/apt.conf.d/00apt_default_release
add  sources.list /etc/apt/sources.list

run  apt-get update -qy
run  apt-get install -y supervisor git sudo ssh \
        rabbitmq-server easy-rsa sshpass rsync suricata wget jq curl nginx \
        tigervnc-standalone-server x11-apps xterm postgresql-11 postgresql-client libpq-dev \
        build-essential python3-dev libffi-dev libldap2-dev libsasl2-dev libssl-dev virtualenv python3-virtualenv \
        python2-dev chromium chromium-driver

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

# Add packages for log shipping
user admin
run cd /home/admin/packages/deb && wget "$FILEBEAT_DEB_URL"
run cd /home/admin/packages/rpm && wget "$FILEBEAT_RPM_URL"
run cd /home/admin/packages/deb && wget "$METRICBEAT_DEB_URL"
run cd /home/admin/packages/rpm && wget "$METRICBEAT_RPM_URL"

# Install Scirius packages
run virtualenv -p python3 /home/admin/venv
add requirements.txt /home/admin/requirements.txt
run /home/admin/venv/bin/pip install -r /home/admin/requirements.txt --upgrade
run /home/admin/venv/bin/pip install coverage pylint pylint-django

# Robotframework
user root
add requirements-rf.txt /root/requirements-rf.txt
run virtualenv -p python3 /root/rf/
run /root/rf/bin/pip install -r /root/requirements-rf.txt --upgrade

# vnc2flv installation requires python2
run virtualenv -p python2 /root/vnc2flv/
run /root/vnc2flv/bin/pip install vnc2flv

run mkdir /root/.ssh
run mkdir -p /etc/ansible

# Postgresql installation
run mkdir -p /var/run/postgresql/11-main.pg_stat_tmp/ && chown postgres:postgres /var/run/postgresql -R
user postgres
run /etc/init.d/postgresql start && \
    psql --command "CREATE USER docker WITH SUPERUSER PASSWORD 'scient';" && \
    createdb -O docker scient && \
    /etc/init.d/postgresql stop
user root
run echo "host all  all    127.0.0.0/8  md5" >> /etc/postgresql/11/main/pg_hba.conf
run sed -e "s/^log_line_prefix .*/log_line_prefix = '%t [%p]: [%l-1] '/" -i /etc/postgresql/11/main/postgresql.conf
run (echo 'log_min_duration_statement = 0' && \
    echo 'log_checkpoints = on' && \
    echo 'log_connections = on' && \
    echo 'log_disconnections = on' && \
    echo 'log_lock_waits = on' && \
    echo 'log_temp_files = 0' && \
    echo 'log_autovacuum_min_duration = 0' && \
    echo 'log_error_verbosity = default') >> /etc/postgresql/11/main/postgresql.conf

# Install pgbadger
env PGBADGER_VER 11.2
run cd /root && wget https://github.com/darold/pgbadger/archive/v${PGBADGER_VER}.tar.gz
run cd /root && tar xzf v${PGBADGER_VER}.tar.gz && mv pgbadger-${PGBADGER_VER} pgbadger && rm v${PGBADGER_VER}.tar.gz

# nginx configuration
add nginx.conf /etc/nginx/sites-enabled/default
run touch /var/log/gunicorn-error.log /var/log/gunicorn-access.log && \
    chown admin:admin /var/log/gunicorn-error.log /var/log/gunicorn-access.log

run make-cadir /etc/nginx/ca && \
    cd /etc/nginx/ca && bash -c "./easyrsa init-pki && \
        echo | ./easyrsa build-ca nopass && \
        ./easyrsa build-server-full stamus nopass"
run mkdir -p /etc/nginx/ssl/ && cp /etc/nginx/ca/pki/issued/stamus.crt /etc/nginx/ssl/manager.crt && \
    cp /etc/nginx/ca/pki/private/stamus.key /etc/nginx/ssl/manager.key
