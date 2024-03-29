# cms-ars-5.0-oracle-mysql-8-stig-overlay
**CMS’ ISPG (Information Security and Privacy Group) decided to discontinue funding the customization of MITRE’s Security Automation Framework (SAF) for CMS after September 2023. This repo is now in archive mode, but still accessible. For more information about SAF with current links, see https://security.cms.gov/learn/security-automation-framework-saf**


InSpec profile overlay to validate the secure configuration of Oracle MySQL 8 against [DISA's](https://public.cyber.mil/stigs/) Oracle MySQL 8 STIG Version 1 Release 1 tailored for CMS ARS 5.0.

#### AWS-RDS-Ready: Profile updated to adapt checks when the running against an AWS RDS instance of MySQL, by setting the input `aws_rds` to `true`. See [Inputs: Tailoring your scan to Your Environment](#inputs-tailoring-your-scan-to-your-environment) and [MySQL client setup](#mysql-client-setup) below.

## Getting Started  
### InSpec (CINC-auditor) setup
For maximum flexibility/accessibility, we’re moving to “cinc-auditor”, the open-source packaged binary version of Chef InSpec, compiled by the CINC (CINC Is Not Chef) project in coordination with Chef using Chef’s always-open-source InSpec source code. For more information: https://cinc.sh/

It is intended and recommended that CINC-auditor and this profile overlay be run from a __"runner"__ host (such as a DevOps orchestration server, an administrative management system, or a developer's workstation/laptop) against the target. This can be any Unix/Linux/MacOS or Windows runner host, with access to the Internet.

__For the best security of the runner, always install on the runner the _latest version_ of CINC-auditor.__ 

__The simplest way to install CINC-auditor is to use this command for a UNIX/Linux/MacOS runner platform:__
```
curl -L https://omnitruck.cinc.sh/install.sh | sudo bash -s -- -P cinc-auditor
```

__or this command for Windows runner platform (Powershell):__
```
. { iwr -useb https://omnitruck.cinc.sh/install.ps1 } | iex; install -project cinc-auditor
```
To confirm successful install of cinc-auditor:
```
cinc-auditor -v
```
> sample output:  _4.24.32_

Latest versions and other installation options are available at https://cinc.sh/start/auditor/.

### MySQL client setup

To run the MySQL profile against an AWS RDS Instance, CINC-auditor expects the mysql client to be readily available on the same runner system it is installed on.
 
For example, to install the mysql client on a Linux runner host:
```
sudo yum install mysql-community-server
```
To confirm successful install of mysql:
```
which mysql
```
> sample output:  _/usr/bin/mysql_
```
mysql –-version
```		
> sample output:  *mysql  Ver 8.0.32 for Linux on x86_64 (MySQL Community Server - GPL)*

Test mysql connectivity to your AWS RDS instance from your InSpec runner host:
```
mysql -u <master user> -p<password>  -h <endpoint>.amazonaws.com -P 3306
```		
> sample output:
>
>  *Welcome to the MySQL monitor.  Commands end with ; or \g.*
>  *Your MySQL connection id is 4035*
>  *Server version: 8.0.32 Source distribution*
>
>  *Copyright (c) 2000, 2023, Oracle and/or its affiliates.*
>
>  *Oracle is a registered trademark of Oracle Corporation and/or its*
>  *affiliates. Other names may be trademarks of their respective*
>  *owners.*
>
>  *Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.*
>
>  *mysql> quit*
>
>  *Bye*

For installation of mysql client on other operating systems for your runner host, visit https://www.mysql.com/

## Specify your BASELINE system categorization as an environment variable:
### (if undefined defaults to Moderate baseline)

```
# BASELINE (choices: Low, Low-HVA, Moderate, Moderate-HVA, High, High-HVA)
# (if undefined defaults to Moderate baseline)

on linux:
BASELINE=Moderate

on Powershell:
$env:BASELINE="Moderate"
```

## Inputs: Tailoring your scan to Your Environment

The following inputs must be configured in an inputs ".yml" file for the profile to run correctly for your specific environment. More information about InSpec inputs can be found in the [InSpec Profile Documentation](https://www.inspec.io/docs/reference/profiles/).

### Inputs You May Tailor (with *example* values you should tailor to your environment):

```yaml
#Description: State if your database is an AWS RDS instance
#Value type: Boolean
aws_rds: false
 
#Description: privileged account username MySQL DB Server
#Value Type: string
user: root

#Description: password specified user
#Value Type: string
password: mysqlrootpass

#Description: hostname of MySQL DB Server
#Value Type:
host: localhost

#Description: port MySQL DB Server
#Value Type: numeric
port: 3306

#Description: Wildcard based path to list all audit log files
#Value Type: string
audit_log_path: /var/lib/mysql/audit*log*

#Description: List of documented audit admin accounts.
#Value Type: array
audit_admins: ["'root'@'localhost'", "'root'@'%'"]

#Description: Name of the documented server cert issuer.
#Value Type: string
org_appoved_cert_issuer: CMS Root CA

#Description: List of documented accounts exempted from PKI authentication.
#Value Type: array
pki_exception_users: ["healthchecker"]

#Description: List of documented accounts allowed to login with password.
#Value Type: array
authorized_password_users: ["healthchecker"]

#Description: List of documented mysql accounts with administrative privileges.
#Value Type: array
mysql_administrative_users: ["root"]

#Description: List of documented mysql administrative role grantees
#Value Type: array
mysql_administrative_grantees: ["'root'@'localhost'"]

#Description: List of approved Plugins
#Value Type: array
approved_plugins: ["audit_log"]

#Description: List of approved components
#Value Type: array
approved_components: ["file://component_validate_password"]

#Description: Authorized MySQL port definitions
#Value Type: Hash
mysql_ports:
  port: 3306
  admin_port: 33062
  mysqlx_port: 33060

#Description: Authorized MySQL socket definitions
#Value Type: Hash
mysql_sockets:
  socket: '/var/lib/mysql/mysql.sock'
  mysqlx_socket: '/var/run/mysqld/mysqlx.sock'

#Description: Location of the my.cnf file
#Value Type: string
mycnf: /etc/my.cnf

#Description: Location of the mysqld-auto.cnf file
#Value Type: string
mysqld_auto_cnf: /var/lib/mysql/auto.cnf

#Description: Location of the mysqld-auto.cnf file
#Value Type: array
authorized_procedures: []

#Description: Location of the mysqld-auto.cnf file
#Value Type: array
authorized_functions: []

#Description: Approved minimum version of MySQL
#Value Type: string
minimum_mysql_version: 8.0.25


```

## Running This Overlay Directly from Github against a hosted (non AWS RDS) instance of MySQL 8: 

Against a remote target using ssh (i.e., cinc-auditor installed on a separate runner host)
```bash
cinc-auditor exec https://github.com/cms-enterprise/cms-ars-5.0-oracle-mysql-8-stig-overlay/archive/main.tar.gz -t ssh://<target_username>:<target_password>@<target_ip>:<target_port> --input-file <path_to_your_input_file/name_of_your_input_file.yml> --reporter=cli json:<path_to_your_output_file/name_of_your_output_file.json> 
```

Against a remote target using a pem key (i.e., cinc-auditor installed on a separate runner host)
```bash
cinc-auditor exec https://github.com/cms-enterprise/cms-ars-5.0-oracle-mysql-8-stig-overlay/archive/main.tar.gz -t ssh://<target_username>@<target_ip>:<target_port> -i <PEM_KEY> --input-file <path_to_your_input_file/name_of_your_input_file.yml> --reporter=cli json:<path_to_your_output_file/name_of_your_output_file.json>  
```

Against a _**locally-hosted**_ instance (i.e., cinc-auditor installed on the target hosting the database)

```bash
cinc-auditor exec https://github.com/cms-enterprise/cms-ars-5.0-oracle-mysql-8-stig-overlay/archive/main.tar.gz --input-file=<path_to_your_inputs_file/name_of_your_inputs_file.yml> --reporter=cli json:<path_to_your_output_file/name_of_your_output_file.json>
```

Against a _**docker-containerized**_ instance (i.e., cinc-auditor installed on the node hosting the container):
```
cinc-auditor exec https://github.com/cms-enterprise/cms-ars-5.0-oracle-mysql-8-stig-overlay/archive/main.tar.gz -t docker://<instance_id> --input-file=<path_to_your_inputs_file/name_of_your_inputs_file.yml> --reporter=cli json:<path_to_your_output_file/name_of_your_output_file.json>
```
## Running This Overlay Directly from Github against an AWS RDS instance of MySQL 8: 

```bash
cinc-auditor exec https://github.com/cms-enterprise/cms-ars-5.0-oracle-mysql-8-stig-overlay/archive/main.tar.gz --input-file <path_to_your_input_file/name_of_your_input_file.yml> --reporter=cli json:<path_to_your_output_file/name_of_your_output_file.json> 
```


### Different Run Options

  [Full exec options](https://docs.chef.io/inspec/cli/#options-3)

## Running This Overlay from a local Archive copy 

If your runner is not always expected to have direct access to GitHub, use the following steps to create an archive bundle of this overlay and all of its dependent tests:

(Git is required to clone the InSpec profile using the instructions below. Git can be downloaded from the [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) site.)

When the __"runner"__ host uses this profile overlay for the first time, follow these steps: 

```
mkdir profiles
cd profiles
git clone https://github.com/cms-enterprise/cms-ars-5.0-oracle-mysql-8-stig-overlay.git
cinc-auditor archive cms-ars-5.0-oracle-mysql-8-stig-overlay
cinc-auditor exec <name of generated archive> --input-file=<path_to_your_inputs_file/name_of_your_inputs_file.yml> --reporter=cli json:<path_to_your_output_file/name_of_your_output_file.json>
```

For every successive run, follow these steps to always have the latest version of this overlay and dependent profiles:

```
cd cms-ars-5.0-oracle-mysql-8-stig-overlay
git pull
cd ..
cinc-auditor archive cms-ars-5.0-oracle-mysql-8-stig-overlay --overwrite
cinc-auditor exec <name of generated archive> --input-file=<path_to_your_inputs_file/name_of_your_inputs_file.yml> --reporter=cli json:<path_to_your_output_file/name_of_your_output_file.json>
```

## Using Heimdall for Viewing the JSON Results

The JSON results output file can be loaded into __[heimdall-lite](https://heimdall-lite.cms.gov/)__ for a user-interactive, graphical view of the InSpec results. 

The JSON InSpec results file may also be loaded into a __[full heimdall server](https://github.com/mitre/heimdall2)__, allowing for additional functionality such as to store and compare multiple profile runs.

## Authors
* Eugene Aronne - [ejaronne](https://github.com/ejaronne)

## Special Thanks
* Aaron Lippold - [aaronlippold](https://github.com/aaronlippold)
* Shivani Karikar - [karikarshivani](https://github.com/karikarshivani)

## Contributing and Getting Help
To report a bug or feature request, please open an [issue](https://github.com/cms-enterprise/cms-ars-5.0-oracle-mysql-8-stig-overlay/issues/new).

### NOTICE

© 2018-2022 The MITRE Corporation.

Approved for Public Release; Distribution Unlimited. Case Number 18-3678.

### NOTICE 

MITRE hereby grants express written permission to use, reproduce, distribute, modify, and otherwise leverage this software to the extent permitted by the licensed terms provided in the LICENSE.md file included with this project.

### NOTICE  

This software was produced for the U. S. Government under Contract Number HHSM-500-2012-00008I, and is subject to Federal Acquisition Regulation Clause 52.227-14, Rights in Data-General.  

No other use other than that granted to the U. S. Government, or to those acting on behalf of the U. S. Government under that Clause is authorized without the express written permission of The MITRE Corporation.

For further information, please contact The MITRE Corporation, Contracts Management Office, 7515 Colshire Drive, McLean, VA  22102-7539, (703) 983-6000.

### NOTICE 

Defense Information Systems Agency (DISA) STIGs are published at: https://public.cyber.mil/stigs/
