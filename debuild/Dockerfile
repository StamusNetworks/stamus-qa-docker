FROM debian:buster

# Work-around broken sources.list from base image (removes jessie-updates repo)
run echo 'APT::Default-Release "buster";' > /etc/apt/apt.conf.d/00apt_default_release
run apt-get update -qy
run apt-get install -y dh-virtualenv dpkg devscripts git python-dev libpq-dev python-setuptools libffi-dev libldap2-dev libsasl2-dev openssh-client apt-utils
run echo 'DEBUILD_DPKG_BUILDPACKAGE_OPTS="-us -uc -I -i"' > /root/.devscripts
run echo 'DEBUILD_LINTIAN_OPTS="-i -I --show-overrides"' >> /root/.devscripts
run git config --global user.email "noreply@stamus-networks.com"
run git config --global user.name "Gitlab CI"
