## LDAP Authentication

KAP supports integration with LDAP servers for user authentication. This validation is achieved through the Spring Security framework, so it has a good versatility. 

#### Setup LDAP server
Before enabling LDAP authentication, you need an running LDAP service. If you already have it, contact the LDAP administrator to get the necessary information including connectivity inforamtion, organization structure, etc.

If you don't have LDAP server, need install one for KAP. The recommended server tool is OpenLDAP Server 2.4, as it is an open source implementation (under OpenLDAP Public License) and is the most popular LDAP server. It has been packaged in some Linux distributions like Red Hat Enterprise Linux; it can also be downloaded from http://www.openldap.org/

The installation may vary with different platform. You may need check different documents or tutorials like. Here we take CentOS 6.4 as an example:  

* Check installation
```
sudo find / -name openldap*
```

If not installed, install with yum:
```
sudo yum install -y openldap openldap-servers openldap-clients
```

* Configure after installation
```
cp /usr/share/openldap-servers/slapd.conf.obsolete /etc/openldap/slapd.conf
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
mv /etc/openldap/slapd.d{,.bak}
```

Edit slapd.conf, as follows：

1．Set the directory suffix

Find this line:
```
suffix "dc=my-domain,dc=com"
```

Change it to:
```
suffix "dc=example,dc=com"
```

2．Set DN for LDAP Administrator:

Find line：
```
rootdn "cn=Manager,dc=my-domain,dc=com"
```

Change it to：
```
rootdn "cn=Manager,dc=example,dc=com"
```

3．Set password for LDAP administrator

Find line：

```
rootpw secret
```

Replacing the default value with an encrypted password string. To create an encrypted password string, type the following command:

```
slappasswd
```

When prompted, type and then re-type a password. The program prints the resulting encrypted password to the shell prompt.

Next, copy the newly created encrypted password into the /etc/openldap/slapd.conf on one of the rootpw lines and remove the hash mark (#). When finished, the line should look similar to the following example:
```
rootpw {SSHA}vv2y+i6V6esazrIv70xSSnNAJE18bb2u
```

4．Change the owner for these configuration files

```
chown ldap.ldap /etc/openldap/*
chown ldap.ldap /var/lib/ldap/*
```

5．Make new directory:
```
mkdir /etc/openldap/cacerts
```

6．Reboot the system, and then start the service:
```
sudo service slapd start
```

After the service is started, you can import some sample data.

7．Create new file example.ldif

```
dn:dc=example,dc=com
objectclass:dcObject
objectclass:organization
o:Example, Inc.
dc:example

dn:cn=Manager,dc=example,dc=com
objectclass:organizationalRole
cn:Manager
```

8．Import it with command:

```
/usr/bin/ldapadd -x -W -D "cn=Manager,dc=example,dc=com" -f example.ldif
```

When prompt the password, enter the password of LDAP administrator. Then the import is succeed.


#### Configure LDAP Information in KAP

First, in conf/kylin.properties, configure the URL of the LDAP server, the necessary username and password (if the LDAP server is not anonymous); for security reason, the password here need be encrypted with AES, you can run "${KYLIN_HOME}/bin/kylin.sh io.kyligence.kap.tool.general.CryptTool AES <your_password>"to get the encrypted value, and then fill in kylin.properties:

```
ldap.server=ldap://<your_ldap_host>:<port>
ldap.username=<your_user_name>
ldap.password=<your_password_hash>
```

Second, provide user retrieval pattern, such as starting organization unit, filtering conditions etc; The following is an example for reference:

```
ldap.user.searchBase=OU=UserAccounts,DC=mycompany,DC=com
ldap.user.searchPattern=(&(AccountName={0})(memberOf=CN=MYCOMPANY-USERS,DC=mycompany,DC=com))
ldap.user.groupSearchBase=OU=Group,DC=mycompany,DC=com
```

If you need service accounts (for system integration) to access KAP, follow the example above to configure `ldap.service. *`, Otherwise leave them blank.

### Configure Administrator Groups and Default Roles

KAP allows mapping an LDAP group to the administrator role: In kylin.properties, set "acl.adminRole" to "ROLE_" + GROUP_NAME (The GROUP_NAME need be in upper case). For example, use the group "KAP-ADMIN-GROUP" to manage all KAP administrators, then this property should be set to

```
acl.adminRole=ROLE_KAP-ADMIN-GROUP
acl.defaultRole=ROLE_ANALYST,ROLE_MODELER
```

The attribute "acl.defaultRole" defines the roles that granted to any logged-on user. By default be both Analyst and Modeler.

#### Enable LDAP

In conf/kylin.properties, set "kylin.security.profile=ldap"，and then restart KAP.


