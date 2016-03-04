1. Introduction
JSNAPy,Junos Snapshot Administrator in Python captures and audit runtime environment of network 
devices running Junos operating system (Junos OS). It automates network state verification by 
capturing and validating status of a device. It is used to take pre-snapshots and then post-snapshots 
after modification and compare them based on test cases provided. It can also be used to audit device's 
runtime environment against pre-defined criteria. 

JSNAPy is supported in two modes:
1. Command line tool 
2. Python Module

Input & Output Components:
1. Input components include all yaml files. It consists of main config files, test files, and other files in yaml like mail files.
2. Output components include snapshot files in xml or text. Jsnapy will further analyse these files and display output in terminal and log all information in log files.  

While installing Jsnapy, it creates Jsnapy folder at /etc and and /etc/logs
1] /etc/jsnapy : It consists of 
                 a) samples: directory consists of sample configs and test files in yaml.
                 b) snapshots: directory containing all snapshots
                 c) testfiles: directory containing all test cases and config files
                 d) jsnapy.cfg: JSNAPy configparser file containing file paths, modify this file to change file paths
                 e) logging.yml: file describing loggging settings
                                 modify this file if you want to change logging levels or output format
2] /var/log/jsnapy : It consists of
                 a) info.log : contain all info messages
                 b) errors.log : contain all error messages
                 c) critical.log : contain all critical messages
                 d) debug.log : contain all debug level messages
                 e) jsnapy.log : default file containing all log messages

Supported test operators and their description:
------------------------------------------------
    For comparing current snapshot with pre-defined criteria:
     - Execute Tests Over Elements with Numeric or String Values
        1. all-same: Check if all content values for the specified element are the same. It can also be used to compare all 
                     content values against another specified element.
               - all-same: flap-count
                 checks if all values of node <flap-count> in given Xpath is same or not.
        2. is-equal: Check if the value (integer or string) of the specified element matches a given value.
               - is-equal: admin-status, down
                 check if value of node <admin-status> in given Xpath is down or not. 
        3. not-equal: Check if the value (integer or string) of the specified element does not match a given value.
               - not-equal: oper-status, down
                 check that value of node <oper-status> should not be equal to down.
        4. exists:  verifies the existence of an XML element in the snapshot.
               - exists: no-active-alarm 
                 checks if node </no-active-alarm> is present or not        
        5. not-exists: verifies xml element should not be present in snapshot
               -not-exists: no-active-alarm 
                checks </no-active-alarm > node should not be present in snapshot.
        6. contains: determines if an XML element string value contains the provided test-string value.
               - contains: //package-information/name[1], jbase
               checks if jbase is present in given node or not. 

     - Execute Tests Over Elements with Numeric Values
        1. is-gt: Check if the value of a specified element is greater than a given numeric value.
                - is-gt: cpu-total, 2
                  checks value of <cpu-total> should be greater than 2
        2. is-lt: Check if the value of a specified element is lesser than a given numeric value.
                - is-lt temperature, 55
                  checks value of <temperature> is 55 or not.
        3. in-range: Check if the value of a specified element is in the given numeric range.
                - in-range memory-buffer-utilization, 20, 70 
                  check if value of <memory-buffer-utilization> is in range of 20, 70
        4. not-range: Check if the value of a specified element is outside of a given numeric range.
                  - not-range: memory-heap-utilization, 5 , 40
                    checks if value of <memory-heap-utilization> is not in range of 5 to 40
  
     - Execute Tests Over Elements with String Values: 
        1.contains: determines if an XML element string value contains the provided test-string value.
               - contains: //package-information/name[1], jbase
                 checks if jbase is present in given node or not. 
        2. is-in: Check if the specified element string value is included in a given list of strings.
               - is-in: oper-status, down, up
                 check if value of <oper-status> in is list (down, up)
        3. not-in: Check if the specified element string value is NOT included in a given list of strings.
               - not-in :oper-status, down, up
                 check if value of <oper-status> in not in list (down, up)
        
    Compare Elements or Element Values in Two Snapshots:
    1. no-diff: Compare data elements present in both snapshots, and verify if their value is the same. 
               - no-diff: mastership-state 
               checks that value of <mastership-state> in two snapshots is not different.
    2. list-not-less: Check if the item is present in the first snapshot but not present in the second snapshot.
           - list-not-less: interface-name
                 checks that if <interface-name> is present in first snapshot, then it should also present in 
                 second snapshot.
                 If nothing is specified, then it will compare all nodes
    3. list-not-more: Check if the item is present in the second snapshot but not present in the first snapshot.
               -list-not-more address-family/address-family-name 
                checks that if node is present in second snapshot then it should be present in first snapshot also.
                If nothing is specified, then it will compare all nodes.
    4. delta: Compare the change in value of an element to a delta. 
          delta can be specified as:
              1. an absolute percentage
              2. positive, negative percentage
              3. an absolute fixed value
              4. a positive negative fixed value
               - delta: memory-heap-utilization, 15% 
                 Value of <memory-heap-utilization> should not differ by more than 15% between pre and post snapshots.


JSNAPy as command line tool:
----------------------------
Jsnapy provides following functionalities for network state verification:
  1. --snap 
  2. --snapcheck
  3. --check 
  4. --diff

1. --snap : this command lets you to take snapshot.
     jsnap --snap <file_name> -f <config_file>

