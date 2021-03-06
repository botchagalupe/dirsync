## Enstratius LDAP Integration

We can use Enstratius to leverage LDAP or Active Directory (AD) as an authoritative source of user data for a customer. Enstratius can synchronize its internal directory with a subset of groups within LDAP or AD and all users in those groups. When a customer adds a user to LDAP, for example, and that user is in one of the synchronized groups, Enstratius will discover the user and add that user and all group memberships for that user in Enstratius. If that user is terminated, Enstratius will remove all the user accesses and group membership in the Enstratius directory as well. 

## Dirsync-Tool
Dirsync-Tool is a mutlinode sbin tool that allows you configure LDAP inetgration with Enstratius and run directory synchronization.

## Usage

```
root@vagrant:/services/console/sbin# ./dirsync-tool.sh -h
Usage: -action

Options:
   -v, --version                      show program's version number and exit.
   -h, --help                         show this help message and exit.

    Configure actions:
    ---------------------------------------

      -l $customerId                  List a LDAP service for a particular customer.
                                      Not specifying customer id returns LDAP services for all the customer.
      -g $directoryServiceID          List group mapping for a specific directory service.
                                      Not specifying directory service id returns all group mapping.
      -a                              Add a LDAP Service by taking LDAP configuration as inputs from user prompt. 
      -f $filePath                    Add a LDAP service by reading LDAP configuration from a properties file.
                                      Not specifying the path will print out a template of the file to read from.
      -u $directoryServiceID          Update existing LDAP Service by taking inputs from user prompt. 
      -m $serviceId:$fieldToUpdate:$newValue
                                      Update a specific field of the specific LDAP service from the database.
      -s $directoryServiceId:$standardGroups
                                      Add standard groups(separated by ',') for a specific LDAP service.
      -k $directoryServiceId:$standardGroups
                                      Remove standard groups(separated by ',') for a specific LDAP service.
      -r $directoryServiceID          Remove specific LDAP service from database. 
      -c                              Check authentication with LDAP credentials.


    Dirsync actions:
    ---------------------------------------

      --prerun                        Pre-run dirsync. 
      --run                           Run dirsync.

```

## Adding LDAP service.
A customer can have multiple LDAP services with different configurations. Before integrating LDAP with Enstratius, we will require gathering of some schema mapping information from the LDAP/AD administrator.

1. <code>customer_admin_group</code>
 : The CN of the group in the directory service (for example, ‘Administrators’) that will always have administrative access across all accounts in the infrastructure. This value may be null. If null, user must have some non-directory service group as admin group elsewhere. Enstratius will search under ldap_group_base for an object with an object class of ldap_ group_object_class and the CN matching this value. That group will be the admin group.

2. <code>standard_groups</code>
 : Similar to customer_admin_group, this value is a comma-separated list of group CNs representing the groups to be replicated between Enstratius and directory service. This may be null if trying to replicate the customer_admin_group. This list should not, however, include the customer_admin_group. Users in these groups will be synchronized into Enstratius, but the groups won’t have any special access except that defined by roles they are assigned.

3. <code>ldap_access_endpoint</code>
 : For example, ldap://ad.example.com/. This is the LDAP server.

4. <code>ldap_access_principal</code>
 : The formal DN for the principal being used to authenticate connections with LDAP. For example, ‘CN=LDAP User,CN=Users,DC=ad,DC=example,DC=com’.

5. <code>ldap access password</code>
 : The directory service access password. DirsyncTool will encrypt the password and generate a secrete access code and store in database as <code>ldap_access_secret</code>.

6. <code>ldap_access_ssl</code>
 : A value of ‘Y’ or ‘N’ depending on whether SSL should be used for the communications with the LDAP server. ‘Y’ is recommended.

7. <code>ldap_group_base</code>
 : The base DN for the object under which group objects are stored. For example, ‘CN=Builtin,D C=ad,DC=example,DC=com’. Enstratius will do a deep search under this object for groups.

8. <code>ldap_group_description</code>
 : The name of the LDAP attribute containing the value to be used as a group description when the group is represented in Enstratius. If you do not have any kind of description attribute, just use the CN value or what is defined in ldap_group_name.

9. <code>ldap_group_name</code>
 : The LDAP attribute containing the name of the group. This value should be unique across all groups. It is typically ‘CN’.

