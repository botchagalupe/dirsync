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



## Listing LDAP services

There are two tables in the console database that represents a LDAP service : customer_ldap_service and customer_ldap_directory. When listing a LDAP service with the tool, it incorporates the contents of both the tables into one. 
