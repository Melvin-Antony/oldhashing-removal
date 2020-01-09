Description:
===========

cPanel servers blocks it updates if the database server running under it is below or equal to Mysql5.5. 
However, you may come to a situation where you are not just able to upgrade the database server especially
in an old server which contains database user password with old 4.1 hashing. Just upgrading the database 
server will result in these old hashings not to work. So you will be forced to rehash the password to the 
latest standards. The best method is to reset the password with the same password to rehash it. However, you may find more than a file under the user which has the credentials of the database user and one may be different. This porbably happens when the user ditches the old contents without properly removing it. In that case you needed to manually check which of the passwords mentioned in the files are correct. Using the passwrod reset scrip, you do not need to manually check each of the passwords. 

Add the database users list that will pop up once you click on the upgrade section in cPanel to a file called as 'dbuser.txt'. The below script will generate output.txt will all the files that contain the string of the databse user under each home folder of each users 

```
tr ' ' '\n' < dbuser.txt| grep "_" | sort | uniq | sort > a.txt
for i in `cat a.txt|awk -F'_' '{print $1}' | sort | uniq`;

 do

j=`grep $i a.txt | tr '\n' ' ' | awk '{$1=$1};1' | replace " " " -e "` ;

if [ ! -d "/var/cpanel/userdata/$i" ];
then
i=`cat /etc/passwd | awk -F"/home/" '{print $2}' | awk -F":" '{print $1}' | grep ^$i`
fi

find /var/cpanel/userdata/$i -maxdepth 1 -type f | xargs grep documentroot | awk -F":" '{print $1}' | egrep -v -e cache -e SSL -e nobody | xargs grep documentroot |  awk -F":"  '{printf ($2"\n"$3"\n")}' | grep -v documentroot| grep -v "public_html\/" | xargs grep -irl --exclude=error_log --exclude=core* --exclude=*sql --exclude=*.zip --exclude=*.tar* --exclude=*.txt -e $j | tee -a output.txt ;

done
```

A simple script that runs continuosuly in an infinite while loop and accepts database username and the current password to rehash it using whmapi only if the input password is the current password.

```
while :
do

echo "Enter Database user and password"
read -r i k
j=`echo $i | awk -F'_' '{print $1}'`
z=$(echo $k | sed 's/\$/\\$/g')
w=`/opt/cpanel/ea-php55/root/usr/bin/php -r 'echo urlencode("'$z'");'`

mysql -u $i -p$k -e"quit"  &>/dev/null

if [ `echo $?` -eq 0 ];

then

whmapi1 set_mysql_password user=$i password=$w cpuser=$j

else

echo "Sorry..Not resetting as the password does not match"

fi

done
```
