FROM debian:stretch

run echo 'APT::Install-Recommends "0";' >> /etc/apt/apt.conf.d/01norecommend
run apt-get update -qy
run apt-get install -y python-docutils texlive-latex-base texlive-latex-recommended texlive-fonts-recommended make python-sphinx texlive-latex-extra latexmk texlive-generic-extra