10. <code>ldap_group_object_class</code>
 : The object class for groups in the LDAP/AD directory. Typically, this value is ‘group’.

11. <code>ldap_group_usernames</code>
 : The LDAP attribute that contains the user names of the users that belong to this group.

12. <code>ldap_object_class</code>
 : The LDAP attribute that defines an object’s object class. For example, ‘objectClass’.

13. <code>ldap_user_base</code>
 : The base DN under which users are stored. For example, ‘CN=Users,DC=ad,DC=example,DC =com’. Enstratius will do a deep search under this object for users.

14. <code>ldap_user_email</code>
 : The LDAP attribute representing a user email. In some organizations, the email address may be embedded inside a more complex text attribute. You can use the values ldap_email_ regex and ldap_email_regex_index to pull the email value from that text. Enstratius requires user emails to be unique and serves as the user’s primary mechanism for identifying themselves to Enstratius.

15. <code>ldap_user_family_name</code>
 : The LDAP attribute that stores a person’s last name/family name.

16. <code>ldap_user_given_name</code>
 : The LDAP attribute that stores a person’s first name/given name.

17. <code>ldap_user_group</code>
 : The LDAP attribute that defines the user’s group membership.

18. <code>ldap_user_object_class</code>
 : The LDAP object class for users. For example, ‘user’.

19. <code>ldap_user_user_name</code>
 : The LDAP attribute that stores the user name a person uses to login with other systems when a user name (as opposed to an email address) is used to identify them.


###### Enstratius also optionally looks for the following attributes:

20. <code>ldap_default_phone_region</code>
 : The default region for interpreting the phone number fields. For example, ‘US’.

23. <code>ldap_email_regex</code>
 : A Java regex that tells Enstratius how to parse the values in the ldap_user_email field.

24. <code>ldap_email_regex_index</code>
 : The index identifying which part of the regex stores the email address.

25. <code>ldap_user_mobile</code>
 : The LDAP attribute containing the user’s mobile phone number. This value is required when using LDAP with an SMS-based Enstratius multi-factor authentication. If the mobile phone number is from a country different than the one defined by ldap_default_phone_region, it should start with a country code. Otherwise, Enstratius will interpret it as belonging in the default phone region country.

26. <code>ldap_user_time_zone</code>
 : The LDAP attribute storing the user’s preferred time zone.


Once we have these information, it can be stored in the database via Dirsyc Tool. The directory synchronization process will extract information from the database to conenect with LDAP server. 

There are two ways we cam add a LDAP service in Enstratius using the tool.

### Using user prompt inputs for LDAP configuration values.

Adding a LDAP service with user input action allows the user to add LDAP service by taking user prompt inputs in succession.

```
dirsync-tool.sh -a

Example: 

root@vagrant:/services/console/sbin# ./dirsync-tool.sh -a

LDAP Access Endpoint    : ldap://ad.example.com/
LDAP Access Principal   : CN=LDAP User,CN=Users,DC=ad,DC=example,DC=com
LDAP Access Password    : ****

Connecting to endpoint  : ldap://ad.example.com/
With principal          : CN=LDAP User,CN=Users,DC=ad,DC=example,DC=com

Authenticated
Exiting authentication.
Connection to ldap://ad.example.com/ with principal  CN=LDAP User,CN=Users,DC=ad,DC=example,DC=com was successful.

Enter customer ID  [Numeric]: 500
Name of service             :TestService
Description of service      :TestDescription
Label of service            :blue
Precedence      [Numeric]   :1
Customer Admin Group        :Admin 
Standard Groups             :Standard
LDAP Access SSL  [y/n]      :n
LDAP Object Class           :objectClass
LDAP Group Base             :CN=Builtin,DC=ad,DC=example,DC=com
LDAP Group Description      :groups
LDAP Group Name             :cn
LDAP Group Usernames        :member
LDAP Group Object Class     :group
LDAP User Base              :CN=Users,DC=ad,DC=example,DC =com
LDAP User Family Name       :sn
LDAP User Given  Name       :givenName
LDAP User Group             :memberOf
LDAP User Email             :mail  
LDAP User Object Class      :user
LDAP User UserName          :sAMAccountName
LDAP Default Phone Region   :US
LDAP Email Regex            :
LDAP Email Regex Index [Numeric] :
LDAP User Mobile            :654654654
LDAP User TimeZone          :TZ


Created LDAP service Testing with ID : 800
```

