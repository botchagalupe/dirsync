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

customer_admin_group
The CN of your the group in your directory service (for example, ‘Administrators’) that will always have administrative access across all accounts in your infrastructure. This value may be null. If null, you must have some non-directory service group as your admin group elsewhere. enStratus will search under ldap_group_base for an object with an object class of ldap_ group_object_class and the CN matching this value. That group will be the admin group.

standard_groups
Similar to customer_admin_group, this value is a comma-separated list of group CNs representing the groups to be replicated between enStratus and your directory service. This may be null if you are just replicating the customer_admin_group. This list should not, however, include the customer_admin_group. Users in these groups will be synchronized into enStratus, but the groups won’t have any special access except that defined by roles you assign.

ldap_access_endpoint
For example, ldap://ad.example.com/. This is your LDAP server.

ldap_access_principal
The formal DN for the principal being used to authenticate connections with LDAP. For example, ‘CN=LDAP User,CN=Users,DC=ad,DC=example,DC=com’.
ldap_access_secret
The encrypted secret password for your directory service access. In order to create this
value, you must run the command java com.enstratus.directory.CustomerLdapDirectory CUSTOMER_ID ENCRYPTION_KEY PASSWORD and use the encryption key defined in the classes/cms/networks.cfg file under the encryptionKey value to perform the encryption. You must have the enstratus-directory-2011.08.1.jar file in your CLASSPATH. This encrypted value cannot be decrypted with the encryption password alone. The output of the command is what you stick in ldap_access_secret.
enStratus User Management ￼ Page 6
Copyright © 2012 enStratus Networks, Inc.
ldap_access_ssl
A value of ‘Y’ or ‘N’ depending on whether SSL should be used for the communications with the LDAP server. ‘Y’ is recommended.
ldap_group_base
The base DN for the object under which group objects are stored. For example, ‘CN=Builtin,D C=ad,DC=example,DC=com’. enStratus will do a deep search under this object for groups.
ldap_group_description
The name of the LDAP attribute containing the value to be used as a group description when the group is represented in enStratus. If you do not have any kind of description attribute, just use the CN value or what is defined in ldap_group_name.
ldap_group_name
The LDAP attribute containing the name of the group. This value should be unique across all groups. It is typically ‘CN’.
ldap_group_object_class
The object class for groups in your LDAP/AD directory. Typically, this value is ‘group’.
ldap_group_usernames
The LDAP attribute that contains the user names of the users that belong to this group.
ldap_object_class
The LDAP attribute that defines an object’s object class. For example, ‘objectClass’.
ldap_user_base
The base DN under which users are stored. For example, ‘CN=Users,DC=ad,DC=example,DC =com’. enStratus will do a deep search under this object for users.
ldap_user_email
The LDAP attribute representing a user email. In some organizations, the email address may be embedded inside a more complex text attribute. You can use the values ldap_email_ regex and ldap_email_regex_index to pull the email value from that text. enStratus requires user emails to be unique and serves as the user’s primary mechanism for identifying themselves to enStratus.
ldap_user_family_name
The LDAP attribute that stores a person’s last name/family name.
ldap_user_given_name
The LDAP attribute that stores a person’s first name/given name.
ldap_user_group
The LDAP attribute that defines the user’s group membership.
ldap_user_object_class
The LDAP object class for users. For example, ‘user’.
ldap_user_user_name
The LDAP attribute that stores the user name a person uses to login with other systems when a user name (as opposed to an email address) is used to identify them.
enStratus User Management ￼ Page 7
Copyright © 2012 enStratus Networks, Inc.
enStratus User Management ￼ Page 8
Copyright © 2012 enStratus Networks, Inc.
enStratus also optionally looks for the following attributes:
ldap_default_phone_region
The default region for interpreting the phone number fields. For example, ‘US’.
ldap_email_regex
A Java regex that tells enStratus how to parse the values in the ldap_user_email field.
ldap_email_regex_index
The index identifying which part of the regex stores the email address.
ldap_user_mobile
The LDAP attribute containing the user’s mobile phone number. This value is required when using LDAP with an SMS-based enStratus multi-factor authentication. If the mobile phone number is from a country different than the one defined by ldap_default_phone_region, it should start with a country code. Otherwise, enStratus will interpret it as belonging in the default phone region country.
ldap_user_time_zone
The LDAP attribute storing the user’s preferred time zone.

## Listing LDAP services

There are two tables in the console database that represents a LDAP service : customer_ldap_service and customer_ldap_directory. When listing a LDAP service with the tool, it incorporates the contents of both the tables into one. 
