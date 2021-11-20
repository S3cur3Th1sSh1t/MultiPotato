# ManualPotato

First of all - credit to [@splinter_code](https://twitter.com/splinter_code) & [@decoder_it](https://twitter.com/decoder_it) for [RoguePotato](https://github.com/antonioCoco/RoguePotato) as this code heavily bases on it.

This is just another Potato to get SYSTEM via SeImpersonate privileges. But this one is different in terms of 

* It doesn't contain any SYSTEM auth trigger for weaponization. Instead the code can be used to integrate your favorite trigger by yourself.
* It's not only using `CreateProcessWithTokenW` to spawn a new process. Instead you can choose between `CreateProcessWithTokenW`, `CreateProcessAsUserW`, `CreateUser` and `BindShell`.

So this project is able to open up a NamedPipe Server, impersonate any user connecting to it and afterwards doing the users option mentioned above. If any new SYSTEM auth triggers are published in the future this tool can still be used to elevate privileges - you just need to use another Pipe-Name in this case.

Examples:

1. CreateUser with modified PetitPotam trigger:

```
c:\temp\MultiPotato> MultiPotato.exe -t CreateUser
```

You have by default values 30 secconds (changable via [THEAD_TIMEOUT](https://github.com/S3cur3Th1sSh1t/MultiPotato/blob/main/Multipotato/common.h) afterwards to let the SYSTEM account or any other account authenticate. This can be done for example via an unpatched MS-EFSRPC function for example. By default MultiPotato listens on the pipename `\\.\pipe\pwned/pipe/srvsvc` which can be used in combination with MS-EFSRPC perfectly fine. For other SYSTEM auth triggers you can adjust this value via parameter.

```
c:\temp\MultiPotato> PetitPotamModified.exe localhost/pipe/pwned localhost
``` 

Using `PetitPotam.py` as trigger from a remote system with a valid low privileged user is of course also possible.

![alt text](https://github.com/S3cur3Th1sSh1t/MultiPotato/raw/main/Images/CreateUser.PNG)

2. CreateProcessAsUserW with SpoolSample trigger:

```
c:\temp\MultiPotato> MultiPotato.exe -t CreateProcessAsUserW -p "pwned\pipe\spoolss" -e "C:\temp\stage2.exe"
```

And trigger it via

```
c:\temp\MultiPotato>MS-RPRN.exe \\192.168.100.150 \\192.168.100.150/pipe/pwned
```

![alt text](https://github.com/S3cur3Th1sSh1t/MultiPotato/raw/main/Images/CreateProcessAsUserW.PNG)

Important: In my testings for MS-RPRN I could not use localhost or 127.0.0.1 as target, this has to be the network IP-Adress or FQDN. In addition the Printer Service needs to be enabled for this to work.


3. BindShell

```
c:\temp\MultiPotato> MultiPotato.exe -t BindShell -p "pwned\pipe\spoolss"
```

To trigger for this pipe MS-RPRN can be used again.

## Why??

I recently had a penetrationtest, where I was able to pwn a MSSQL Server via SQL-Injection and XP_CMDShell. But all public Potatoes failed on this target system to elevate privileges from service-account to SYSTEM. The System auth trigger was not the problem - instead `CreateProcessWithTokenW` failed all the time with NTSTATUS Code 5 - access forbidden. This doesn't really makes sense and may be an edge case but one reason for that `could be` the local endpoint protection which may have blocked this.

Therefore I searched for alternatives - and asked some people on Twitter about it. Again Credit to [@splinter_code](https://twitter.com/splinter_code) for explaining me how to do it via `CreateProcessAsUserW` which worked fine on the pwned MSSQL server to get a SYSTEM C2-Callback.