### Reading LDAP configuration values from a file.

The other way to add a LDAP service is by reading LDAP configuration values from a file. You can prinout the template of file to be read by calling the tool with '-f' action without any parameters.

```
dirsync-tool.sh -f 

Example :

root@vagrant:/services/console/sbin# ./dirsync-tool.sh -f
 
## Template for adding LDAP service in Enstratius. Copy the contents of this file, add values and save.

customerId=<Numberic>
name=<String>
description=<String>
label=<String>
precedence=<Numeric>
customerAdminGroup=<String>
standardGroups=<String>
ldapAccessEndpoint=<String>
ldapAccessPrincipal=<String>
ldapAccessPassword=<String>
ldapAccessSsl=<Boolean>
ldapObjectClass=<String>
ldapGroupBase=<String>
ldapGroupDescription=<String>
ldapGroupName=<String>
ldapGroupUsernames=<String>
ldapGroupObjectClass=<String>
ldapUserBase=<String>
ldapUserEmail=<String>
ldapUserFamilyName=<String>
ldapUserGivenName=<String>
ldapUserGroup=<String>
ldapUserObjectClass=<String>
ldapUserUserName=<String>
ldapDefaultPhoneRegion=<String [2]>
ldapEmailRegex=<String>
ldapEmailRegexIndex=<Numeric>
ldapUserMobile=<String>
ldapUserTimeZone=<String>

```

Once you have saved the file with all the respective values filled out, you can specify the path of the file as a parameter  in order to load the LDAP configuration values into the database. 

```
dirsync-tool.sh -r <path/of/file>

Example: 

root@vagrant:/services/console/sbin# ./dirsync-tool.sh -f /services/console/sbin/addService 

Connecting to endpoint  : ldap://ad.example.com/
With principal          : CN=LDAP User,CN=Users,DC=ad,DC=example,DC=com
Failed connecting to directory at endpoint ldap://ad.example.com/ : ad.example.com:389


Created LDAP service TestService with ID : 400


***************************************************************
      directoryServiceId          : 400
      customerId                  : 500
      active                      : true
      name                        : TestService
      description                 : TestDescription
      precedence                  : 1
      label                       : blue    
      type                        : LDAP

      == LDAP Configuration ==

      customerAdminGroup          : Admin
      standardGroups              : Standard
      ldapAccessEndpoint          : ldap://ad.example.com/
      ldapAccessPrincipal         : CN=LDAP User,CN=Users,DC=ad,DC=example,DC=com
      ldapAccessSecret            : 4a5f69089a092bd0
      ldapAccessSsl               : false
      ldapObjectClass             : objectClass
      ldapGroupBase               : CN=Builtin,DC=ad,DC=example,DC=com
      ldapGroupDescription        : groups    
      ldapGroupName               : cn      
      ldapGroupUsernames          : member  
      ldapGroupObjectClass        : group   
      ldapUserBase                : CN=Users,DC=ad,DC=example,DC =com
      ldapUserEmail               : mail
      ldapUserFamilyName          : sn      
      ldapUserGivenName           : givenName
      ldapUserGroup               : memberOf
      ldapUserObjectClass         : user    
      ldapUserUserName            : sAMAccountName
      ldapDefaultPhoneRegion      : US          
      ldapEmailRegex              : null
      ldapEmailRegexIndex         : null
      ldapUserMobile              : 654654654        
      ldapUserTimeZone            : TZ 
```

## Listing LDAP services

 Listing a LDAP service with the tool will incorporates the contents of both the tables,<code>customer_ldap_service</code> and <code>customer_ldap_directory</code>, into one. Providing customer id will print all LDAP services for that customer.

