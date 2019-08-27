FROM debian:stretch

ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update
RUN apt-get dist-upgrade -y
RUN apt-get install -y procps samba samba-dsdb-modules samba-vfs-modules easy-rsa
RUN apt-get install -y vim net-tools ldap-utils
RUN rm -f /etc/samba/smb.conf

# Work-around to use ntvfs, when samba gets upgraded, this may be replaced by passing --use-ntvfs
# to the "samba-tool domain provision" command below
RUN sed -e 's/use_ntvfs=use_ntvfs/use_ntvfs=True/' -i /usr/lib/python2.7/dist-packages/samba/provision/__init__.py

RUN samba-tool domain provision --server-role=dc --dns-backend=SAMBA_INTERNAL --realm=STAMUS.COM --domain=STAMUS --adminpass=Nupik4Ever
RUN sed -e '9i    ldap server require strong auth = no' -i /etc/samba/smb.conf
RUN sed -e '9i    tls cafile = /etc/samba/tls/keys/ca.crt' -i /etc/samba/smb.conf
RUN sed -e '9i    tls certfile = /etc/samba/tls/keys/samba.crt' -i /etc/samba/smb.conf
RUN sed -e '9i    tls keyfile = /etc/samba/tls/keys/samba.key' -i /etc/samba/smb.conf
RUN sed -e '9i    tls enabled = yes' -i /etc/samba/smb.conf
RUN rmdir /etc/samba/tls && make-cadir /etc/samba/tls
RUN cp /etc/samba/tls/openssl-1.0.0.cnf /etc/samba/tls/openssl.cnf
RUN bash -c "cd /etc/samba/tls/ && source vars && /etc/samba/tls/clean-all"
RUN bash -c "cd /etc/samba/tls/ && source vars && /etc/samba/tls/pkitool --initca"
RUN bash -c "cd /etc/samba/tls/ && source vars && /etc/samba/tls/pkitool --server samba"
RUN samba-tool domain passwordsettings set --max-pwd-age=0
RUN samba-tool user create ad-user Password1
RUN samba-tool user create ad-user-inactive Password1
RUN samba-tool user create ad-staff Password1
RUN samba-tool user create ad-staff-inactive Password1
RUN samba-tool user create ad-admin Password1
RUN samba-tool user create ad-admin-inactive Password1
RUN samba-tool group add scirius-active
RUN samba-tool group addmembers scirius-active ad-user,ad-staff,ad-admin
RUN samba-tool group add scirius-staff
RUN samba-tool group addmembers scirius-staff ad-staff,ad-staff-inactive
RUN samba-tool group add scirius-admin
RUN samba-tool group addmembers scirius-admin ad-admin,ad-admin-inactive

COPY start.sh /

ENTRYPOINT ["/start.sh"]
