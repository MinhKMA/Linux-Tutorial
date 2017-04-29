## Modify the Command Line Prompt
The PS1 variable is the character string that is displayed as the prompt on the command line. Most distributions set PS1 to a known default value, for example, the user and the hostname as in:
```
[root@caldera01 ~]#
```
This could prove useful if you are working in multiple roles and want to be always reminded of who you are and what machine you are on.

```
[root@caldera01 ~]# echo $PS1
[\u@\h \W]\$
[root@caldera01 ~]#
[root@caldera01 ~]# export PS1='[\u@\h \W(customt)]# '
[root@caldera01 ~(customt)]#
[root@caldera01 ~(customt)]# echo $PS1
[\u@\h \W(customt)]#
[root@caldera01 ~(customt)]#
[root@caldera01 ~(customt)]# export PS1='[\u@\h \W]# '
[root@caldera01 ~]#
```
