 
# added line for develop branch
#3 added more comment
# feature branch
uwsgi -s ./myproject.sock --wsgi-file wsgi.py -H ./myprojectenv/ --http-processes=4 --chmod-socket=777 --master
 
 To host nginx service, do the following
 
 1. install uwsgi 
 pip install uwsgi (no need to activate virtualenv)
 2. create uwsgi.ini aka config file like so:

	[uwsgi]
	module = wsgi
	master = true
	processes = 5
	threads=2
	venv=/home/chandresh/myproject/myprojectenv
	socket =/home/chandresh/myproject/myproject.sock
	chmod-socket = 777
	vacuum = true
	die-on-term = true
	logto=/home/chandresh/myproject/%n.log

put it in the same dir as project

3. also create wsgi.py file for callling main py app object

	from myproject import app as application

	if __name__ == "__main__":
	    application.run()

Note: usgi uses "application" to run. Hence don't use this name in main py file.

4. Install nginx via apt-get or yum
5. Look for nginx.conf file in /etc/nginx/ or some other place depending on the architecture

6. add the following server block in the http block

	 server{
		 listen 80; # (or any port you like);
		 server_name ckm; #( your server_name or ip if domain name not configured in /etc/hosts)
		location / { try_files $uri @yourapplication; }
	    location @yourapplication {
		include uwsgi_params;
		uwsgi_pass unix:/path/to/sock/file/myproject.sock;

	    }
Note: nginx communicates to uwsgi via wsgi protocal and sock file. Make sure nginx has read/write/ex permission to that file and parent folders or it will fail

For error related to nginx: see log file at  /var/log/nginx/error.log;
For granting  nginx access to sock file do this:

see my ans here https://stackoverflow.com/questions/29872174/wsgi-nginx-error-permission-denied-while-connecting-to-upstream/51324847#51324847

sudo usermod -a -G $USER nginx

Now, we can give our user group execute permissions on our home directory. This will allow the Nginx process to enter and access content within:

chmod 710 /path/to/project/dir

If the permission denied error is still there: then the hack sudo setenforce 0 (SELinux issue) will do the trick.


7. Create service file to restart uwsgi service during system boot in /etc/systemd/system/
name this file as your_proj_name.service

	Unit]
	Description=uWSGI instance to serve myproject
	After=network.target

	[Service]
	User=chandresh
	Group=nginx
	WorkingDirectory=/home/chandresh/myproject
	Environment="PATH=/home/chandresh/myproject/myprojectenv/bin"
	ExecStart=/home/chandresh/myproject/myprojectenv/bin/uwsgi --ini myproject.ini

	[Install]
	WantedBy=multi-user.target
	                                
and start/stop and enable this service via
sudo systemctl start/stop/restart/enable/disable/status service_name

8. By default, service can be accessd via http://server_name:port (if running on  port 80, can skip port)













