FROM debian:stretch
env DEBIAN_FRONTEND noninteractive
env HOSTNAME rules.apache-rules.com
env CERT_CN *.apache-rules.com
run apt-get update -qy
run apt-get upgrade -y
run apt-get install -y apache2 easy-rsa wget
run a2enmod ssl && a2ensite default-ssl
run make-cadir /etc/rules-ca
run cp /etc/rules-ca/openssl-1.0.0.cnf /etc/rules-ca/openssl.cnf
run sed -e "s/^subjectAltName=.*/subjectAltName=\"DNS:$CERT_CN\"/" -i /etc/rules-ca/openssl.cnf
run cd /etc/rules-ca && bash -c "source vars && ./clean-all && \
        yes $'\n' | ./build-ca && \
        (yes $'\n' | head -n 5 ; echo $CERT_CN ; \
        yes $'\n' | head -n 4 ; sleep 5s ; echo y; echo y; sleep 5s) | ./build-key-server $HOSTNAME"
run sed -e "s#SSLCertificateFile.*#SSLCertificateFile /etc/rules-ca/keys/$HOSTNAME.crt#" -e "s#SSLCertificateKeyFile.*#SSLCertificateKeyFile /etc/rules-ca/keys/$HOSTNAME.key#" -i /etc/apache2/sites-available/default-ssl.conf
run cp /etc/rules-ca/keys/ca.crt /var/www/html/
copy emerging.rules.tar.gz /var/www/html/emerging.rules.tar.gz
run chmod a+r /var/www/html/emerging.rules.tar.gz
cmd ["bash", "-c", "/etc/init.d/apache2 start && tail -f /var/log/apache2/access.log"]
