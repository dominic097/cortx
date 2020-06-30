# Cortx-S3Server QuickStart guide
This is a step by step guide to get Cortx-S3Server ready for you on your system.
Before cloning, however, you need to have an SSC / Cloud VM or a local VM setup in either VMWare Fusion or Oracle VirtualBox [LocalVMSetup](LocalVMSetup.md).

## Accessing the code right way
The latest code which is getting evolved and contributed is on the github.
Seagate contributors will be referencing, cloning and committing their code to/from this https://github.com/Seagate/cortx-s3server

To simply pull the code in which to build `git clone --recursive "https://github.com/Seagate/cortx-s3server" -b main`

Following steps will make your access to server hassle free.
1. From here on all the steps needs to be followed as the root user.
  * Set the root user password using `sudo passwd` and enter the required password.
  * Type `su -` and enter the root password to switch to the root user mode.
2. Create SSH Public Key
  * [SSH generation](https://git-scm.com/book/en/v2/Git-on-the-Server-Generating-Your-SSH-Public-Key) will make your key generation super easy. Follow the instructions throughly.
3. Add SSH Public Key on [github](https://github.com/settings/keys).
  * Log into the github with your github account credentials.
  * On right top corner you will see your name, open drop down menu by clicking and choose settings.
  * In the menu on the left, click on the SSH and GPG keys, and add your public key (which is generated in step one) right there.

WoW! :sparkles:
You are all set to fetch Cortx-S3Server repo now! 

## Cloning Cortx-S3Server Repository
Getting the main Cortx-S3Server code on your system is straightforward.
1. `$ cd path/to/your/dev/directory`
2. `$ git clone git@github.com:Seagate/cortx-s3server.git -b main` ( It has been assumed that `git` is preinstalled.If not then follow git installation specific steps. Recommended git version is 2.x.x . Check your git version using `$ git --version` command.)
3. `$ cd cortx-s3Server`
4. `$ git submodule update --init --recursive && git status`

## Installing dependency
This is a one time initialization when we do clone the repository or there is a changes in dependent packages.

  * At some point during the execution of the `init.sh` script, it will prompt for the following passwords. Enter them as mentioned below.
    * SSH password: `<Enter root password of VM>`
    * Enter new password for openldap rootDN:: `seagate`
    * Enter new password for openldap IAM admin:: `ldapadmin`

1. `$ cd ./scripts/env/dev`
2. `$ ./init.sh`, For some system `./init.sh` fails sometimes. If it is failing run `./upgrade-enablerepo.sh` and re run `./init.sh`. Refer below image of successful run of `./init.sh` where `failed` field should be zero.

<p align="center"><img src="../../assets/images/init_script_output.PNG?raw=true"></p>

## Compilation and Running Unit Test
All the following commands assume that user is already in its main source directory.
### Running Unit test and System test
1. Setup the host system
  * `$ ./update-hosts.sh`
2. Following script by default will build the code, run the unit test and system test in your local system. Check for help to get more details.  
  * `$ ./jenkins-build.sh`. 
  * You may have to add `/usr/local/bin` to PATH variable using command `$PATH=$PATH:/usr/local/bin` if it is not there already.
  
  Make sure the output log has a message as shown in below image to ensure successful execution of system test in `./jenkins-build.sh`.
  
<p align="center"><img src="../../assets/images/jenkins_script_output.PNG?raw=true"></p>

### Testing using S3CLI
1. Installation and configuration
  * Make sure you have `easy_install` installed using `$ easy_install --version`. If it is not installed run the following command.
    * `$ yum install python-setuptools python-setuptools-devel`
  * Make sure you have `pip` installed using `$ pip --version`. If it is not installed run the following command.
    * `$ python --version`, if you don't have python version 2.6.5+ then install python.
    * `$ python3 --version`, if you don't have python3 version 3.3+ then install python3.
    * `$ easy_install pip`
  * Make sure Cortx-S3Server and it's dependent services are running.
    * `$ ./jenkins-build.sh --skip_build --skip_tests` so that it will start Cortx-S3Server and it's dependent services.
    * `$ pgrep s3`, it should list the `PID` of S3 processes running.
    * `$ pgrep m0`, it should list the `PID` of motr processes running.
  * Install aws client and it's plugin
    * `$ pip install awscli`
    * `$ pip install awscli-plugin-endpoint`
  * Generate aws access key id and aws secret key
    * `$ s3iamcli -h` to check help messages.
    * `$ s3iamcli CreateAccount -n < Account Name > -e < Email Id >` to create new user. Enter the ldap `User Id` and `password` as mentioned below. Along with other details it will generate `aws access key id` and `aws secret key` for new user, make sure you save those.
      * Enter Ldap User Id: `sgiamadmin`
      * Enter Ldap password: `ldapadmin`
  * Configure AWS
    * `$ aws configure` and enter the following details.
      * AWS Access Key ID [None]: < ACCESS KEY >
      * AWS Secret Access Key [None]: < SECRET KEY >
      * Default region name [None]: US
      * Default output format [None]: text
  * Set Endpoint
    * `$ aws configure set plugins.endpoint awscli_plugin_endpoint`
    * `$ aws configure set s3.endpoint_url http://s3.seagate.com`
    * `$ aws configure set s3api.endpoint_url http://s3.seagate.com`
  * Make sure aws config file has following content
    * `$ cat ~/.aws/config`
  ```
  [default]
  output = text
  region = US
  s3 =
      endpoint_url = http://s3.seagate.com
  s3api =
      endpoint_url = http://s3.seagate.com
  [plugins]
  endpoint = awscli_plugin_endpoint
  ```
  * Make sure aws credential file has your access key Id and secret key.
    * `$ cat ~/.aws/credentials`
2. Test cases
  * Make Bucket
    * `$ aws s3 mb s3://seagatebucket`, should be able to get following output on the screen
      * `make_bucket: seagatebucket`
  * List Bucket
    * `$ aws s3 ls`, should be able to see the bucket we have created.
  * Remove Bucket
    * `$ aws s3 rb s3://seagatebucket`, bucket should get remove and should not be seen if we do list bucket.
  * Copy local file to remote(PUT)
    * `$ aws s3 cp test_data s3://seagatebucket/`, will copy the local file test_data and test_data object will be created under bucket.
  * List Object
    * `$ aws s3 ls s3://seagatebucket`, will show the object named as test_data
  * Move local file to remote(PUT)
    * `$ aws s3 mv test_data s3://seagatebucket/`, will move the local file test_data and test_data object will be created under bucket.
  * Remove object 
    * `$ aws s3 rm  s3://seagatebucket/test_data`, the test_data object should be removed and it should not be seen if we do list object.

KABOOM!!!

## Testing specific MOTR version with Cortx-S3Server
For this demand also we are having solution :
1. Search for specific commit-id in search box and choose type = 'Commits' , click on  search result (specific commit) and copy associated change-id
2. `$ cd third_party/mero` (It is assumed that you are into main directory of your Cortx-S3Server repo)
3. Use copied commit HASH/REFSPEC in step 1 as shown below.
   
 > git checkout Id41cd2b41cb77f1d106651c267072f29f8c81d0f
   
4. Update submodules 
> `$ git submodule update --init --recursive`
5. Build motr

> `cd ..`

> `./build_mero.sh`

6. Run jenkins script to make sure that your build & tests passes.

> `cd ..`

> `./jenkins-build.sh`

* success logs looks like this :point_down:

<p align="center"><img src="../../assets/images/jenkins_script_output.PNG?raw=true"></p>

### You're all set & You're awesome

In case of any queries, feel free to write to our [SUPPORT](SUPPORT.md).

Let's start without a delay to contribute to Seagate's open source initiative and join this movement with us, keeping a common goal of making data storage better, more efficient and more accessible.

Seagate welcomes You! :relaxed: