## Setting up NSO 

 also be a demo where we will summarize known best practices for work in areas including:

- Data modelling
- Service templates
- Using the Python and RESTCONF API

! Services and Data Modeling CREATE SERVICE

```

ncs_cli
devices sync-from
conf
svi_verify_example first-instance vlan-id 1337 vlan-name SVI-DEMO switches first-sw switch-device dist-sw01
exit
switches second-sw switch-device dist-sw02
exit
firewalls first-fw firewall-device edge-firewall01
exit
commit dry-run outformat native
commit
end

```

Config We want:
https://github.com/jabelk/svi_verify_example 

Service Templates
https://github.com/jabelk/svi_verify_example/blob/master/templates/svi_verify_example-template.xml

Data Modeling
https://github.com/jabelk/svi_verify_example/blob/master/src/yang/svi_verify_example.yang

Python
https://github.com/NSO-developer/selftest/blob/master/python/selftest.py

! VERIFY WITH PYTHON
```


show running-config svi_verify_example first-instance | display xpath
conf
selftest svi-tests service /svi_verify_example[name='first-instance']
commands ping-test command ping arguments 172.16.107.2 failstring "No route to host" devices dist-sw01
exit
commands sh-ip-int-br command show arguments "ip interface brief" devices dist-sw01
exit
commands test-svi-fail command ping arguments 172.16.110.2 devices dist-sw01 failstring "No route to host"
top
commit
selftest svi-tests execute
end
show svi_verify_example selftest-result

```

POSTMAN

GET

DELETE

PATCH (Create)







 
