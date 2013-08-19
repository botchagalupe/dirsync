## Dirsync-Tool
Dirsync-Tool is a mutlinode sbin tool that allows you configure LDAP inetgration with Enstratius and run directory synchronization.

## Usage

```
root@vagrant:/services/console/sbin# ./dirsync-tool.sh -h
Usage: -action

Options:
   -v, --version              show program's version number and exit.
   -h, --help                 show this help message and exit.

    Main actions:
    ---------------------------------------

      -l $customerId          List a LDAP service for a particular customer from console database.
                              Not specifying customer id returns LDAP services for all the customer.

      -g $directoryServiceID  List group mapping for a specific directory service
                              Not specifying directory service id returns all group mapping.

      -a                      Add a LDAP Service. 
      -u $directoryServiceID  Update existing LDAP Service in database. 
      -r $directoryServiceID  Remove sepcific LDAP service from database. 
      -p                      Pre-run dirsync that logs the changes in the dirsync run. 
      -f                      Run an actual dirsync
      -c                      Check authentication with LDAP credentials.
      -m $directoryServiceID  Update customer admin and standard groups for a LDAP service.

```

## Adding LDAP service.
Dirsync-Tool i.0 only supports user prompt feature for data intake. A customer can have multiple LDAP services with different configurations. Here are the attribute values the user should gather from LDAP/AD administrator before integrating LDAP with Enstratius.

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


Once we have these information, it can be stored in the database via Dirsyc Tool. The tool will prompt user to enter all of the above information along with other general high level information.

NOTE: The tool will validate the LDAP credentials first before passsing some other LDAP configuration values.

```
root@vagrant:/services/console/sbin# ./dirsync-tool.sh -a
```

## Listing LDAP services


There are two tables in the console database that represents a LDAP service : <code>customer_ldap_service</code> and <code>customer_ldap_directory</code>. When listing a LDAP service with the tool, it incorporates the contents of both the tables into one. 

```
root@vagrant:/services/console/sbin# ./dirsync-tool.sh -l
```

## Removing a LDAP service 

Removing a LDAP service will remove the contents from both <code>customer_ldap_service</code> and <code>customer_ldap_directory</code> related that directory service.

``

``

## Updating LDAP service

The tool allows a user to update exisitng LDAP services. It will prompt user to enter new values for each of the mutable attributes in code>customer_ldap_service</code> and <code>customer_ldap_directory</code> 

``
``

## Updating groups of a LDAP service

Groups in AD is one of the attributes that changes more frequently. Customer can add/remove groups and these group name information will have to be changed manually for now. The tool allows you to change the change just the groups attributes of the LDAP service in Enstratius.

``
``

# Listing group mapping 

The contents of group mappings is managed by the dirsync code. No manual steps for populating the database is required. However we can view it's contents by the tool for debugging puroposes.

```
```



