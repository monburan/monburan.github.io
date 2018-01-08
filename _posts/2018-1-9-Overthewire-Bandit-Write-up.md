---
layout: post
title: Overthewire Bandit Write up(0-20)
categories: [wargame]
tags: [wargame, writeup]
description: "做題學知識(2018.1.9)"
fullview: false
comments: true
---
# 0
The password for the next level is stored in a file called readme located in the home directory. Use this password to log into bandit1 using SSH. Whenever you find a password for a level, use SSH (on port 2220) to log into that level and continue the game.

    cat readme 

[//]: boJ9jbbUNNfktd78OOpsqOltutMc3MY1

# 1
The password for the next level is stored in a file called - located in the home directory

    cat ./-

[//]: CV1DtqXWVFXTvM2F0k09SHz0YwRINYA9

# 2
The password for the next level is stored in a file called spaces in this filename located in the home directory

    cat spaces\ in\ this\ filename

[//]: UmHadQclWmgdLOKQ3YNgjWxGoRMb5luK

# 3
The password for the next level is stored in a hidden file in the inhere directory.

    cat inhere/.hidden

[//]:pIwrPrtPN36QITSp3EQaw936yaFoFgAB

# 4
The password for the next level is stored in the only human-readable file in the inhere directory. Tip: if your terminal is messed up, try the “reset” command.


    file inhere/*
    #  inhere/-file07: ASCII text
    cat inhere/-file07

[//]:koReBOKuIDDepwhWk7jZC0RTdopnAYKh

# 5
The password for the next level is stored in a file somewhere under the inhere directory and has all of the following properties:

human-readable

1033 bytes in size

not executable

    find . -size 1033c|xargs cat

[//]:DXjZPULLxYr17uwoI01bNLQbtFemEgo7

# 6
The password for the next level is stored somewhere on the server and has all of the following properties:

owned by user bandit7

owned by group bandit6

33 bytes in size

     find / -size 33c -user bandit7 -group bandit6 2>/dev/null|xargs cat

[//]:HKBPTKQnIay4Fw76bEy8PVxKEDQRKTzs

# 7
The password for the next level is stored in the file data.txt next to the word millionth

    cat data.txt |grep millionth

[//]:cvX2JJa4CFALtqS87jk27qwqGhBM9plV

# 8
The password for the next level is stored in the file data.txt and is the only line of text that occurs only once

    sort data.txt |uniq -c|grep "[1] "

[//]: UsvVyFSfZZWbi6wgC7dAFyFuR6jQQUhR

# 9
The password for the next level is stored in the file data.txt in one of the few human-readable strings, beginning with several ‘=’ characters.

    strings data.txt |grep =

[//]: truKLdjsbJ5g7yyJ2X2R0o3a5HQJFuLk

# 10
The password for the next level is stored in the file data.txt, which contains base64 encoded data

     base64 -d data.txt

[//]: IFukwKGsFW8MOq3IRFqrxE1hxTNEbUPR

# 11:
The password for the next level is stored in the file data.txt, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions

    cat data.txt|tr 'n-za-mN-ZA-M' 'a-zA-Z'

[//]: 5Te8Y4drgCRfCx8ugdwuEX8KFC6k2EUu

# 12
The password for the next level is stored in the file data.txt, which is a hexdump of a file that has been repeatedly compressed. For this level it may be useful to create a directory under /tmp in which you can work using mkdir. For example: mkdir /tmp/myname123. Then copy the datafile using cp, and rename it using mv (read the manpages!)

    mkdir /tmp/monburan
    cp data.txt /tmp/monburan/
    cd /tmp/monburan
    xxd -r data.txt data
    file data
    # data: gzip compressed data, was "data2.bin", last modified: Thu Dec 28 13:34:36 2017, max compression, from Unix
    cp data data1.gz
    gzip -d data1.gz
    file data1
    # data1: bzip2 compressed data, block size = 900k
    cp data1 data2.bz2
    bzip2 -d data2.bz2
    # data2: gzip compressed data, was "data4.bin", last modified: Thu Dec 28 13:34:36 2017, max compression, from Unix
    cp data2 data3.gz
    gzip -d data3.gz
    file data3
    # data3: POSIX tar archive (GNU)
    cp data3 data4.tar
    tar xvf data4.tar
    file data5.bin
    # data5.bin: POSIX tar archive (GNU)
    cp data5.bin data5.tar
    tar xvf data5.tar
    file data6.bin
    # data6.bin: bzip2 compressed data, block size = 900k
    cp data6.bin data6.bz2
    file data6
    # data6: POSIX tar archive (GNU)
    cp data6 data7.tar
    tar xvf data7.tar
    file data8
    # data8.bin: gzip compressed data, was "data9.bin", last modified: Thu Dec 28 13:34:36 2017, max compression, from Unix
    cp data8.bin data9.gz
    gzip -d data9.gz
    file data9
    # data9: ASCII text
    cat data9

[//]: 8ZjyCRiBWFYkneahHwxCv3wb2a1ORpYL

# 13
The password for the next level is stored in /etc/bandit_pass/bandit14 and can only be read by user bandit14. For this level, you don’t get the next password, but you get a private SSH key that can be used to log into the next level. Note: localhost is a hostname that refers to the machine you are working on

    ssh -i sshkey.private bandit14@localhost
    cat /etc/bandit_pass/bandit14

[//]: 4wcYUJFw0k0XLShlDzztnTBHiqxU3b3e

# 14
The password for the next level can be retrieved by submitting the password of the current level to port 30000 on localhost.

    telnet localhost 30000

[//]: BfMYroe26WYalil77FoDi9qh59eK5xNr

# 15
The password for the next level can be retrieved by submitting the password of the current level to port 30001 on localhost using SSL encryption.

Helpful note: Getting “HEARTBEATING” and “Read R BLOCK”? Use -ign_eof and read the “CONNECTED COMMANDS” section in the manpage. Next to ‘R’ and ‘Q’, the ‘B’ command also works in this version of that command…

    openssl s_client -quiet -connect localhost:30001

[//]: cluFn7wTiGryunymYOu4RcffSxQluehd

# 16
The credentials for the next level can be retrieved by submitting the password of the current level to a port on localhost in the range 31000 to 32000. First find out which of these ports have a server listening on them. Then find out which of those speak SSL and which don’t. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.

    nmap -v localhost -p 31000-32000
    # PORT      STATE SERVICE
    # 31046/tcp open  unknown
    # 31518/tcp open  unknown
    # 31691/tcp open  unknown
    # 31790/tcp open  unknown
    # 31960/tcp open  unknown
    echo test|nc localhost 31046
    # test
    echo test|nc localhost 31518
    echo test|nc localhost 31691
    # test
    echo test|nc localhost 31790
    echo test|nc localhost 31960
    # test
    echo cluFn7wTiGryunymYOu4RcffSxQluehd|openssl s_client -quiet -connect localhost:31790
    # -----BEGIN RSA PRIVATE KEY-----
    # MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
    # imZzeyGC0gtZPGujUSxiJSWI/oTqexh+cAMTSMlOJf7+BrJObArnxd9Y7YT2bRPQ
    # Ja6Lzb558YW3FZl87ORiO+rW4LCDCNd2lUvLE/GL2GWyuKN0K5iCd5TbtJzEkQTu
    # DSt2mcNn4rhAL+JFr56o4T6z8WWAW18BR6yGrMq7Q/kALHYW3OekePQAzL0VUYbW
    # JGTi65CxbCnzc/w4+mqQyvmzpWtMAzJTzAzQxNbkR2MBGySxDLrjg0LWN6sK7wNX
    # x0YVztz/zbIkPjfkU1jHS+9EbVNj+D1XFOJuaQIDAQABAoIBABagpxpM1aoLWfvD
    # KHcj10nqcoBc4oE11aFYQwik7xfW+24pRNuDE6SFthOar69jp5RlLwD1NhPx3iBl
    # J9nOM8OJ0VToum43UOS8YxF8WwhXriYGnc1sskbwpXOUDc9uX4+UESzH22P29ovd
    # d8WErY0gPxun8pbJLmxkAtWNhpMvfe0050vk9TL5wqbu9AlbssgTcCXkMQnPw9nC
    # YNN6DDP2lbcBrvgT9YCNL6C+ZKufD52yOQ9qOkwFTEQpjtF4uNtJom+asvlpmS8A
    # vLY9r60wYSvmZhNqBUrj7lyCtXMIu1kkd4w7F77k+DjHoAXyxcUp1DGL51sOmama
    # +TOWWgECgYEA8JtPxP0GRJ+IQkX262jM3dEIkza8ky5moIwUqYdsx0NxHgRRhORT
    # 8c8hAuRBb2G82so8vUHk/fur85OEfc9TncnCY2crpoqsghifKLxrLgtT+qDpfZnx
    # SatLdt8GfQ85yA7hnWWJ2MxF3NaeSDm75Lsm+tBbAiyc9P2jGRNtMSkCgYEAypHd
    # HCctNi/FwjulhttFx/rHYKhLidZDFYeiE/v45bN4yFm8x7R/b0iE7KaszX+Exdvt
    # SghaTdcG0Knyw1bpJVyusavPzpaJMjdJ6tcFhVAbAjm7enCIvGCSx+X3l5SiWg0A
    # R57hJglezIiVjv3aGwHwvlZvtszK6zV6oXFAu0ECgYAbjo46T4hyP5tJi93V5HDi
    # Ttiek7xRVxUl+iU7rWkGAXFpMLFteQEsRr7PJ/lemmEY5eTDAFMLy9FL2m9oQWCg
    # R8VdwSk8r9FGLS+9aKcV5PI/WEKlwgXinB3OhYimtiG2Cg5JCqIZFHxD6MjEGOiu
    # L8ktHMPvodBwNsSBULpG0QKBgBAplTfC1HOnWiMGOU3KPwYWt0O6CdTkmJOmL8Ni
    # blh9elyZ9FsGxsgtRBXRsqXuz7wtsQAgLHxbdLq/ZJQ7YfzOKU4ZxEnabvXnvWkU
    # YOdjHdSOoKvDQNWu6ucyLRAWFuISeXw9a/9p7ftpxm0TSgyvmfLF2MIAEwyzRqaM
    # 77pBAoGAMmjmIJdjp+Ez8duyn3ieo36yrttF5NSsJLAbxFpdlc1gvtGCWW+9Cq0b
    # dxviW8+TFVEBl1O4f7HVm6EpTscdDxU+bCXWkfjuRb7Dy9GOtt9JPsX8MBTakzh3
    # vBgsyi/sN3RqRBcGU40fOoZyfAMT8s1m/uYv52O6IgeuZ/ujbjY=
    # -----END RSA PRIVATE KEY-----
    mkdir /tmp/monburan
    cd /tmp/monburan
    echo cluFn7wTiGryunymYOu4RcffSxQluehd|openssl s_client -quiet -connect localhost:31790 > sshkey.private
    # use vi remove unuseful
    ssh -i sshkey.private bandit17@localhost
    # if you need input password you need this -> https://stackoverflow.com/questions/9270734/ssh-permissions-are-too-open-error
    chmod 400 sshkey.private
    ssh -i sshkey.private bandit17@localhost

# 17
There are 2 files in the homedirectory: passwords.old and passwords.new. The password for the next level is in passwords.new and is the only line that has been changed between passwords.old and passwords.new

NOTE: if you have solved this level and see ‘Byebye!’ when trying to log into bandit18, this is related to the next level, bandit19

    diff passwords.new passwords.old

[//]: kfBf3eYk5BPBRzwjqutbbfE887SVc5Yd
# 18
The password for the next level is stored in a file readme in the homedirectory. Unfortunately, someone has modified .bashrc to log you out when you log in with SSH.

    ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme

[//]: IueksS7Ubh8G3DCwVzrTd8rAVOwq3M5x
# 19
To gain access to the next level, you should use the setuid binary in the homedirectory. Execute it without arguments to find out how to use it. The password for this level can be found in the usual place (/etc/bandit_pass), after you have used the setuid binary.

     ./bandit20-do cat /etc/bandit_pass/bandit20
    
[//]: GbKksEFF4yrVs6il55v6gwY5aVje5f0j
# 20
There is a setuid binary in the homedirectory that does the following: it makes a connection to localhost on the port you specify as a commandline argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the next level (bandit21).

NOTE: Changes to the infrastructure made this level more difficult. You will need to figure out a way to launch multiple commands in the same Docker instance.

NOTE 2: Try connecting to your own network daemon to see if it works as you think

    echo GbKksEFF4yrVs6il55v6gwY5aVje5f0j | nc -l 10010

    ./suconnect 10010

[//]: gE269g2h3mw3pwgrj0Ha9Uoqen1c9DGr