```
dirsync-tool.sh -l 
dirsync-tool.sh -l <customer_id>

Example: 

root@vagrant:/services/console/sbin# ./dirsync-tool.sh -l 500


***************************************************************
      directoryServiceId          : 400
      customerId                  : 500
      active                      : true
      name                        : TestService
      description                 : TestDescription
      precedence                  : 1
      label                       : blue
      type                        : LDAP

      == LDAP Configuration ==

      customerAdminGroup          : Admin
      standardGroups              : Standard
      ldapAccessEndpoint          : ldap://ad.example.com/
      ldapAccessPrincipal         : CN=LDAP User,CN=Users,DC=ad,DC=example,DC=com
      ldapAccessSecret            : 4a5f69089a092bd0
      ldapAccessSsl               : false
      ldapObjectClass             : objectClass
      ldapGroupBase               : CN=Builtin,DC=ad,DC=example,DC=com
      ldapGroupDescription        : groups
      ldapGroupName               : cn
      ldapGroupUsernames          : member
      ldapGroupObjectClass        : group
      ldapUserBase                : CN=Users,DC=ad,DC=example,DC =com
      ldapUserEmail               : mail
      ldapUserFamilyName          : sn
      ldapUserGivenName           : givenName
      ldapUserGroup               : memberOf
      ldapUserObjectClass         : user
      ldapUserUserName            : sAMAccountName
      ldapDefaultPhoneRegion      : US
      ldapEmailRegex              : null
      ldapEmailRegexIndex         : null
      ldapUserMobile              : 654654654
      ldapUserTimeZone            : TZ

```

## Removing a LDAP service 

Removing a LDAP service will remove the contents from both <code>customer_ldap_service</code> and <code>customer_ldap_directory</code> tables in the database.

```
dirsync-tool.sh -r <directory_service_id>

Example :

root@vagrant:/services/console/sbin# ./dirsync-tool.sh -r 500
INFO: LDAP service with id : 500 has been removed.
```

## Updating LDAP service
There are two ways on how a user can update an existing LDAP service.

#### Using user prompt inputs for updating LDAP configuration values.

It will prompt user to enter new LDAP configuration values for a LDAP service in succession. Pressing enter will not make changes to the attribute.

```
dirsync-tool.sh -u <directory_service_id>

Example : 

root@vagrant:/services/console/sbin# ./dirsync-tool.sh -u 400
*****************************************************
 Updating LDAP Service : (TestService:400:500)
*****************************************************
Directory Service Id (400)
Customer ID (500) : 
Active      (true) : 
Name        (TestService) : 
Description (TestDescription) : 
Label       (blue) : 
Precedence  (1) :
*****************************************************
 Updating LDAP directory : (400:500)
*****************************************************
Directory Service Id (400)
Customer ID (500) : 
Customer Admin Group        (Admin) :
Standard Groups             (Standard) :
LDAP Access Endpoint        (ldap://ad.example.com/) :
LDAP Access Principal       (CN=LDAP User,CN=Users,DC=ad,DC=example,DC=com) :
LDAP Access Secret          (c95aabc96b947353) :
LDAP Access SSL             (Y/N) :
LDAP Group Base             (CN=Builtin,DC=ad,DC=example,DC=com) :
LDAP Group Description      (groups) :
LDAP Group Name             (cn) :
LDAP Group Usernames        (member) :
LDAP Group Object Class     (group) :

LDAP User Base              (CN=Users,DC=ad,DC=example,DC =com) :
LDAP User Family Name       (sn) :
LDAP User Given  Name       (givenName) :
LDAP User Group             (memberOf) :
LDAP User Email             (mail) :
LDAP User Object Class      (user) :

LDAP Object Class      (objectClass) :
LDAP User UserName          (sAMAccountName) :
LDAP Default Phone Region   (US) :
LDAP Email Regex            (null) :
LDAP Email Regex Index      (null) :
LDAP User Mobile            (654654654) :
LDAP User TimeZone          (TZ) :

Updated Directory Service [400]
```


#### Updating specific attributes of a LDAP service.

```
dirsync-tool.sh -m <directory_service_id>:<attribute_name>:<new_value>

Example:

root@vagrant:/services/console/sbin# ./dirsync-tool.sh -m 400:name:TestChange

Updated Directory Service [400] Field [name] with new value [TestChange].



***************************************************************
      directoryServiceId          : 400
      customerId                  : 500
      active                      : true
      name                        : TestChange
      description                 : TestDescription
      precedence                  : 1
      label                       : blue
      type                        : LDAP

      == LDAP Configuration ==

      customerAdminGroup          : Admin
      standardGroups              : Standard
      ldapAccessEndpoint          : ldap://ad.example.com/
      ldapAccessPrincipal         : CN=LDAP User,CN=Users,DC=ad,DC=example,DC=com
      ldapAccessSecret            : c95aabc96b947353
      ldapAccessSsl               : false
      ldapObjectClass             : objectClass
      ldapGroupBase               : CN=Builtin,DC=ad,DC=example,DC=com
      ldapGroupDescription        : groups
      ldapGroupName               : cn
      ldapGroupUsernames          : member
      ldapGroupObjectClass        : group
      ldapUserBase                : CN=Users,DC=ad,DC=example,DC =com
      ldapUserEmail               : mail
      ldapUserFamilyName          : sn
      ldapUserGivenName           : givenName
      ldapUserGroup               : memberOf
      ldapUserObjectClass         : user
      ldapUserUserName            : sAMAccountName
      ldapDefaultPhoneRegion      : US
      ldapEmailRegex              : null
      ldapEmailRegexIndex         : null
      ldapUserMobile              : 654654654
      ldapUserTimeZone            : TZ
```

