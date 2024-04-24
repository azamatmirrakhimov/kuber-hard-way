# Here are the commands you can use to set up the nginx load balancer. Run these on the server that you have designated as your load balancer server:
~~~
sudo apt-get install -y nginx-full
~~~
~~~
sudo systemctl enable nginx
~~~
~~~
sudo mkdir -p /etc/nginx/tcpconf.d
~~~
~~~
sudo vi /etc/nginx/nginx.conf
~~~
# Add the following to the end of nginx.conf:
~~~
include /etc/nginx/tcpconf.d/*;
~~~
# Set up some environment variables for the lead balancer config file:
~~~
CONTROLLER1_IP=172.31.18.85
~~~
~~~
CONTROLLER2_IP=172.31.24.235
~~~
~~~
CONTROLLER3_IP=172.31.17.121
~~~
# Create the load balancer nginx config file:
~~~
cat << EOF | sudo tee /etc/nginx/tcpconf.d/kubernetes.conf
stream {
    upstream kubernetes {
        server $CONTROLLER1_IP:6443;
        server $CONTROLLER2_IP:6443;
        server $CONTROLLER3_IP:6443;
    }

    server {
        listen 6443;
        listen 443;
        proxy_pass kubernetes;
    }
}
EOF
~~~

# Reload the nginx configuration:
~~~
sudo nginx -s reload
~~~
# You can verify that the load balancer is working like so:
~~~
curl -k https://localhost:6443/version
~~~
# You should get back some json containing version information for your Kubernetes cluster.
