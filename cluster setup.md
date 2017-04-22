
#A tutorial to install and setup couchdb clusters on Ubuntu 16.04

Installing and setup a couchdb cluster cost me three days due to less resource and poor documentation in the Internet.After rolling back my three VMs for more than 20 times...I finally set it up. This post is for any couchdb beginner to build his/her own database cluster system. If you prefer a Chinese version, plase see this link.[CLICK ME](http://101.100.232.7)

##Fist you should have three Ubuntu 16.04 virtual machines :)

Login to your virtual machine, I believe you are currently ***using 'ubuntu' user.***

Now enter"```sudo passwd```, and enter your password for your root user and confirm it by enterning password again.
run command ```su -``` and login.
Now you should being using your root user.

##Now we start to install couchdb
*run following command:
```
mkdir temp
cd temp
wget https://raw.githubusercontent.com/afiskon/install-couchdb/master/install-couchdb.sh
wget https://raw.githubusercontent.com/afiskon/install-couchdb/master/install-couchdb.sh
sh install-couchdb.sh
```
You don't need to worry abouth what excatly they are.They are but some bash command in a file to make it easier to be used by linux beginner, everything got finined in 5 seconds.
***Then do the same things for all VMs***

After doing aboves, You have three couchdb instances running on your VMs individually!

##Now we CLUSTER it:

Begin with your first virtual machine:
run the commands:
```
su -
cd temp/apache-couchdb-2.0.0/rel/couchdb/etc/
ls
```
***now you can see three files in the folder, let's modify two of them:

**For the local.ini:

run command : 
```
nano local.ini
```
then change:
```
;port = 5984
;bind_address=127.0.0.1
```
to:
```
port=5984
bind_address= [your ip address]
```

**For the vm.args:
run command: 
```nano local.ini
```
then change:
```
-name couchdb@localhost
```

to 
```
-name couchdb@[your ip address]
```
***(this is very important and it MUST be excatly like 'couchdb@xxx.xxx.xxx.xxx'.)***


then we build up our erlang config, run command:
cd /root/temp/apache-couchdb-2.0.0/rel/couchdb/releases/2.0.0
ls
nano sys.args
then change the "[]."to this:

*******copy and paste everything between these two lines********
[
    {lager, [
        {error_logger_hwm, 1000},
        {error_logger_redirect, true},
        {handlers, [
            {lager_console_backend, [debug, {
                lager_default_formatter,
                [
                    date, " ", time,
                    " [", severity, "] ",
                    node, " ", pid, " ",
                    message,
                    "\n"
                ]
            }]}
        ]},
        {inet_dist_listen_min, 9100},
        {inet_dist_listen_max, 9200}
    ]}
].
*******copy and paste everything between these two lines********

Then we return to our temp director with commands and reinitial it:
cd /root/temp
sh reinit.sh
then we should see something like this:
{"all_nodes":["couchdb@111.222.333.44"],"cluster_nodes":["couchdb@111.222.333.44"]}
the it is "couchdb@localhost" rather than couchdb@111.222.333.44, this is because your database did not successfully
configure your name in vm.args file. you may consider try to bind it to the proper name it mentioned before.

now we install curl:
apt-get install curl
then run command:

curl -X PUT http://127.0.0.1:5984/_node/couchdb@[your ip address]/_config/admins/admin1 -d '"password"'
curl -X PUT http://127.0.0.1:5984/_node/couchdb@[your ip address]/_config/chttpd/bind_address -d '"0.0.0.0"'

Do above things for all your VMs. we are about there!

Now. On your LOCAL MACHINE a mac or window:
Let's build a ssh tunnel:

ssh -N -L 9000:localhost:5984 ubuntu:[your vm password]@[your VM ip address] ## you may need a private key to login, though actually we
                                                                             ## are not logging in.
then use a web browser, I prefer Chrome, enter the address:

http://localhost:9000/_utils/

then login and setup as a cluster.
enter the admin and password,
add other two nodes then click config the cluster.

ALl DONE!
To check if you made it properly:
http://localhost:9000/_membership/
you are suppose to see this:

{"all_nodes":["couchdb@111.222.333.44","couchdb@123.423.25.534.62","couchdb@22.66.55.33"],
"cluster_nodes":["couchdb@111.222.333.44","couchdb@123.423.25.534.62","couchdb@22.66.55.33"]}

If not so.....You may consider do it again or check it whether the database config your '-name' or try restart it by 
'sv restasrt couchdb'

If I were you, I would definitly not choose CouchDB. It's dirty!!