2. --check: this command compares two snapshots based on given test cases.
     jsnap --check <pre_snap> <post_snap> -f <config_file>
     if test cases are not specified in test files, then it will compare pre and post snap files, node by node

3. --snapcheck: compares the current configuration against pre defined criteria
     jsnap --snapcheck <snap_fila_name> -f <config_file>

4. --diff : compares two snapshots (either in xml or text format) word by word
     jsnap --diff <pre_snap> <post_snap> -f <config_file>
     This operator is supported only in command line mode.

Output format:
    Output will be displayed using Jinja template.
    For printing any node value from snapshot, specify pre or post and then node name
    For example:
    {{pre['admin-status']}}  : This will print admin status from pre snapshot
    {{post['admin-status']}} : This will print admin status from post snapshot
    can also specify id using:
    {{id_0}} : for id 0
    {{id_1}} : for id 1

For example:
    Input consist of main config file and test files in yaml
     Config File Example:
     --------------------- 
    # for one device, can be given like this:
    hosts:
      - devices: router 
            username : abc
            passwd: pqr
    tests:
      - test_no_diff.yml 
      - test_delta.yml
    # can use sqlite to store data and compare them  
    sqlite:
      - store_in_sqlite: True
            check_from_sqlite: True
            database_name: jbb.db
            compare: 1,0
       # can send mail by specifying mail
       mail: send_mail.yml
  
    Test file Example:
    --------------------
       tests_include:
    - test_flap_count

       test_flap_count:
         - rpc: get-bgp-neighbor-information
         - iterate:
             xpath: '//bgp-information/bgp-peer'
             tests:
               - all-same: flap-count
                 err: "Test Succeeded!!! flap count are all same, it is <{{post['flap-count']}}>"
                 info: "Test Failed!! flap count are all different <{{post['flap-count']}}>"


    When used as command line:
    --------------------------

     [jpriyal-mba13:git_jsnap_py/latest/jsnap_test] jpriyal% jsnap --snapcheck pre -f config_single_snapcheck.yml
     Connecting to device abc.englab.juniper.net ................

        Taking snapshot for get-bgp-neighbor-information................
    ****************************************
    Performing test on Device: router 
        ****************************************

    Tests Included: test_interfaces_terse 
    ****************************************
    rpc is get-bgp-neighbor-information
    ****************************************
    ----------------------Performing all-same Test Operation----------------------
    Test Succeeded !! flap count are all same, it is <0> 
    Final result of all-same: PASSED 
    ------------------------------- Final Result!! -------------------------------
    Total No of tests passed: 1 
    Total No of tests failed: 0 
    Overall Tests passed!!! 


JSNAPy as module
------------------
For using it as module, it provides following functions under class Jsnapy:
  1. snap(snap_file_name, config_file_name/ config_yaml_data, dev)
       snap_file_name : User can give either
                     - full file path (but file should exists)
                     - prefix, in this case file name is formed automatically (<devicename>_<prefix>_<command/rpc>.<xml/text>)
                       all snapshots are taken from '/etc/jsnapy/snapshots' unless complete path of snapshot file is specified.
       <config_file> : User can give either full path or name of config file, in that case it should be present in current 
                       working directory. Inside this config file, user can give test files by giving:
                          - full path of test file
                          - only name of test file, this file should present in configs folder in '/etc/Jsnapy/configs'
                        configs folder in '/etc/Jsnapy' is default folder containing test files.
       config_yaml_data : instead of test details in config file, user can also specify yaml content in some variable.
       dev : device object, by default it is None 
             If device object is given then it will connect to given device otherwise take device details from config file. 

  2. snapcheck (snap_file_name, config_file_name/ config_yaml_data, dev)
           parameters mean same as snap function

  3. check (snap_filename1, snap_filename2, config_file_name, dev)
           parameters mean same as snap function

  Function of snap, check, snapcheck is same as mentioned in command line argument.
  Output format:
       These functions will return dictionary as output.
       Dictionary will contain all details about total number of test cases passed/ failed.
       Result of each command and rpc. 

Sample Program:
from jnpr.jsnapy import SnapAdmin
from pprint import pprint
from jnpr.junos import Device

dev1 = Device(host='device', user='abc', password='pqr')
dev1.open()

js = SnapAdmin()
config_file = "/etc/jsnapy/config_single_check.yml"
snapvalue = js.snapcheck(config_file, "pre", "post", dev=dev1)
print "snap value is:", snapvalue

for snapcheck in snapvalue:
    print "\n -----------snapcheck----------", snapcheck
    print "Tested on", snapcheck.device
    print "Final result: ", snapcheck.result
    print "Total passed: ", snapcheck.no_passed
    print "Total failed:", snapcheck.no_failed
    pprint(dict(snapcheck.test_details))

Sample Output:
Tested on device
Final result:  Failed
Total passed:  1
Total failed: 1
{'show interfaces terse lo*': [{'element_list': ['admin-status', 'down'],
                                                 'result': False,
                                                 'testoperation': 'is-equal',
                                                'xpath': '//physical-interface[normalize-space(name) = "lo0"]'},
                                {'element_list': ['oper-status’, 'downoo’, 'up'],
                                                 'result': True,
                                                 'testoperation': 'is-in',
                                                'xpath': '//physical-interface[normalize-space(name) = "lo0"]’}
                                      ]
                                 }