## Updating standard groups of a LDAP service

Standard groups in AD is one of the attributes that changes more frequently. There are cases where rather than replacing the standard groups of a LDAP service, we require adding more groups to the existing list. We can add/remove groups from existing list with the help of the dirsycn-tool.

#### Adding standard groups

```
dirsync-tool.sh -s <dirctory_service_id>:<standard_groups to add separated by commas.>

Example :

root@vagrant:/services/console/sbin# ./dirsync-tool.sh -s 400:standard2,standard3,standard4
New standard group list for LDAP service 400 will be Standard,standard2,standard3,standard4

Added new standard groups.



***************************************************************
      directoryServiceId          : 400
      customerId                  : 500
      active                      : true
      name                        : TestChange
      description                 : TestDescription
      precedence                  : 1
      label                       : blue
      type                        : LDAP

      == LDAP Configuration ==

      customerAdminGroup          : Admin
      standardGroups              : Standard,standard2,standard3,standard4
      ldapAccessEndpoint          : ldap://ad.example.com/
      ldapAccessPrincipal         : CN=LDAP User,CN=Users,DC=ad,DC=example,DC=com
      ldapAccessSecret            : c95aabc96b947353
      ldapAccessSsl               : false
      ldapObjectClass             : objectClass
      ldapGroupBase               : CN=Builtin,DC=ad,DC=example,DC=com
      ldapGroupDescription        : groups
      ldapGroupName               : cn
      ldapGroupUsernames          : member
      ldapGroupObjectClass        : group
      ldapUserBase                : CN=Users,DC=ad,DC=example,DC =com
      ldapUserEmail               : mail
      ldapUserFamilyName          : sn
      ldapUserGivenName           : givenName
      ldapUserGroup               : memberOf
      ldapUserObjectClass         : user
      ldapUserUserName            : sAMAccountName
      ldapDefaultPhoneRegion      : US
      ldapEmailRegex              : null
      ldapEmailRegexIndex         : null
      ldapUserMobile              : 654654654
      ldapUserTimeZone            : TZ
```

#### Removing standard groups

```
dirsync-tool.sh -s <dirctory_service_id>:<standard_groups to remove separated by commas.>

Example : 

root@vagrant:/services/console/sbin# ./dirsync-tool.sh -k 400:standard3
Removing 'standard3' from existing standard group list.
New standard group list for LDAP service 400 will be Standard,standard2,standard4

Removed specified standard groups.



***************************************************************
      directoryServiceId          : 400
      customerId                  : 500
      active                      : true
      name                        : TestChange
      description                 : TestDescription
      precedence                  : 1
      label                       : blue
      type                        : LDAP

      == LDAP Configuration ==

      customerAdminGroup          : Admin
      standardGroups              : Standard,standard2,standard4
      ldapAccessEndpoint          : ldap://ad.example.com/
      ldapAccessPrincipal         : CN=LDAP User,CN=Users,DC=ad,DC=example,DC=com
      ldapAccessSecret            : c95aabc96b947353
      ldapAccessSsl               : false
      ldapObjectClass             : objectClass
      ldapGroupBase               : CN=Builtin,DC=ad,DC=example,DC=com
      ldapGroupDescription        : groups
      ldapGroupName               : cn
      ldapGroupUsernames          : member
      ldapGroupObjectClass        : group
      ldapUserBase                : CN=Users,DC=ad,DC=example,DC =com
      ldapUserEmail               : mail
      ldapUserFamilyName          : sn
      ldapUserGivenName           : givenName
      ldapUserGroup               : memberOf
      ldapUserObjectClass         : user
      ldapUserUserName            : sAMAccountName
      ldapDefaultPhoneRegion      : US
      ldapEmailRegex              : null
      ldapEmailRegexIndex         : null
      ldapUserMobile              : 654654654
      ldapUserTimeZone            : TZ
      
```

