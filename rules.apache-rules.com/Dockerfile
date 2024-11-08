FROM debian:bookworm
env DEBIAN_FRONTEND noninteractive
env HOSTNAME rules.apache-rules.com
env CERT_CN *.apache-rules.com
run apt-get update -qy
run apt-get upgrade -y
run apt-get install -y apache2 easy-rsa wget
run a2enmod ssl && a2ensite default-ssl
run make-cadir /etc/rules-ca
run ls /etc/rules-ca/
run cp /etc/rules-ca/openssl-easyrsa.cnf /etc/rules-ca/openssl.cnf
run sed -e "s/^subjectAltName=.*/subjectAltName=\"DNS:$CERT_CN\"/" -i /etc/rules-ca/openssl.cnf
run export EASYRSA_BATCH=1
WORKDIR /etc/rules-ca
run ./easyrsa init-pki nopass
run echo "gitlab-ci" | ./easyrsa build-ca nopass
run echo "yes" | ./easyrsa build-server-full $HOSTNAME nopass
run sed -e "s#SSLCertificateFile.*#SSLCertificateFile /etc/rules-ca/pki/issued/$HOSTNAME.crt#" -e "s#SSLCertificateKeyFile.*#SSLCertificateKeyFile /etc/rules-ca/pki/private/$HOSTNAME.key#" -i /etc/apache2/sites-available/default-ssl.conf
run find /etc/rules-ca/
run cp /etc/rules-ca/pki/ca.crt /var/www/html/
copy emerging.rules.tar.gz /var/www/html/emerging.rules.tar.gz
run chmod a+r /var/www/html/emerging.rules.tar.gz
cmd ["bash", "-c", "/etc/init.d/apache2 start && tail -f /var/log/apache2/access.log"]
