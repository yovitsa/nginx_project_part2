
useradd -m /var/lib/webgen/bin/HTML login_shell webgen# 2420_Assignment_3_Part_1


First issue that I have encouter that my previous apache server that we used in previous classes was running actually I had 5 apache servers runnig, and my port 80 was in use I could not access it
I had first to stop the apache server and the disable it in porder to make space nginx to run on port 80
hi
I also installe dthe lsof package to my drolet in order to see all teh servers runnign, this package was quite usefull as detailed all teh servers running, and to check which ports are open as my port 80 was conflicting with nginx and that was teh reason why the nginx did not run

My second issue was that my server block was asking me for a certificate, while I was following the arch wiki tutorial for creating a seperate server block file, teh example has ssl addition on the port syntax, after oding some research I have removed this adn the server was running smothley

ANother issues was that my nginx file thorwing the erro and I had to increase my Nginx Setting server_names_hash_max_size and server_names_hash_bucket_size the message in journactl was indicating to increase to 2048 and 128 from 64 nad 1024 respectivly. After handling all of these erros my server was running proprely. 

MY issue was that my creatin generate-index.service was failing, I had to debug this by adding a ReaminAfterExit=yes directive to let the system to keep the service active nad restart on  failure is that the serice will restart if it fails


#Ip tables filter issue
#refernecne https://bbs.archlinux.org/viewtopic.php?pid=1333991#p1333991
# Reference
  
  #Use the section 3.2.3.1 to setup your server block
  1. https://wiki.archlinux.org/title/Nginx 3.2.3.1 

  #lsof
  https://www.liquidweb.com/blog/how-to-locate-open-ports-in-linux/#:~:text=The%20six%20primary%20methods%20for,re%20correctly%20configured%20and%20secure.
