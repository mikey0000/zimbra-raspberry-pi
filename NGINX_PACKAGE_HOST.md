
############################################
# INSTRUCTIONS TO SETUP NGINX PACKAGES HOST
############################################

# You might need to resolve network, firewall, selinux, permissions issues appropriately before proceeding:

# sudo sed -i -e s/^SELINUX=enforcing/SELINUX=permissive/ /etc/selinux/config
# sudo setenforce permissive
# sudo systemctl stop firewalld
# sudo ufw disable

sudo bash -s <<"EOM_SCRIPT"
[ -f /etc/redhat-release ] && ( yum install -y epel-release && yum install -y nginx && service nginx start )
[ -f /etc/redhat-release ] || ( apt-get -y install nginx && service nginx start )
tee /etc/nginx/conf.d/zimbra-pkg-archives-host.conf <<EOM
server {
listen 8008;
location / {
root /home/git/installer-build/BUILDS;
autoindex on;
}
}
EOM
service httpd stop 2>/dev/null
service nginx restart
service nginx status
EOM_SCRIPT


=========================================================================================================