## Listing group mapping 

The contents of group mappings is managed by the dirsync code. No manual steps for populating the database is required. However we can view it's contents by the tool for debugging puroposes.

```
root@vagrant:/services/console/sbin# ./dirsync-tool.sh -g <directory_service_id>

Example : 

root@vagrant:/services/console/sbin# ./dirsync-tool.sh -g 400
Group Id               : 300
Directory Service Id   : 400
Directory Group Id     : CN=Admin,CN=Builtin,DC=ad,DC=example,DC=com

```

## Pre-Run Dirsync

The tool allows you to run the directory  in a "Pre-Run" mode that only logs the changes that are going to be made if an actual run of a dirsync happened. This mode will not make any changes to the Database. 

Also make sure the log4j settings for console is configured to print out dirsync logs.

Add the following category in /services/console/resources/log4j.xml to print out the pre-run logs of a dirsync.

```
  <category name="es.prerun">
    <priority value="TRACE" />
  </category>
```

Then execute the command to pre-run dirsync.

```
dirsync-tool.sh --prerun

Example :

root@vagrant:/services/console/sbin# ./dirsync-tool.sh --prerun
Running dirsync with prerun  = true
com.valtira.cms.types.AdAttributeFactory
com.valtira.cms.types.AdTypeAttributeFactory
com.valtira.cms.types.CategoryAttributeFactory
com.valtira.cms.types.LinkAttributeFactory
com.valtira.cms.types.MediaAttributeFactory
com.valtira.cms.types.MediaGroupAttributeFactory
com.valtira.cms.types.MediaTypeAttributeFactory
com.valtira.cms.types.PageAttributeFactory
com.valtira.cms.types.ColorAttributeFactory

com.valtira.blog.types.ForumAttributeFactory

Dirsync Run [Pre-run=true]  took : 16 seconds.

```
The logs for the pre-run of a dirsync can be found under /services/console/logs/dirsynclogs directory.


## Run Dirsync

Actual run of a directory synchronization process. 

Make sure the log4j settings for console is configured to print out dirsync logs.

Add the following category in /services/console/resources/log4j.xml to print out the logs of a dirsync run.

```
  <category name="es.directory">
    <priority value="TRACE" />
  </category>
```

Then execute the command to run dirsync.


```
dirsync-tool.sh --run

Example : 

root@vagrant:/services/console/sbin# ./dirsync-tool.sh --run
Running dirsync with prerun  = false
com.valtira.cms.types.AdAttributeFactory
com.valtira.cms.types.AdTypeAttributeFactory
com.valtira.cms.types.CategoryAttributeFactory
com.valtira.cms.types.LinkAttributeFactory
com.valtira.cms.types.MediaAttributeFactory
com.valtira.cms.types.MediaGroupAttributeFactory
com.valtira.cms.types.MediaTypeAttributeFactory
com.valtira.cms.types.PageAttributeFactory
com.valtira.cms.types.ColorAttributeFactory

com.valtira.blog.types.ForumAttributeFactory

Dirsync Run [Pre-run=false]  took : 16 seconds.

```
The logs for the pre-run of a dirsync can be found under /services/console/logs/dirsynclogs directory.

## Validating LDAP credentials

The tool allows the user to validate LDAP credentials separately. It only requires the user to provide the LDAP endpoint, LDAP principal/username and LDAP password.

```
dirsync-tool.sh -c

Example : 

root@vagrant:/services/console/sbin# ./dirsync-tool.sh -c
LDAP Endpoint           : ldap://ad.example.com/
LDAP Access Principal   : CN=LDAP User,CN=Users,DC=ad,DC=example,DC=com
LDAP Password           : testtesttest
Connecting to endpoint  : ldap://ad.example.com/
With principal          : CN=LDAP User,CN=Users,DC=ad,DC=example,DC=com
Failed connecting to directory at endpoint ldap://ad.example.com/ : ad.example.com:389
ERROR: Connection to ldap://ad.example.com/ with principal CN=LDAP User,CN=Users,DC=ad,DC=example,DC=com failed.

```





