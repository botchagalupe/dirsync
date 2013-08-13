## Dirsync-Tool
Dirsync-Tool is a mutlinode sbin tool that allows you configure LDAP inetgration with Enstratius and run directory synchronization.

## Usage

```
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
